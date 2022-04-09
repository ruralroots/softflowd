# Softflowd/Softflowctl – ***NetFlow™*** Flow Processor & Exporter

## Description
Softflowd is a software implementation of Cisco's *NetFlow™* traffic accounting protocol. It collects and tracks traffic flows by listening on promiscuous interfaces. It is designed for minimal CPU load on busy networks.

Softflowd semi-statefully tracks traffic flows recorded by listening on network interface(s) or by reading pcap packet capture files. These flows may be exported via *NetFlow™* to a collecting host or summarized within Softflowd itself.

Softflowd can also read, analyze, and export pcap packet capture files.

The Softflowd package on OpenWrt supports the runtime Job Control functions of Softflowctl, a companion program to Softflowd that directly interfaces to the flow control files for each active monitoring instance.

Any *NetFlow™* compatible collector should work with Softflowd. A search for *NetFlow Analysis software* will yield multiple choices for various platforms, both commercial and open source. Many commercial entities provide a complimentary collector/analysis application.

Original creator:
Damien Miller <djm@mindrot.org>
Developer website  http://mindrot.org

Current maintainer:
Hitoshi Irino <irino@sfc.wide.ad.jp><br>https://github.com/irino/softflowd/<br>
Issues to https://github.com/irino/softflowd/issues

Documentation: See Linux Man Pages - softflowd(8), softflowctl(8), bpf(4)


## Installation & Usage ##
* Update your local opkg repository (`opkg update`)
* Install Softflowd (`opkg install softflowd`). The Softflowd service is enabled by default and a sample config file is included in `/etc/config/softflowd`
* The important Softflowctl job control functions are called from CLI.
* Support for filtering flow data with Berkeley Packet Filter **bpf(4)** is not supported.


## **Softflowd/Softflowctl Available Runtime Commands**

	start		->  Start softflowd Service if not configured as auto start

			SOFTFLOWCTL RUNTIME COMMANDS

	statistics	->  Show Interface Statistics
	dump	        ->  Dump Interface Flows
	pause	 	->  Pause Interface Flow Monitoring
	resume		->  Resume Interface Flow Monitoring
	expire		->  Expire Interface Flows
	delete		->  Delete All Interface Flows
	timeouts	->  Show Interface Timeout Settings
	active		->  Show All Active Interfaces
	shutdown	->  Exit Gracefully & close softflowd
	update		->  Enable/Disable An Interface & Restart softflowd Monitoring
	debugUp         ->  Increase softflowd Debug Verbosity
	debugDown       ->  Decrease softflowd Debug verbosity`

**Syntax:** `/etc/init.d/softflowd [command] [ctlsock] [bool]`

**Hint:** You can create a persistent shortcut command alias in your shell by editing `/etc/shinit` and adding<br>
`alias sf='/etc/init.d/softflowd'`

Commands can now be executed by entering:<br>`sf [command] [ctlsock] [option] [bool]`

Entering: full pathname `/etc/init.d/softflowd` or *alias* without a command on the comand line will prompt with available commands.

### softflowd Config File – /etc/config/softflowd ###

The first line in each configured interface section (`config ctlsock`) delineates each configuration section and declares the softflowd *Control Socket File* name. Softflowd listens on this **Control Socket** for ***Softflowctl*** *Runtime Job Control Commands*. Multiple configuration sections for the various interfaces configured in OpenWrt can be created, and support for concurrent monitoring instances on different interfaces is possible.

Please be well aware that softflowd can put a significant load on your router and collector host especially when running multiple instances in high usage environments. Use caution when configuring multiple instances to run concurrently.
## Configuration File Options <br>

|Option| Default |    Description/Valid Values <cr> |
| :---:|:---:|:--- |
enabled|1|Interface to be monitored - enabled=1/disabled=0
|interface||Interface to be monitored per ifconfig
|pcap_file||    Full path to the pcap packet capture file to be analyzed. <br>Softflowd processes the whole file and displays summary information before exiting the instance.<br>**Note:**<br> *Either the host_port option OR pcap_file option MUST be configured.*|
|timeout1<br>timeout2<br>.<br>.<br>.<br>.<br>timeout8||These values refer to the timeout overrides as explained in the softflowd(8) manpage. If entered, they override the default value for the specified flow type, otherwise the default value prevails. There is no enforcement of entry order. You may configure none, one, many, or all possible timeout overrides in any order.<br>**tcp.rst**=120s	 **tcp_fin**=300s<br>**expint**=160s	**icmp**=300s<br>  **tcp**=3600s	**maxlife**=604800s<br>**general**=3600s	**udp**=300s
max_flows|8192|Maximum Flows – Specify the maximum number of flows to concurrently track. If exceeded, the flows which have least recently seen traffic are forcibly expired. The default setting corresponds to ~800K of working data.
host_port||Collector IP:Port. This specifies the Host IP and Listening Port on the Host that will analyze the exported flow data. <br>**Note:**<br>*Either the host_port option OR pcap_file option MUST be configured.*
export_version|9|NetFlow export versions 1, 5, and 9 are supported.
hoplimit ||Sets ipv4 TTL, or ipv6 hop limit to *hop_limit*.<br>Defaults to the system TTL when exporting flows to a unicast host.
tracking_level|full|Full, proto, ip - *full* (track everything in the flow), *proto* (track source and destination and protocol), and *ip* (only track source and destination addresses)
track_ipv6 |0|Set to 1  to Track ipv6 regardless whether or not it is supported. Currently only Netflow V9 supports IPv6. Primarily used for debugging,
sampling_rate|100|Periodic sampling rate (denominator).<br>Note that this is a *sampling rate* ie. if *sampling_rate* value is set to 100, 1 of every 100 flow packets will be sampled.

## Softflowctl Usage
***Syntax:***`/etc/init.d/softflowd [command] [ctlsock] [bool]`

The examples below use the example shell command alias notation (see ***Hint*** above)

* Show ***Statistics*** for the specified monitored interface flow:<br>`sf statistics wan`

  * Reports statistics for the specified interface flows.

* ***Dump*** an Active Interface flow:<br>`sf dump wan`

  * Dumps all currently active flows for the specified interface and resumes monitoring.<br>Dumped flow data is Reported - No flow processing occurs.

* ***Pause*** an Active Interface flow:<br>`sf pause wan`

  * Temporarily Pauses flow collection for the specified interface

* ***Resume*** an Active Interface flow:<br>`sf resume wan`

  * Resumes flow collection for the specified interface

* ***Expire*** an Active Interface flow:<br>`sf expire wan`

  * Expires currently active flows for the specified interface and processes and exports the data.

* ***Delete*** an Active Interface flow:<br>`sf delete wan`

  * Deletes currently active flows for the specified interface. No processing or exporting of the flow data occurs.

* Displays All ***Timeout*** Values for an Active Interface:<br>`sf timeouts wan`

  * Reports the current timeout values for the specified interface.

* Display all ***Active*** Interfaces:    `sf active`

  * Reports all currently active interfaces.

* ***Shutdown*** All Monitored Instances Gracefully and Close softflowd:<br>`sf shutdown`

  * Exits all instances of softflowd gracefully, closes softflowd processes, and cleans up the run enviroment.

  * All currently active interface flows are expired, processed, and exported.

  * To immediately terminate softflowd without expiring, processing, and exporting active flows, issue:<br>`sf stop && sf shutdown`  This is the equivalent of issuing the softflowctl:<br>`exit` command.

* ***DebugUp/DebugDown*** Increase OR Decrease the Softflowd Debug Verbosity

  `sf debugUp wan`
  * Runtime Command to increase softflowd debug verbosity level of the *wan* instance.

  `sf debugDown vpn0`
  * Runtime Command to decrease softflowd debug verbosity level of the *vpn0* instance.

* ***Update*** *Enable/Disable* An Interface and Restart Softflowd Monitoring:

	The *update* command takes a boolean operator appended to the command line<br>1=Enable/0=Disable

	Enable wan interface `sf update wan 1`

	Disable vpn0 interface  `sf update vpn0 0`
	 * This runtime command allows the user to *temporarily enable* and begin monitoring a currently disabled interface, or *temporarily disable* and shutdown a currently active interface gracefully, and restart Softflowd.

	 * To enable or disable an interface to be monitored permanently, you must edit this option in the `/etc/config/softflowd` file.
