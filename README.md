# srsRAN Emulator

This repository holds a tutorial on how to create an emulated setup having an RU-DU-CU, to study the O-FH and F1 interfaces. Furthermore, this tutorial demonstrates a signaling storm attack and provides a mitigation strategy.

## System Requirements
Visit installation page for more info: https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html
## Step 1: Install Dependencies

Run the following commands to update and install required dependencies:

```bash
sudo apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev
```

```bash
sudo apt install -y tcpdump wireshark iperf3 net-tools
```

```bash
sudo apt-get install libzmq3-dev
```

```bash
sudo apt install git
```

## Step 2: Clone and Build srsRAN 5G

### 1. Clone the latest srsRAN 5G repository and built:

```bash
cd ~
git clone https://github.com/srsran/srsRAN_Project.git
cd srsRAN_Project
mkdir build
cd build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j`nproc`
```
```bash
sudo make install
sudo ldconfig
```

## Step 3: Build 5G core 
Install docker: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

### 1. Navigate to the docker directory and build:
```bash
cd ~/srsRAN_Project/docker
sudo docker compose up --build 5gc
```

## Step 4: Build and compile a gNB
Install dependencies
```bash
sudo apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev libzmq3-dev git curl jq -y
```
Compile the project
```bash
cd srsRAN_Project/build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j`nproc`
```
```bash
cd srsRAN_Project/build/apps
gnb -h (to get the subcommands that can be used)
```
Download the two config files for gnb and ue from here: https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html 

Put the config files in srsrRAN_Project\configs directory 

Run the gnb application
```bash
gnb -c /home/srsran-zmq/srsRAN_Project/configs/gnb_zmq.yaml
```

## Step 5: Build and compile a UE 
Install dependencies
```bash
sudo apt-get install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev git curl jq -y
```
Clone srsRAN_Project 4G since the ue is located there

```bash
git clone https://github.com/srsRAN/srsRAN_4G.git
cd srsRAN_4G/
mkdir build
cd build/
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j`nproc`
```
Move to correct directory
```bash
cd /srsRAN_4G/build/srsue/
```
When using the ZMQ-based RF driver in the srsUE, it is important to create an appropriate network namespace in the host machine. This is achieved with the following command:
```bash
sudo ip netns add ue1
```

To verify the new “ue1” network namespace exists, run:
```bash
sudo ip netns list
```


