---
layout: post
title:  "Boost Tx-Power for region-locked wifi adapter (Alfa AWUS036NHA)"
date:   2016-08-25 11:05:20 +0700
categories: jekyll update
---
By following this tutorial, you will rebuild the Tx-Power restriction information in the OS to lift the 20 dBm restrictions in some region. We have to use this method for devices that contain hard-burned region information and couldn't be modify by commandline (eg. Alfa AWUS036NHA). For devices that can modify region by commandline, please use Google to find another tutorial because I won't cover it here

There've been a few topics regarding above issue but some packages aren't available anymore and some doesn't work with current version of Kali anymore (2016.1). I used following topics for references:

- https://forums.kali.org/showthread.php?19481-Alfa-AWUS036NHA-Tx-Power-Boost-Guide
- http://null-byte.wonderhowto.com/how-to/set-your-wi-fi-cards-tx-power-higher-than-30-dbm-0149606/

{% highlight bash %}
apt-get install python-m2crypto libgcrypt11-dev libnl-3-dev pkg-config libnl-genl-3-dev

# Build working dir
mkdir ~/regionhack
cd ~/regionhack
wget https://www.kernel.org/pub/software/network/wireless-regdb/wireless-regdb-2016.06.10.tar.gz # 10-Jun-2016 13:05
tar -zxvf wireless-regdb-2016.06.10.tar.gz

wget https://www.kernel.org/pub/software/network/crda/crda-3.18.tar.gz #30-Jan-2015 19:14
tar -zxvf crda-3.18.tar.gz

# Modify region restriction data
cd wireless-regdb-2016.06.10/
gedit db.txt
{% endhighlight %}

You can check what region your card is in by plugging in the Wireless adapter and run `dmesg`. You might see this line

```
cfg80211: Regulatory domain changed to country: GB
```

In this case, my adapter's region is GB. So I will replace the `00:` and `GB:` block as following to lift the 20dBm restriction 

```
	(2402 - 2482 @ 40), (N/A, 30)
	(5735 - 5835 @ 40), (N/A, 30)
```

After above modification, `db.txt` will contain following lines
```
country 00:
	(2402 - 2482 @ 40), (N/A, 30)
	(5735 - 5835 @ 40), (N/A, 30)
```

```
country GB:
	(2402 - 2482 @ 40), (N/A, 30)
	(5735 - 5835 @ 40), (N/A, 30)
```

Now, run following lines. Note that in Kali, there's no `/usr/lib/crda` directory, any tutorial that uses `/usr/lib/crda` will not work on Kali.

{% highlight bash %}

# Build and copy new regulatory.bin to the system
make
cp regulatory.bin /lib/crda/regulatory.bin
cp *.pem ~/regionhack/crda-3.18/pubkeys

# Copy system default trusted keys
cd /lib/crda/pubkeys
cp benh@debian.org.key.pub.pem ~/regionhack/crda-3.18/pubkeys

# Change default destination to `/lib/crda` (It was `/usr/lib/crda` orignally which isn't available in new Kali)


{% endhighlight %}

You have to replace `REG_BIN?=/usr/lib/crda/regulatory.bin` by `REG_BIN?=/lib/crda/regulatory.bin`. Save and exit.
We've reached the final building step

{% highlight bash %}
cd ~/regionhack/crda-3.18/
gedit Makefile
make
make install
{% endhighlight %}

Reboot Kali and run following command

```
iw reg set BO
iwconfig wlan0 txpower 30
# Check the Tx-Power again with
iwconfig
```