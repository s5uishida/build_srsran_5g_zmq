# Build srsRAN_Project 5G RAN with ZeroMQ
srsRAN_Project software suite includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications. Therefore, in order to confirm the facilities of 5GC, I will describe the simple procedure for building the virtual gNodeB instead of the real device.
**Note that this section describes how to build srsRAN_Project in Virtualbox VM.**

Please refer to the following for building srsRAN_Project 5G RAN with ZeroMQ.
- Installation Guide - https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html
- ZeroMQ-based Setup - https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup
- Configuration Reference - https://docs.srsran.com/projects/project/en/latest/user_manuals/source/config_ref.html

**4GB or more memory is required to build. And 2 CPU cores or more are required to run.**

Also, when connecting by 5G NR-UE with ZeroMQ, see [here](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins) for how to build and configure this RF simulated UE.

---

<a id="toc"></a>

## Table of Contents

- [Install the required libraries including ZeroMQ](#install_libs)
- [Clone srsRAN_Project](#clone_srsran)
- [Build srsRAN_Project 5G RAN on Virtualbox VM](#build)
- [Create the configuration file of gNodeB](#create_gnb_config)
- [Issues](#issues)
- [Confirmed Version List](#ver_list)
- [Changelog (summary)](#changelog)

---

<a id="install_libs"></a>

## Install the required libraries including ZeroMQ

```
apt install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev libzmq3-dev
```

<a id="clone_srsran"></a>

## Clone srsRAN_Project

```
git clone https://github.com/srsran/srsRAN_Project.git
```

<a id="build"></a>

## Build srsRAN_Project 5G RAN on Virtualbox VM

According to [this](https://github.com/srsran/srsRAN_Project/discussions/151#discussioncomment-6576652), when building with Virtualbox VM, if you don't worry about running it in real-time, you could add `-DAUTO_DETECT_ISA=OFF` to cmake options.
```
cd srsRAN_Project
mkdir build
cd build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON -DAUTO_DETECT_ISA=OFF
make -j`nproc`
```

<a id="create_gnb_config"></a>

## Create the configuration file of gNodeB

Get `gNB config` of [ZeroMQ-based Setup](https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup) as the original file.
```
cd srsRAN_Project/build/apps/gnb
wget <link of "gNB config">
```
For reference, `gnb_zmq.yaml` on 2023.10.24 is as follows.
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
  prach:
    prach_config_index: 1           # Set PRACH configuration index to 1. 

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

<a id="issues"></a>

## Issues

If the latest `main` branch doesn't work, you may try the latest `release` version.

1. If the `gnb` on Virtualbox VM fails to do `NGSetup`, [this](https://github.com/srsran/srsRAN_Project/issues/172#issuecomment-1681908406) might help to solve it.
2. According to [here](https://github.com/srsran/srsRAN_4G/issues/1213), there is an issue where downlink packets stop flowing between srsue and srsgnb via ZeroMQ. The hotfix is [here](https://github.com/srsran/srsRAN_4G/issues/1213#issuecomment-1703512937).
3. According to [here](https://github.com/srsran/srsRAN_Project/issues/241#issuecomment-1756599022), 2 CPU cores or more are required to run.
4. According to [here](https://github.com/srsran/srsRAN_Project/issues/263), UE(`ue_zmq.conf`) failed to connect to Open5GS via gNodeB of srsRAN_Project 23.10. [This](https://github.com/srsran/srsRAN_Project/issues/263#issuecomment-1773756230) solved it.

<a id="ver_list"></a>

## Confirmed Version List

I simply confirmed the operation of the following versions.

| Version | Commit | Date | Issues |
| --- | --- | --- | -- |
| 23.10.1 | `374200deefd8e1b96fab7328525fd593a808a641` | 2023.10.23 | 3 |
| 23.10 | `e38e418bda8432397b2fa7dc399cb7afde3c3b95` | 2023.10.20 | 3, 4 |
| 23.5+ | `5e6f50a202c6efa671d5b231d7c911dc6c3d86ed` | 2023.09.20 | 3 |
| 23.5+ | `1afd7240f2b5e2061ab4158e8fcdacb15961813a` | 2023.08.07 | 1, 2 |

<a id="changelog"></a>

## Changelog (summary)

- [2023.12.02] Updated a list of confirmed versions.
- [2023.11.02] Updated `gnb_zmq.yaml`.
- [2023.10.21] Added the case of srsRAN_Project 23.10.
- [2023.10.10] Added a list of confirmed versions.
- [2023.08.10] Initial release.
