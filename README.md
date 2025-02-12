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

```bash
sudo make install
sudo ldconfig
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

Start the UE
```bash
sudo srsue /home/srsran-zmq/srsRAN_Project/configs/ue_zmq.conf
```

## Signaling Storm Attack 
Here we are going to attempt to change the random number used in the UL RRC Setup Request used by the UE to initiate the RRC connection. 

The code we are interested is located in **srsRAN_4G/srsue/src/stack/rrc_nr/** and is the **rrc_nr.cc** script.

Line code **617** assigns the random value to the **ue_id** as follows:  **rrc_setup_req->ue_id.set_random_value();**.

Since the ASN.1 formal notation is used for describing data transmitted by telecommunications protocols, we need to find out the structure. 

We observe the header file **rrc_nr.h** located in **srsRAN_4G/lib/include/srsran/asn1/**, in line **7686** we can see the struct for the **init_ue_id_c**. 
The previous header file is used by the other **rrc_nr.h** located in **srsRAN_4G/srsue/hdr/stack/rrc_nr/**.

We appended the code line below in the **rrc_nr.cc** script to print the random value chosen as initial UE id. 
```bash
#include <bitset>
// Print UE ID as a decimal  and bitstring
std::cout << "UE ID (random_id - decimal): " << random_id << std::endl;
std::bitset<40> binary_representation(random_id);
std::cout << "UE ID (random_id - binary): " << binary_representation << std::endl;
```
Compile again and run
```bash
cd /srsRAN_4G/build/
make -j`nproc`
sudo make install
sudo srsue /home/srsran-zmq/srsRAN_Project/configs/ue_zmq.conf
```
Find the ASN.1 file for the **init_ue_id_c** struct located in **srsRAN_4G/lib/include/srsran/asn1/rrc/ul_ccch_msg.h**
```
// InitialUE-Identity ::= CHOICE
struct init_ue_id_c {
  struct types_opts {
    enum options { s_tmsi, random_value, nulltype } value;

    const char* to_string() const;
  };
  typedef enumerated<types_opts> types;

  // choice methods
  init_ue_id_c() = default;
  init_ue_id_c(const init_ue_id_c& other);
  init_ue_id_c& operator=(const init_ue_id_c& other);
  ~init_ue_id_c() { destroy_(); }
  void        set(types::options e = types::nulltype);
  types       type() const { return type_; }
  SRSASN_CODE pack(bit_ref& bref) const;
  SRSASN_CODE unpack(cbit_ref& bref);
  void        to_json(json_writer& j) const;
  // getters
  s_tmsi_s& s_tmsi()
  {
    assert_choice_type(types::s_tmsi, type_, "InitialUE-Identity");
    return c.get<s_tmsi_s>();
  }
  fixed_bitstring<40>& random_value()
  {
    assert_choice_type(types::random_value, type_, "InitialUE-Identity");
    return c.get<fixed_bitstring<40> >();
  }
  const s_tmsi_s& s_tmsi() const
  {
    assert_choice_type(types::s_tmsi, type_, "InitialUE-Identity");
    return c.get<s_tmsi_s>();
  }
  const fixed_bitstring<40>& random_value() const
  {
    assert_choice_type(types::random_value, type_, "InitialUE-Identity");
    return c.get<fixed_bitstring<40> >();
  }
  s_tmsi_s&            set_s_tmsi();
  fixed_bitstring<40>& set_random_value();

private:
  types                                          type_;
  choice_buffer_t<fixed_bitstring<40>, s_tmsi_s> c;

  void destroy_();
};
```

Also **rrc_nr.h** exists in **srsRAN_4G/lib/include/srsran/asn1/** that we have a struct for the **initial_ue_id_c**
```
// InitialUE-Identity ::= CHOICE
struct init_ue_id_c {
  struct types_opts {
    enum options { ng_minus5_g_s_tmsi_part1, random_value, nulltype } value;
    typedef int8_t number_type;

    const char* to_string() const;
    int8_t      to_number() const;
  };
  typedef enumerated<types_opts> types;

  // choice methods
  init_ue_id_c() = default;
  init_ue_id_c(const init_ue_id_c& other);
  init_ue_id_c& operator=(const init_ue_id_c& other);
  ~init_ue_id_c() { destroy_(); }
  void        set(types::options e = types::nulltype);
  types       type() const { return type_; }
  SRSASN_CODE pack(bit_ref& bref) const;
  SRSASN_CODE unpack(cbit_ref& bref);
  void        to_json(json_writer& j) const;
  // getters
  fixed_bitstring<39>& ng_minus5_g_s_tmsi_part1()
  {
    assert_choice_type(types::ng_minus5_g_s_tmsi_part1, type_, "InitialUE-Identity");
    return c.get<fixed_bitstring<39> >();
  }
  fixed_bitstring<39>& random_value()
  {
    assert_choice_type(types::random_value, type_, "InitialUE-Identity");
    return c.get<fixed_bitstring<39> >();
  }
  const fixed_bitstring<39>& ng_minus5_g_s_tmsi_part1() const
  {
    assert_choice_type(types::ng_minus5_g_s_tmsi_part1, type_, "InitialUE-Identity");
    return c.get<fixed_bitstring<39> >();
  }
  const fixed_bitstring<39>& random_value() const
  {
    assert_choice_type(types::random_value, type_, "InitialUE-Identity");
    return c.get<fixed_bitstring<39> >();
  }
  fixed_bitstring<39>& set_ng_minus5_g_s_tmsi_part1();
  fixed_bitstring<39>& set_random_value();

private:
  types                                 type_;
  choice_buffer_t<fixed_bitstring<39> > c;

  void destroy_();
};
```
