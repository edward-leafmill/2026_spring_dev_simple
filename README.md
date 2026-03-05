# hello_simple

VS Code + CMake로 C++ 프로젝트를 빌드·실행·디버깅하는 방법을 배우는 예제입니다.

---

## 프로젝트 구조

```
hello_simple/
├── CMakeLists.txt          ← 빌드 설정 (어떤 파일을 컴파일할지)
├── main.cpp                ← 소스 코드
├── .vscode/
│   ├── tasks.json          ← VS Code 빌드 작업 정의
│   └── launch.json         ← VS Code 디버깅/실행 설정
└── build/                  ← CMake가 만드는 빌드 결과물 폴더
    └── hello_simple.exe    ← 최종 실행 파일
```

---

## 핵심 개념: 파일들이 어떻게 연결되는가

아래 흐름을 이해하면 전체 구조가 보입니다.

```
CMakeLists.txt          tasks.json              launch.json
─────────────          ──────────              ───────────
타깃 이름 정의    →    타깃을 빌드하는 Task  →  빌드 후 실행/디버깅
(hello_simple)         (preLaunchTask로 연결)
```

### 1단계: CMakeLists.txt — "무엇을 만들 것인가"

```cmake
cmake_minimum_required(VERSION 3.16)
project(hello_simple LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)        # C++23 사용
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(hello_simple main.cpp)   # ← 이 줄이 핵심!
```

`add_executable(hello_simple main.cpp)` 의 의미:
- **hello_simple** = 실행 파일(타깃) 이름 → `build/hello_simple.exe`가 만들어짐
- **main.cpp** = 컴파일할 소스 파일

> 파일이 여러 개면 `add_executable(hello_simple main.cpp utils.cpp header.h)` 처럼 추가합니다.

### 2단계: tasks.json — "어떻게 빌드할 것인가"

```jsonc
{
    "label": "CMake: build launch target",
    "command": "cmake",
    "args": [
        "--build",
        "${command:cmake.buildDirectory}",    // ← 빌드 폴더 (자동)
        "--target",
        "${command:cmake.launchTargetName}"   // ← 타깃 이름 (자동)
    ]
}
```

핵심 포인트:
- `${command:cmake.buildDirectory}` → CMake Tools 확장이 알려주는 빌드 폴더 경로
- `${command:cmake.launchTargetName}` → 상태바에서 선택한 타깃 이름

**왜 이렇게 하는가?** 경로나 이름을 직접 적으면("하드코딩"), 타깃 이름이 바뀌거나 빌드 폴더가 달라질 때마다 수정해야 합니다. `${command:cmake.*}` 변수를 쓰면 자동으로 맞춰집니다.

### 3단계: launch.json — "어떻게 실행/디버깅할 것인가"

```jsonc
{
    "name": "C/C++: Debug CMake launch target (gdb)",
    "type": "cppdbg",
    "program": "${command:cmake.launchTargetPath}",       // 실행할 파일 (자동)
    "cwd": "${command:cmake.launchTargetDirectory}",      // 작업 디렉터리 (자동)
    "preLaunchTask": "CMake: build launch target",        // ← 먼저 빌드!
    "MIMode": "gdb",
    "miDebuggerPath": "C:\\msys64\\ucrt64\\bin\\gdb.exe"  // 디버거 경로
}
```

핵심 포인트:
- `preLaunchTask` → 실행 전에 tasks.json의 빌드 Task를 먼저 실행
- `program` → 빌드된 실행 파일 경로를 CMake Tools에서 자동으로 가져옴
- 디버거는 MSYS2/UCRT64의 `gdb.exe` 사용

---

## 실습 절차 (따라하기)

### 사전 준비

- VS Code 설치
- VS Code 확장: **C/C++**, **CMake Tools** 설치
- MSYS2 설치 후 `ucrt64/bin`에 `g++.exe`, `gdb.exe` 있는지 확인

### 단계별 실행

```
① VS Code에서 폴더 열기 (File → Open Folder → C:\hello_simple)

② Command Palette (Ctrl+Shift+P) → "CMake: Configure" 실행
   → build/ 폴더에 빌드 파일들이 생성됨

③ VS Code 하단 상태바에서 Launch Target이 "hello_simple"인지 확인
   → 아니면 클릭해서 선택

④ F5 누르기 = 디버깅 모드 실행
   → 빌드 → gdb 연결 → 프로그램 실행
   → 브레이크포인트를 걸면 멈춤

⑤ Ctrl+F5 누르기 = 디버깅 없이 실행
   → 빌드 → 프로그램 바로 실행 (브레이크포인트 무시)
```

---

## F5 vs Ctrl+F5 차이

| 키 | VS Code 명령 | 동작 |
|---|---|---|
| **F5** | `workbench.action.debug.start` | 디버거(gdb) 연결 후 실행. 브레이크포인트에서 멈춤 |
| **Ctrl+F5** | `workbench.action.debug.run` | 디버거 없이 실행. 브레이크포인트 무시 |

> 두 경우 모두 `preLaunchTask`에 의해 **빌드가 먼저** 실행됩니다.

### 키가 제대로 작동하는지 확인하는 법

1. Command Palette → `Developer: Toggle Keyboard Shortcuts Troubleshooting`
2. F5 또는 Ctrl+F5를 누르면 하단 패널에 어떤 명령이 실행되는지 로그가 출력됨
3. 최종 매칭된 `commandId`를 확인하면 됨

> `Keyboard event cannot be dispatched` 메시지가 Ctrl/Shift 단독 입력 시 반복되는 것은 정상입니다. 조합키의 최종 해석 라인만 확인하세요.

---

## 자주 하는 실수 & 헷갈리는 점

### "active file 빌드"와 "CMake 타깃 빌드"는 다릅니다

tasks.json에는 두 가지 빌드 방식이 있습니다:

| 방식 | Task 이름 | 설명 |
|---|---|---|
| **CMake 타깃 빌드** | `CMake: build launch target` | CMake가 관리하는 전체 프로젝트 빌드 |
| **Active file 빌드** | `C/C++: g++.exe build active file` | 현재 열린 파일 하나만 g++로 직접 컴파일 |

- 파일이 1개뿐인 간단한 예제에서는 둘 다 동작하지만, **파일이 여러 개인 프로젝트에서는 반드시 CMake 타깃 빌드를 사용**하세요.
- launch.json은 CMake 타깃 빌드 Task(`preLaunchTask`)를 사용하도록 설정되어 있습니다.

### 브레이크포인트가 안 걸려요

- 실행 중인 바이너리와 소스 파일이 일치하는지 확인하세요.
- 빌드가 오래된 상태라면 `CMake: Configure` → `F5`로 다시 빌드 후 실행하세요.
- Release 모드가 아닌 **Debug 모드**로 빌드되었는지 확인하세요 (상태바의 빌드 타입).

### 상태바의 Play 버튼(벌레 아이콘)

- 아이콘 모양은 현재 선택된 실행 모드(디버그/일반)를 보여주는 UI일 뿐입니다.
- 실제 동작은 `launch.json` 설정이 결정합니다. 아이콘만 보고 판단하지 마세요.

---

## 설정의 설계 의도

이 프로젝트의 `.vscode` 설정은 다음 원칙으로 만들어졌습니다:

1. **하드코딩 최소화** — 경로·이름을 직접 쓰지 않고 `${command:cmake.*}` 변수 사용
2. **타깃 이름이 바뀌어도 동작** — CMake Tools 상태바에서 선택만 바꾸면 됨
3. **빌드 폴더가 달라도 동작** — 다른 환경에서도 같은 설정 파일 공유 가능

이 덕분에 나중에 타깃 이름을 바꾸거나, 소스 파일을 추가하거나, 빌드 폴더를 변경해도 `.vscode` 설정 파일을 수정할 필요가 없습니다.
