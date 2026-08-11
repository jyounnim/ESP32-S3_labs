# Zephyr RTOS 커리큘럼 개요

`freertos_curriculum/`의 22개 실습과 동일한 주제를, 이번엔 Zephyr RTOS의 API와 사상에 맞춰 다시 구성한 시리즈입니다. 단순히 함수 이름만 바꾼 것이 아니라, **Zephyr만의 설계(협조적/선점형 스레드 구분, 통합된 k_poll, k_event 등)가 FreeRTOS와 실제로 어떻게 다른지** 확인하는 데 초점을 맞췄습니다.

> ⚠️ **PlatformIO는 ESP32 계열의 Zephyr 프레임워크를 지원하지 않습니다.** `platformio.ini`에 `framework = zephyr`를 지정해도 espressif32 플랫폼에서는 빌드되지 않는다는 점이 PlatformIO 커뮤니티 및 GitHub 이슈에서 확인되었습니다. 따라서 이 커리큘럼은 Zephyr 공식 도구인 **west** 기반으로 진행하며, VS Code는 "Zephyr IDE" 확장을 통해 에디터 겸 빌드/플래시 GUI로 사용합니다. (지금까지 GPIO/I2C/SPI/Wi-Fi/BLE 등 다른 커리큘럼에서 써온 PlatformIO + Arduino 프레임워크와는 완전히 다른 빌드 체계입니다)

## 사전 준비물

- PC (Windows / macOS / Linux, WSL2 권장 for Windows)
- Python 3, Git 설치되어 있을 것
- ESP32-S3 보드 (데이터 전송 지원 USB 케이블)
- 디스크 여유 공간 5GB 이상 (Zephyr SDK + 모듈 소스 다운로드용)
- 인터넷 연결 (최초 설치 시 다운로드 용량이 큼 — 네트워크 상태 확인 권장)

---

## Step 1. VS Code에 IDE for Zephyr Extension 설치(경로에 한글이 없는 경우에 가능)

1. VS Code Extensions(`Ctrl+Shift+X`)에서 `IDE for Zephy` 검색
<img width="500" height="100" alt="image" src="https://github.com/user-attachments/assets/822d1164-277b-44ba-9607-d19ecd86975f" />
2. 제작사 **mylonics**의 **IDE for Zephyr** Extension 설치
3. 설치 후 VS Code 재시작

---

## Step 2. west 환경 & Zephyr SDK 설치

### 방법 A — Zephyr IDE Extension 사용 (권장, GUI 기반)

1. 좌측 Activity Bar에서 Zephyr IDE 아이콘 클릭
2. Command Palette(`Ctrl+Shift+P`) → **Zephyr IDE: Setup Zephyr IDE** 실행
3. west workspace 초기화, Zephyr 소스 다운로드, SDK 설치가 자동으로 진행됨 (수 분~수십 분 소요, 네트워크 속도에 따라 다름)

### 방법 B — 수동 설치 (터미널 CLI)

```bash
pip install west
west init ~/zephyrproject
cd ~/zephyrproject
west update
west zephyr-export
pip install -r zephyr/scripts/requirements.txt
west sdk install
```

**설치 확인**:

```bash
west --version
west boards | grep esp32s3
```

`esp32s3_devkitc` 등 ESP32-S3 관련 보드 타겟이 출력되면 정상입니다. (Zephyr 버전에 따라 보드 타겟 이름이 `esp32s3_devkitc/esp32s3/procpu` 형태로 표기될 수 있습니다 — 정확한 이름은 이 명령으로 확인하세요.)

---

## Step 3. 새 프로젝트 생성

### Zephyr IDE 확장 사용 시

1. Zephyr IDE 패널 → **Create Project** 클릭
2. 템플릿 선택 (Hello World 또는 Blank)
3. Board 검색 → `esp32s3_devkitc` 계열 선택
4. 프로젝트 이름 입력 후 생성

### 프로젝트 구조

```
my_app/
├── CMakeLists.txt
├── prj.conf              ← 커널 설정 (Kconfig) — 이 커리큘럼의 CONFIG_EVENTS, CONFIG_PM 등이 여기 들어감
├── boards/
│   └── esp32s3_devkitc.overlay   ← 보드별 하드웨어 설정 (07번 실습 등에서 사용)
└── src/
    └── main.c
```

> ⚠️ 사용 중인 보드가 16MB Flash / 8MB PSRAM(N16R8)처럼 특정 메모리 구성이라면, `west build` 명령에 `-S flash-16M -S psram-8M` 같은 snippet 플래그를 추가하거나 overlay 파일에 반영해야 합니다. 보드 데이터시트를 먼저 확인하세요.

---

## Step 4. Hello World 작성 & 실행

### `src/main.c`

```c
#include <zephyr/kernel.h>

int main(void) {
    while (1) {
        printk("Hello, ESP32-S3! (Zephyr)\n");
        k_sleep(K_SECONDS(1));
    }
    return 0;
}
```

### 빌드 · 플래시 · 모니터

```bash
west build -b esp32s3_devkitc
west flash
west espressif monitor
```

Zephyr IDE 확장을 사용 중이라면 GUI의 **Build** / **Flash** / **Monitor** 버튼으로 동일하게 실행할 수 있습니다.

### 실행 & 확인

- 시리얼 모니터에 `Hello, ESP32-S3! (Zephyr)`가 1초 간격으로 출력되는지 확인
- 안 뜨면 보드의 RESET 버튼을 눌러보고, 그래도 안 되면 USB 케이블/포트를 확인

## 환경구축 체크리스트

- [ ] Zephyr IDE 확장 설치 및 west workspace 구축 (Step 1~2)
- [ ] `west boards | grep esp32s3`로 보드 타겟 이름 확인
- [ ] esp32s3_devkitc 보드로 새 프로젝트 생성 (Step 3)
- [ ] Hello World 코드 작성 → `west build` → `west flash` → 모니터 확인 (Step 4)

> `west sdk install`과 최초 `west update`는 다운로드 용량이 커서 시간이 걸릴 수 있습니다. 네트워크 상태를 미리 확인하세요.

## 트러블슈팅

| 증상 | 원인 / 해결 |
|---|---|
| `west boards`에 esp32s3 관련 보드가 없음 | `west update`가 완료되지 않았거나 Espressif HAL 모듈이 누락 — `west update` 재실행 |
| 빌드 시 Espressif HAL 관련 blob 에러 | Espressif HAL은 RF 관련 바이너리 blob이 별도로 필요 — `west blobs fetch hal_espressif` 실행 |
| west build 실패 (보드를 찾을 수 없음) | 정확한 보드 타겟 이름을 `west boards \| grep esp32s3`로 재확인 (Zephyr 버전에 따라 표기 형식이 다를 수 있음) |
| 새 터미널에서 `west: command not found` | west 환경이 활성화되지 않음 — `source ~/zephyrproject/zephyr/zephyr-env.sh`(또는 사용한 venv) 재실행 |
| 시리얼 출력이 안 보임 | `west espressif monitor` 대신 `screen`/`minicom`으로 baud rate 115200 확인, RESET 버튼으로 재시작 |

---

## FreeRTOS와의 첫 번째 큰 차이 — 우선순위 숫자

**Zephyr는 숫자가 작을수록 우선순위가 높습니다** (FreeRTOS와 정반대). 게다가 **음수 우선순위는 "협조적(cooperative)" 스레드**, **0 이상은 "선점형(preemptible)" 스레드**로 아예 종류가 나뉩니다. 이건 FreeRTOS에는 없는 Zephyr만의 핵심 개념이라, 커리큘럼 초반부(02, 04번)에서 집중적으로 다룹니다.

## 공통 사항 (01번부터 적용)

- 대부분 `printk()`만으로 확인 가능하고, 하드웨어 배선이 필요한 실습(07번)은 별도 명시
- 코드의 출력 문자열은 영어로 작성되어 있습니다
- 이후 실습들의 빌드 명령은 위 Step 4와 동일하게 `west build -b esp32s3_devkitc`(또는 `esp32s3_devkitc/esp32s3/procpu`) → `west flash` → `west espressif monitor` 순서입니다

## 목차

| 번호 | 파일 | 주제 | FreeRTOS 커리큘럼 대응 |
|---|---|---|---|
| 01 | `01_THREAD_CREATION_LAB.md` | Thread 생성 기초 (K_THREAD_DEFINE / k_thread_create) | 01 |
| 02 | `02_THREAD_PRIORITY_LAB.md` | 우선순위 체계와 협조적/선점형 스레드 | 02 |
| 03 | `03_THREAD_LIFECYCLE_LAB.md` | Thread 동적 생성/종료 | 03 |
| 04 | `04_COOPERATIVE_YIELD_LAB.md` | 협조적 스레드와 k_yield — 반드시 양보해야 하는 이유 | 04 |
| 05 | `05_PRIORITY_INVERSION_LAB.md` | 우선순위 역전 재현 | 05 |
| 06 | `06_IDLE_THREAD_LAB.md` | Idle Thread와 CPU 유휴 시간 | 06 |
| 07 | `07_ISR_SEMAPHORE_LAB.md` | 인터럽트(ISR) + k_sem | 07 |
| 08 | `08_COUNTING_SEMAPHORE_LAB.md` | Counting Semaphore | 08 |
| 09 | `09_MUTEX_LAB.md` | k_mutex vs k_sem, Priority Inheritance | 09 |
| 10 | `10_MSGQ_BASICS_LAB.md` | Message Queue 기본 (k_msgq) | 10 |
| 11 | `11_K_POLL_LAB.md` | k_poll — 여러 커널 객체 동시 대기 | 11 (Queue Set) |
| 12 | `12_POLL_SIGNAL_LAB.md` | Poll Signal — 경량 이벤트 | 12 (Task Notification) |
| 13 | `13_K_EVENT_LAB.md` | k_event — 다중 조건 대기 | 13 (Event Group) |
| 14 | `14_K_TIMER_LAB.md` | k_timer (One-shot / Periodic) | 14 |
| 15 | `15_STACK_MONITORING_LAB.md` | 스택 사용량 모니터링 | 15 |
| 16 | `16_DEADLOCK_LAB.md` | Deadlock 재현과 회피 | 16 |
| 17 | `17_CRITICAL_SECTION_LAB.md` | irq_lock / k_sched_lock / k_spinlock | 17 |
| 18 | `18_MULTICORE_REALITY_LAB.md` | ESP32-S3에서의 멀티코어 — AMP vs SMP | 18 |
| 19 | `19_POWER_MANAGEMENT_LAB.md` | Zephyr Power Management (prj.conf) | 19 |
| 20 | `20_RUNTIME_STATS_LAB.md` | Thread Runtime Stats — 정식 CPU 사용률 API | 20 |
| 21 | `21_PRODUCER_CONSUMER_LAB.md` | Producer-Consumer 종합 패턴 | 21 |
| 22 | `22_ZEPHYR_VS_FREERTOS_LAB.md` | Zephyr vs FreeRTOS 종합 비교 | 22 |

## 학습 흐름

- **01~06**: Thread 자체의 특성 — 특히 Zephyr 고유의 협조적/선점형 구분
- **07~13**: 동기화/통신 수단 — k_sem/k_mutex/k_msgq는 FreeRTOS와 유사하지만, k_poll·k_event는 Zephyr가 더 통합적으로 설계한 부분
- **14~17**: 타이밍 제어와 자원 보호
- **18~20**: ESP32-S3에서 Zephyr가 실제로 어떻게 다르게 동작하는지 (특히 18번 — 멀티코어 모델 자체가 FreeRTOS/ESP-IDF와 다릅니다)
- **21~22**: 종합 응용 및 FreeRTOS 커리큘럼과의 최종 비교
