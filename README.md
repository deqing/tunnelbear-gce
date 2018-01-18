# TunnelBear-GCE

## The Problem

[Google Compute Engine][4] provides an easy way to deploy virtual machines. One problem it currently has, is although it provides regions and zones for the physical deployment, the allocated external IP is still in the US as described in [this stackoverflow question][5] and [this Google grouop thread][6]. For example, even an instance is created in Australia region, it still not able to visit Australia-only websites.

## Install An VPN on Google Compute Engine

A way to solve the problem above is to use VPN. Following describes how to install [TunnelBear VPN][1] on a GCE instance that runs on Ubuntu, the idea comes from [tunnelbear-helper][3].

### Initial setup

Install `openvpn` and `ruby` if they are not already installed.

	sudo apt-get install openvpn ruby

The TunnelBear OpenVPN config files need to be downloaded. They can be found at a link on the [Linux support page][2]. The file is named `openvpn.zip` or similar.

After downloading, unzip the file and rename the folder.

	unzip openvpn.zip
	mv openvpn tunnelbear.d

### Authorization file

TunnelBear uses user/password authentication on top of the provided key files. OpenVPN can load this information from a file when it’s started. The TunnelBear `systemd` unit file expects a key file, if you don’t want to use one, delete the `--auth-user-pass /etc/openvpn/tunnelbear.d/tb-auth.key \` line from that file. But then the username and password will have to entered everytime a connection is started.

Create the auth file in the same folder as the config files.

	touch tunnelbear.d/tb-auth.key
	vi tunnelbear.d/tb-auth.key

The auth file is two lines only. This is the same information that is used to log into the TunnelBear website:

	email
	password

### Configuration file

Copy one of the ovpn files to your own file, e.g. `aus.ovpn`, add following two lines:

	keepalive 10 30
	auth-user-pass tb-auth.key

### File permissions

The files need to be owned by the root account, and not otherwise readable. Change the permissions, and then the ownership:

	chmod 600 tunnelbear.d/*
	sudo chown root:root tunnelbear.d/*

### Installation

Finally! First copy the the config folder into place.

	sudo cp -r tunnelbear.d /etc/openvpn/

Then, download `tunnelbear@.service` and `tunnelbear` from this repo, and:

1. Copy the systemd unit file into place.

	sudo cp tunnelbear@.service /usr/lib/systemd/system/

2. Copy the tunnelbear script into place.

	sudo cp tunnelbear /usr/local/bin/

### Usage

IMPORTANT: For an Ubuntu GCE instance we usually connect via SSH, when VPN turns on, the SSH will not be able to reach the instance, so need to turn it off after using it.

Here is one example of using it:

	sudo systemctl start tunnelbear@aus; sleep 10; wget "www.luxbet.com"; sudo systemctl stop tunnelbear@aus

### Feedbacks

Note that this tutorial might not be correct, as several methods had been tried out on the instance, I might be missing or mixed with configurations from other VPNs. Please feel free to raise on issue if you see any errors.


[1]: https://www.tunnelbear.com
[2]: https://www.tunnelbear.com/updates/linux_support/
[3]: https://github.com/JenniferMack/tunnelbear-helper
[4]: https://cloud.google.com/compute/
[5]: https://stackoverflow.com/a/47291482/558892
[6]: https://groups.google.com/forum/#!topic/gce-discussion/RjzyHRBRujg

