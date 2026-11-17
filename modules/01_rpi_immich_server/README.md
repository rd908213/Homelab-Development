# Chapter 1: Raspberry Pi Immich NAS Server (`01_rpi_immich_server`)

> **Self-Hosted Raspberry Pi 4 Photo Cloud & Network-Attached Storage (NAS)**  
> *A 7-step, 16-notebook engineering guide taking a Raspberry Pi 4 (4GB) and external NVMe SSD from bare metal to a zero-trust, Tailscale-connected Immich photo server.*

---

## 🎯 Chapter Overview

In Chapter 1, we engineered a dedicated, self-hosted photo cloud and NAS from the ground up using a **Raspberry Pi 4 Model B (4GB RAM)** and an external **Western Digital WD Blue SN5000 NVMe SSD** in a **UGREEN 10Gbps USB 3.0 UASP enclosure**. 

Following the **Purdue ME597 (IIoT)** laboratory standard, the build is broken down into **7 modular steps** and **16 self-contained Jupyter notebooks** with inline verification checkpoints.

---

## 🗺️ Chapter 1 Notebook Index

| Step | Module | Notebook Guide | Key Milestone Objective | Verification Proof |
| :---: | :--- | :--- | :--- | :--- |
| **1** | **OS Setup** | [Module 1.1: OS Selection & Prep](./01_os_setup/01_1_os_selection_and_prep.ipynb) | Evaluate OS choices; configure 64-bit ARM image in Imager | Imager customization panel |
| **1** | **OS Setup** | [Module 1.2: Headless Flashing & Boot](./01_os_setup/01_2_headless_flashing_and_boot.ipynb) | Flash media, verify power delivery, headless initial boot | Flashing verification & LED status |
| **1** | **OS Setup** | [Module 1.3: Post-Boot Security & Password](./01_os_setup/01_3_post_boot_security_and_password.ipynb) | Rotate bootstrap password, verify PAM & `sudo -k` elevation | `passwd` & `sudo whoami` verification |
| **2** | **Networking** | [Module 2.1: LAN Discovery & Static IP](./02_network_configuration/02_1_network_discovery_and_ip.ipynb) | mDNS scan, router DHCP static lease / MAC reservation | Initial ping, router DHCP table & static IP ping |
| **2** | **Networking** | [Module 2.2: Hardening SSH & Baseline](./02_network_configuration/02_2_secure_ssh_and_baseline.ipynb) | Ed25519 key login, disable password SSH, package upgrade | SSH login banner & `htop` dashboard |
| **3** | **Storage** | [Module 3.1: SSD Hardware & Partitioning](./03_storage_expansion/03_1_ssd_hardware_and_partitioning.ipynb) | USB 3.0 / PCIe UASP check, GPT partition, format `ext4` | Hardware setup photo & `lsblk -f` partition |
| **3** | **Storage** | [Module 3.2: Persistent Mount (`fstab`)](./03_storage_expansion/03_2_persistent_mount_fstab.ipynb) | UUID query, `/etc/fstab` `nofail` entry, reboot resilience | `df -h /mnt/homelab_storage` post-reboot |
| **4** | **Personal VPN** | [Module 4.1: VPN & Mesh Architecture](./04_personal_vpn/04_1_vpn_architecture_tailscale.ipynb) | Understand CGNAT, WireGuard mesh vs port forwarding | Tailscale admin console setup |
| **4** | **Personal VPN** | [Module 4.2: Tailscale Node Provisioning](./04_personal_vpn/04_2_tailscale_setup_and_verification.ipynb) | Install Tailscale, register node, Avahi mDNS fix, peer ping | `tailscale status` & client ping |
| **5** | **Docker** | [Module 5.1: Docker Engine Installation](./05_docker_runtime/05_1_docker_engine_installation.ipynb) | 2GB SSD swapfile, official Docker repo, non-root group | `docker run hello-world` output |
| **5** | **Docker** | [Module 5.2: Docker Compose & Homelab Layout](./05_docker_runtime/05_2_docker_compose_and_homelab_layout.ipynb) | Compose plugin, directory tree, test container | `docker compose ps` test stack |
| **6** | **Immich NAS** | [Module 6.1: Immich Architecture & `.env`](./06_immich_nas_deployment/06_1_immich_architecture_and_env.ipynb) | Microservices overview, Pi thread tuning, SSD path map | Custom `.env` storage mapping |
| **6** | **Immich NAS** | [Module 6.2: Compose Stack & First Boot](./06_immich_nas_deployment/06_2_immich_compose_and_first_boot.ipynb) | Image pull, database migrations, container healthchecks | `docker compose ps` all healthy |
| **6** | **Immich NAS** | [Module 6.3: Admin Setup & Initial Library](./06_immich_nas_deployment/06_3_immich_admin_and_photo_library.ipynb) | Admin onboarding, test photo ingest, resource profiling | Immich web UI & uploaded albums |
| **7** | **Remote Access** | [Module 7.1: Remote Access Security](./07_remote_access/07_1_remote_access_security_and_options.ipynb) | Zero-trust threat model, Cloudflare ToS limits, MagicDNS | Web access over Tailscale FQDN |
| **7** | **Remote Access** | [Module 7.2: Mobile Sync & Maintenance](./07_remote_access/07_2_mobile_app_sync_and_outside_testing.ipynb) | iOS/Android app config, off-network cellular sync test | Mobile app sync confirmation |

---

## 🧠 Key Learnings & Engineering Takeaways from Chapter 1

Building and refining Chapter 1 surfaced several critical real-world operational lessons that carry forward into future multi-node expansions:

### 1. Networking, mDNS & Tailscale Pitfalls
- **Avahi + Tailscale Hostname Collisions (`homelab-pi-2.local`):** When Tailscale starts its `tailscale0` virtual interface, `avahi-daemon` on Raspberry Pi OS can misinterpret reflected packets on `tailscale0` as a conflicting device on the network, silently renaming the host to `homelab-pi-2.local` and breaking SSH shortcuts.
  - **Fix:** Point `~/.ssh/config` `HostName` directly to the router-reserved static IP (e.g., `10.0.0.200`) rather than `.local`, and set `deny-interfaces=tailscale0` in `/etc/avahi/avahi-daemon.conf`.
- **Zero-Trust Mesh over Port Forwarding & Cloudflare Tunnels:** Serving bulk media/video through free Cloudflare Tunnels violates Section 2.8 of their Terms of Service, and public port forwarding exposes the server to brute-force scanning. Tailscale's WireGuard mesh with **MagicDNS** provides encrypted, zero-port-forwarded remote access from anywhere.

### 2. Storage Reliability & Boot Safety
- **Boot-Safe `/etc/fstab` Mounts (`nofail` & `noatime`):** External USB/NVMe drives must always be mounted by immutable filesystem `UUID` (never `/dev/sda1`, which can shift across reboots) with `defaults,noatime,nofail`. Without `nofail`, if the USB enclosure is unplugged or slow to initialize on power-up, `systemd` drops the headless Pi into an unreachable emergency shell.
- **Filesystem Ownership & Scheduled TRIM:** Newly formatted `ext4` partitions default to `root:root` (`755`). Delegating ownership (`chown -R pi:pi` and `chmod 775`) is mandatory before launching non-root Docker workloads. Furthermore, verifying `fstrim.timer` ensures weekly background SSD garbage collection without inline `discard` latency.
- **Hardware Power & UASP:** Connecting the UGREEN RTL9210 NVMe enclosure strictly to the **blue USB 3.0 ports** ensures UASP speed (~300–400 MB/s vs. 40 MB/s on USB 2.0), and pairing a DRAM-less NVMe SSD (WD Blue SN5000) with an official 15W/27W power supply prevents brownouts during heavy write bursts.

### 3. Memory Management & OS Evolution on SBCs
- **Swap File Provisioning on Bookworm (`fallocate` vs. `dphys-swapfile`):** Newer 64-bit Raspberry Pi OS (Debian Bookworm) releases deprecated the legacy `dphys-swapfile` service. Allocating a persistent **2GB (`2048MB`) swap file directly on the external NVMe SSD** (`/mnt/homelab_storage/swapfile`) using `fallocate`, `mkswap`, and `swapon` prevents the Linux OOM killer from terminating PostgreSQL or Immich during heavy indexing—while completely sparing the MicroSD boot card from swap wear.
- **Immich ML Concurrency Throttling:** On a 4GB RAM board, pinning `MACHINE_LEARNING_WORKERS=1` in `.env` keeps CLIP vector embedding and facial recognition jobs well within physical RAM envelopes (~3.8GB usable on headless OS Lite).

### 4. Container Runtime & Stack Architecture
- **Upstream Docker & Modern Compose Plugin:** Debian default apt repositories ship outdated `docker.io` packages. Installing from Docker's official GPG-signed apt repository gives access to modern `docker-ce` and the integrated `docker compose` CLI plugin (Compose v5+).
- **Decoupling Compose Configs from Bulk Data:** Keeping declarative stack definitions (`docker-compose.yml` and `.env`) organized in `~/homelab/stacks/<service>` while mapping both `UPLOAD_LOCATION` and `DB_DATA_LOCATION` (`pgvector`) to `/mnt/homelab_storage/` cleanly separates ephemeral container logic from persistent SSD storage and simplifies 3-2-1 backup snapshots.

