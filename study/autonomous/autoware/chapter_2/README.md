# 시뮬레이션 실행

Autoware 내에 waypoint_follower 시뮬레이터를 이용해서 Motion Planning, Mission Planning의 알고리즘을 테스트할 수 있다. op_global_planner를 실행해야 하기 때문에 벡터맵이 필요하다.



## 가) 시뮬레이션 실행방법

1. 필요한 데이터 준비
   
   - vectory map
   - tf(world -> map) 

2. 벡터맵, TF 읽어오기

   벡터맵은 vector_map_loader 노드에 의해 읽어온다. map 좌표계는 tf 패키지의 static_transform_publisher 노드를 이용한다.

   Runtime Manager -> Map -> Vector Map : vector map 파일을 실행함

   Runtime Manager -> Map -> TF : tf.launch 파일 실행(map좌표계가 정의되어야함)

   > pcd 파일이 있으면 pcd파일도 launch 해줌(선택)

3. 전역경로(Global Path) 생성

   읽어온 벡터맵을 기반으로 초기위치, 목표위치를 Rviz의 2D Pose Estimate, 2D Nav Goal 버튼을 이용해서 지정해주면 전역경로 생성함 

   Runtime Manager -> Computing -> op_global_planner 실행

   RVIZ에서 초기위치, 목표위치 설정

   OUTPUT

   - Rviz용 시각화 토픽 : /global_wayoints_rviz , /vector_map_center_lines_rviz

   - **전역경로 토픽 : /lane_waypoints_array(autoware_msgs/LaneArray)**

4. waypoint_follower 시뮬레이션 실행

   wf_simulator를 통해서 좌표계(sim_base_link,sim_lidar)를 생성하고, 시뮬레이션의 위치와 속도를 autoware 내부 알고리즘에 사용하는 (/current_vel,/current_pose)에 연결해주는 vel_pose_connect 이용한다.

   Runtime Manager -> Computing -> vel_pose_connect 실행(app 눌러서 시뮬레이션모드 체크)

   Runtime Manager -> Computing -> wf_simulator 실행

   Rviz에서 초기위치 설정(2D Pose Estimate)

   OUTPUT
   
   - **위치 : /current_pose(geometry_msgs/PoseStamped)**
   - **속도 : /current_vel(geometry_msgs/TwistStamped)**


5. 지역경로(Local Path) 생성

   전역경로(/lane_waypoints_array), 차량 위치(/current_pose)가 있으므로, Local planning을 이용해서 후보 경로를 만들 수 있다. 

   Runtime Manager -> Computing -> op_common_params 실행(제어 파라미터 설정)

   Runtime Manager -> Computing -> op_trajectory_generator 실행(후보경로 생성)
   > OUTPUT : /local_trajectories

   Runtime Manager -> Computing -> op_trajectory_evaluator 실행(후보경로 COST 계산)
   > OUTPUT : /local_trajectory_cost, local_weighted_trajectories
 
   Runtime Manager -> Computing -> op_behavior_selector 실행(상태 결정, 최종 경로 결정)
   > OUTPUT : /behavior_state , /final_waypoints

 
6. 경로 추종 
   
   최종 경로(/final_waypoints)가 나왔으므로, 경로추종 알고리즘을 이용해서 경로를 추종할 수 있다. 제어커맨드(/ctrl_cmd)가 계산된다. 하지만 wf_simulator는 /vechicle_cmd 토픽으로 제어하기 때문에 twist_fiilter를 실행해준다.

   Runtime Manager -> Computing -> pure_pursuit 

   Runtime Manager -> Computing -> twist_filter 

   OUTPUT
   
   - 제어 커맨드 : **/ctrl_cmd(autoware_msgs/ControlCommandStamped)**  
   - 시뮬레이터 제어 커맨드 : **/vehicle_cmd(autoware_msgs/VehicleCmd)**



