# hello_simple

VS Code + CMake 기반 C++ 예제 프로젝트입니다.  
현재 설정은 `launch.json` + `tasks.json` + CMake Tools 변수(`cmake.*`)를 함께 사용합니다.

---

## 프로젝트 구성

- `CMakeLists.txt`: 빌드 타깃 정의 (`hello_simple`)
- `.vscode/tasks.json`: 빌드 작업 정의
- `.vscode/launch.json`: 디버그/실행(launch) 정의
- `.vscode/settings.json`: CMake 실행 정책(실행 전 빌드 등) 정의
- `main.cpp`: 예제 소스

---

## 현재 VS Code 실행 단축키 정리

이 프로젝트에서 자주 쓰는 키는 다음 3개입니다.

1. `F5`
- 명령: `Debug: Start Debugging`
- 의미: 디버거(gdb)를 붙여 실행
- 동작: `cmake.buildBeforeRun=true` 기준으로 변경분 빌드 후 디버그 실행

2. `Ctrl+F5`
- 명령: `Debug: Start Without Debugging`
- 의미: 디버거 없이 실행
- 동작: 선택된 launch configuration을 그대로 사용하며, `cmake.buildBeforeRun=true` 기준으로 변경분 빌드 후 실행

3. `Ctrl+Shift+F5`
- 명령: `CMake: Run Without Debugging` (CMake Tools)
- 의미: CMake Tools 경로로 디버거 없이 실행
- 특징: 일반적으로 `Ctrl+F5`보다 오버헤드가 적어 더 빠르게 체감될 수 있음

---

## launch 기반 실행 vs CMake 실행 차이

### 1) launch 기반 (`F5`, `Ctrl+F5`)

`launch.json`의 현재 선택된 configuration을 실행합니다.

현재 설정:
- `program`: `${command:cmake.launchTargetPath}`
- `cwd`: `${command:cmake.launchTargetDirectory}`
- `.vscode/settings.json`: `"cmake.buildBeforeRun": true`

즉, 실행 전 빌드를 거친 뒤(증분 빌드) launch 규칙대로 실행합니다.

### 2) CMake 실행 (`Ctrl+Shift+F5`)

CMake Tools가 선택된 launch target을 직접 실행합니다.

- launch 인프라를 덜 거치므로 빠르게 느껴질 수 있음
- 현재 팀 워크플로우에서는 이 키를 "빠른 실행" 용도로 사용

---

## tasks.json과 launch.json 역할 분리

### tasks.json (빌드)

현재 task는 아래와 같은 의미입니다.

- 라벨: `CMake: build launch target`
- 실제 명령:
  - `cmake --build ${command:cmake.buildDirectory} --target ${command:cmake.launchTargetName}`

핵심:
- 항상 전체 재빌드가 아니라, CMake/빌드 시스템이 판단한 **증분 빌드(변경분 중심)** 를 수행

### launch.json (실행/디버그)

현재 configuration은 CMake Tools 변수를 이용해 실행 파일 경로를 동적으로 가져옵니다.

- 실행 파일: `${command:cmake.launchTargetPath}`
- 디버거: gdb (`miDebuggerPath`)
- 실행 전 빌드: CMake Tools 설정(`cmake.buildBeforeRun=true`)으로 일원화

### settings.json (CMake 실행 정책)

현재 프로젝트는 아래 설정으로 빌드 트리거를 1곳으로 통일합니다.

- `"cmake.buildBeforeRun": true`

의미:
- `F5`, `Ctrl+F5`, `Ctrl+Shift+F5`에서 CMake 경로로 실행할 때 실행 전 변경분 빌드를 수행
- `launch.json`의 `preLaunchTask`와 중복되지 않도록 `preLaunchTask`는 사용하지 않음

---

## "현재 선택된 launch configuration"이란?

- 열린(active) 파일이 아니라,
- Run and Debug 패널 상단 드롭다운에서 선택된 `launch.json` 항목을 뜻합니다.

이 프로젝트는 현재 configuration이 1개이므로 사실상 그 항목이 항상 선택됩니다.

---

## 이 프로젝트의 실행 타깃

`CMakeLists.txt`에서 다음 타깃을 정의합니다.

- `add_executable(hello_simple main.cpp)`

따라서 launch/CMake 실행 모두 기본적으로 `hello_simple`를 대상으로 동작합니다.

---

## 팀 운영 메모

- 전역 `keybindings.json` 변경은 프로젝트 범위를 넘으므로 사용하지 않음
- 프로젝트 실행은 `Ctrl+Shift+F5` 기준으로 사용
- 디버깅이 필요할 때만 `F5` 사용
