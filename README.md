# Docker Container for Autonomous Risk Framework

This readme document is intended to lead through the installation of Docker and the necessary packages. 

## Installation of Docker and NVIDIA Docker on the Host Machine
To fully Carla, the NVIDIA extension for Docker must be installed. For this section, it is not assumed that Docker is installed on the host macine at all. The installation has been tested on Ubuntu 20.04 as host system. The following instructions can also be found with more explanations [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker). Using the following command (assuming curl is installed with `sudo apt-get install curl`) will install Docker.

    curl https://get.docker.com | sh && sudo systemctl --now enable docker

The following command will make the NVIDIA Container Toolkit available for installation on the system.

    distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
    && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
    && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

Finally the NVIDIA Docker Toolkit can be installed on the system.

    sudo apt-get update
    
    sudo apt-get install -y nvidia-docker2

To apply the changes, it is necessary to restart the Docker service.

    sudo systemctl restart docker
 
To check if the installation was successful, the following command can be run:

    sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
    
This should create an output that looks approximately like the following:

    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 460.80       Driver Version: 460.80       CUDA Version: 11.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  GeForce GTX 1080    Off  | 00000000:01:00.0  On |                  N/A |
    | 31%   59C    P2    45W / 180W |    821MiB /  8111MiB |      4%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+

    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    +-----------------------------------------------------------------------------+

## Building the Docker Image from the Dockerfile
The file to build the image, the `Dockerfile` is located in the `Docker` directory. Building the image can be done using the following command:

    sudo docker build -t autonomous_risk_framework:0.1 /PATH_TO_AUTONOMOUSRISKFRAMEWORK/Docker
    
To verify that the build was successful, the following command can be used to list locally available images.

    sudo docker images
    
The output should look similar to this:

    REPOSITORY                  TAG       IMAGE ID       CREATED          SIZE
    autonomous_risk_framework   0.1       bc29d0c5140a   21 seconds ago   10.8GB
    carlasim/carla              0.9.11    95ae4b17d967   6 months ago     9.86GB

## Configuration Inside the Container
As the github repository is not public yet and building the Docker image is a non-interactive process, some configuration must be done manually at this point inside the Docker container. This mainly includes downloading the repository and installing all the dependencies.
To open the image and create a new container, use the following command (the options are not necessary at the moment, but it is assumed that we want to open Carla later).

    sudo docker run --runtime=nvidia --gpus all -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -it autonomous_risk_framework:0.1 bash
    
The output should look like

    carla@a182e68e7243:~$
    
where `a182e68e7243` is the container ID. In the next step, the github repository is cloned in the home directory and the branch is changed to the branch dedicated for the docker container.

    git clone https://github.com/sisl/AutonomousRiskFramework.git
    
    cd AutonomousRiskFramework
    
    git checkout marc/carla_docker
   
To install the necessary python packages, we install

    pip3 install -r /home/carla//AutonomousRiskFramework/CARLAIntegration/scenario_runner/requirements.txt

*Installation of the Julia packages needs to be included here*

## Running Carla
Assuming that the terminal that was used to configure the container, was not closed and the container ID still is `a182e68e7243`, Carla can be launched using the following command.

    /home/carla/CarlaUE4.sh -carla-rpc-port=2222 -windowed -ResX=320 -ResY=240 -benchmark -fps=10 -quality-level=Low -opengl

The Carla window should open at this point and (for Carla 0.9.11) show a roundabout with a fountain. 

To interact with the server, a new terminal window is required. With this window, it is possible to connect to the current session of the docker container by typing in the **new** terminal:

    sudo docker exec -it a182e68e7243 bash
 
where `a182e68e7243` should be replaced with the correct container ID. We can now change the map and the position of the spectator using the modified `config.py` file that is located in the `./CARLAIntegration/util/` directory. Such a command could look like:

    python3 /home/carla/AutonomousRiskFramework/CARLAIntegration/util/config.py --map Town01 --port 2222 --spectator-loc 80.37 25.30 0.0    

*Continue with how to execute AST*

## Contacts
- Stanford Intelligent Systems Laboratory (SISL)
    - Marc Schlichting: [MarcSchlichting](https://github.com/MarcSchlichting)
