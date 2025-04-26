## RealSense D435i
### Upgrade/downgrade the firmware to version 5.15.1.55 (Support for more recent versions coming soon)
1. Download the zip file for [Version-5_15_1_55](https://www.intelrealsense.com/download/24238/?tmstv=1725546235) from the [firmware page](https://dev.intelrealsense.com/docs/firmware-releases-d400)
2. Unzip the file: `unzip d400_series_fw_5_15_1_55-1.zip` 
3. Navigate to the path of the bin file: `cd d400_series_fw_5_15_1_55-1`
4. Update by running the following command: `rs-fw-update -f Signed_Image_UVC_5_15_1_55.bin`