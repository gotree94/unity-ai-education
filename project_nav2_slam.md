# 프로젝트 3: Nav2 SLAM — 자율주행 로봇 매핑 및 위치추정

> **원본**: [Robotics-Nav2-SLAM-Example](https://github.com/Unity-Technologies/Robotics-Nav2-SLAM-Example)
>
> **목표**: Unity 환경을 Gazebo 대신 사용하여 Turtlebot3 로봇으로 Navigation2의 SLAM 튜토리얼 수행

---

## 개요

이 프로젝트는 **Unity**를 **Gazebo** 대체 시뮬레이션 환경으로 사용하여, **Turtlebot3** 로봇이 **Navigation2 (Nav2) SLAM**을 수행하도록 합니다. Unity Robotics Warehouse 환경에서 LiDAR 스캔을 생성하고, ROS 2를 통해 Nav2 SLAM 시스템과 통신합니다.

### 전체 아키텍쳐

```
┌──────────────────────────────────────────────────┐
│                  Docker / Host                     │
│  ┌─────────────────────┐  ┌──────────────────┐   │
│  │   ROS 2 (Galactic)   │  │  Unity Editor    │   │
│  │                      │  │                  │   │
│  │  ┌───────────────┐  │  │  ┌────────────┐  │   │
│  │  │ Nav2 SLAM     │  │  │  │ Robotics    │  │   │
│  │  │ - map_server  │──┼──┼─►│ Warehouse   │  │   │
│  │  │ - amcl        │  │  │  │ 환경        │  │   │
│  │  │ - slam_toolbox│  │  │  ├────────────┤  │   │
│  │  ├───────────────┤  │  │  │ LiDAR      │  │   │
│  │  │ Turtlebot3    │  │  │  │ (Raycast)  │  │   │
│  │  │ (Model/제어)  │◄─┼──┼──┤ 시뮬레이터  │  │   │
│  │  ├───────────────┤  │  │  ├────────────┤  │   │
│  │  │ ROS TCP       │  │  │  │ ROS TCP    │  │   │
│  │  │ Endpoint      │◄─┼──┼──► Connector  │  │   │
│  │  └───────────────┘  │  │  └────────────┘  │   │
│  └─────────────────────┘  └──────────────────┘   │
└──────────────────────────────────────────────────┘
```

---

## Unity AI 활용 프로세스

### Phase 1: ROS 2 + Unity 개발 환경 설정

**Assistant 프롬프트:**

```
"Nav2 SLAM 프로젝트 개발 환경을 설정해줘.
1. Docker 환경 설정 스크립트 생성:
   - ROS 2 Galactic 기반
   - Nav2, slam_toolbox, turtlebot3 패키지 포함
   - Unity ROS TCP endpoint 설정
2. Unity 프로젝트 생성:
   - Robotics Warehouse 패키지 설치
   - ROS-TCP-Connector 패키지 설치
   - Visualizations 패키지 설치
3. docker-compose.yml 생성:
   - ros2_master 서비스
   - Unity 서비스 (선택)"
```

**AI 생성 Docker 설정:**

```yaml
# docker-compose.yml — AI Assistant 생성
version: '3.8'
services:
  ros2:
    image: ros:galactic-ros-base
    container_name: ros2_nav2
    network_mode: host
    environment:
      - ROS_DOMAIN_ID=0
      - RMW_IMPLEMENTATION=rmw_fastrtps_cpp
    volumes:
      - ./ros2_ws:/ros2_ws
      - ./unity_scripts:/unity_scripts
    command: >
      bash -c "cd /ros2_ws &&
               colcon build &&
               source install/setup.bash &&
               ros2 launch nav2_bringup slam_launch.py"
```

### Phase 2: Unity Warehouse 환경 설정

**Assistant 프롬프트 (Plan Mode ON):**

```
"Plan Mode ON. Robotics Warehouse 환경을 설정해줘:
1. Robotics Warehouse 프리팹을 씬에 배치
2. Warehouse 크기: 20x20 미터
3. 선반과 장애물 랜덤 배치 활성화
4. Turtlebot3 URDF 임포트
   - base_link 위치: (0, 0, 0)
   - LiDAR 링크에 Raycast Sensor 컴포넌트 추가
5. 바닥에 NavMesh Surface 추가
6. 조명 설정 (환경 전체 균일 조명)"
```

**AI 구성 상세:**

```
Assistant Plan:
  ✅ Robotics Warehouse: Assets/Prefabs/Warehouse 배치
  ✅ Warehouse 크기 20x20 설정 완료
  ✅ Random Obstacle Generation 활성화
  ✅ Turtlebot3 URDF Import 완료
  ✅ LiDAR Sensor: RaycastSensor.cs 컴포넌트
      - Range: 12m
      - Angle: -135° ~ 135°
      - Resolution: 1°
  ✅ NavMesh Surface: 바닥에 Bake 완료
```

### Phase 3: LiDAR 및 센서 시뮬레이션

**Assistant 프롬프트:**

```
"LiDAR 센서 스크립트를 생성해줘:
1. RaycastSensor.cs:
   - 270도 스캔 (1도 간격)
   - 최대 거리 12m
   - sensor_msgs/LaserScan 발행
   - ROS 2 토픽: /scan
   - 주파수: 10Hz
2. OdometryPublisher.cs:
   - Turtlebot3의 속도/위치 추적
   - nav_msgs/Odometry 발행
   - 토픽: /odom
3. VelocitySubscriber.cs:
   - geometry_msgs/Twist 구독
   - /cmd_vel 토픽
   - Unity 물리 기반 이동 구현
4. TF 브로드캐스터 추가
   - odom → base_link 변환"
```

**AI 생성 LiDAR 스크립트:**

```csharp
// RaycastSensor.cs — AI Assistant 생성
using UnityEngine;
using Unity.Robotics.ROSTCPConnector;
using RosMessageTypes.Sensor;
using RosMessageTypes.Std;

public class RaycastSensor : MonoBehaviour
{
    [Header("LiDAR Configuration")]
    public float maxRange = 12f;
    public int numRays = 271;  // -135° to 135°
    public float frequency = 10f;

    private ROSConnection ros;
    private string topicName = "/scan";
    private float timer;

    void Start()
    {
        ros = ROSConnection.GetOrCreateInstance();
        ros.RegisterPublisher<LaserScanMsg>(topicName);
    }

    void Update()
    {
        timer += Time.deltaTime;
        if (timer < 1f / frequency) return;
        timer = 0f;

        PublishLaserScan();
    }

    void PublishLaserScan()
    {
        var scan = new LaserScanMsg
        {
            header = new HeaderMsg
            {
                frame_id = "base_scan",
                stamp = new TimeMsg(
                    (uint)Time.time, 0)
            },
            angle_min = -135f * Mathf.Deg2Rad,
            angle_max = 135f * Mathf.Deg2Rad,
            angle_increment = 270f / (numRays - 1)
                           * Mathf.Deg2Rad,
            time_increment = 0f,
            scan_time = 1f / frequency,
            range_min = 0.01f,
            range_max = maxRange,
            ranges = new float[numRays],
            intensities = new float[numRays]
        };

        for (int i = 0; i < numRays; i++)
        {
            float angle = -135f + (270f * i / (numRays - 1));
            Vector3 dir = Quaternion.Euler(0, angle, 0)
                        * transform.forward;

            if (Physics.Raycast(transform.position, dir,
                    out RaycastHit hit, maxRange))
            {
                scan.ranges[i] = hit.distance;
                scan.intensities[i] = 1f;
            }
            else
            {
                scan.ranges[i] = maxRange;
                scan.intensities[i] = 0f;
            }
        }

        ros.Publish(topicName, scan);
    }
}
```

### Phase 4: Nav2 SLAM 실행

**Assistant 프롬프트:**

```
"Nav2 SLAM 실행 스크립트를 생성해줘:
1. Unity-ROS 연결 검증 스크립트:
   - /scan, /odom, /cmd_vel 토픽 통신 확인
2. Turtlebot3 제어 스크립트:
   - 키보드로 /cmd_vel 발행 (ros2 teleop)
   - 또는 Nav2 nav_through_poses 자동 주행
3. SLAM 시각화:
   - Unity Visualizations 패키지로
   - 실시간 맵 데이터(/map 표시)
   - 로봇 궤적 표시
   - LiDAR 스캔 포인트 시각화
4. Docker 실행 명령 확인"
```

**AI 생성 실행 워크플로우:**

```bash
# Terminal 1: Docker + ROS 2 실행
docker compose up -d

# Terminal 2: Unity Editor 실행 (수동)
# Unity Hub에서 Nav2SLAMExampleProject 열기
# Play 모드 진입

# Terminal 3: Nav2 SLAM 실행
docker exec -it ros2_nav2 bash
source /ros2_ws/install/setup.bash
ros2 launch nav2_bringup slam_launch.py \
    slam:=True \
    use_sim_time:=True

# Terminal 4: 로봇 제어 (teleop)
docker exec -it ros2_nav2 bash
ros2 run turtlebot3_teleop teleop_keyboard
# 또는 자동 주행:
ros2 run nav2_bringup navigation_launch.py
```

### Phase 5: Unity 시각화 확장

**Assistant 프롬프트:**

```
"Unity Visualizations 패키지를 활용한 SLAM 시각화를 추가해줘:
1. LaserScan 시각화:
   - /scan을 Unity 씬에 실시간 포인트 클라우드 표시
   - 색상: 거리별 그라디언트 (가까움=초록, 멂=빨강)
2. Occupancy Grid 시각화:
   - /map 토픽을 2D 그리드로 표시
   - 벽은 검정, 빈 공간은 흰색
3. 로봇 Pose 시각화:
   - /amcl_pose를 Unity 씬에 화살표로 표시
4. 사용자 정의 시각화:
   - ROS TCP Connector Visualizations 확장"
```

**AI 생성 시각화 설정:**

```csharp
// SLAM Visualization Config — AI Assistant
// Unity Visualizations 패키지 설정

// 1. LaserScan Visualizer
//    - Topic: /scan
//    - Type: PointCloud
//    - Color: DistanceGradient

// 2. OccupancyGrid Visualizer
//    - Topic: /map
//    - Type: Grid2D
//    - Origin: map frame

// 3. PoseVisualizer
//    - Topic: /amcl_pose
//    - Type: Arrow3D
//    - Color: Blue
```

---

## AI 활용 효과

| 작업 | 수동 (Gazebo 기준) | AI + Unity 활용 | 차이 |
|------|-------------------|----------------|------|
| Docker/ROS 2 환경 설정 | 60분 | 10분 | 6x |
| Warehouse 환경 구성 | 90분 | 15분 | 6x |
| LiDAR 시뮬레이터 | 120분 | 12분 | 10x |
| Odometry/TF/제어 스크립트 | 90분 | 15분 | 6x |
| Unity 시각화 | 60분 | 10분 | 6x |
| 통합 테스트 및 디버깅 | 120분 | 30분 | 4x |
| **전체** | **~540분** | **~92분** | **5.9x** |

---

## AI 활용 확장 아이디어

```
멀티 로봇 SLAM:
"2대의 Turtlebot3를 추가해서 멀티 로봇 SLAM 환경을 만들어줘.
각 로봇에 다른 네임스페이스 할당"

커스텀 환경 생성:
"Robotics Warehouse 대신 사무실 환경을 AI Generators로 생성해줘
- 책상, 의자, 칸막이 배치
- 실제 사무실 LiDAR 스캔과 유사하도록"

성능 분석:
"SLAM 매핑 정확도를 평가하는 스크립트 생성:
- Ground Truth 맵과 SLAM 맵 비교
- RMSE 계산
- 시각적 차이 히트맵 표시"
```

---

## 참고

- **원본 저장소**: [Robotics-Nav2-SLAM-Example](https://github.com/Unity-Technologies/Robotics-Nav2-SLAM-Example)
- **Navigation2**: [navigation.ros.org](https://navigation.ros.org)
- **slam_toolbox**: [github.com/SteveMacenski/slam_toolbox](https://github.com/SteveMacenski/slam_toolbox)
- **Robotics Warehouse**: [github.com/Unity-Technologies/Robotics-Warehouse](https://github.com/Unity-Technologies/Robotics-Warehouse)
- **Turtlebot3**: [robots.ros.org/turtlebot3](https://robots.ros.org/turtlebot3/)
- **Unity Visualizations**: [ROS-TCP-Connector Visualizations](https://github.com/Unity-Technologies/ROS-TCP-Connector/tree/main/com.unity.robotics.visualizations)
