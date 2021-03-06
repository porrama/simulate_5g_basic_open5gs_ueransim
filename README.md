## Simulation 5G network with Virtual RAN and UE

---

## Table of Contents
- [Software and Hardware](#id-specification)
- [Network Topology and Simulation Model](#id-overview)
- [Initial Preparation](#id-init)
- [Software Configulation](#id-configure)
- [Simulation Testing](#id-testing)
- [Reference](#id-reference)

---

<div id='id-specification'/>

## Software and Hardware

### Hardware Specification
| Module           | Detail                                         |
| -----------      | -----------                                    |
| Operating System | Windows 11 Home 21H2                           |
| Processer        | 11th Gen Intel(R) Core (TM) i5-1135G7 @2.40GHz |
| RAM              | 16.0 GB                                        |
| Graphics Card    | Intel(R) Iris(R) Xe Graphics                   |
| Connectivity     | Intel(R) Wi-Fi 6 AX201 160MHz                  |

### Software Used
| Software        | Version                     |
| -----------     | -----------                 |
| VirtualBox      | VirtualBox 6.1              |
| Open5GS         | Updated Jan, 2022           |
| UERANSIM        | Release v3.2.5              |
| Ubuntu Server   | Ubuntu Server 20.04 LTS     |
| Ubuntu Desktop  | Ubuntu Desktop 20.04.4 LTS  |

---

<div id='id-overview'/>

## Network Topology and Simulation Model

<img
  src="/Images/network_overview.jpg"
  title="Network Overview"
  style="display: inline-block; margin: 0 auto; max-width: 300px">
  
---

<div id='id-init'/>

## Initial Preparation

### 1. [Open5GS software](https://github.com/porrama/install_open5gs) 
- VM#1 (Core Network)

### 2. [UERANSIM software](https://github.com/porrama/install_ueransim) 
- VM#2 (RAN gNodeB)
- VM#3 (RAN UE 1)
- VM#4 (RAN UE 2)

---

<div id='id-configure'/>

## Software Configuration

Clone this project
~~~
cd ~
git clone https://github.com/porrama/simulate_5g_basic_open5gs_ueransim
~~~

### 1. Network Setting of in VirtualBox

Setting -> Network -> Choose the Adapter and ***Enable Network Adapter*** Option

| Virsual Box Number | Role               | Adapter                                       | Operation System  |
| -----------        | -----------        | -----------                                   | -----------       |
| VM#1               | Core Network - CP  | #1 Host Only Adapter                          | Ubuntu Server     |
| VM#2               | Core Network - UP  | #1 Host Only Adapter <br> #2 NAT Network      | Ubuntu Server     |
| VM#3               | RAN gNodeB         | #1 Host Only Adapter                          | Ubuntu Server     |
| VM#4               | RAN UE 1           | #1 Host Only Adapter                          | Ubuntu Desktop    |
| VM#5               | RAN UE 2           | #1 Host Only Adapter                          | Ubuntu Desktop    |

### 2. IP Setting

- Configuration File of **Core Network - VM#1 and VM#2**
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file
sudo rm /etc/netplan/00-installer-config.yaml
~~~
Core Network - CP (VM#1)
~~~
sudo cp 00-installer-config-control.yaml /etc/netplan/00-installer-config.yaml
sudo netplan apply
~~~
Core Network - UP (VM#2)
~~~
sudo cp 00-installer-config-user.yaml /etc/netplan/00-installer-config.yaml
sudo netplan apply
~~~

- Configuration File of **RAN gNodeB - VM#3** 
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/ran_configuration_file
sudo rm /etc/netplan/00-installer-config.yaml
~~~
~~~
sudo cp 00-installer-config-gnodeb.yaml /etc/netplan/00-installer-config.yaml
sudo netplan apply
~~~

### 3. Network Function & Network Setting (Open5GS)

- Access and Mobility Management Function (AMF)
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file
rm ../../open5gs/install/etc/open5gs/amf.yaml
cp amf.yaml ../../open5gs/install/etc/open5gs/amf.yaml
~~~

-  Session Management Function (SMF)
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file
rm ../../open5gs/install/etc/open5gs/smf.yaml
cp smf.yaml ../../open5gs/install/etc/open5gs/smf.yaml
~~~

- WebUI

[Run WebUI](#id-webui) -> **http://192.168.157.111:3000** -> Login -> Subscriber Menu -> Click + Button -> Fill **IMSI** -> SAVE 

> Username : admin <br>
> Password : 1423

| UE              | IMSI                | DNN                | OP/ OPc            |
| -----------     | -----------         | -----------        | -----------        |
| 1               | 001010000000000     | Internet           | OPc                | 
| 2               | 001010000000001     | Internet           | OPc                |

- User Plane Function (UPF)
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file
rm ../../open5gs/install/etc/open5gs/upf.yaml
cp upf.yaml ../../open5gs/install/etc/open5gs/upf.yaml
~~~

- Network setting < net.ipv4.ip_forward=1 >
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file
sudo rm /etc/sysctl.conf
sudo cp sysctl.conf /etc/sysctl.conf
~~~

- ogstun interface
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file
sudo sh ogstuni.sh
~~~

### 4. RAN gNodeB (UERANSIM)

- open5gs-gnb.yaml
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/ran_configuration_file
rm ../../UERANSIM/config/open5gs-gnb.yaml
cp open5gs-gnb.yaml ../../UERANSIM/config/open5gs-gnb.yaml
~~~

### 5. RAN UE (UERANSIM)

- open5gs-ue1.yaml
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/ran_configuration_file
rm ../../UERANSIM/config/open5gs-ue.yaml
cp open5gs-ue1.yaml ../../UERANSIM/config/open5gs-ue.yaml
~~~

- open5gs-ue2.yaml
~~~
cd ~/simulate_5g_basic_open5gs_ueransim/ran_configuration_file
rm ../../UERANSIM/config/open5gs-ue.yaml
cp open5gs-ue2.yaml ../../UERANSIM/config/open5gs-ue.yaml
~~~

- set default interface (UE1)
~~~
sudo route add default gw 10.45.0.2
~~~

- set default interface (UE2)
~~~
sudo route add default gw 10.45.0.3
~~~

- set DNS
~~~
cat /etc/resolv.conf
~~~

---

<div id='id-testing'/>

## Simulation Testing

Run **runnfv_open5gs.sh**
~~~
cd ~/open5gs/install/bin
sudo sh ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file/runnfv_open5gs_control.sh
~~~ 
~~~
cd ~/open5gs/install/bin
sudo sh ~/simulate_5g_basic_open5gs_ueransim/core_configuration_file/runnfv_open5gs_user.sh
~~~ 

<div id='id-webui'/>

Run **runwebui_open5gs.sh** 
~~~
cd ~/open5gs/webui
sudo sh ~/install_open5gs/runwebui_open5gs.sh
~~~

Run **gNodeB**
~~~
cd ~/UERANSIM/build/
sudo ./nr-gnb -c ../config/open5gs-gnb.yaml
~~~

Run **UE1 & UE2**
~~~
cd ~/UERANSIM/build/
sudo ./nr-ue -c ../config/open5gs-ue.yaml
~~~

---

<div id='id-reference'/>

## Reference
- [Open5GS Documentation](https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources)
- [UERANSIM Documentation](https://github.com/aligungr/UERANSIM)
- [s5uishida Project](https://github.com/s5uishida)

---
