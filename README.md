# srsRAN Emulator

This repository holds a tutorial on how to create an emulated setup having an RU-DU-CU, to study the O-FH and F1 interfaces. Furthermore, this tutorial demonstrates a signaling storm attack and provides a mitigation strategy.

## System Requirements

## Step 1: Install Dependencies

Run the following commands to update and install required dependencies:

```bash
sudo apt update && sudo apt upgrade -y
```

```bash
sudo apt install -y build-essential cmake git libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev ninja-build
```

```bash
sudo apt install -y tcpdump wireshark iperf3 net-tools
```

## Step 2: Clone and Build srsRAN 5G

### 1. Clone the latest srsRAN 5G repository:

```bash
cd ~
git clone https://github.com/srsran/srsRAN_Project.git
cd srsRAN_Project
```

### 2. Build and install:

```bash
mkdir build
cd build
cmake ..
make -j$(nproc)
sudo make install
```

### 3. Update system libraries:

```bash
sudo ldconfig
```

### 4. Verify Installation

```bash
srsran_install_test
```

## Step 3: Configure RU, DU, and CU

### 1. Navigate to the configuration directory:
```bash
cd ~/srsRAN_Project/configs
```
### *Configure RU (Radio Unit) Emulator*
### 2. Open the RU configuration file:
```bash
nano ru_emulator.conf
```
### 3. Ensure ZeroMQ (ZMQ) is set up for Open Fronthaul communication:
```bash
zmq.tx_port0=tcp://*:2001
zmq.rx_port0=tcp://localhost:2000
```
### 4. Save and exit (CTRL+X, then Y, then ENTER).
### 5. Start the RU emulator:
```bash
sudo srsruemu --args "tx_port0=tcp://localhost:2001,rx_port0=tcp://*:2000,id=1"
```

### *Configure DU (Distributed Unit)*
