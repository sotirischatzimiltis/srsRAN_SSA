# srsRAN_SSA

This repository holds a tutorial on how to create an emulated setup having an RU-DU-CU, to study the O-FH and F1 interfaces. Furthermore, this tutorial demonstrates a signaling storm attack and provides a mitigation strategy.

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

(Instructions for configuring RU, DU, and CU go here.)
