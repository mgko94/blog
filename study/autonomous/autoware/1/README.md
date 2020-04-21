# 개발환경 구성

컴퓨터 스펙

    - OS : Ubuntu 18.04 LTS
    - CPU : I7-8750
    - 그래픽카드 : GTX1060
    - USER NAME : autoware(USER NAME이 다르면 추 후에 경로수정이 필요함)



## 가) Host에 개발환경 구성
    
Host에 직접 Nvidia 그래픽 드라이버, ROS, Cuda, 종속라이브러리 등을 직접 설치한다.

1. NVIDIA 그래픽 드라이버, 쿠다(10.0) 설치
    
        sudo apt-get update 
        sudo apt-get upgrade

        lspci | grep -i nvidia


        uname -m && cat /etc/*release

        sudo apt-get purge nvidia*
        sudo apt-get autoremove
        sudo apt-get autoclean
        sudo rm -rf /usr/local/cuda*

        sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
        echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" | sudo tee /etc/apt/sources.list.d/cuda.list

        sudo apt-get update 
        sudo apt-get -o Dpkg::Options::="--force-overwrite" install cuda-10-0 cuda-drivers

        echo 'export PATH=/usr/local/cuda-10.0/bin${PATH:+:${PATH}}' >> ~/.bashrc
        echo 'export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}' >> ~/.bashrc
        source ~/.bashrc
        sudo ldconfig

        nvidia-smi
        nvcc -V
    > https://m.blog.naver.com/sw4r/221744342510

2. ROS Melodic 설치
    
        sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
        sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
        sudo apt update
        sudo apt install ros-melodic-desktop-full
        echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
        source ~/.bashrc
        source /opt/ros/melodic/setup.bash
        sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
        sudo rosdep init
        rosdep update
    > http://wiki.ros.org/melodic/Installation/Ubuntu
    

3. 아이젠 3.3.7 버전 설치


        cd && wget http://bitbucket.org/eigen/eigen/get/3.3.7.tar.gz
        mkdir eigen && tar --strip-components=1 -xzvf 3.3.7.tar.gz -C eigen
        cd eigen && mkdir build && cd build && cmake .. && make 
        sudo make install
        cd && rm -rf 3.3.7.tar.gz && rm -rf eigen

        grep "#define EIGEN_[^_]*_VERSION" /usr/local/include/eigen3/Eigen/src/Core/util/Macros.h
    > https://gitlab.com/autowarefoundation/autoware.ai/autoware/-/wikis/Source-Build

4. Autoware 다운 및 빌드

        mkdir -p Autoware/src && cd Autoware
        wget -O autoware.ai.repos "https://gitlab.com/autowarefoundation/autoware.ai/autoware/raw/1.12.0/autoware.ai.repos?inline=false"
        vcs import src < autoware.ai.repos
        rosdep update
        rosdep install -y --from-paths src --ignore-src --rosdistro melodic

        AUTOWARE_COMPILE_WITH_CUDA=1 colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release #cuda version

        colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release #without cuda


5. Autoware 패키지 수정

    ROS Melodic 버전에서 Runtime manager Gui 문제가 발생 runtime_manager_dialog.py, tmgr.py파일 수정이 필요함 아래링크보고 수정


        gedit ~/Autoware/src/autoware/utilities/runtime_manager/scripts/rtmgr.py 
        gedit ~/Autoware/src/autoware/utilities/runtime_manager/scripts/runtime_manager_dialog.py  
    >https://gitlab.com/autowarefoundation/autoware.ai/utilities/-/merge_requests/25/diffs 


    runtime_manager, ndt_matching, lidar_euclidean_cluster_detect 소스코드 수정 및 다른 설정 파일 복사.


    
        cp ~/shared_dir/src/autoware/utilities/runtime_manager/scripts/run ~/Autoware/src/autoware/utilities/runtime_manager/scripts/run
        cd ~/Autoware/build/runtime_manager
        make install 

        cp ~/shared_dir/src/autoware/core_perception/lidar_localizer/nodes/ndt_matching/ndt_matching.cpp ~/Autoware/src/autoware/core_perception/lidar_localizer/nodes/ndt_matching/ndt_matching.cpp
        cd ~/Autoware/build/lidar_localizer 
        make install 


        cp ~/shared_dir/src/autoware/core_perception/lidar_euclidean_cluster_detect/launch/lidar_euclidean_cluster_detect.launch ~/Autoware/src/autoware/core_perception/lidar_euclidean_cluster_detect/launch/lidar_euclidean_cluster_detect.launch 
        cd ~/Autoware/build/lidar_euclidean_cluster_detect 
        make install

        cd 
        mkdir autoware_openplanner_logs
        cd autoware_openplanner_logs && mkdir SimulationData
        cp ~/shared_dir/autoware_openplanner_logs/SimulationData/EgoCar.csv  ~/autoware_openplanner_logs/SimulationData/

        cp -rf ~/shared_dir/default.rviz ~/.rviz/default.rviz 
        echo 'source /home/autoware/Autoware/install/setup.bash' >> ~/.bashrc
        source ~/.bashrc

    > 유저이름이 autoware 이여야 한다. 다르다면 자신의 유저이름에 맞게 바꿔서 명령어를 입력


6. Autoware 실행

        cd ~/Autoware
        roslaunch runtime_manager runtime_manager.launch

## 나) Docker의 컨테이너에 개발환경 구성

Image 파일을 만들어 컨테이너에 ROS, Cuda, 종속라이브러리를 설치를 해야 하지만 Image 파일은 Autoware에서 제공하기 때문에 다운받아서 바로 사용할 수 있다. 다만 Nvidia 그래픽 드라이버는 호스트 설치한다. Docker-nvidia를 통해 Host에 있는 그래픽카드를 컨테이너에서 잡아서 사용할 수 있다.


