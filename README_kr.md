# ESP32-S3

**Language:** [English](./README.md) | **한국어**

---

# 00. 개발환경 셋업 — VS Code + PlatformIO (ESP32-S3)

Non-OS(bare-metal, RTOS 없이 `setup()`/`loop()` 구조) 기준 ESP32-S3 실습 전체 시리즈의 시작 파일입니다. 이 문서로 개발환경을 구축한 뒤, 목차 순서대로 진행하시면 됩니다.

## 사전 준비물

- PC (Windows / macOS / Linux)
- ESP32-S3 개발보드 (본 시리즈는 **ESP32-S3-DevKitC-1, N16R8** 기준으로 작성 — 16MB Quad Flash + 8MB Octal PSRAM)
- 데이터 전송을 지원하는 USB 케이블 (충전 전용 케이블 아님)

---

## Step 1. VS Code 설치

[code.visualstudio.com](https://code.visualstudio.com)에서 OS에 맞는 버전 설치. 이미 있다면 생략.

## Step 2. PlatformIO IDE 확장 설치

1. VS Code Extensions(`Ctrl+Shift+X`) → `PlatformIO IDE` 검색 → 설치
2. 설치 후 VS Code 재시작 (최초 로딩에 수 분 소요)
3. 좌측 Activity Bar에 PlatformIO 개미 아이콘이 보이면 설치 완료

## Step 3. 새 프로젝트 생성

1. PlatformIO Home → **New Project**
2. Board: `Espressif ESP32-S3-DevKitC-1` 검색 후 선택
3. Framework: **Arduino**
4. 생성 완료 시 `platformio.ini`, `src/main.cpp`가 자동 생성됨

## Step 4. N16R8 Flash/PSRAM 설정

기본 보드 정의는 N8(8MB Flash, PSRAM 없음) 기준입니다. N16R8(16MB Quad Flash + 8MB Octal PSRAM)을 쓰신다면 `platformio.ini`에 아래 설정을 추가하세요.

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200

; ---- N16R8 (16MB Quad Flash + 8MB Octal PSRAM) 설정 ----
board_build.flash_mode = qio
board_upload.flash_size = 16MB
board_build.partitions = default_16MB.csv
board_build.arduino.memory_type = qio_opi
build_flags =
    -DBOARD_HAS_PSRAM
```

> Flash는 Quad(`qio`), PSRAM은 Octal(`opi`)이라 `memory_type = qio_opi`를 씁니다. 다른 모듈(N16R8V, N32R8V 등)을 쓰신다면 Flash/PSRAM 구성이 다를 수 있으니 실물 각인을 확인하세요.

## Step 5. Hello World

```cpp
#include <Arduino.h>

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("Hello, ESP32-S3!");
}

void loop() {
  Serial.println("Running...");
  delay(1000);
}
```

Build(하단 상태바 체크 아이콘) → Upload(화살표 아이콘) → Serial Monitor(플러그 아이콘)로 확인.

## Step 6. 코어 버전 확인 (중요)

`ledcAttach` 같은 최신 PWM API를 쓸지, `ledcSetup`+`ledcAttachPin` 같은 구버전 API를 쓸지는 **설치된 Arduino-ESP32 코어 버전**에 따라 갈립니다. 아래 코드로 확인하세요.

```cpp
void setup() {
  Serial.begin(115200);
  delay(1000);
#if defined(ESP_ARDUINO_VERSION_MAJOR)
  Serial.printf("Arduino-ESP32 core: %d.%d.%d\n", ESP_ARDUINO_VERSION_MAJOR, ESP_ARDUINO_VERSION_MINOR, ESP_ARDUINO_VERSION_PATCH);
#endif
}
void loop() {}
```

- **core 2.x**: PWM은 `ledcSetup(channel, freq, res)` + `ledcAttachPin(pin, channel)` + `ledcWrite(channel, duty)`
- **core 3.x 이상**: PWM은 `ledcAttach(pin, freq, res)` + `ledcWrite(pin, duty)`

이 시리즈의 PWM 관련 예제는 **core 2.x(구 API) 기준으로 작성**되어 있습니다 — 만약 3.x를 쓰고 계시다면 신버전 API로 바꿔서 진행하세요 (예제마다 두 방식을 함께 표기).

## 확인용 체크리스트

- [ ] VS Code + PlatformIO IDE 설치
- [ ] 프로젝트 생성 및 N16R8 Flash/PSRAM 설정 반영
- [ ] Hello World 빌드/업로드/시리얼 출력 확인
- [ ] 사용 중인 Arduino-ESP32 코어 버전 확인

---
