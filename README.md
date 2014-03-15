foscam-customweb
================

Replaces the default Foscam network camera web interface with a more modern interface. The foscam_customweb script automates the process of modifying the Foscam Web UI firmware and uploading it to the device.

Requirements
===
* Foscam network camera (only tested with 8910w)
* Linux (main script 'foscam_customweb' is shell script)
* Gawk
```
sudo apt-get install gawk
```

Getting Project and Web UI Firmware
===
```
git clone https://github.com/dan-v/foscam-customweb
cd foscam-customweb
wget -O /tmp/foscam_firmware.zip "http://foscam.us/downloads/MJPEG%20indoor%20PT%20camera-11.37.2.55%20NA.zip"
unzip -o -d /tmp /tmp/foscam_firmware.zip
cp "/tmp/Web UI/2.0.10.9.bin" .
```

Web UI Customizations
===
All Web UI customizations are by default in custom_web directory. Make any changes that you want here. By default, there is just a modified index.htm page (the default page displayed on the camera). This modified index.htm contains:
* Simple default twitter bootstrap theme
* Javascript to control the camera with arrow keys

Script Usage
===
```
./foscam_customweb options
This script unpacks the Foscam Web UI, add customizations, and re-packages it.

OPTIONS:
   -h      Show this message
   -w      Web UI file (e.g. 2.0.10.9.bin)
   -s      Foscam IP Address (e.g. 192.168.1.100)
   -u      Username with admin privileges (e.g. admin)
   -c      Custom Web UI directory. Optional, defaults to custom_web.
   -p      Corresponding password for user. Optional, defaults to prompting for password. 
   -o      Name of output customized WebUI file. Optional, defaults to naming custom_<webuifilename>.
   -v      Verbose
   
Example: ./foscam_customweb -w 2.0.10.9.bin -s 192.168.1.100 -u admin
```

