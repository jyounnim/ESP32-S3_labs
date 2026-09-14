# 02. 웨이크워드(음성 키워드) 감지

## ⚠️ 하드웨어 필요

DevKitC-1에는 마이크가 없습니다. **I2S 디지털 마이크**(예: INMP441, ICS-43434)가 필요합니다. I2S 방식은 아날로그 마이크보다 노이즈에 강하고 배선이 단순해서 TinyML 음성 예제에서 표준으로 씁니다.

일반적인 배선 (예시, 실제 사용 핀은 코드에서 지정):

| 마이크 핀 | ESP32-S3 GPIO(예시) |
|---|---|
| WS (LRCL) | GPIO15 |
| SCK (BCLK) | GPIO16 |
| SD (DOUT) | GPIO17 |
| L/R | GND (모노, 왼쪽 채널 고정) |
| VDD | 3.3V |
| GND | GND |

## 정직하게 말씀드릴 부분

`esp-tflite-micro`에 포함된 `micro_speech` 예제는 **"yes"/"no" 단 두 단어만** 인식하는 20KB짜리 데모 모델입니다. 이게 "웨이크워드 감지"의 기본 파이프라인(오디오 캡처 → 특징 추출 → 신경망 추론 → 판정)을 보여주는 표준 예제이긴 하지만, **"헤이 XX" 같은 커스텀 웨이크워드를 인식하려면 직접 모델을 학습**시켜야 합니다. 이 문서는 먼저 기본 예제로 파이프라인을 검증한 뒤, 커스텀 웨이크워드로 확장하는 경로까지 안내합니다.

## Step 1. micro_speech 예제로 파이프라인 검증

`00_LITERT_TFLM_SETUP.md`에서 만든 프로젝트에서:

```bash
cp -r managed_components/espressif__esp-tflite-micro/examples/micro_speech/main/* main/
```

`main/audio_provider.cc`(또는 유사한 이름의 파일)에서 I2S 핀 번호를 실제 배선에 맞게 수정합니다 — 정확한 파일 위치와 변수명은 받아온 예제 버전에 따라 다를 수 있으니, `main/` 폴더에서 `i2s`로 검색해 핀 설정 부분을 찾으세요.

```bash
idf.py build
idf.py -p <포트> flash monitor
```

마이크에 대고 "yes" 또는 "no"라고 말하면, 시리얼 모니터에 감지 결과와 확신도(score)가 출력됩니다.

```
Detected yes, score: 201
Detected no, score: 214
```

## Step 2. 커스텀 웨이크워드로 확장 — Edge Impulse

직접 데이터셋을 모으고 CNN을 설계하는 건 상당한 작업이라, 실무에서는 **Edge Impulse**(웹 기반 TinyML 플랫폼)로 이 과정을 크게 단축합니다.

1. [edgeimpulse.com](https://edgeimpulse.com)에 가입, 새 프로젝트 생성
2. **데이터 수집**: 스마트폰 앱 또는 ESP32-S3에 연결한 마이크로 원하는 웨이크워드(예: "헤이 에스프")를 수십~수백 번 녹음, "noise"/"unknown" 카테고리도 함께 수집 (오탐 방지에 중요)
3. **Impulse Design**: Audio(MFCC) 전처리 블록 + Classification(작은 CNN) 블록 구성
4. **학습 & 테스트**: 웹에서 자동으로 학습, 정확도/혼동행렬 확인
5. **Deployment**: "C++ Library" 또는 "TensorFlow Lite Micro" 형식으로 내보내기 → 다운로드한 라이브러리를 `main/` 폴더에 포함

Edge Impulse가 내보낸 코드는 `run_inference()`류의 함수를 제공하므로, Step 1에서 검증한 오디오 캡처 파이프라인에 yes/no 모델 대신 이 함수를 연결하면 됩니다.

## 관찰 포인트

- 웨이크워드 모델은 보통 "1초 분량의 오디오 슬라이딩 윈도우"를 200ms 간격으로 계속 추론합니다 — `FSS Technology`의 참고 벤치마크에 따르면 ESP32-S3에서 이런 소형 CNN(약 80KB, INT8)은 240MHz 기준 약 19ms 안에 추론이 끝나 실시간 처리에 충분히 여유가 있습니다
- "noise"/"unknown" 데이터를 충분히 모으지 않으면, 조용한 환경에서도 오탐(false positive)이 잦아집니다 — 실제 사용 환경의 배경 소음을 데이터셋에 포함시키는 게 정확도보다 더 중요한 경우가 많습니다
- 검출 후 바로 다음 검출까지 약간의 디바운스(예: 1~2초)를 코드에 넣어두면, 같은 발화 하나가 여러 윈도우에 걸쳐 중복 감지되는 것을 방지할 수 있습니다

## 트러블슈팅

| 증상 | 원인 / 해결 |
|---|---|
| 계속 "unknown"만 나옴 | I2S 핀 배선/설정 오류로 오디오 자체가 안 들어오고 있을 가능성 — 오디오 raw 값을 시리얼로 찍어서 마이크 신호가 실제로 들어오는지 먼저 확인 |
| yes/no 정확도가 낮음 | 마이크와 입 사이 거리, 배경 소음 확인 — 데모 모델은 조용한 환경 기준으로 학습되어 있음 |
| Edge Impulse 모델 통합 시 컴파일 에러 | Edge Impulse SDK와 `esp-tflite-micro`의 TFLM 버전이 다를 수 있음 — Edge Impulse는 자체 TFLM 사본을 함께 내보내므로, 두 버전이 충돌하지 않게 프로젝트 구조를 분리하는 게 안전 |

## 다음

03번 파일(`03_SENSOR_ANOMALY_LAB.md`)에서 진동/센서 이상 탐지를 다룹니다 — 기존 가변저항으로 바로 실습 가능합니다.
