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



1. autoware 유저 생성

    다음과 같이 변경해줌 원래 user: Rubicom이라고 한다면 autoware 유저랑 pid 번호를 바꿔줌 
    autoware 유저번호를 1000번으로 바꿔줘야함.

        sudo adduser autoware  
        sudo edit /etc/passwd
        
        ##다음과 같이 편집##
        rubicom:x:1001:1001:Rubicom,,,:/home/rubicom:/bin/bash 
        autoware:x:1000:1000:Autoware,,,:/home/autoware:/bin/bash




    group도 에디터를 열어서 수정해줌. autoware에게 sudo 권한을 준다.  


        sudo gedit /etc/group 

        ##다음과 같이 편집##
        sudo:x:27:autoware # add ‘autoware’ into group ‘sudo’ 
        rubicom:x:1001 # previously 1000 
        autoware:x:1000 # previously 1001


    해당 폴더의 소유권을 바꿔줌

        sudo chown –R 1000:1000 /home/autoware
        sudo chown –R 1001:1001 /home/rubicom 

    autoware 유저로 자동로그인 설정을 해준다.

        sudo gedit /etc/gdm3/custom.conf 



2. docker ce 설치
   
    도커 명령어 및 사용법 [Link](study\docker\README.md)

        sudo apt-get update # update package lists 
        sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
        sudo apt-key fingerprint 0EBFCD88 
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" 
        sudo apt-get update 
        sudo apt-get install docker-ce 
        sudo docker run hello-world
    >https://gitlab.com/autowarefoundation/autoware.ai/autoware/-/wikis/docker-installation


3. nvidia 그래픽카드 설치
   
        sudo lshw -C display
        ubuntu-drivers devices
        sudo add-apt-repository ppa:graphics-drivers/ppa
        sudo apt update
        sudo ubuntu-drivers autoinstall
        sudo reboot


4. docker-nvidia 설치
   
        curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo aptkey add -
        distribution=$(. /etc/os-release;echo $ID$VERSION_ID) 
        curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidiadocker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list 
        sudo apt-get update 

        curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo aptkey add  -
        sudo apt-get install nvidia-docker2 
        sudo pkill -SIGHUP dockerd 
        docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
    >https://github.com/NVIDIA/nvidia-docker


5. 공유파일 설정

        cp –r target_dir ~/shared_dir  
        # target_dir이 복사할 대상 : 제공받은 폴더경로를 적어주면된다. 
        cd ~/shared_dir && sudo chown -R $(id -u):$(id -g) * 


6. autoware docker 이미지 생성 후 컨테이너 실행

        cd ~ && git clone https://gitlab.com/autowarefoundation/autoware.ai/docker.git
        cd ~/docker/generic
        sudo ./run.sh -r melodic -s





