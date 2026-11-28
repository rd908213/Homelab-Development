# Chapter 2: Multi-Node Homelab Expansion — Repurposing a 2019 ROG Zephyrus (`02_second_node`)

> **Hybrid ARM + x86_64/NVIDIA Homelab Architecture**  
> *Transforming a retired 2019 ASUS ROG Zephyrus gaming laptop into a battery-bypassed, 24/7 GPU compute node (`homelab-gpu`), uniting the lab with a Ubiquiti UniFi Switch Enterprise 8 PoE (`USW-Enterprise-8-PoE`), offloading heavy Immich AI/CUDA workloads, and dedicating the Raspberry Pi (`homelab-pi`) to ultra-reliable network core services like Pi-hole DNS.*

---

## 🎯 Why This Move Makes Sense: The Hybrid Homelab Philosophy

In Chapter 1, our **Raspberry Pi 4 (4GB RAM)** proved that an ARM single-board computer can run a complete self-hosted cloud stack while sipping ~5W of power. However, running **everything** on a single 4GB ARM board introduces two engineering bottlenecks:
1. **Compute & Memory Pressure During Heavy Indexing:** When uploading hundreds of high-resolution photos or 4K videos to Immich, the `immich-machine-learning` container (running CLIP semantic vector embeddings and facial recognition) and FFmpeg video transcoding saturate the Pi's 4 ARM Cortex-A72 cores and push RAM near its 4GB limit—even with `MACHINE_LEARNING_WORKERS=1` and a 2GB SSD swapfile.
2. **Blast Radius (Fate Sharing):** If a heavy container spikes memory and triggers the Linux Out-Of-Memory (OOM) killer or requires a host reboot, any core network services running on that same Pi go offline simultaneously.

By introducing your **2019 ASUS ROG Zephyrus** (`GA502` / `GU502` / `GX502` series) as **Node 2 (`homelab-gpu`)**, you unlock the best of both architectures:

| Dimension | Node 1: Raspberry Pi 4 (`homelab-pi`) | Node 2: 2019 ROG Zephyrus (`homelab-gpu`) |
| :--- | :--- | :--- |
| **CPU Architecture** | 64-bit ARM (`aarch64`, 4 cores) | 64-bit x86 (`x86_64`, 6–8 cores / 12–16 threads) |
| **Memory (RAM)** | 4 GB LPDDR4 | 16 GB – 32 GB DDR4 |
| **GPU Acceleration** | VideoCore VI (no CUDA support) | **NVIDIA GeForce GTX 1660 Ti / RTX 2060 / 2070 (6GB–8GB VRAM, CUDA + NVENC)** |
| **Idle Power Draw** | ~4W – 6W | ~12W – 18W (Lid closed, dGPU in P8 idle state) |
| **Optimal Homelab Role** | **24/7 Storage & Network Core:** Primary NAS storage (`/mnt/homelab_storage`), Immich Core & PostgreSQL, **Pi-hole DNS**, Tailscale Subnet/DNS Router, Uptime Monitoring | **Heavy Compute & AI Worker:** Remote `immich-machine-learning` (`-cuda`), Hardware Video Transcoding (`NVENC`), Local LLMs (Ollama), Media Transcoding |

---

## 🏗️ Chapter 2 Multi-Node Architecture

```mermaid
graph TD
    subgraph LAN ["Unified Home Lab Network (UniFi USW-Enterprise-8-PoE)"]
        Router["Home Router / Gateway<br>(DHCP Static Reservations)"]
        Switch["UniFi Switch Enterprise 8 PoE<br>(8x 2.5GbE PoE+ | 2x 10G SFP+<br>80 Gbps Switching / 40 Gbps Non-Blocking)"]
        
        subgraph Node1 ["Node 1: Raspberry Pi 4 (homelab-pi | 10.0.0.200)"]
            PiOS["Raspberry Pi OS Lite 64-bit"]
            Pihole["Pi-hole v6 + Unbound<br>(Port 53 LAN & Tailnet DNS)"]
            Uptime["Uptime Kuma<br>(Port 3001 Lab Monitor)"]
            ImmichCore["immich-server + postgres + redis<br>(Port 2283)"]
            SSD["WD Blue SN5000 NVMe SSD<br>(/mnt/homelab_storage)"]
        end
        
        subgraph Node2 ["Node 2: 2019 ROG Zephyrus (homelab-gpu | 10.0.0.201)"]
            Ubuntu["Ubuntu Server 24.04 LTS (x86_64)<br>(Headless, Lid-Closed, Battery Bypassed)"]
            Nvidia["NVIDIA Driver + Container Toolkit<br>(CUDA & NVENC Runtime)"]
            ImmichML["immich-machine-learning:release-cuda<br>(Port 3003 Remote AI Worker)"]
        end
    end
    
    Router --- Switch
    Switch ===|"Port 2 (1 GbE)"| Node1
    Switch ===|"Port 3 (1 GbE / 2.5 GbE)"| Node2
    ImmichCore --> SSD
    ImmichCore ===|"Port 3003 (LAN Sub-ms RPC)"| ImmichML
    ImmichML --> Nvidia
    Uptime -. "Health Checks" .-> Node2
```

---

## 🛒 Master Shopping List & Bill of Materials (BOM)

Before beginning the hands-on notebooks, gather the following tools, thermal consumables, and networking hardware:

### 1. Laptop Battery Removal, Bypass & Thermal Maintenance Kit
| Item | Purpose | Est. Cost |
| :--- | :--- | :--- |
| **Precision Screwdriver Set (`PH0`, `PH1` Phillips + Plastic Pry Spudgers)** | Removing the 14–15 bottom chassis screws (note varying lengths!) and unclipping the Zephyrus bottom plate without marring the magnesium-alloy/plastic clips. | \$12 – \$20 |
| **High-Viscosity Non-Conductive Thermal Paste** *(PTM7950 Phase-Change Pad, Arctic MX-6, or Noctua NT-H2)* | Repasting the bare CPU and NVIDIA GPU silicon dies. High-viscosity paste or PTM7950 prevents "pump-out" during 24/7 thermal cycling. | \$8 – \$15 |
| **Viscous Thermal Putty (`K5 PRO` or `Upsiren U6 Pro`, 20g)** | Replacing dried/torn factory VRM and GDDR6 VRAM thermal pads. Eliminates pad thickness guesswork (0.5mm vs 1.0mm) so the copper heatsink seats flush against the CPU/GPU dies. | \$12 – \$18 |
| **99% Isopropyl Alcohol (IPA) + Lint-Free Swabs & Microfiber** | Dissolving baked-on 2019 factory thermal compound and flux residue from the dies and copper heatpipes. | \$6 – \$10 |
| **Kapton Polyimide Tape (10mm) & ESD Tweezers** | Insulating the exposed motherboard battery header pins after battery removal and securing internal cables. | \$6 – \$8 |
| **Adhesive Nylon P-Clips / Zip-Tie Mounts + Heavy-Duty Adhesive Velcro** | **Fixing "Table-Bump" DC Barrel Shutoffs:** Anchors the heavy ASUS AC power brick flat to the table/shelf and locks the DC barrel cable in a zero-tension strain-relief loop so bumping the desk can never wiggle the plug loose. | \$6 – \$10 |
| **Pure Sine-Wave UPS (600VA–1000VA)** *(e.g., CyberPower CP1000PFCLCD or APC BE600M1)* | **The External "Battery Bypass":** Replaces the dead internal Li-Po battery with safe, external battery backup, surge protection, and prevents abrupt Linux filesystem corruption or AC power-spike throttling during brownouts. | \$75 – \$140 |

### 2. Multi-Node Networking Expansion Kit
| Item | Purpose | Status / Est. Cost |
| :--- | :--- | :--- |
| **Ubiquiti UniFi Switch Enterprise 8 PoE (`USW-Enterprise-8-PoE`)** | Enterprise Layer-3 fanless multi-gig switch featuring **8× 2.5 GbE RJ45 PoE+ ports** (802.3af/at, 120W budget, max 32W/port), **2× 10G SFP+ ports**, 80 Gbps switching capacity (40 Gbps non-blocking), internal 150W AC PSU, and a 1.3" color LCM touchscreen. Works immediately as a wire-speed L2 switch in Chapter 2 and unlocks full UniFi Network management in Chapter 3. | **Owned (Acquired)** *(MSRP \$479)* |
| **USB 3.0 / USB-C to 2.5GbE or Gigabit Ethernet Adapter** *(Optional — unlocks full 2.5GbE on `homelab-gpu`, or required if your 2019 Zephyrus is a `GA502`/`GU502` without built-in RJ45)* | Realtek `RTL8156` (2.5GbE) or `RTL8153` (1GbE) native Linux kernel plug-and-play adapter. Pairing an `RTL8156` 2.5GbE dongle with the `USW-Enterprise-8-PoE` lets `homelab-gpu` communicate at 2.5 Gbps with the Chapter 3 `UNAS 2` NAS! | \$15 – \$25 |
| **UniFi Patch Cables (`UACC-CABLE-PATCH-0.4M-BL-10`, 0.4m Blue 10-Pack) or Cat 6 Patch Cables** | Ultra-thin 34 AWG bendable-boot patch cables rated for up to 10 GbE and PoE+++. Clean, low-latency wiring for Switch $\rightarrow$ Router, Switch $\rightarrow$ `homelab-pi`, Switch $\rightarrow$ `homelab-gpu`, and Chapter 3 `UNAS 2` expansion. | **Owned (Acquired)** |
| **32GB USB 3.0 Flash Drive** | Flashing the Ubuntu Server 24.04 LTS (`x86_64`) installer via Ventoy or BalenaEtcher to overwrite the unlicensed Windows 11 install. | \$7 – \$10 |

> 💡 **Compatibility Note (`USW-Enterprise-8-PoE` Active PoE+):** All 8 RJ45 ports support **active** IEEE 802.3af/at (PoE+, max 32W/port), meaning the switch negotiates before energizing a port—plugging non-PoE devices (`homelab-pi`, `homelab-gpu`, home router) is 100% safe. Note that because it is passively cooled (fanless), leave breathable clearance above its aluminum chassis. In **Chapter 3**, when adding the **UniFi UNAS 2** (which requires 60W **PoE++**), you will power the UNAS 2 via its included 60W PoE++ injector inline with a 2.5GbE switch port.

---

## 🗺️ Chapter 2 Notebook Index

| Step | Module | Notebook Guide | Key Milestone Objective | Verification Proof |
| :---: | :--- | :--- | :--- | :--- |
| **1** | **Hardware & Battery Bypass** | [Module 1.1: Hardware Evaluation, Battery Removal & AC Bypass](./01_hardware_prep_and_battery_bypass/01_1_hardware_eval_and_battery_bypass.ipynb) | Safe Li-Po removal, Kapton header insulation, EC reset, UPS power bridging | Internal battery bay photo & AC-only BIOS post |
| **1** | **Hardware & Battery Bypass** | [Module 1.2: Deep Thermal Maintenance & BIOS Server Tuning](./01_hardware_prep_and_battery_bypass/01_2_thermal_maintenance_and_bios_prep.ipynb) | Heatsink teardown, CPU/GPU repaste, K5 Pro VRM putty, Secure Boot off | Repasted heatsink photo & BIOS settings check |
| **2** | **OS & Headless Setup** | [Module 2.1: Replacing Windows 11 with Ubuntu Server 24.04 LTS](./02_os_installation_and_headless_setup/02_1_os_selection_and_ubuntu_install.ipynb) | Why Ubuntu Server 24.04 LTS beats Windows 11; full NVMe wipe & OpenSSH install | `hostnamectl` & `lsblk` on `homelab-gpu` |
| **2** | **OS & Headless Setup** | [Module 2.2: Clamshell Mode, Thermal Capping & SSH Baseline](./02_os_installation_and_headless_setup/02_2_headless_lid_close_and_power_tuning.ipynb) | `HandleLidSwitch=ignore`, console screen blanking, power limits, SSH keys | Closed-lid SSH session & `sensors` thermal check |
| **3** | **Lab Networking** | [Module 3.1: UniFi Switch Enterprise 8 PoE Topology & Static IP Plan](./03_network_expansion_and_cluster_topology/03_1_switch_topology_and_subnet_design.ipynb) | Wire `USW-Enterprise-8-PoE` (2.5GbE), reserve `10.0.0.201`, join Tailscale mesh & SSH jump aliases | Switch wiring photo, `iperf3` & `tailscale status` |
| **4** | **GPU & Migration** | [Module 4.1: NVIDIA Drivers, Container Toolkit & Docker](./04_gpu_compute_and_workload_migration/04_1_nvidia_drivers_and_container_toolkit.ipynb) | Headless NVIDIA driver install, `nvidia-ctk`, CUDA test in Docker, `nvidia-smi` | `docker run --gpus all nvidia/cuda nvidia-smi` |
| **4** | **GPU & Migration** | [Module 4.2: Offloading Immich Machine Learning & Transcoding](./04_gpu_compute_and_workload_migration/04_2_immich_ml_and_transcoding_offload.ipynb) | Deploy `immich-machine-learning:release-cuda` on laptop, strip ML from Pi, point Pi to `:3003` | CUDA ML logs on laptop & reduced RAM on Pi |
| **5** | **Pi Core Services** | [Module 5.1: Workload Split Strategy & Pi-hole v6 DNS](./05_pi_core_services/05_1_pi_workload_strategy_and_pihole.ipynb) | What stays on the Pi & why; deploy Pi-hole v6 in Docker + Tailscale global DNS | Pi-hole v6 dashboard & blocked query log |
| **5** | **Pi Core Services** | [Module 5.2: Unified Homelab Monitoring with Uptime Kuma](./05_pi_core_services/05_2_homelab_monitoring_uptime_kuma.ipynb) | Deploy Uptime Kuma on Pi to monitor both nodes, Immich, CUDA ML port, and DNS | Uptime Kuma multi-node status dashboard |

