---
layout: post
title: "Getting Your Raspberry Pi Serial Number Using Python"
description: "Getting Your Raspberry Pi Serial Number Using Python"
category: [raspberry pi]
tags: [python, raspberry pi]
---

---------------------------------------

###Solution

----------------------------------------

####Bash/Perl

In Bash, it is very simple to extract... by using Perl. Use

````cat /proc/cpuinfo | perl -n -e '/^Serial[ ]*: ([0-9a-f]{16})$/ && print "$1\n"'````

For example,

````$ cat cpuinfo | perl -n -e '/^Serial[ ]*: ([0-9a-f]{16})$/ && print "$1\n"'````

000000000000000d


####Python

Raspberry Spy provide a very useful Python example.

      def getserial():
        # Extract serial from cpuinfo file
        cpuserial = "0000000000000000"
        try:
          f = open('/proc/cpuinfo','r')
          for line in f:
            if line[0:6]=='Serial':
              cpuserial = line[10:26]
          f.close()
        except:
          cpuserial = "ERROR000000000"

        return cpuserial

####References

[Licence key product pages](http://www.raspberrypi.com/mpeg-2-license-key/)

[Raspberry Spy: Getting Your Raspberry Pi Serial Number Using Python](http://www.raspberrypi-spy.co.uk/2012/09/getting-your-raspberry-pi-serial-number-using-python/)
