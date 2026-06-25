# 프로젝트 1: Pick-and-Place 로봇 시뮬레이션

> **원본**: [Unity-Robotics-Hub Pick-and-Place Tutorial](https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/tutorials/pick_and_place/README.md)
>
> **목표**: Niryo One 로봇 팔을 Unity로 가져와 ROS + MoveIt 기반 Pick-and-Place 작업 수행

---

## 개요

이 프로젝트는 **Niryo One** 로봇 팔의 URDF를 Unity로 가져오고, ROS와 TCP로 통신하여 MoveIt 경로 계획을 통해 물체를 집고 옮기는(Pick-and-Place) 작업을 수행합니다.

### 전체 아키텍처

```
┌─────────────────────┐      ROS TCP       ┌─────────────────────┐
│   ROS (Noetic)      │ ◄─────────────────► │   Unity Editor      │
│                     │                     │                     │
│  ┌───────────────┐  │                     │  ┌───────────────┐  │
│  │ MoveIt        │  │                     │  │ Niryo One     │  │
│  │ (경로 계획)    │  │                     │  │ (URDF 임포트)  │  │
│  ├───────────────┤  │                     │  ├───────────────┤  │
│  │ Mover Service │──┼─────────────────────┼─►│ Trajectory    │  │
│  │ (trajectory)  │  │                     │  │ Planner.cs    │  │
│  ├───────────────┤  │                     │  ├───────────────┤  │
│  │ Server Endpoint│ │                     │  │ Articulation  │  │
│  │ (TCP bridge)  │  │                     │  │ Body          │  │
│  └───────────────┘  │                     │  └───────────────┘  │
└─────────────────────┘                     └─────────────────────┘
```

---

## Unity AI 활용 프로세스

### Phase 1: 프로젝트 설정 및 URDF 임포트

**Assistant 프롬프트:**

```
"로봇 Pick-and-Place 프로젝트를 설정해줘.
1. 새 3D 프로젝트 생성
2. URDF-Importer 패키지 설치
3. ROS-TCP-Connector 패키지 설치
4. 빈 씬 생성 후 테이블 프리팹 배치
5. 메인 카메라 위치를 (0, 1.4, -0.7), 회전 (45, 0, 0)으로 설정
6. Physics Solver Type을 'Temporal Gauss Seidel'로 변경"
```

**AI 동작:**

| 단계 | AI 작업 | 설명 |
|------|---------|------|
| 1 | 프로젝트 생성 | 새 3D 프로젝트 생성 |
| 2 | 패키지 설치 | Package Manager에서 `com.unity.robotics.urdf-importer` 설치 |
| 3 | 패키지 설치 | `com.unity.robotics.ros-tcp-connector` 설치 |
| 4 | 씬 설정 | 테이블, Target, TargetPlacement 프리팹 배치 |
| 5 | 카메라 설정 | Transform 값 조정 |
| 6 | 물리 설정 | Project Settings > Physics > Solver Type 변경 |

### Phase 2: URDF 로봇 임포트

**Assistant 프롬프트 (Plan Mode ON):**

```
"Plan Mode를 켜고 Niryo One 로봇을 URDF에서 임포트해줘.
URDF 파일 위치: Assets/URDF/niryo_one/niryo_one.urdf
임포트 옵션: Y축 기본, VHACD 메쉬 분해 사용
임포트 후 base_link의 Immovable을 켜고,
Controller 컴포넌트에서 Stiffness=10000, Damping=100, Force Limit=1000,
Speed=30, Acceleration=10으로 설정해줘"
```

**AI Plan 예시:**

```
Plan:
  ✅ 1. Assets > Import Robot from URDF 실행
  ✅ 2. niryo_one.urdf 선택, Y-axis, VHACD 옵션
  ✅ 3. 임포트 완료 후 niryo_one 오브젝트 선택
  ✅ 4. base_link > Immovable = true
  ✅ 5. Controller 컴포넌트 파라미터 설정
```

### Phase 3: ROS TCP 연결 설정

**Assistant 프롬프트:**

```
"ROS TCP 연결을 설정해줘:
1. ROS/IP 주소를 127.0.0.1로 설정
2. ROS TCP 포트를 10005로 설정
3. 'pose' 토픽을 구독하는 C# 스크립트 생성
4. SourceDestinationPublisher.cs 스크립트 생성
   - 조인트 상태를 ROS로 발행
   - UI 버튼 클릭 시 발행
5. 버튼 UI (Canvas) 생성
   - 'Publish' 텍스트 버튼
   - OnClick에 SourceDestinationPublisher.Publish() 연결"
```

**AI 생성 스크립트 예시:**

```csharp
// SourceDestinationPublisher.cs — AI Assistant 생성
using UnityEngine;
using Unity.Robotics.ROSTCPConnector;
using Unity.Robotics.ROSTCPConnector.ROSGeometry;
using RosMessageTypes.Geometry;
using RosMessageTypes.Std;
using RosMessageTypes.Sensor;

public class SourceDestinationPublisher : MonoBehaviour
{
    [Header("Publishing Settings")]
    public string topicName = "source_destination";
    public GameObject targetObject;
    public GameObject targetPlacement;

    private ROSConnection ros;

    void Start()
    {
        ros = ROSConnection.GetOrCreateInstance();
        ros.RegisterPublisher<MsgType>(topicName);
    }

    public void Publish()
    {
        // 조인트 상태 및 Pick/Place 포즈 전송
        Debug.Log("Publishing source and destination poses...");
        // ... (상세 구현)
    }
}
```

### Phase 4: Pick-and-Place 로직 구현

**Assistant 프롬프트:**

```
"Pick-and-Place 로직을 구현해줘:
1. TrajectoryPlanner.cs 생성
   - MoverService의 서비스 이름: 'niryo_moveit'
   - Pick Pose와 Place Pose 정의
   - 조인트 경로를 ArticulationBody xDrive.target에 할당
   - Gripper 열기/닫기 함수 포함
2. ROS 서비스 요청 구조체 MoverServiceRequest 생성
3. 응답 처리 콜백 TrajectoryResponse 구현
4. ExecuteTrajectories 코루틴 구현
   - 각 조인트 위치를 Rad2Deg 변환하여 할당
   - Grasp 포즈에서 CloseGripper() 호출
   - 시퀀스 종료 후 OpenGripper() 호출"
```

**AI 생성 핵심 코드:**

```csharp
// TrajectoryPlanner.cs — AI 핵심 생성
private IEnumerator ExecuteTrajectories(MoverServiceResponse response)
{
    if (response.trajectories == null) yield break;

    for (int poseIndex = 0;
         poseIndex < response.trajectories.Length; poseIndex++)
    {
        for (int jointConfigIndex = 0;
             jointConfigIndex < response.trajectories[poseIndex]
                 .joint_trajectory.points.Length;
             jointConfigIndex++)
        {
            var jointPositions = response.trajectories[poseIndex]
                .joint_trajectory.points[jointConfigIndex].positions;
            float[] result = jointPositions
                .Select(r => (float)r * Mathf.Rad2Deg).ToArray();

            for (int joint = 0;
                 joint < jointArticulationBodies.Length; joint++)
            {
                var drive = jointArticulationBodies[joint].xDrive;
                drive.target = result[joint];
                jointArticulationBodies[joint].xDrive = drive;
            }
            yield return new WaitForSeconds(jointAssignmentWait);
        }

        if (poseIndex == (int)Poses.Grasp)
            CloseGripper();

        yield return new WaitForSeconds(poseAssignmentWait);
    }
    OpenGripper();
}
```

### Phase 5: ROS 워크스페이스 설정

**Assistant 프롬프트:**

```
"ROS catkin 워크스페이스를 설정하는 스크립트를 생성해줘:
1. niryo_moveit 패키지 구조 생성
2. scripts/mover.py 생성
   - MoveIt move_group 초기화
   - plan_pick_and_place() 함수
   - Pre-grasp → Grasp → Pick-up → Place 4단계 경로 계획
3. launch/part_3.launch 생성
   - roscore, server_endpoint, Mover Service, MoveIt 실행
4. config/params.yaml 생성
   - ROS IP, TCP 포트 설정"
```

**AI 생성 ROS 스크립트 예시:**

```python
# mover.py — AI 생성
#!/usr/bin/env python3
import rospy
import moveit_commander
from moveit_commander import MoveGroupCommander, RobotCommander
from niryo_moveit.srv import MoverService, MoverServiceResponse
from trajectory_msgs.msg import JointTrajectoryPoint

def plan_trajectory(move_group, destination_pose, start_joint_angles):
    current_joint_state = JointState()
    current_joint_state.name = joint_names
    current_joint_state.position = start_joint_angles

    moveit_robot_state = RobotState()
    moveit_robot_state.joint_state = current_joint_state
    move_group.set_start_state(moveit_robot_state)
    move_group.set_pose_target(destination_pose)
    plan = move_group.plan()

    return plan

def plan_pick_and_place(req):
    phases = ['pre_grasp', 'grasp', 'pick_up', 'place']
    trajectories = []

    for phase in phases:
        pose = get_pose_for_phase(phase, req)
        traj = plan_trajectory(move_group, pose, req.joints_input)
        trajectories.append(traj)

    return MoverServiceResponse(trajectories=trajectories)
```

---

## 실행 순서

```
1. 터미널 1: ROS 실행
   $ roslaunch niryo_moveit part_3.launch
   
2. Unity Editor: Play 모드 진입

3. Unity UI: Publish 버튼 클릭
   → 조인트 상태가 ROS로 전송
   → MoveIt이 경로 계획
   → Trajectory를 Unity로 반환
   → 로봇이 물체 집기/옮기기 수행
```

---

## AI 활용 효과

| 작업 | 수동 소요 시간 | AI 활용 시간 | 차이 |
|------|--------------|-------------|------|
| 프로젝트 설정 + 패키지 설치 | 15분 | 3분 | 5x |
| URDF 임포트 + 설정 | 20분 | 5분 | 4x |
| C# 스크립트 3종 생성 | 60분 | 10분 | 6x |
| ROS 파이썬 스크립트 | 45분 | 8분 | 5.6x |
| 디버깅 및 테스트 | 30분 | 15분 | 2x |
| **전체** | **~170분** | **~41분** | **4.1x** |

---

## 문제 해결 (AI 활용)

```
Assistant: "로봇 조인트가 흔들리는데 원인을 찾아줘"
→ AI: Controller Stiffness=10000, Damping=100 확인 제안

Assistant: "MoveIt connection timeout 에러가 발생해"
→ AI: roslaunch 실행 여부, IP/포트 일치 확인 제안
```

---

## 참고

- **원본 튜토리얼**: [Pick-and-Place Tutorial](https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/tutorials/pick_and_place/README.md)
- **URDF Importer**: [github.com/Unity-Technologies/URDF-Importer](https://github.com/Unity-Technologies/URDF-Importer)
- **ROS-TCP-Connector**: [github.com/Unity-Technologies/ROS-TCP-Connector](https://github.com/Unity-Technologies/ROS-TCP-Connector)
- **Niryo One**: [niryo.com](https://niryo.com/niryo-one/)
- **MoveIt**: [moveit.ros.org](https://moveit.ros.org/)
