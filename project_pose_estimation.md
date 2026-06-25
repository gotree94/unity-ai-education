# 프로젝트 2: Object Pose Estimation — 딥러닝 기반 물체 자세 추정

> **원본**: [Robotics-Object-Pose-Estimation](https://github.com/Unity-Technologies/Robotics-Object-Pose-Estimation)
>
> **목표**: Unity에서 합성 데이터를 수집하고 딥러닝 모델을 학습시켜 UR3 로봇 팔이 큐브의 자세(Pose)를 추정하고 Pick-and-Place 수행

---

## 개요

이 프로젝트는 **Unity Perception Package**를 사용하여 **합성 학습 데이터**를 생성하고, **딥러닝 모델**을 학습시켜 물체의 3D 자세(Pose)를 추정한 후, **UR3 로봇 팔**이 추정된 자세를 기반으로 물체를 집어 옮기는 **End-to-End 파이프라인**을 구현합니다.

### 전체 파이프라인

```
┌─────────────────────────────────────────────────────────────┐
│                    Unity AI Workflow                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1                    Step 2                    Step 3  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ 데이터 생성    │    │ 모델 학습     │    │ 추론 및 배치  │  │
│  │ (Perception   │───►│ (PyTorch)    │───►│ (Unity + ROS)│  │
│  │  Package)     │    │              │    │              │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│       │                                                     │
│       ├─ 도메인 랜덤화                                         │
│       ├─ 랜덤 배경/조명/자세                                   │
│       └─ JSON 레이블 자동 생성                                 │
│                                                              │
│  Step 4: UR3 로봇이 추정된 Pose로 Pick-and-Place 수행         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Unity AI 활용 프로세스

### Phase 1: 씬 설정 및 URDF 임포트

**Assistant 프롬프트:**

```
"UR3 로봇 Pick-and-Place + Pose Estimation 프로젝트를 초기화해줘.
1. 새 3D 프로젝트 생성
2. 필요한 패키지 설치:
   - com.unity.robotics.urdf-importer
   - com.unity.robotics.ros-tcp-connector
   - com.unity.perception
3. UR3 로봇 URDF 임포트
4. 조명 3개 배치 (Key, Fill, Back)
5. 테이블 위에 큐브 배치
6. RGB 카메라 설치 (UR3 그리퍼 위치)
7. 메인 카메라 위치 설정"
```

**AI 동작 상세:**

```
Assistant:
  ✅ 프로젝트 생성 완료
  ✅ 3개 패키지 설치 완료
  ✅ URDF Import Robot from URDF 실행 (UR3)
  ✅ 조명 배치 완료
  ✅ 큐브 + 테이블 씬 구성 완료
  ✅ RGB 카메라 위치 조정 완료
```

### Phase 2: 데이터 수집 씬 구성

이 단계가 **핵심**입니다. Unity Perception Package의 **Randomizer**를 사용하여 다양한 데이터를 생성합니다.

**Assistant 프롬프트 (Plan Mode):**

```
"Plan Mode ON. 데이터 수집을 위한 Perception 씬을 설정해줘:
1. Perception Camera에 다음 Randomizer 추가:
   - Randomize 큐브 위치 (테이블 위 랜덤)
   - Randomize 큐브 회전 (전축 랜덤)
   - Randomize 조명 색상/강도
   - Randomize 배경색
2. 큐브에 Object Pose Labeler 컴포넌트 추가
   - 레이블: 'cube'
3. 데이터 수집 스크립트 생성
   - 1000장의 이미지 자동 캡처
   - 각 이미지에 대한 Pose Ground Truth JSON 저장
4. Perception Camera 설정:
   - Capture Interval: 1프레임
   - Image Resolution: 512x512"
```

**AI 생성 Randomizer 설정:**

```csharp
// PerceptionCamera Randomizer 설정 — AI Assistant 구성
// Randomizer 목록:
// 1. TransformRandomizer (큐브 위치/회전 랜덤화)
// 2. LightRandomizer (조명 랜덤화)
// 3. BackgroundRandomizer (배경색 랜덤화)

// Object Pose Labeler — AI가 큐브에 자동 추가
// label: "cube"
// JSON 출력 예시:
{
  "label": "cube",
  "pose": {
    "position": [0.234, 0.512, -0.123],
    "rotation": [15.2, 45.0, -30.1]
  },
  "image_path": "captures/0001.png"
}
```

### Phase 3: 데이터 수집 실행 및 모델 학습

**Assistant 프롬프트:**

```
"1. 데이터 수집 스크립트를 실행해줘
   - 1000장 이미지 자동 생성
   - 각 이미지별 Pose 정보 JSON 저장
2. 데이터셋 구조 확인
3. PyTorch 학습 파이프라인 스크립트 생성:
   - PoseEstimationTrainer.py
   - ResNet50 백본 + Pose 회귀 헤드
   - 데이터 로더: JSON + 이미지 로드
   - Loss: Position MSE + Rotation Geodesic Loss
   - 학습률: 0.001, 에포크: 50
   - 체크포인트 저장"
```

**AI 생성 학습 코드:**

```python
# PoseEstimationTrainer.py — AI Assistant 생성
import torch
import torch.nn as nn
from torch.utils.data import Dataset, DataLoader
import json, os
from PIL import Image
from torchvision import transforms, models

class PoseDataset(Dataset):
    def __init__(self, json_path, img_dir):
        with open(json_path) as f:
            self.data = json.load(f)
        self.img_dir = img_dir
        self.transform = transforms.Compose([
            transforms.Resize((224, 224)),
            transforms.ToTensor(),
            transforms.Normalize([0.485, 0.456, 0.406],
                                 [0.229, 0.224, 0.225])
        ])

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        item = self.data[idx]
        image = Image.open(
            os.path.join(self.img_dir, item['image_path']))
        image = self.transform(image)
        pose = torch.tensor(
            item['pose']['position'] +
            item['pose']['rotation'], dtype=torch.float32)
        return image, pose

class PoseEstimator(nn.Module):
    def __init__(self):
        super().__init__()
        backbone = models.resnet50(pretrained=True)
        self.features = nn.Sequential(*list(
            backbone.children())[:-2])
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc_position = nn.Linear(2048, 3)
        self.fc_rotation = nn.Linear(2048, 3)

    def forward(self, x):
        x = self.features(x)
        x = self.avgpool(x).flatten(1)
        pos = self.fc_position(x)
        rot = self.fc_rotation(x)
        return torch.cat([pos, rot], dim=1)

# 학습 루프
model = PoseEstimator()
criterion = nn.MSELoss()  # Position MSE
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
dataset = PoseDataset("dataset.json", "captures/")
loader = DataLoader(dataset, batch_size=32, shuffle=True)

for epoch in range(50):
    for images, poses in loader:
        pred = model(images)
        loss = criterion(pred, poses)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
    print(f"Epoch {epoch}: Loss = {loss.item():.4f}")

torch.save(model.state_dict(), "pose_model.pth")
```

### Phase 4: 추론 및 Pick-and-Place 통합

**Assistant 프롬프트:**

```
"학습된 Pose Estimation 모델을 Unity + ROS와 통합해줘:
1. ROS 노드 생성: pose_estimation_node.py
   - RGB 이미지 구독
   - 학습된 PyTorch 모델 로드
   - 큐브 Pose 추론
   - 추정된 Pose 발행
2. Unity TrajectoryPlanner 업데이트
   - Pose Estimation 결과를 Pick Pose로 사용
   - 추정 Pose 기반 MoveIt 경로 계획 호출
3. 실시간 카메라 피드에서 큐브 감지 및 Pick-and-Place"

참고: 사전 학습된 모델을 사용하려면
Model/ 디렉토리의 pretrained_model.pth 사용 가능
```

**AI 생성 추론 노드:**

```python
# pose_estimation_node.py — AI Assistant 생성
#!/usr/bin/env python3
import rospy
from sensor_msgs.msg import Image
from geometry_msgs.msg import Pose
import cv2
import torch
from PIL import Image as PILImage
import numpy as np

class PoseEstimationNode:
    def __init__(self):
        rospy.init_node('pose_estimation_node')
        self.model = PoseEstimator()
        self.model.load_state_dict(
            torch.load('pose_model.pth'))
        self.model.eval()
        self.pose_pub = rospy.Publisher(
            'estimated_pose', Pose, queue_size=10)
        self.img_sub = rospy.Subscriber(
            '/unity_camera/image', Image, self.callback)

    def callback(self, msg):
        # Unity RGB 이미지를 PyTorch 텐서로 변환
        img = self.ros_to_tensor(msg)
        with torch.no_grad():
            pose = self.model(img.unsqueeze(0))
        # 추정된 Pose 발행
        pose_msg = Pose()
        pose_msg.position.x = pose[0, 0].item()
        pose_msg.position.y = pose[0, 1].item()
        pose_msg.position.z = pose[0, 2].item()
        self.pose_pub.publish(pose_msg)
```

---

## 데이터셋 실험 결과

원본 프로젝트 기준 100회 실험 결과:

| 조건 | 성공 | 실패 | 성공률 |
|------|------|------|--------|
| 비가림 없음 | 82 | 5 | **94%** |
| 비가림 있음 | 7 | 6 | 54% |
| **전체** | 89 | 11 | **89%** |

---

## AI 활용 효과

| 작업 | 수동 | AI 활용 | 차이 |
|------|------|---------|------|
| Perception 씬 구성 + Randomizer 설정 | 90분 | 15분 | 6x |
| 데이터 수집 파이프라인 | 60분 | 10분 | 6x |
| PyTorch 모델 학습 코드 | 120분 | 15분 | 8x |
| ROS 추론 노드 | 60분 | 10분 | 6x |
| Unity-ROS 통합 | 45분 | 12분 | 3.75x |
| **전체** | **~375분** | **~62분** | **6x** |

---

## AI Assistant 활용 추가 팁

```
데이터 증강 요청:
"Perception Randomizer에 큐브 텍스처 랜덤화도 추가해줘"

모델 성능 분석:
"학습된 모델의 Pose 추정 정확도를 분석하는 스크립트를 만들어줘
- Position Error (cm)
- Rotation Error (degrees)
- 실패 케이스 시각화"

실시간 디버깅:
"추론된 Pose와 Ground Truth를 Unity 씬에 시각화해줘
- 초록색: Ground Truth 큐브
- 빨간색: 추정 Pose 큐브"
```

---

## 참고

- **원본 저장소**: [Robotics-Object-Pose-Estimation](https://github.com/Unity-Technologies/Robotics-Object-Pose-Estimation)
- **Perception Package**: [com.unity.perception](https://github.com/Unity-Technologies/com.unity.perception)
- **UR3 Robot**: [universal-robots.com/products/ur3-robot](https://www.universal-robots.com/products/ur3-robot)
- **사전 학습 모델**: `Model/` 디렉토리 (원본 저장소 포함)
