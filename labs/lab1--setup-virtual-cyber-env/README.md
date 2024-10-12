# Setup a Cyber Lab

![[Sans-titre-2023-11-02-1412.png]]

#### Prerequisites

- VMware Workstation Pro installed
- Two Ubuntu ISO images (one for the Gateway VM and one for VM1)

**NOTE:** Network interfaces may changes depending on you installations

#### Create the Gateway VM

- Open VMware Workstation and click on `Create a New Virtual Machine`
- Select the ISO Image for Ubuntu, then follow the installation steps to install Ubuntu.
- After installation, configure two network adapters for the Gateway VM: - Go to the `VM Settings` of the Gateway VM. - **Network Adapter 1**: Set it to `NAT`. - **Network Adapter 2**: Set it to `Host-Only (VMNet1)`.
  This setup enables the gateway to route traffic from the host VM (VM1) to the internet.

#### Configure Ubuntu on the Gateway VM

- Run the following commands to install necessary packages

```bash
sudo apt update
sudo apt install net-tools iptables-persistent
```

- **Configure IP forwarding**: Open the sysctl configuration file:

```bash
sudo nano /etc/sysctl.conf
```

Uncomment the line `net.ipv4.ip_forward=1`, then save and close the file. Activate it immediately by running: `sudo sysctl -p`

- **Configure NAT**: Use iptables to enable NAT for forwarding traffic from the `inet2` (host-only) network to the internet:

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
```

Replace `enp0s3` with the NAT interface and `enp0s8` with the host-only interface. Save the rules: `sudo sh -c "iptables-save > /etc/iptables/rules.v4"`

### 2. Create the Host VM (VM1)

1. **Create a second VM** for VM1 (Ubuntu Desktop) by selecting `Create New Virtual Machine` in VMware.
2. **Network Adapter**: Ensure VM1 has only **one network adapter**, set to `Host-only (VMNet1)`.
3. **Install Ubuntu** using the provided ISO image.
4. **Assign Static IP to VM1**:
   - Edit the network configuration file: `sudo nano /etc/netplan/00-installer-config.yaml`
   - Configure it to have a static IP:

```
	network:
		ethernets:
			ens33:
				addresses: [10.1.1.10/24]
				gateway4: 10.1.1.1
				nameservers:
					addresses: [8.8.8.8, 8.8.4.4]
		version: 2
```

- Save the file and apply the changes: `sudo netplan apply`

### 3. Test Connectivity Between VMs

1. **Ping from VM1 to the Gateway**: On VM1, test the connection to the gateway with the following command:
2. **Test Internet Access from VM1**: On VM1, ping a public IP like Google's DNS:

### Exercise

- **Objective**: Ensure that the VM1 Apache server is accessible from the public internet via the Gateway VM.

1. Configure all components as described.
2. Install apache2 web server
3. Test and document the following:
   - Connectivity between VM1 and Gateway.
   - Internet access from VM1.
   - Apache server running on VM1 and accessible locally.
   - Public access to the Apache server via the Gateway's IP.
4. **Extra Task**: Secure the Apache server using basic firewall rules (use `ufw` to allow port 80 and deny other ports).
