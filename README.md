foscam-customweb
================

Customized default website for Foscam network cameras. This is for the Foscam 8910W camera, but it could work with others as well.

Requirements
===
* Linux (main script 'foscam_customweb' is shell script)
* Gawk (sudo apt-get install gawk)

Getting Environment Setup
===
* git clone https://github.com/dan-v/foscam-customweb
* cd foscam-customweb
* wget -O /tmp/foscam_firmware.zip "http://foscam.us/downloads/MJPEG%20indoor%20PT%20camera-11.37.2.51%20(1).zip"
* unzip -o -d /tmp /tmp/foscam_firmware.zip
* cp "/tmp/Web UI/2.4.10.5.bin" .

Web UI Customizations
===
* All Web UI customizations are by default in custom_web directory. Make any changes that you want here.

Script Usage
===
./foscam_customweb options
This script unpacks the Foscam Web UI, add customizations, and re-packages it.

OPTIONS:
   -h      Show this message
   -w      Web UI file (e.g. 2.4.10.5.bin)
   -s      Foscam IP Address (e.g. 192.168.1.100)
   -u      Username with admin privileges (e.g. admin)
   -c      Custom Web UI directory. Optional, defaults to custom_web.
   -p      Corresponding password for user. Optional, defaults to prompting for password. 
   -o      Name of output customized WebUI file. Optional, defaults to naming custom_<webuifilename>.
   -v      Verbose
   
Example: ./foscam_customweb -w 2.4.10.5.bin -s 192.168.1.100 -u admin

