piglow-sysmon
-------------
A system monitor application for the Raspberry Pi / PiGlow.
Displays CPU usage, network usage and temperature as bar charts on the PiGlow

piglow-sysmon is written in C++ and is distributed under the GNU General Public License v3

Installation
------------
I2C needs to be enabled via `sudo raspi-config` and enabling via `Interfacing options`.

piglow-sysmon requires the WiringPi library (http://wiringpi.com/)
Instructions for building and installing WiringPi are at http://wiringpi.com/download-and-install/ or you can install via:

````
git clone git://git.drogon.net/wiringPi
cd wiringPi
./build
````

piglow-sysmon can be downloaded using git

	git clone git://github.com/raspberrycoulis/piglow-sysmon.git

or by getting the zip file at

    https://github.com/raspberrycoulis/piglow-sysmon/archive/master.zip

To compile, simply run

    make

in the program directory.

Running
-------

Run with

	./piglow-sysmon

Note that piglow-sysmon requires the i2c_dev and i2c_bcm2708 kernel modules to be loaded, in order to communicate with the PiGlow. piglow-sysmon will attempt to load these modules if they are not already loaded, but this will succeed only if it is run as root (i.e. `sudo ./piglow-sysmon`). Alternatively, these modules can be loaded with the WiringPi utility gpio,

	gpio load i2c

prior to running piglow-sysmon.

Usage
-----

Run with no command-line arguments, piglow-sysmon spawns a background process which displays the SoC temperature, the CPU usage and the network usage of eth0 (the wired network jack on a Model B), updating once per second.

The upper 'arm' of LEDs indicates temperature, ranging from 40°C (outer red LED just illuminated) to 80°C (all LEDs fully illuminated.

The lower left 'arm' indicates the sum of incoming and outgoing bandwidth, on a logarithmic ranging from 100 bytes/s (red LED just illuminated) to 100Mbytes/sec (all LEDs fully illuminated).

The lower right 'arm' indicates CPU usage, from 0% (all LEDs off) to 100% (all LEDs fully illuminated).

The full command-line parameters are:

	./piglow-sysmon [-b brightness] [-n interface] [-d delay] [-c] [-h] [-?]

		   -b brightness
		        Sets the maximum LED brightess (between 1 and 100).
		        Default is 20.
		   -n interface
		        Specifies the network interface to monitor.
		        Default is eth0.
		   -d delay
		        Specifies the delay in milliseconds between updates.
		        Minimum is 10. Default is 1000. 
		   -c
		        Runs program at the console (when omitted, the default
		        behaviour is to fork a background process)
		   -? -h
		        Show this help screen

## Running as a Service
To run the script on boot, do:

````
sudo nano /lib/systemd/system/piglow-sysmon.service
````

Then paste or type the following:

````
[Unit]
Description=PiGlow System Monitor Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/home/pi/piglow-sysmon/piglow-sysmon -b 100 -n eth0 -d 250 -c > /home/pi/piglow-sysmon/piglow-sysmon.log 2>&1
Restart=always
RestartSec=5


[Install]
WantedBy=multi-user.target
````

Change permissions and enable script:

````
sudo chmod 644 /lib/systemd/system/piglow-sysmon.service
sudo systemctl daemon-reload
sudo systemctl enable piglow-sysmon.service
````

Then either `sudo reboot` or run `sudo systemctl start piglow-sysmon.service`.
