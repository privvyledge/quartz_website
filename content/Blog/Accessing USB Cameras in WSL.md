## Links
* Check my yellow flash drive
## Steps
1. Install WSL
2. Install USBIPD
3. Compile the WSL kernel with USB (and camera) support
4. Attach the camera to WSL
	1. List all connected USB devices and note the busid: `usbipd list`
	2. Share the camera with WSL in an administrative powershell. Using a sample busid of "7-1": `usbipd bind --busid 7-1`
	3. Attach to WSL: `usbipd attach --wsl --busid 7-1`