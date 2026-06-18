# DellPowerEdge-730xd_ServerSetupConfig
Setting up my Dell PowerEdge-730xd server.

Creating an Internal DHCP Server in ProxmoxYes, you can absolutely do this. It is a standard design pattern for building isolated enterprise testing environments.
                                                  [ Proxmox Hypervisor Host ]
                                                                                  │
┌───────────────┴───────────────┐
▼                                                                                                                                                               ▼
[ Linux Bridge: vmbr0 ]                                                                                        [ Linux Bridge: vmbr1 ]
                      (Connected to Home Router)            (Isolated Private Network)
                                         │                                                                                  │
┌───────┴───────┌───────┴───────┐
▼                                                                            ▼ ▼                                                                            ▼
[ EVE-NG VM ]      [ DHCP VM/LXC ] ──► [ New Lab VM ] [ New Lab VM ]
(WAN Management) (Interface 2)                         (Gets Local IP) (Gets Local IP)

The Setup: Inside Proxmox, you create a secondary virtual network bridge (e.g., vmbr1) but do not link it to a physical network card. This creates an entirely isolated, software-only virtual switch inside the server memory.The Infrastructure: Create a lightweight Linux VM or container (like a simple Ubuntu Server or a pfSense firewall VM) and connect its virtual network interface to vmbr1. Install a DHCP server application (like dnsmasq or isc-dhcp-server) on it and configure an internal subnet pool (e.g., 192.168.20.100 to 192.168.20.200).The Automation: Whenever you spin up a new virtual machine inside Proxmox for your lab testing, map its network card to vmbr1. As soon as it boots, it will broadcast a request across the isolated bridge, communicate with your DHCP virtual machine, and instantly receive a local lab IP address.

Port Speed Differences: You are entirely correct that the ports have different maximum speeds. The dedicated iDRAC port is physically limited to 100 Mbps (Fast Ethernet) because managing hardware requires very little data. Your server's main LOM data ports are much faster, usually 1 Gbps (1000 Mbps) or 10 Gbps.How Speed is Maintained: When you enable Shared LOM 1, the iDRAC management chip piggybacks onto the high-speed data port. The port continues to run at its full speed (e.g., 1 Gbps or 10 Gbps). Your Proxmox hypervisor gets access to the full bandwidth of that port.Traffic Allocation: The hardware chip on the motherboard automatically handles the traffic splitting at the physical layer. It intercepts any packets specifically addressed to the iDRAC IP (e.g., 10.10.10.10) and forwards them to the management processor. All other internet and VM data passes straight through to Proxmox at full wire speed.

1) The Setup: Inside Proxmox, you create a secondary virtual network bridge (e.g., vmbr1) but do not link it to a physical network card. This creates an entirely isolated, software-only virtual switch inside the server memory.

2) The Infrastructure: Create a lightweight Linux VM or container (like a simple Ubuntu Server or a pfSense firewall VM) and connect its virtual network interface to vmbr1. Install a DHCP server application (like dnsmasq or isc-dhcp-server) on it and configure an internal subnet pool (e.g., 192.168.20.100 to 192.168.20.200).

3) The Automation: Whenever you spin up a new virtual machine inside Proxmox for your lab testing, map its network card to vmbr1. As soon as it boots, it will broadcast a request across the isolated bridge, communicate with your DHCP virtual machine, and instantly receive a local lab IP address

If Tailscale is running as software inside Proxmox, and the server is shut down, the Tailscale instance is dead. You will not be able to tunnel into your house to reach the iDRAC web page to power it back on.Furthermore, you cannot install Tailscale directly onto iDRAC. iDRAC runs a locked-down, proprietary firmware security operating system that does not allow you to install third-party Linux packages or applications.

** The Solutions
1) The Smart Home Hardware Fix (Easiest & Most Reliable): Buy a cheap smart Wi-Fi plug (such as a TP-Link Kasa or Sonoff plug) for your home setup. Plug the server's power supplies into it. In the physical Dell server BIOS settings (under Power Management), change the setting for AC Power Recovery to "Last State" or "On". If you are away from home and the server is completely shut down, you can open your smart plug app on your phone, toggle the outlet power off and then back on, and the physical server will instantly boot up from scratch.

2) The Always-On Lightweight Pivot (The Professional Method): Run Tailscale on a separate, tiny, low-power device in your house that stays turned on 24/7, such as a Raspberry Pi or an old, energy-efficient mini-PC. You configure Tailscale on that device to act as a Subnet Router for your entire home network range (10.10.10.0/24). When you are at a coffee shop, you connect your laptop to Tailscale. The always-on Raspberry Pi acts as a jump box, routing your traffic directly to your iDRAC's private IP (10.10.10.10), allowing you to hit the "Power On" button cleanly from anywhere in the world

-- How Smart Plugs Work Globally Outside Your Home Network??
Smart home companies like TP-Link (Kasa), Sonoff, and Tuya solve this problem by using an architecture called a Persistent Outbound Cloud Connection. This eliminates the need for risky configurations like port forwarding.

Step-by-Step Data Flow
The Local Phone Setup: When you first configure the smart plug at home, you link it to a free personal account (e.g., your email) inside the TP-Link Kasa mobile app.

The Outbound Connection: Once the smart plug connects to your home Wi-Fi, it instantly initiates a connection outward to the public TP-Link cloud servers. Because this connection starts inside your house and goes out to the internet, your home router's firewall allows it through automatically.

The Persistent Tunnel: The smart plug keeps this tiny outbound connection open 24/7. It acts as an open, encrypted two-way tunnel. The plug constantly sends a tiny "heartbeat" signal to the cloud server every few seconds saying, "I am online, pointing to account username@email.com, waiting for instructions.

"Remote Execution: When you are at a coffee shop or on cellular data and press "Power On" in your app, your phone sends that command to the public TP-Link cloud server. The cloud server looks up your account ID, locates the open tunnel connected to your specific smart plug, and drops the command down the tunnel.

The Result: The smart plug receives the instruction and clicks the physical relay switch over to supply power to your server. This entire round-trip process takes less than a second.

To Safely Power Down: Never use the smart plug app to turn the outlet off while Proxmox is actively running. Cutting physical power abruptly to an active server can corrupt your Proxmox file system, crash your EVE-NG virtual disk drives, and damage data. Always log into the Proxmox Web GUI interface first and click "Shutdown".

DIMM (Dual In-line Memory Module): The full-sized RAM stick profile used in traditional Desktop PCs and Rack Servers.
RAM is never backward compatible. A DDR4 motherboard slot cannot accept or downgrade itself to run DDR3. Every generation represents a clean break in hardware design.
Dell 12th Generation servers (e.g., R720, R620) run on Intel Xeon v1/v2 processors, meaning they only support DDR3.
Dell 13th Generation servers (e.g., R730, R630) run on Intel Xeon v3/v4 processors, meaning they only support DDR4

DDR4 processes memory operations roughly 50% faster, eliminating internal bottlenecks. DDR4 consumes less power, reducing the heat output and keeping server fans quieter.

[ DDR3 RAM Stick ] ───► [ Offset Key Notch ] ──► (Will not fit into a DDR4 Slot)
[ DDR4 RAM Stick ] ───► [ Center Key Notch ] ──► (Will not fit into a DDR3 Slot)

Yes, enterprise servers like the Dell PowerEdge or HPE ProLiant require ECC Registered RAM to boot. If you attempt to harvest cheap, non-ECC memory sticks from a standard consumer desktop and insert them into an enterprise server, the motherboard will flag a critical POST error and refuse to start up

ECC stands for Error-Correcting Code. It is a specialized memory architecture that detects and automatically corrects internal data corruption on the fly.Deep space cosmic rays and minor electrical fluctuations can occasionally flip a binary 0 to a 1 inside a memory chip (a phenomenon known as a Bit Flip).
In a standard PC (Non-ECC): A bit flip in memory causes an instantaneous Blue Screen of Death (BSOD), system crash, or file corruption.
In an Enterprise Server (ECC): The RAM stick contains an extra physical memory chip that stores parity code.
If a bit flips, the ECC algorithm detects it, fixes the error in real-time without interrupting the operating system, and alerts the administrator via iDRAC.

A dual-processor server (like the Dell R730 or R630) typically has 24 slots total (12 slots dedicated to CPU 1, and 12 slots dedicated to CPU 2).
If you only install one physical CPU chip into the server, the other 12 RAM slots are completely disabled. You must have both CPUs installed to use all 24 slots.

No. You cannot mix slots randomly or leave accidental spaces. Server motherboards use strict multi-channel architecture.
If you insert a RAM stick into an incorrect slot, the server will throw a configuration error during boot and disable that memory entirely.

The Ordering Rule: Slots are color-coded (usually white, black, and green) and labeled alphabetically (A1, A2, A3... for CPU 1, and B1, B2, B3... for CPU 2).
The Process: You must populate the slots with the lowest numbers first, balanced evenly between both processors.
For example, if you have 4 identical sticks of RAM, you must place them in slots A1 and A2 (for CPU 1) and B1 and B2 (for CPU 2).
Leaving random empty spaces between active channels breaks the memory controller's balance.

Supported RAM Type: DDR4 ECC Registered (RDIMM) or Load Reduced (LRDIMM) memory modules. It cannot run DDR3 or desktop RAM.

Can RAM Be Overprovisioned Like a CPU?
Yes, but with a major catch. While a CPU handles overprovisioning smoothly via time-sharing, RAM is physical, rigid data workspace. Two different applications cannot occupy the exact same physical byte of memory at the exact same millisecond.Hypervisors like Proxmox handle RAM overprovisioning using a feature called Memory Ballooning and Swap Space.

How Memory Ballooning Works in Your ScenarioLet's analyze your scenario: You have a physical 64 GB RAM baseline. Proxmox needs a safe 4 GB overhead to run stably. You have three large virtual machines configured, but you promise you will never run them all at the exact same time:EVE-NG VM: Assigned 56 GBUbuntu VM: Assigned 8 GBParrot OS VM: Assigned 8 GBTotal Configured RAM: 72 GB (which exceeds your physical 64 GB limit).If you change your workflow and boot all three VMs simultaneously, Proxmox uses these preservation mechanics:

The Balloon Driver: Proxmox inflates a virtual "balloon" inside the Ubuntu and Parrot OS idle states. It forces those guest operating systems to flush their internal caches and surrender their unused RAM back to the hypervisor pool so the active EVE-NG VM can use it.

The Swap Space Layer: If all three VMs actively demand their full allocated RAM at the exact same time and exceed 64 GB, Proxmox will not instantly crash. Instead, it begins writing the excess memory data onto your Physical SSD storage drive (a backup file called Swap).
The Homelab Warning: Because an SSD is significantly slower than physical RAM chips, your entire server will experience a severe performance drop (known as disk thrashing). Your virtual network routers inside EVE-NG will lag, and your command line interface will freeze.

The Structural Solution: If you only have 64 GB of physical RAM, you can absolutely configure 72 GB worth of VMs inside Proxmox. However, you must manually turn off the Ubuntu and Parrot OS VMs before starting up your heavy EVE-NG topology to avoid hitting the slow swap file barrier.

4. Advanced CPU Overprovisioning LimitsCan I Assign 40 vCPUs to a 32-Thread Server Matrix?Yes, absolutely. The math does not stop at your physical hardware thread count.If you have a dual-CPU server yielding 16 physical cores and 32 virtual hyperthreads, you can easily provision a single EVE-NG virtual machine to use 40 vCPUs, or even create four separate 10-vCPU virtual machines.How Proxmox Executes This: """Proxmox does not tie a virtual vCPU to a physical core permanently. """
It treats virtual CPUs as processing tasks. If you assign 40 vCPUs, Proxmox breaks the workload down into 40 execution queues and schedules them across your 32 physical threads using millisecond-level time slices.
The Operational Reality: Since 90% of your networking nodes inside EVE-NG sit idle waiting for configuration commands, they rarely use their assigned CPU slices simultaneously. The system will easily handle this allocation.

5. RDIMM vs. LRDIMM Server Memory
When purchasing refurbished DDR4 server RAM, you will encounter two distinct enterprise engineering formats:
[ Motherboard Memory Channel ]
 ├──► RDIMM: Connects data lines directly to the CPU register (Moderate loads)
└──► LRDIMM: Uses a buffer chip to isolate all data lines electrically (Max loads)

--RDIMM (Registered DIMM)
How it works: Features a hardware register chip directly on the RAM stick that buffers the command and address signals coming from the CPU.Characteristics: It is the standard, default choice for 95% of homelabs. It balances high speeds with low cost and reliable error correction.

--LRDIMM (Load-Reduced DIMM)
How it works: Includes a specialized memory buffer chip that handles command signals and buffers the actual data lines. This completely isolates the CPU's memory controller from the physical electrical load of the memory chips.
Characteristics: Engineered for massive enterprise databases. It allows a server to support maximum memory capacity (e.g., filling all 24 slots with heavy 64GB or 128GB sticks) without forcing the system to downclock the memory speed to cope with the electrical resistance.

The Crucial Rules for Buying MemoryDo Not Mix Formats: You cannot mix RDIMMs and LRDIMMs inside the same server. If you place an RDIMM stick next to an LRDIMM stick, the motherboard will trigger an incompatibility halt and refuse to pass POST.

The Budget Guide: For your EVE-NG virtualization lab, buy RDIMMs. They are highly abundant, significantly cheaper on the refurbished market, and run at lower latencies than LRDIMMs under standard homelab capacities (128GB to 256GB).

Video Transcoding :
================
Software Transcoding -- (Deconde / Encode in real time).
Most burden on CPU. CPU intense, quality same.

Hardware Transcoding -- Needs dedicated Hardware.
Ex : 
Intel Quick Sync Video --> Intel CPU + Dedicated Graphics.
AMD or NVidea -- Graphic cards.

Use Case : 
Movie stored as 4K HEVC
Phone only supports 1080p H.264
Plex uses Quick Sync to convert it in real time
Without Quick Sync, the CPU would have to do all the work. (It also supports - Live Streaming).
Programs like: OBS Studio. --> can use Quick Sync to encode your stream for platforms like YouTube or Twitch while keeping CPU usage low.
✅ Most Intel Core i3/i5/i7/i9 processors with integrated graphics

❌ CPUs with an F suffix usually do not have Quick Sync because the integrated GPU is disabled.

Home Lab Services :
================
Common homelab services include:

1) Plex Media Server (media streaming)
2) Jellyfin (open-source Plex alternative)
3) Nextcloud (your own Google Drive)
4) Home Assistant (smart home control)
5) Pi-hole (network ad blocking)

400GB SSD #1
├── Proxmox OS
├── Plex
├── Nextcloud
├── Home Assistant
├── DNS
├── DHCP
 |── Small utility VMs

400GB SSD #2
└── Free for future projects

400GB NVMe #1
└── EVE-NG

400GB NVMe #2
└── Free for future projects

200GB SSD #1
└── ISO images
└── Templates
└── VM backups

200GB SSD #2
└── Free for future projects
└── Docker lab
└── Test VMs
└── Scratch space

300GB HDD
└── Shared family storage
├── Wedding videos
├── Wedding photos
├── Documents
 |── Nextcloud data


Proxmox
├── EVE-NG
├── Home Assistant
├── Plex
├── Nextcloud
├── Nginx
├── WireGuard VPN
├── Docker
├── Portainer
├── Grafana
├── Prometheus
├── DNS
 |── DHCP

Linux Bonding.  (port binding / aggregation)
Use all 4 ports and combine them as 1 single port. What happens here, you can use 1 single ethernet port and connect all the VM's to the same port. When all VM's are idle and only VM is running and it is downloading it now downloads at a max of 40GB/sec -- i.e -- 4 x 10GB Ethernet connections combined together.

LACP + multi-queue + multi-stream traffic

========================================
Port Config :
========================================

Notes: Here we have total 5 Ethernet ports.
1 - iDRAC port -- Remote Access Control to view BIOS setting and recover Proxmox and other BIOS settings.
2,3,4,5 - 10 GB Ethernet ports.
FYI, Each Ethernet ports have separate MacAddresses and IP addresses canbe assigned individually.

Step1: You can configure the reserved IP address by logging into the Router and under connected devices. Add the MacAddress, device name and add a fixed IP.
Step 2: Now on the Dell server in the iDRAC IP address setting.
You have two modes. 
a) IP Address Allocation Mode 
b) Network Card (NIC) selection Mode.

When you select IP address Allocation Mode, you have two options.
1) Static --> To set a Static IP address --> Add the same IP address that you provided on the IP under reserved IP.
2) DHCP(Default) : Here the iDRAC contacts the DHCP server(in our case the Router) and requets an IP. As there is an existing Mac address entry of the iDRAC port with a dedicated reserved IP address the DHCP server assigns the ssame IP to the iDRAC port.

--b) When you select the other option - Network Interface Card (NIC) selection mode :
1) Dedicated (default on Enterprise) -- iDRAC traffic uses only the single, dedicated iDRAC network port on the back of the server(marked with the wrench icon). This keeps your remote management traffic completely physically separated from your main server traffic data.

2) Shared(LOM1 / LOM2 / LOM3 / LOM4): In this case iDRAC traffic runs parallel to any of the network ports. This happens when the original iDRAC port has been damaged or you only have one network cable to connect.
- Here two IP address resides on the same port1/2/3/4 of the NIC. And on the router you can give two separte MAC address and reserved IP's and connect via a single cable and the traffic gets split. physical macaddress of the prot1 and the virtual mac address of iDRAC sits on the same physical NIC port.

When you share a port, that single physical ethernet cable will carry traffic for two completely independent devices (the physical iDRAC processor and your Proxmox operating system), each with its own IP and MAC address.
The network card inside your Dell server acts like a tiny, built-in network switch.

FYI, You cannot use both Dedicated iDRAC port and the LOM1/LOM2 port together and can be only used simultaneously.
How the packet routing handles this without conflicts:When a packet arrives at the LOM1 port from your switch, the network card checks the VLAN ID.If the packet is tagged VLAN 10, the network card diverts it internally straight to the iDRAC processor. Proxmox will never even see this packet.If the packet is tagged VLAN 20, the network card passes it directly up to the Proxmox hypervisor. The iDRAC completely ignores it.
3) Shared (LOM All): All ports act as iDRAC ports + server ports.
4) FailOver mode : To the point iDRAC port fails it works as the primary iDRAC port. But when it fails. It chooses one of the active LOM ports to be the IDRAC port in that case.

