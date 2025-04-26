## ROS2 DDS Configuration
1. Increase the default Cyclone DDS Performance:
	1. Download the config file: `sudo wget https://raw.githubusercontent.com/privvyledge/autoware.f1tenth/refs/heads/main/cyclone_dds/10-cyclone-max.conf -O /etc/sysctl.d/10-cyclone-max.conf`
	2. Apply the configuration: `sudo sysctl -p /etc/sysctl.d/10-cyclone-max.conf`
	3. (optional) Append the following to your ~/.bashrc file:
	4. ```if [ ! -e /tmp/cycloneDDS_configured ]; then
    sudo sysctl -w net.core.rmem_max=2147483647
    sudo sysctl -w net.ipv4.ipfrag_time=3
    sudo sysctl -w net.ipv4.ipfrag_high_thresh=134217728     # (128 MB)
    sudo ip link set lo multicast on
    touch /tmp/cycloneDDS_configured
fi```
      5. source ~/.bashrc
2. (optional but recommended) Setup static IPs to avoid in a lab to avoid needing multi-cast and reducing the total ROS traffic.
	1. Download my cyclonedds_config_static.xml template file
	2. Add/replace the IP addresses with the IP address of your robots and remote PC. NOTE: the order is irrelevant.
	3. Add the following environment variable either in your ~/.bashrc or as a docker --env: `CYCLONEDDS_URI=file:///mnt/shared_dir/cyclonedds_config_static.xml`
3. Disable ubuntus default WiFi powersaving feature as it reduces the default throughput to conserve power:
	1. Set the value to 2: `sudo sed -i 's/wifi.powersave = 3/wifi.powersave = 2/' /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf`
	2. Restart the network manager service: `sudo systemctl restart NetworkManager`
4. Install Chrony and configure the server/client for accurate time synchronization across all machines
	1. Install chrony: `sudo apt update && sudo apt install -y chrony`
	2. Modify chrony setting on the server and PC by editing the conf file: `sudo nano /etc/chrony/chrony.conf`
		1. On the server (main PC)
			1. Add the following at the end of the file: `allow 192.168.2.194/24`
		2. On each jetson:
			1. Add the following under the 'pool' section (usually line 24): `server 192.168.2.194 iburst minpoll 0 maxpoll 5 maxdelay .05`
5. Restart the computer