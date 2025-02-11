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
```bash
apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev libzmq3-dev git curl jq -y
git clone https://github.com/srsran/srsRAN_Project.git
cd srsRAN_Project/
mkdir build
cd build/
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j`nproc`
cp /root/srsRAN_Project/build/apps/gnb /usr/bin/gnb
```
