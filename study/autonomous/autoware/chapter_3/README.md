# 로그파일 실행

실제 로그파일을 이용해서 Autoware의 알고리즘을 실행할 수 있다. 로그파일은 GNSS, Lidar 두 개의 토픽만 있어도 실행가능하다. 



## 가) 로그 파일 실행방법

1. 필요한 데이터 준비

   pcd 맵기반으로 라이다 Localization을 하기 때문에 라이다 데이터와 pcd 맵이 필요함. gnss_pose는 초기위치를 잡거나 라이다 matching이 안될 때 위치 초기화하는데 사용함. vector_map은 global path planning에 사용 

   - vector_map
   - pcd map
   - bag file(/gnss_pose, /points_raw)

2. 벡터맵, TF 읽어오기

   벡터맵은 vector_map_loader 노드에 의해 읽어온다. map 좌표계는 tf 패키지의 static_transform_publisher 노드를 이용한다.

   Runtime Manager -> Map -> Vector Map : vector map 파일을 실행함

   Runtime Manager -> Map -> TF : tf.launch 파일 실행(map좌표계가 정의되어야함)

   Runtime Manager -> Map -> Point Cloud : pcd 맵을 불러옴

   > pcd맵은 한번 실행했을 때 load해서 rviz를 초기화하거나 껐다 키면 다시 실행해서 viualize 해준다.



