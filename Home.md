This project provides a hardware accelerated URL extraction system. The NetFPGA reference router has been modified to identify HTTP packets containing URLs and send a copy to the host system. Software running on the host extracts and stores URLs and search terms into a database, and then displays them through a graphical user interface.

# Project summary #

Status: Release pending

Version: 1.0

Authors:
Michael Ciesla (m.ciesla@student.unsw.edu.au), Vijay Sivaraman

NetFPGA base source: 2.0

## Download ##

Release date is 13th July 2009.

## Description ##
### System Overview ###
The URL extraction system consists of two main components: hardware and software. The hardware component is an extended NetFPGA IPv4 reference router that identifies packets containing a HTTP GET request in hardware and sends a copy to the host. The software component is composed of three parts: URL Extractor, database, and graphical user interface. The URL Extractor parses HTTP GET packets, extracts the contained URLs and search terms, and then stores them into a database. The GUI queries the database for top occurring URLs and search terms, and displays them on-screen. A system diagram is shown in below.

![http://netfpga.org/netfpgawiki/images/6/6e/Netfpga-urlx.png](http://netfpga.org/netfpgawiki/images/6/6e/Netfpga-urlx.png)
Image adapted from [1](1.md)

### Packet Life Cycle ###

The packet life cycle of the URL extraction system can be explained in the following six step sequence:

  1. A packet enters the NetFPGA through the gigabit Ethernet ports and is put in a MAC RxQ .
  1. It then traverses the User Data Path, which processes the packet to determine the output port, and places the packet in the TxQ corresponding to the output port. The User Data Path duplicates HTTP GET packets, sending a copy up to the host by placing it into the CPU TxQ, and forwarding the other along its normal path.
  1. The packet in the MAC TxQ is sent out onto the Ethernet, whereas the packet in the CPU TxQ is transfered across the PCI Bus to the NetFPGA kernel driver.
  1. The URL Extractor software then receives the packet by reading from a socket bound to the NetFPGA software interface (nf2c0).
  1. The URL Extractor parses the HTTP GET packet and extracts the contained URL, storing it into the database. The URL is then checked for embedded search engine terms, and if found, they are also extracted and stored into the database.
  1. Finally, the GUI queries the database for top occuring URLs and search terms, displaying them on-screen.

### URL Extractor ###
Below is a screenshot of the output produced by the URL Extractor.

![http://netfpga.org/netfpgawiki/images/7/74/Urlx.png](http://netfpga.org/netfpgawiki/images/7/74/Urlx.png)

### GUI ###
Below is a screenshot of the GUI. The left pane displays the top occurring URLs in the the database. The right pane displays the top occurring Google search terms (some asian characters distort the search term count alignment).

![http://netfpga.org/netfpgawiki/images/c/c9/Urlx-gui.png](http://netfpga.org/netfpgawiki/images/c/c9/Urlx-gui.png)

## Regression Tests ##

The regression tests verify the functionality of the hardware component of the URL extractor system.  In order to run the tests, you need to have the machine connected for the regression tests as stated in the [Run Regression Tests](http://netfpga.org/netfpgawiki/index.php/Guide#Run_Regression_Tests) section of the Guide.

After connecting the cables, ensure dhclient is not running. Then execute the following command to run the regression tests.

```
 nf21_regression_tests.pl --project url_extraction
```
### Regression Tests ###

The URL extraction router contains all the same regression tests as the reference router, with the addition of three new test (below). The definition of the reference router regression tests can be found on [Router Tests](http://netfpga.org/netfpgawiki/index.php/Beta_Release_Regression_Tests#Router_Tests) wiki page.

#### Test 1: Verify duplication of Unix GET packets ####
> Name: test\_get\_unix
> Description: Tests the identification and duplication of packets containing a HTTP GET request method from a Unix client (TCP header length = 32B).
  1. Initialize netfpga hardware (same as test\_packet\_forwarding)
  1. Send 20 Unix GET packets from eth1 to eth2 and nf2c0.
  1. Send 20 Unix GET packets from eth2 to eth1 and nf2c0.
  1. Check the number of forwarded packets register and verify the value is correct.
> Location
projects/url\_extraction/regress/test\_get\_unix
> Output
```
 SUCCESS!
```
#### Test 2: Verify duplication of Windows GET packets ####
> Name: test\_get\_win

> Description: Tests the identification and duplication of packets containing a HTTP GET request method from a Windows client (TCP header length = 20B).
  1. Initialize netfpga hardware (same as test\_packet\_forwarding)
  1. Send 20 Windows GET packets from eth1 to eth2 and nf2c0.
  1. Send 20 Windows GET packets from eth2 to eth1 and nf2c0.
  1. Check the number of forwarded packets register and verify the value is correct.
> Location
projects/url\_extraction/regress/test\_get\_win
> Output
```
 SUCCESS!
```
#### Test 3: Verify non-GET packets are not duplicate ####
> Name: test\_get\_nondup

> Description: Tests that packets not containing a HTTP GET request method are forwarded correctly without being duplicated.
  1. Initialize netfpga hardware (same as test\_packet\_forwarding)
  1. Send 20 packets from eth1 to eth2 with an ip\_len < MIN\_LEN, and proto = TCP.
  1. Send 20 packets from eth2 to eth1 with an ip\_len < MIN\_LEN, and proto = TCP.
  1. Send 20 packets from eth1 to eth2 with an ip\_len < MIN\_LEN, and proto != TCP.
  1. Send 20 packets from eth2 to eth1 with an ip\_len < MIN\_LEN, and proto != TCP.
  1. Send 20 packets from eth1 to eth2 with an ip\_len > MIN\_LEN, and proto != TCP.
  1. Send 20 packets from eth2 to eth1 with an ip\_len > MIN\_LEN, and proto != TCP.
  1. Send 20 packets from eth1 to eth2 with an ip\_len > MIN\_LEN, proto = TCP,  and dst port != HTTP.
  1. Send 20 packets from eth2 to eth1 with an ip\_len > MIN\_LEN, proto = TCP, and dst port != HTTP
  1. Check the number of forwarded packets register and verify the value is correct.
> Location
projects/url\_extraction/regress/test\_get\_nondup
> > Output
```
 SUCCESS!
```
## Usage ##

### Installation ###
  * Install packages required by software components:

> yum install mysql-server mysql-devel gtk2-devel
  * Start the MySQL server:
```
 service mysqld start
```
#### Setup the MySQL Database ####
  * Set a password for the root database user:
```
 mysqladmin -u root password netfpga
 mysqladmin -u root --password reload
```
  * Create the database:
```
mysqladmin -u root -p create db
```
  * Create the database tables:
```
 cd projects/url_extraction/sw/db
 mysql -u root -p db < search_term_table.sql 
 mysql -u root -p db < url_tbl.sql 
```
#### Compile the URL Extractor ####
  * Compile the URL Extractor from the source:
```
cd projects/url_extraction/sw/urlx
 make
```
#### Compile the GUI ####
  * Compile the GUI from the source:
```
 cd projects/url_extraction/sw/gui
 make
```
### Hardware Component ###
  1. Ensure that the NetFPGA kernel driver is loaded and that the CPCI has been reprogrammed.
  1. Download the URL extraction bitfile:
```
 nf2_download url_extraction.bit
```
#### Network Configuration ####
There are two main ways to configure the router:
  1. Use the modified version of SCONE located in <tt>sw</tt> directory. This version of SCONE has been modified to drop all GET packets received on the nf2c0 interface, i.e so they don't interfere with the SCONE processes (the URL Extractor still receives the GET packets).
  1. Statically configure all networking information using the <tt>cli</tt> or Java gui. Adjacent nodes will also require a static ARP entry for the router.

### Software Component ###
#### URL Extractor ####
The URL Extractor is started by running the `urlx` binary. The `interface_name` argument specifies the network interface to receive GET packets from, e.g. nf2c0.
```
 cd project/url_extraction/sw/urlx
 ./urlx 
    Usage: ./urlx interface_name
```
#### GUI ####
The GUI is started by running the <tt>gui</tt> binary:
```
 cd project/url_extraction/sw/gui
 ./gui
```
### Testbed Setup ###
The URL Extraction system can be tested using the below topology. The NetFPGA interfaces use IP addresses 192.168.x.1, where 'x' is the interface number (starting at 1). Connect the PC to the 2nd NetFPGA port. Connect the NAT router to the 3rd NetFPGA port.

![http://netfpga.org/netfpgawiki/images/1/10/Testbed.png](http://netfpga.org/netfpgawiki/images/1/10/Testbed.png)]

On the **host system**:
  * Run SCONE (./scone). The cpuhw and rtable files have been provided for this topology. The NetFPGA has a default route through the NAT router.
```
 cpuhw:
 eth0 192.168.1.1 255.255.255.0 00:00:00:00:00:01
 eth1 192.168.2.1 255.255.255.0 00:00:00:00:00:02
 eth2 192.168.3.1 255.255.255.0 00:00:00:00:00:03
 eth3 192.168.4.1 255.255.255.0 00:00:00:00:00:04
```
```
 rtable: 
 0.0.0.0 192.168.3.2 0.0.0.0 eth2
```
  * Run the URL Extractor:
```
 ./urlx nf2c0
```
  * Run the GUI:
```
 ./gui
```

  * View accessed URLs and search terms.

On the **PC**:
  * Set the default route to 192.168.2.1
  * Start web browsing.

**NAT Router**:
  * he NAT router can be a PC running iptables or a home router/gateway. Here is a sample bash script to enable NAT where eth1 is connected to the LAN and eth0 has the public IP:
```
 #!/bin/sh
 echo "Enabling IP forwarding...\n"
 echo 1 > /proc/sys/net/ipv4/ip_forward
 echo "Flashing iptables...\n" 
 iptables -F
 echo "Adding iptables rules...\n"
 iptables -A FORWARD -i eth1 -j ACCEPT
 iptables -A FORWARD -o eth1 -j ACCEPT
 iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
## References ##
[1](1.md) J.W. Lockwood, J. Naous, G. Gibb. (2008, Aug) Building Gigabit-rate Routers with the NetFPGA: NICTA Tutorial at UNSW. Sydney, Australia. [Online](Online.md). Available: http://netfpga.org/tutorials/NICTA2008/NICTA-NetFPGA_Tutorial-Ver_2-2008_02_3.ppt