# 2025 Capstone Climbing Beta

## 요약

YOLO 기반 클라이밍 홀드 검출과 MediaPipe 기반 사용자 관절 추적을 결합하여, 사용자의 현재 위치와 목표 홀드의 관계를 분석하고 레이저로 다음 동작을 안내하는 초보자용 볼더링 보조 시스템 개발

- **프로젝트명**: Climbing Beta
- **주제**: 초보자를 위한 AI 기반 클라이밍 루트 안내 및 성공/실패 판단 시스템
- **개발 환경**
  - NVIDIA Jetson Orin
  - Intel RealSense
  - Arduino
  - Ubuntu
  - Python
  - C++
- **구현 기술**
  - YOLO 기반 클라이밍 홀드 검출
  - MediaPipe 기반 사용자 관절 추적
  - Intel RealSense 기반 공간 위치 추정
  - YOLO 검출 결과와 사용자 위치 데이터 융합
  - Arduino + MG996R 서보모터 기반 레이저 제어
  - gTTS 기반 음성 안내
- **데이터**
  - 직접 촬영 및 세그멘테이션한 클라이밍 홀드 이미지 3,700장 이상
  - 13회 이상의 클라이밍장 방문을 통한 데이터 수집
- **성과**
  - 200개 이상의 테스트 데이터에서 약 98%의 홀드 인식 정확도 달성

## 개요

Climbing Beta는 클라이밍 초보자가 루트를 쉽게 이해하고 수행할 수 있도록 **카메라를 통해 클라이밍 홀드와 사용자의 위치를 실시간으로 인식하고, 다음 목표 홀드를 레이저로 안내하는 시스템**이다.

YOLO를 활용하여 클라이밍 벽에 존재하는 홀드의 위치와 색상을 검출하고, MediaPipe를 이용하여 사용자의 주요 관절 위치를 추적한다.

이후 Intel RealSense의 깊이 정보를 활용하여 검출된 홀드와 사용자의 상대적인 위치를 분석하고, 현재 사용자의 동작이 목표 루트를 정상적으로 수행하고 있는지를 판단한다.

최종적으로 Arduino와 MG996R 서보모터를 이용하여 레이저의 방향을 제어하고, gTTS를 통해 사용자에게 음성 안내를 제공한다.

![Climbing Beta Overview](images/climbing_beta_overview.png)

---

## 구현 기술

### 1. YOLO 기반 클라이밍 홀드 검출

클라이밍 벽에 존재하는 홀드를 자동으로 인식하기 위해 YOLO 기반 객체 검출 모델을 적용하였다.

실제 클라이밍장에서 직접 데이터를 수집하고 홀드 영역을 세그멘테이션하여 학습 데이터셋을 구축하였다.

총 **3,700장 이상의 클라이밍 홀드 이미지를 직접 수집 및 전처리**하였으며, 다양한 조명과 홀드 형태에 대응할 수 있도록 데이터를 구성하였다.

검출된 홀드에 대해서는 위치와 색상 정보를 추출하여 이후 사용자의 위치와 목표 루트 판단에 활용하였다.

![YOLO Hold Detection](images/yolo_hold_detection.png)

```python
# YOLO 기반 홀드 검출 예시

results = model(frame)

for result in results:
    boxes = result.boxes

    for box in boxes:
        x1, y1, x2, y2 = box.xyxy[0]
        confidence = box.conf[0]

        if confidence > CONF_THRESHOLD:
            hold_center = (
                (x1 + x2) / 2,
                (y1 + y2) / 2
            )