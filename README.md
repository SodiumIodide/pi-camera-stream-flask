# LALT Stagecam

The setup is a Raspberry Pi 3B with a legacy Arducam attached by 3 feet of ribbon cable to the onboard SATA port. The device uses Raspbian OS Bullseye as an operating system. The box dimensions that it lives in should be 8" x 7.5", or approximately that. I believe that is what I cut, as it rests on two side-by-side 8" long 2x4s. I drilled two 1" holes into the siding of the box so ventilating with a foam lid on to keep dust out should not be a problem.

## Here's the important part

To access the stagecam on any device connected to the local network (meaning the LALT Wi-Fi network), a web browser should be opened and the URL typed in as:

`stagecam.local:1943`

## Here's how to connect if you can't

It is important to include the port number, 1943, as I'm using Flask to push the server which doesn't support production ports. If you still cannot connect, type the protocol in as well:

`http://stagecam.local:1943`

If there are issues with connecting to that address, the user may have to allow insecure connections (e.g. `http` addresses as opposed to `https` addresses). This should be a straightforward process on most browsers, but depending on certain user security settings it may not be possible. To enable insecure localhost connections on a Chromium-based browser, you could type into the browser URL:
`chrome://flags/#allow-insecure-localhost`

To enable insecure connections on a Firefox-based browser, type into the URL bar:
`about:config`
Then search for `security.ssl.enable_ocsp_stapling`, double click this item to set its value to false. Firefox will still prompt you to agree to connecting to any `http` address, but will allow you to do so.

The reason for this is simply that generating and serving an SSL certification for a localhost address is incredibly extra and unnecessary, and I'm trying not to tax the localhost server too much for every connection that it takes.

## Here's the less important part

For any troubleshooting that arises, a secure shell may be opened into the device using the admin account name and the machine name:

```bash
ssh lalt@stagecam.local
```

The password to the admin account is known to maintainers and board members.

For reasons I'll get into a little later, I would not suggest simply closing the secure shell connection via `exit`. I would suggest closing the connection via `sudo reboot`.

The stagecam program itself is located in the only directory in the `~/Documents` folder. It is a Python script that uses OpenCV to process the video and Flask to host and distribute it over the local network. It's a small machine so I do not know how many simultaneous connections it can handle. I wouldn't expect it to distribute to a whole audience, but I'd imagine at least 10-20 devices can readily connect simultaneously.

The process runs at bootup through a single line in the `/etc/profile` file. At the very end of the file there should be a `sudo python3 [stagecam-directory]/main.py`{:.language-bash .highlight} call. This will spit out some errors about missing resources when first connecting via a secure shell due to the process already running at startup. This is expected as this file is sourced at both startup and login, and as such the stagecam process is already running as the machine boots up. Logging in is simply trying to run it again. Trying to spin up a new process could cause hosting errors which is why I suggest rebooting the machine after doing any maintenance on it.

In general, if there's ever any issues with it, you should be able to readily reboot the machine just by shutting the power off and turning it back on at the outlet location. A secure shell connection is not required but is an alternative to reboot via the `sudo reboot`{:.language-bash .highlight} command. After rebooting the device, it may take 10-15 seconds for the OS to load and for the stagecam process to spin up. You will know it's successful by the camera itself showing a solid red light.

## Here's how to recreate it

If the OS ever needs to be reconstructed or the Pi3 board replaced, please fork or clone this repository for an up-to-date codebase.

Alternatively, you may use a secure shell to copy over the entire stagecam program directory that lives in `/home/lalt/Documents` before decommissioning the OS or board. An example command would look like:

```bash
cd ~/Documents
scp -r lalt@stagecam.local:/home/lalt/Documents/* .
```

The required packages on the APT package manager are as follows:

### Update the package manager

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Install libraries

```bash
sudo apt-get install libatlas-base-dev
sudo apt-get install libjasper-dev
sudo apt-get install libqtgui4
sudo apt-get install libqt4-test
sudo apt-get install libhdf5-dev
```

### Install Python packages

```bash
sudo apt-get install python3-flask
sudo apt-get install python3-numpy
sudo apt-get install libopencv-dev python3-opencv
```

### Install last package via PyPI

```bash
sudo python3 -m pip install -U imutils
```

A different package manager should have similar such packages but perhaps named differently. You may try to install the Python dependencies using the PyPI package manager via `pip` or `pip3` commands, but I found that certain dependencies like OpenCV take hours to build on a smallboard device and there's no reason to not use the precompiled versions. One important aspect is that you may have to enable legacy camera support on a Raspbian OS installation by running the command `raspi-config`. This setting is located in the "Interface Devices" menu.

After installing these dependencies on a new OS, you should be able to simply edit the `/etc/profile` file and put the same call to `sudo python3 [stagecam-directory]/main.py`{:.language-bash .highlight} at the tail end of it to have the process run at startup.

## The end

Forked for the Los Alamos Little Theater.
