# Build srsRAN_Project 5G RAN with ZeroMQ
srsRAN_Project software suite includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications. Therefore, in order to confirm the facilities of 5GC, I will describe the simple procedure for building the virtual gNodeB instead of the real device.
**Note that this section describes how to build srsRAN_Project in Virtualbox VM.**

Please refer to the following for building srsRAN_Project 5G RAN with ZeroMQ.
- Installation Guide - https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html
- ZeroMQ-based Setup - https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup
- Configuration Reference - https://docs.srsran.com/projects/project/en/latest/user_manuals/source/config_ref.html

**4GB or more memory is required to build.**

---

<h2 id="install_libs">Install the required libraries including ZeroMQ</h2>

```
apt install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev libzmq3-dev
```

<h2 id="clone_srsran">Clone srsRAN_Project</h2>

```
git clone https://github.com/srsran/srsRAN_Project.git
```

<h2 id="build">Build srsRAN_Project 5G RAN on Virtualbox VM</h2>

According to [this](https://github.com/srsran/srsRAN_Project/discussions/151#discussioncomment-6576652), when building with Virtualbox VM, if you don't worry about running it in real-time, you could add `-DAUTO_DETECT_ISA=OFF` to cmake options.
```
cd srsRAN_Project
mkdir build
cd build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON -DAUTO_DETECT_ISA=OFF
make -j`nproc`
```

<h2 id="create_gnb_config">Create the configuration file of gNodeB</h2>

Get `gNB config` of [ZeroMQ-based Setup](https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup) as the original file.
```
cd srsRAN_Project/build/apps/gnb
wget <link of "gNB config">
```
For reference, `gnb_zmq.yaml` on 2023.07.06 is as follows.
```yaml
# This configuration file example shows how to configure the srsRAN Project gNB to allow srsUE to connect to it. 
# This specific example uses ZMQ in place of a USRP for the RF-frontend, and creates an FDD cell with 10 MHz bandwidth. 
# To run the srsRAN Project gNB with this config, use the following command: 
#   sudo ./gnb -c gnb_zmq.yaml

amf:
  addr: 10.53.1.2                  # The address or hostname of the AMF.
  bind_addr: 10.53.1.1             # A local IP that the gNB binds to for traffic from the AMF.

ru_sdr:
  device_driver: zmq                # The RF driver name.
  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=11.52e6 # Optionally pass arguments to the selected RF driver.
  srate: 11.52                      # RF sample rate might need to be adjusted according to selected bandwidth.
  tx_gain: 75                       # Transmit gain of the RF might need to adjusted to the given situation.
  rx_gain: 75                       # Receive gain of the RF might need to adjusted to the given situation.

cell_cfg:
  dl_arfcn: 368500                  # ARFCN of the downlink carrier (center frequency).
  band: 3                           # The NR band.
  channel_bandwidth_MHz: 10         # Bandwith in MHz. Number of PRBs will be automatically derived.
  common_scs: 15                    # Subcarrier spacing in kHz used for data.
  plmn: "00101"                     # PLMN broadcasted by the gNB.
  tac: 7                            # Tracking area code (needs to match the core configuration).
  pdcch:
    dedicated:
      ss2_type: common              # Search Space type, has to be set to common
      dci_format_0_1_and_1_1: false # Set correct DCI format (fallback)

log:
  filename: /tmp/gnb.log            # Path of the log file.
  all_level: info                   # Logging level applied to all layers.
  hex_max_size: 0

pcap:
  mac_enable: false                 # Set to true to enable MAC-layer PCAPs.
  mac_filename: /tmp/gnb_mac.pcap   # Path where the MAC PCAP is stored.
  ngap_enable: false                # Set to true to enable NGAP PCAPs.
  ngap_filename: /tmp/gnb_ngap.pcap # Path where the NGAP PCAP is stored.
```
Then, edit according to your environment.

---

<h2 id="changelog">Changelog (summary)</h2>

- [2023.08.10] Initial release.
