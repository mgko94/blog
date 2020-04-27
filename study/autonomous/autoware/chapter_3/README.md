# 로그파일 실행

Autoware 내에 waypoint_follower 시뮬레이터를 이용해서 Motion Planning, Mission Planning의 알고리즘을 테스트할 수 있다. op_global_planner를 실행해야 하기 때문에 벡터맵이 필요하다.



## 가) 시뮬레이션 실행방법

1. autoware 실행

    [세미나](https://github.com/khkim545/autoware_workshop_2019) 자료를 참고



## 나) 실행 노드 분석

1. map

   -  points_map_loader  : pcd파일을 읽어와서 포인트 클라우드 메시지(points_map)를 생성
   -  vector_map_loader : 벡터맵 파일을 읽어와서 벡터맵(vector_map_msgs) 메시지를 만든다.
   -  tf publisher : world -> map,  map -> mobility 좌표계를 publish해준다.



2. mission_planning
   

   -  op_global_planner : 벡터맵(/vector_map/area), 현재속도(/current_velocity), 현재 위치(current_pose)를 받아서 전역경로(/lane_waypoints_array)를 생성
   -  vel_pose_connect : 속도(velocity), 위치(pose)를 autoware에서 사용하는 토픽(/current_velocity, current_pose)으로 remap 한다.
  


3. motion_planning


    - op_common_params : 제어 파라미터 설정
    - op_trajectory_generator : 전역경로(/lane_waypoints_array), 위치(/current_pose), 속도(/current_velocity), 좌표계(tf)를 받아서 후보 경로(/local trajectories) 생성
    - op_trajectory_evaluator : 위치(/current_pose), 속도(/current_velocity), 좌표계(tf), 후보 경로(/local trajectories)를 받아서 가중치를 계산한 후보경로( /local_weighted_trajectories) 생성
    - op_behavior selector : 위치(/current_pose), 속도(/current_velocity), 좌표계(tf),가중치 후보경로( /local_weighted_trajectories)를 받아서 최종 경로(/final_waypoints) 선택 후 경로추종 알고리즘에 전달
    - pure_puresuit : 최종 경로(/final_waypoints), 위치(/current_pose), 속도(/current_velocity), 좌표계(tf)를 받아서 제어값(/ctrl_cmd, /twist_raw ) 계산
    - twist_filter : 계산결과를 twist_gate 노드를 거쳐서 실제 제어 값을 전달

    - wf_simulator : 제어값(/ctrl_cmd)을 받아서 vehicle_status 메시지를 만들어서 시뮬레이션기능 구현


