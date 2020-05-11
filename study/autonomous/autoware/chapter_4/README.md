# 코어 알고리즘 분석

### Autoware1.12.0 버전 기준

## 목록
### 1. Sensing
- [Points Downsampler](##-ㅁ-Points-Downsampler)
- [Points Preprocessor](##-ㅁ-Points-Preprocessor)
- Fusion

### 2. Localization
- [gnss  localizer](##-ㅁ-gnss-localizer)
- [lidar localizer](##-ㅁ-lidar-localizer)
- fusion localizer

### 3. Detection
- vision detector
- vision tracker
- lidar_detector
- lidar_tracker
- fusion_tools
- trafficlight_recognizer

### 4. Semantics
- laserscan2costmap
- potential_feild
- grid_map_filter
- wayarea2grid
- road_occupancy_processor


### 5.Prediction
- naive_motion_predict



### 6. Mission Planning
- lane_planner
- freespace_planner
- OpenPlanner - Global Planning


### 7. Motion Planning
- waypoint_maker
- waypoint_planner
- waypoint_follower
- OpenPlanner - Local Planning
- OpenPlanner - Utilites
- OpenPlanner - Simulator
- Lattice_planner


***

## ㅁ Points Downsampler

라이다 포인트 필터로 샘플 개수를 줄여준다. voxel_grid, ring, distance, random 4가지의 필터를 지원함



1. **voxel_grid_filter**


    voxel_grid 필터

    ![](img/voxel_grid_filter.gif)

    **INPUT**

   - /points_raw(sensor_msgs::PointCloud2)

    **Parameter**
   - Voxel Leaf Size : 
   - Measurment Range : 
  

    **OUTPUT**

   - /filtered_points(sensor_msgs::PointCloud2) : 필터 결과 포인트




2. **ring_filter**

    ring 필터

    ![](img/ring_filter.gif)

    **INPUT**

   - /points_raw(sensor_msgs::PointCloud2)

    **Parameter**
   - Ring Div:
   - Voxel Leaf Size : 
   - Measurment Range : 
  

    **OUTPUT**

   - /filtered_points(sensor_msgs::PointCloud2) : 필터 결과 포인트


3. **distance_filter**

    distance 필터

    ![](img/distance_filter.gif)

    **INPUT**

   - /points_raw(sensor_msgs::PointCloud2)

    **Parameter**
   - Sample Num :
   - Measurment Range : 
  

    **OUTPUT**

   - /filtered_points(sensor_msgs::PointCloud2) : 필터 결과 포인트

4. **random_filter**

    random 필터

    ![](img/random_filter.gif)

    **INPUT**

   - /points_raw(sensor_msgs::PointCloud2)

    **Parameter**
   - Sample Num :
   - Measurment Range : 
  

    **OUTPUT**

   - /filtered_points(sensor_msgs::PointCloud2) : 필터 결과 포인트


## ㅁ Points Preprocessor

라이다 포인트에서 ground, no_ground를 분리해준다.

1. **ring_ground_filter**
   
   ring_ground_filter 

   ![](img/ring_ground_filter.gif)

    **INPUT**

   - /points_raw(sensor_msgs::PointCloud2)

    **Parameter**

   - Sensor Height :
   - Max Slope :
   - Vertical Thres :
   - Sensor Model :
  

    **OUTPUT**

   - /points_ground(sensor_msgs::PointCloud2) : ground 부분 포인트
   - /points_no_ground(sensor_msgs::PointCloud2) : ground가 아닌 부분 포인트

2. **ray_ground_filter**

   ray_ground_filter 

   ![](img/ray_ground_filter.gif)

    **INPUT**

   - /points_raw(sensor_msgs::PointCloud2)

    **Parameter**

   - 사이트 참고
  

    **OUTPUT**

   - /points_ground(sensor_msgs::PointCloud2) : ground 부분 포인트
   - /points_no_ground(sensor_msgs::PointCloud2) : ground가 아닌 부분 포인트




***

## ㅁ gnss localizer

GGA(fix), NMEA 두 가지 프로토콜로 들어오는 데이터를 지원함. 
    
1. fix2tfpose

    GGA 프로토콜로 들어오는 데이터를 받아 LLT(위도,경도,고도)좌표계에서 UTM(x,y,z)로 바꾸고, 위치기반으로 헤딩을 추정한다. 

    > 이전 위치와 현재위치가 0.2m 보다 크면 두점의 방향으로 헤딩을 추정함

    **INPUT**
    
    - /fix(sensor_msgs::NavSatFix)

    **Parameter**
    - plane : utm 평면 번호(1~19번 모두 일본 지역)

    **OUTPUT**
    
    - /gnss_pose(geometry_msgs::PoseStamped) : 차량의 POSE
    - /gnss_stat(std_msgs::Bool) : gnss 데이터를 사용하는지에 대한 토픽(true : 사용, false : 사용안함)
    - /tf (map -> gps) : gps좌표계 생성하고, tf 업데이트해줌
  

2. nmea2tfpose

    NMEA 프로토콜로 들어오는 데이터를 받아 LLT(위도,경도,고도)좌표계에서 UTM(x,y,z)로 바꾸고, 위치기반으로 헤딩을 추정한다. 

    > 이전 위치와 현재위치가 0.2m 보다 크면 두점의 방향으로 헤딩을 추정함

    **INPUT**
    
    - /nmea_sentence(nmea_msgs::Sentence)

    **Parameter**
    - plane : utm 평면 번호(1~19번 모두 일본 지역)

    **OUTPUT**
    
    - /gnss_pose(geometry_msgs::PoseStamped) : 차량의 POSE
    - /tf (map -> gps) : gps좌표계 생성하고, tf 업데이트해줌


***

## ㅁ lidar localizer



1. ndt_mapping 
   
   라이다 