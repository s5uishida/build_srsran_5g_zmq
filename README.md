# Build srsRAN_Project 5G RAN with ZeroMQ
srsRAN_Project software suite includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications. Therefore, in order to confirm the facilities of 5GC, I will describe the simple procedure for building the virtual gNodeB instead of the real device.

Please refer to the following for building srsRAN_Project 5G RAN with ZeroMQ.
- Installation Guide - https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html
- ZeroMQ-based Setup - https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup
- Configuration Reference - https://docs.srsran.com/projects/project/en/latest/user_manuals/source/config_ref.html

The specification of the VM that have been confirmed to work is as follows.
| OS | CPU (Min) | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- |
| Ubuntu 24.04 | 4 | 4GB | 10GB |

**4GB or more memory is required to build. And 4 CPU cores or more are required to run.**

Also, when connecting by 5G NR-UE with ZeroMQ, see [here](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins) for how to build and configure this RF simulated UE.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Install the required libraries including ZeroMQ](#install_libs)
- [Clone srsRAN_Project](#clone_srsran)
- [Build srsRAN_Project 5G RAN](#build)
- [Create the configuration file of gNodeB](#create_gnb_config)
- [Issues](#issues)
- [Confirmed Version List](#ver_list)
- [Sample Configurations](#sample_conf)
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

## Build srsRAN_Project 5G RAN

```
cd srsRAN_Project
mkdir build
cd build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j`nproc`
```
If you do not want to build `tests` target, add `-DBUILD_TESTS=OFF` option.
```
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON -DBUILD_TESTS=OFF
```
According to [this](https://github.com/srsran/srsRAN_Project/discussions/151#discussioncomment-6576652), when building with Virtualbox VM, if you don't worry about running it in real-time, you could add `-DAUTO_DETECT_ISA=OFF` option.
```
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON -DAUTO_DETECT_ISA=OFF
```

<a id="create_gnb_config"></a>

## Create the configuration file of gNodeB

Get `gNB config` of [ZeroMQ-based Setup](https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup) as the original file.
```
cd srsRAN_Project/build/apps/gnb
wget <link of "gNB config">
```
For reference, `gnb_zmq.yaml` on 2024.12.17 is as follows.
```yaml
# This configuration file example shows how to configure the srsRAN Project gNB to allow srsUE to connect to it. 
# This specific example uses ZMQ in place of a USRP for the RF-frontend, and creates an FDD cell with 10 MHz bandwidth. 
# To run the srsRAN Project gNB with this config, use the following command: 
#   sudo ./gnb -c gnb_zmq.yaml

cu_cp:
  amf:
    addr: 10.53.1.2                 # The address or hostname of the AMF.
    port: 38412
    bind_addr: 10.53.1.1            # A local IP that the gNB binds to for traffic from the AMF.
    supported_tracking_areas:
      - tac: 7
        plmn_list:
          - plmn: "00101"
            tai_slice_support_list:
              - sst: 1
  inactivity_timer: 7200            # Sets the UE/PDU Session/DRB inactivity timer to 7200 seconds. Supported: [1 - 7200].

ru_sdr:
  device_driver: zmq                # The RF driver name.
  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=23.04e6 # Optionally pass arguments to the selected RF driver.
  srate: 23.04                      # RF sample rate might need to be adjusted according to selected bandwidth.
  tx_gain: 75                       # Transmit gain of the RF might need to adjusted to the given situation.
  rx_gain: 75                       # Receive gain of the RF might need to adjusted to the given situation.

cell_cfg:
  dl_arfcn: 368500                  # ARFCN of the downlink carrier (center frequency).
  band: 3                           # The NR band.
  channel_bandwidth_MHz: 20         # Bandwith in MHz. Number of PRBs will be automatically derived.
  common_scs: 15                    # Subcarrier spacing in kHz used for data.
  plmn: "00101"                     # PLMN broadcasted by the gNB.
  tac: 7                            # Tracking area code (needs to match the core configuration).
  pdcch:
    common:
      ss0_index: 0                  # Set search space zero index to match srsUE capabilities
      coreset0_index: 12            # Set search CORESET Zero index to match srsUE capabilities
    dedicated:
      ss2_type: common              # Search Space type, has to be set to common
      dci_format_0_1_and_1_1: false # Set correct DCI format (fallback)
  prach:
    prach_config_index: 1           # Sets PRACH config to match what is expected by srsUE
  pdsch:
    mcs_table: qam64                # Sets PDSCH MCS to 64 QAM

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
Then, edit `gnb_zmq.yaml` with reference to [this](https://docs.srsran.com/projects/project/en/latest/user_manuals/source/config_ref.html#id1) according to your environment.

<a id="issues"></a>

## Issues

If the latest `main` branch doesn't work, you may try the latest `release` version.

1. If the `gnb` on Virtualbox VM fails to do `NGSetup`, [this](https://github.com/srsran/srsRAN_Project/issues/172#issuecomment-1681908406) might help to solve it.
2. According to [here](https://github.com/srsran/srsRAN_4G/issues/1213), there is an issue where downlink packets stop flowing between srsue and srsgnb via ZeroMQ. The hotfix is [here](https://github.com/srsran/srsRAN_4G/issues/1213#issuecomment-1703512937).
3. According to [here](https://github.com/srsran/srsRAN_Project/issues/241#issuecomment-1756599022), 2 CPU cores or more are required to run.
4. According to [here](https://github.com/srsran/srsRAN_Project/issues/263), UE(`ue_zmq.conf`) failed to connect to Open5GS via gNodeB of srsRAN_Project 23.10. [This](https://github.com/srsran/srsRAN_Project/issues/263#issuecomment-1773756230) solved it.
5. According to [here](https://github.com/srsran/srsRAN_Project/discussions/995#discussioncomment-11634006), 4 CPU cores or more are required to run.

<a id="ver_list"></a>

## Confirmed Version List

I simply confirmed the operation of the following versions.

| Version | Commit | Date | Issues |
| --- | --- | --- | -- |
| 24.10+ | `e5d5b44b92cf18d1bd1736da0148e5f9cce3721d` | 2024.12.03 | 5 |
| 24.10 | `9d5dd742a70e82c0813c34f57982f9507f1b6d5d` | 2024.10.14 | 3 |
| 24.04+ | `4ac5300d4927b5199af69e6bc2e55d061fc33652` | 2024.07.31 | 3 |
| 24.04+ | `c33cacba7d940e734ac7bad08935cbc35578fad9` | 2024.06.10 | 3 |
| 24.04+ | `78238fd15e4cd82a6324d6dbbb612ac5584b13ea` | 2024.05.13 | 3 |
| 24.04 | `1483bda3091420cf7270eacdf31de932865c6294` | 2024.04.22 | 3 |
| 23.10.1+ | `2f90c8b60e9396a7aed59645c98dbcbccda2bf7c` | 2024.03.25 | 3 |
| 23.10.1 | `374200deefd8e1b96fab7328525fd593a808a641` | 2023.10.23 | 3 |
| 23.10 | `e38e418bda8432397b2fa7dc399cb7afde3c3b95` | 2023.10.20 | 3, 4 |
| 23.5+ | `5e6f50a202c6efa671d5b231d7c911dc6c3d86ed` | 2023.09.20 | 3 |
| 23.5+ | `1afd7240f2b5e2061ab4158e8fcdacb15961813a` | 2023.08.07 | 1, 2 |

<a id="sample_conf"></a>

## Sample Configurations

- [Open5GS 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration](https://github.com/s5uishida/open5gs_5gc_srsran_sample_config)
- [free5GC 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration](https://github.com/s5uishida/free5gc_srsran_sample_config)

<a id="changelog"></a>

## Changelog (summary)

- [2024.12.21] Updated a list of confirmed versions.
- [2024.10.16] Updated a list of confirmed versions.
- [2024.08.31] Updated a list of confirmed versions.
- [2024.05.14] Updated a list of confirmed versions. And changed the OS from Ubuntu 22.04 to 24.04.
- [2024.05.02] Updated a list of confirmed versions.
- [2024.03.29] Updated a list of confirmed versions.
- [2023.12.02] Updated a list of confirmed versions.
- [2023.11.02] Updated `gnb_zmq.yaml`.
- [2023.10.21] Added the case of srsRAN_Project 23.10.
- [2023.10.10] Added a list of confirmed versions.
- [2023.08.10] Initial release.
