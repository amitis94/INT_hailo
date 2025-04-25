# Step

1. PC
2. Raspberry Pi5

## PC

### 수행 동작

1. Object Detection(Classification 등) 모델 학습(GPU 사용 가능)
2. ONNX 기반 모델 .hef 변환 (일부 모델은 tflite 및 tfrecord 활용해야 하는 경우 존재)

### 요구 환경

참조 : [https://note.mmmsk.myds.me/Projects/Embedded-AI/(Hailo)-Hailo-Dataflow-Compiler-설치](https://note.mmmsk.myds.me/Projects/Embedded-AI/(Hailo)-Hailo-Dataflow-Compiler-%EC%84%A4%EC%B9%98)

1. Hailo Dataflow Compiler (version 3.28)
참조 : https://hailo.ai/developer-zone/software-downloads/
venv requirements.txt : 
    
    [requirements.txt](attachment:476fbf7b-7eb8-4b7f-b135-1a7c2b2bf462:requirements.txt)
    

1. Hailo Model Zoo
참조 : https://github.com/hailo-ai/hailo_model_zoo

### 모델 변환 방법

의견 1 : https://docs.ultralytics.com/ko/models/yolov8/에서 제공하는 일반적인 Object Detection 학습 방법 활용 후 `onnx` 내보내기 한 파일을 사용해도 관계 없음(YOLOv8, opset version 11인 환경에서 성공)

의견 2 : Model Zoo 활용하는 가이드 https://github.com/hailo-ai/hailo_model_zoo/tree/master/training 참조하여 진행 가능

1. `onnx` 파일 준비
2. Hailo Dataflow Compiler 내 python venv 설치 후 `onnx` 파일을 `hef` 변환 명령어 실행

```python
hailomz compile \
  --ckpt models.onnx \
  --calib-path IMAGE_FOLDER_PATH \
  --yaml models.yaml \
  --classes 999 \
  --hw-arch=hailo8
```

- ckpt : `onnx` 파일 경로
- calib-path : quantization 활용 목적 학습된 이미지 폴더 경로
- yaml : Hailo Model Zoo에서 특정 모델 (가이드에서는 YOLOv8 기준)의 `yaml` 파일 경로
- classes : 모델에 학습 시킨 클래스 수
- hw-arch : hailo 모델 버전에 맞는 값 입력(default : hailo8)

`yaml` 파일 내 설정 값으로 들어있는 `alls` 수정이 필요할 수 있음, 질문 후 답변을 받은 내용 참조
(https://community.hailo.ai/t/model-training-gpu/13623/5)

여기까지 성공했다면 `hef` 파일이 생성됐음

관련 질의 사항은 https://community.hailo.ai/ 에서 직접 질문하고 답변 받아야 함

## Raspberry Pi5

### 수행 동작

1. `hef` 파일로 만들어진 모델 Hailo NPU 추론

### 요구 환경

참조 : [https://note.mmmsk.myds.me/Projects/Embedded-AI/(Hailo)-Hailo-Dataflow-Compiler-설치](https://note.mmmsk.myds.me/Projects/Embedded-AI/(Hailo)-Hailo-Dataflow-Compiler-%EC%84%A4%EC%B9%98)

1. HailoRT (version 4.20)
참조 : https://hailo.ai/developer-zone/software-downloads/

1. Hailo-Application-Code-Examples
참조 : https://github.com/hailo-ai/Hailo-Application-Code-Examples

### Object Detection 추론
참조 : [GitHub Hailo-Application-Code-Examples/runtime/python/object_detect…](https://github.com/hailo-ai/Hailo-Application-Code-Examples/tree/main/runtime/python/object_detection)
```python
./object_detection.py \
	-n models.hef \
	-i inference_image.jpeg \
	-l model_trained_labels.txt
```

- n : `hef`모델 파일 경로
- -i : 입력 이미지(연결된 카메라나 특정 폴더 내 이미지들 전체 추론 가능, RAM 용량에 영향)
- -l : 모델 학습에 쓰였던 라벨 파일(coco.txt로 저장된 형식 참조)
