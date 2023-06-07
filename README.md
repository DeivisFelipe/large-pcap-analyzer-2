[![Build Status](https://travis-ci.com/f18m/large-pcap-analyzer.svg?branch=master)](https://travis-ci.com/f18m/large-pcap-analyzer)

# Large PCAP file analyzer
Large PCAP file analyzer is a command-line utility program that performs some simple operations
on .PCAP files very quickly. This allows you to manipulate also very large PCAP files 
that cannot be easily handled with other software like <a href="https://www.wireshark.org/">Wireshark</a>.

Currently it builds and works on Linux but actually nothing prevents it from running on Windows.
It is based over the well-known libpcap.

Some features of this utility: 

1. Extract packets matching a simple BPF filter (tcpdump syntax).
2. Extract packets matching plain text.
3. Computes the tcpreplay speed required to respect packet timestamps.
4. Understands GTPu tunnelling and allows filtering via BPF filters (tcpdump syntax) the encapsulated (inner) GTPu frames.
5. Changes PCAP duration, changing the timestamp inside each packet.
6. Provides "traffic reports" e.g. the connections that transport the most bytes or packets.


# Table of Contents

* [How to install](#how-to-install)
* [Command line help](#command-line-help)
* [Example run 1: time analysis](#example-run-1-time-analysis)
* [Example run 2: raw search](#example-run-2-raw-search)
* [Example run 3: tcpdump-like](#example-run-3-tcpdump-like)
* [Example run 4: GTPu filtering](#example-run-4-gtpu-filtering)
* [Example run 5: valid TCP stream filtering](#example-run-5-valid-tcp-stream-filtering)
* [Example run 6: set PCAP duration resetting IFG](#example-run-6-set-pcap-duration-resetting-ifg)
* [Example run 7: set PCAP duration preserving IFG](#example-run-7-set-pcap-duration-preserving-ifg)
* [Example run 8: change PCAP timestamps](#example-run-8-change-pcap-timestamps)
* [Example run 9: generate traffic reports](#example-run-9-generate-traffic-reports)



# How to install

As for most Linux software, you can install the software just running:

```
	$ wget https://github.com/f18m/large-pcap-analyzer/archive/3.8.0.tar.gz
	$ tar xvzf 3.8.0.tar.gz
	$ cd large-pcap-analyzer-3.8.0/
	$ ./configure && make
	$ sudo make install
```

Or you can use one of the following installation options:

| Build Status  | Applies to |
|:-------------:|:----------:|
| [![RPM Repositories](https://copr.fedorainfracloud.org/coprs/f18m/large-pcap-analyzer/package/large-pcap-analyzer/status_image/last_build.png)](https://copr.fedorainfracloud.org/coprs/f18m/large-pcap-analyzer/) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | CentOS 7,  Fedora 27,  Fedora 28, openSUSE Leap 15.0 and openSUSE Tumbleweed. Click on the badge to reach the page with the RPM repository informations. |
| [![Snap Status](https://build.snapcraft.io/badge/f18m/large-pcap-analyzer.svg)](https://snapcraft.io/large-pcap-analyzer) | Arch Linux, Debian, Fedora, Gentoo, Linux Mint, openSUSE, Raspbian, Ubuntu. If you have [snapd](https://docs.snapcraft.io/core/install) installed, just run ```snap install large-pcap-analyzer``` |

For developers: link to [Snapcraft page for large PCAP analyzer](https://build.snapcraft.io/user/f18m/large-pcap-analyzer)


# Command line help

```
	large-pcap-analyzer version 3.8.0, built with libpcap libpcap version 1.9.1 (with TPACKET_V3)
	by Francesco Montorsi, (c) 2014-2023
	Usage:
	large-pcap-analyzer [options] somefile.pcap ...
	Miscellaneous options:
	-h,--help                this help
	-v,--verbose             be verbose
	-V,--version             print version and exit
	-q,--quiet               suppress all normal output, be script-friendly
	-w <outfile.pcap>, --write <outfile.pcap>
							where to save the PCAP containing the results of filtering/processing
	-a,--append              open output file in APPEND mode instead of TRUNCATE
	Filtering options (i.e., options to select the packets to save in <outfile.pcap>):
	-Y <tcpdump_filter>, --display-filter <tcpdump_filter>
							the PCAP filter to apply on packets (will be applied on outer IP frames for GTPu pkts)
	-G <gtpu_tcpdump_filter>, --inner-filter <gtpu_tcpdump_filter>
							the PCAP filter to apply on inner/encapsulated GTPu frames (or outer IP frames for non-GTPu pkts)
	-C <conn_filter>, --connection-filter <conn_filter>
							4-tuple identifying a connection to filter; syntax is 'IP1:port1 IP2:port2'
	-S <search-string>, --string-filter <search-string>
							a string filter that will be searched inside loaded packets
	-T <syn|full3way|full3way-data>, --tcp-filter  <syn|full3way|full3way-data>
							filter for entire TCP connections having 
								-T syn: at least 1 SYN packet
								-T full3way: the full 3way handshake
								-T full3way-data: the full 3way handshake and data packets
	Timestamp processing options (i.e., options that might change packets saved in <outfile.pcap>):
	-t,--timing              provide timestamp analysis on loaded packets
	--set-duration <HH:MM:SS>
							alters packet timestamps so that the time difference between first and last packet
							matches the given amount of time. All packets in the middle will be equally spaced in time.
	--set-duration-preserve-ifg <HH:MM:SS>
							alters packet timestamps so that the time difference between first and last packet
							matches the given amount of time. Interframe gaps (IFG) are scaled accordingly.
	--set-timestamps-from <infile.txt>
							alters all packet timestamps using the list of Unix timestamps contained in the given text file;
							the file format is: one line per packet, a single Unix timestamp in seconds (floating point supported)
							per line; the number of lines must match exactly the number of packets of the filtered input PCAP.
	Reporting options:
	-p,--stats               provide basic parsing statistics on loaded packets
	--report <report-name>
							provide a report on loaded packets; list of supported reports is:
								allflows_by_pkts: print in CSV format all the flows sorted by number of packets
								top10flows_by_pkts: print in CSV format the top 10 flows sorted by number of packets
								allflows_by_pkts_outer: same as <allflows_by_pkts> but stop at GTPu outer tunnel, don't parse the tunneled packet
								top10flows_by_pkts_outer: same as <top10flows_by_pkts> but stop at GTPu outer tunnel, don't parse the tunneled packet
	--report-write <outfile.csv>
							save the report specified by --report in CSV format into <outfile.csv>
	Inputs:
	somefile.pcap            the large PCAP trace to analyze; more than 1 file can be specified.

	Note that:
	-Y and -G options accept filters expressed in tcpdump/pcap_filters syntax. See http://www.manpagez.com/man/7/pcap-filter/ for more info.
	A 'flow' is defined as a unique tuple of (srcIP, srcPort, dstIP, dstPort) for UDP,TCP,SCTP protocols.
	Other PCAP utilities you may be looking for are:
	* mergecap: to merge PCAP files
	* tcpdump: can be used to split PCAP files (and more)
	* editcap: can be used to manipulate timestamps in PCAP files (and more)
	* tcprewrite: can be used to rewrite some packet fields in PCAP files (and more)
```

# Example run 1: time analysis

In this example we are interested in understanding how many seconds of traffic are contained in a PCAP file:

```
$ large_pcap_analyzer -t large.pcap 

No PCAP filter set: all packets inside the PCAP will be loaded.
8M packets (8751268 packets) were loaded from PCAP.
Tcpreplay should replay this PCAP at an average of 73.34Mbps / 14580.72pps to respect PCAP timings.
```

Note that to load a 5.6GB PCAP only 1.9secs were required (on a 3GHz Intel Xeon CPU).
This translates to a processing throughput of about 3GB/sec (in this mode).
RAM memory consumption was about 4MB.


# Example run 2: raw search

In this example we are interested in selecting any packet that may contain inside it the string "youtube":

```
$ large_pcap_analyzer -v -S "youtube" -w out.pcap bigcapture.pcap

Analyzing PCAP file 'bigcapture.pcap'...
The PCAP file has size 5.50GiB = 5636MiB.
No PCAP filter set: all packets inside the PCAP will be loaded.
Successfully opened output PCAP 'out.pcap'
1M packets loaded from PCAP...
2M packets loaded from PCAP...
3M packets loaded from PCAP...
4M packets loaded from PCAP...
5M packets loaded from PCAP...
6M packets loaded from PCAP...
7M packets loaded from PCAP...
8M packets loaded from PCAP...
Processing took 5 seconds.
8M packets (8751268 packets) were loaded from PCAP.
0M packets (9825 packets) matched the filtering criteria (search string / PCAP filters / valid TCP streams filter) and were saved into output PCAP.
```

Note that to load, search and extract packets from a 5.6GB PCAP only 5secs were required (on a 3GHz Intel Xeon CPU).
This translates to a processing throughput of about 1GB/sec (in this mode).
RAM memory consumption was about 4MB.


# Example run 3: tcpdump-like

In this example we are interested in selecting packets having a VLAN tag and directed or coming from an HTTP server:

```
$ large_pcap_analyzer -v -Y 'vlan and tcp port 80' -w out.pcap bigcapture.pcap

Successfully compiled PCAP filter: vlan and tcp port 80
Analyzing PCAP file 'bigcapture.pcap'...
The PCAP file has size 5.50GiB = 5636MiB.
Successfully opened output PCAP 'out.pcap'
1M packets loaded from PCAP (matching PCAP filter)...
2M packets loaded from PCAP (matching PCAP filter)...
3M packets loaded from PCAP (matching PCAP filter)...
4M packets loaded from PCAP (matching PCAP filter)...
5M packets loaded from PCAP (matching PCAP filter)...
6M packets loaded from PCAP (matching PCAP filter)...
7M packets loaded from PCAP (matching PCAP filter)...
8M packets loaded from PCAP (matching PCAP filter)...
Processing took 3 seconds.
8M packets (8751268 packets) were loaded from PCAP (matching PCAP filter).
0M packets (1147 packets) matched the filtering criteria (search string / PCAP filters / valid TCP streams filter) and were saved into output PCAP.
```

Note that to load, search and extract packets from a 2GB PCAP only 1sec was required (on a 3GHz Intel Xeon CPU).
RAM memory consumption was about 4MB.


# Example run 4: GTPu filtering

In this example we are interested in selecting packets GTPu-encapsulated for a specific TCP flow between the
IP address 1.1.1.1 <-> 1.1.1.2, on TCP ports 80 <-> 10000:


```
$ large_pcap_analyzer -v -G '(host 1.1.1.1 or host 1.1.1.2) and (port 80 or port 10000)' -w out.pcap bigcapture.pcap

Successfully compiled GTPu PCAP filter: (host 1.1.1.1 or host 1.1.1.2) and (port 80 or port 10000)
Analyzing PCAP file 'bigcapture.pcap'...
The PCAP file has size 5.50GiB = 5636MiB.
Successfully opened output PCAP 'out.pcap'
1M packets loaded from PCAP...
2M packets loaded from PCAP...
3M packets loaded from PCAP...
4M packets loaded from PCAP...
5M packets loaded from PCAP...
6M packets loaded from PCAP...
7M packets loaded from PCAP...
8M packets loaded from PCAP...
Processing took 3 seconds.
8M packets (8751268 packets) were loaded from PCAP.
8M packets (8501213 packets) loaded from PCAP are GTPu packets (97.1%).
0M packets (0 packets) matched the filtering criteria (search string / PCAP filters / valid TCP streams filter) and were saved into output PCAP.
```


# Example run 5: valid TCP stream filtering

In this example we are interested in selecting packets of TCP connections that have at least 1 SYN and 1 SYN-ACK packet
(if GTPu packets are found this analysis is done for the encapsulated TCP connections):

```
$ large_pcap_analyzer -v -T -w out.pcap bigcapture.pcap

Analyzing PCAP file 'bigcapture.pcap'...
The PCAP file has size 5.50GiB = 5636MiB.
Successfully opened output PCAP 'out.pcap'
Valid TCP filtering enabled: performing first pass
1M packets loaded from PCAP...
2M packets loaded from PCAP...
3M packets loaded from PCAP...
4M packets loaded from PCAP...
5M packets loaded from PCAP...
6M packets loaded from PCAP...
7M packets loaded from PCAP...
8M packets loaded from PCAP...
Processing took 2 seconds.
Detected 1 invalid packets, 721214 non-TCP packets and 37436 valid TCP flows (on a total of 85878 flows).
Valid TCP filtering enabled: performing second pass
Analyzing PCAP file 'bigcapture.pcap'...
The PCAP file has size 5.50GiB = 5636MiB.
1M packets loaded from PCAP...
2M packets loaded from PCAP...
3M packets loaded from PCAP...
4M packets loaded from PCAP...
5M packets loaded from PCAP...
6M packets loaded from PCAP...
7M packets loaded from PCAP...
8M packets loaded from PCAP...
Processing took 2 seconds.
8M packets (8751268 packets) were loaded from PCAP.
0M packets (4498 packets) matched the filtering criteria (search string / PCAP filters / valid TCP streams filter) and were saved into output PCAP.
```

Note that to load, search and extract packets from a 5.6GB PCAP only 4.5secs were required (on a 3GHz Intel Xeon CPU).
This translates to a processing throughput of about 1GB/sec (in this mode).


# Example run 6: set PCAP duration resetting IFG

In this example a PCAP that would take 8 minutes to be replayed (without top speed option) will be
modified to take just 1.2 seconds to replay.
To better explain the result of the processing consider the following table where the original PCAP duration
is reset from 20secs down to 10secs using `--set-duration` option:

| Frame index | Frame relative time in original PCAP | Frame relative time in output PCAP |
|-------------|--------------------------------------|------------------------------------|
| 1           | +0.0                                 | +0.0                               |
| 2           | +1.0                                 | +2.5                               |
| 3           | +15.0                                | +5.0                               |
| 4           | +18.0                                | +7.5                               |
| 5           | +20.0                                | +10.0                              |

See the following example session:

```
$ large_pcap_analyzer --timing test-pcaps/ipv4_gtpu_https.pcap
0M packets (18201 packets) were loaded from PCAP.
Last packet has a timestamp offset = 473.48sec = 7.89min = 0.13hours
Tcpreplay should replay this PCAP at an average of 0.27Mbps / 38.44pps to respect PCAP timings.

$ large_pcap_analyzer --set-duration 1.2 --write /tmp/test.pcap test-pcaps/ipv4_gtpu_https.pcap 
PCAP duration will be set to: 1.200000 secs
Successfully opened output PCAP '/tmp/test.pcap'
Packet processing operations require 2 passes: performing first pass
0M packets (18201 packets) were loaded from PCAP.
Packet processing operations require 2 passes: performing second pass
0M packets (18201 packets) were loaded from PCAP.
0M packets (18201 packets) were processed and saved into output PCAP.

$ large_pcap_analyzer --timing /tmp/test.pcap 
0M packets (18201 packets) were loaded from PCAP.
Last packet has a timestamp offset = 1.20sec = 0.02min = 0.00hours
Tcpreplay should replay this PCAP at an average of 105.00Mbps / 15167.50pps to respect PCAP timings.
```

Note that using `--set-duration` all timestamps in the resulting PCAP will have an equal inter-frame-gap (IFG). 
In other words the original IFGs will be lost.


# Example run 7: set PCAP duration preserving IFG

Repeating example #6 using `--set-duration-preserve-ifg` instead of `--set-duration` will give the same
result as far as the total PCAP duration is concerned, but the ratio between the new PCAP IFGs and the original
PCAP IFGs will be preserved.
To better explain the result of the processing consider the following table where the original PCAP duration
is scaled down by a factor of 10 using `--set-duration-preserve-ifg`:

| Frame index | Frame relative time in original PCAP | Frame relative time in output PCAP |
|-------------|--------------------------------------|------------------------------------|
| 1           | +0.0                                 | +0.0                               |
| 2           | +1.0                                 | +0.1                               |
| 3           | +15.0                                | +1.5                               |
| 4           | +16.0                                | +1.6                               |

As you can see the inter-frame-gaps (IFGs) among the packets are preserved: the packet #4 in the original PCAP
has a timestamp difference from packet #1 equal to 16secs that become 1.6secs in the rescaled PCAP.
The same ratio is found considering the timestamp difference between packet #4 and packet #3: it is 1sec in
the original PCAP and 0.1sec in the rescaled output PCAP.

```
$ large_pcap_analyzer --set-duration-preserve-ifg 1.2 --write /tmp/test.pcap test-pcaps/ipv4_gtpu_https.pcap 
PCAP duration will be set to: 1.200000 secs
Successfully opened output PCAP '/tmp/test.pcap'
Packet processing operations require 2 passes: performing first pass
0M packets (18201 packets) were loaded from PCAP.
Packet processing operations require 2 passes: performing second pass
0M packets (18201 packets) were loaded from PCAP.
0M packets (18201 packets) were processed and saved into output PCAP.

$ large_pcap_analyzer --timing /tmp/test.pcap 
0M packets (18201 packets) were loaded from PCAP.
Last packet has a timestamp offset = 1.20sec = 0.02min = 0.00hours
Tcpreplay should replay this PCAP at an average of 105.00Mbps / 15167.50pps to respect PCAP timings.
```


# Example run 8: change PCAP timestamps

In this example the timestamps of 2 packets are manually tweaked.
First of all current timestamps are extracted using a tool like [tshark](https://www.wireshark.org/docs/man-pages/tshark.html),
in Epoch format:

```
$ tshark -F pcap -r test-pcaps/timing-test.pcap -Tfields -e frame.time_epoch >pkts_timings.txt
```

Then the timestamps of the 10-th packet and 11-th packet are replaced with the absolute time "Saturday 9 February 2019 19:20:00",
corresponding to the Unix timestamp value 1549740000 (you can use an online tool like https://www.epochconverter.com/),
in the dump of packet timestamps:

```
$ sed -i '10s/.*/1549740000.000000000/' pkts_timings.txt
$ sed -i '11s/.*/1549740000.100000000/' pkts_timings.txt
```

Finally using the Large PCAP file analyzer tool, the capture trace is actually modified and the result is saved into the
"out.pcap" file:

```
$ large_pcap_analyzer --write out.pcap --set-timestamps-from pkts_timings.txt test-pcaps/timing-test.pcap
```


# Example run 9: generate traffic reports

In this example we are interested in having a quick overview of the top 10 connections contained in the PCAP file
that carry the highest number of packets.
To achieve that, from the --help overview, the "top10flows_by_pkts" traffic report is selected:

```
$ ./large_pcap_analyzer --report top10flows_by_pkts test-pcaps/ipv4_gtpu_https.pcap 
0M packets (18201 packets) were loaded from PCAP.
Packet parsing failed for 0/18201 pkts. Total number of packets/flows detected: 18201/213.
Traffic report in CSV format:
flow_num,num_pkts,%pkts,num_bytes,%bytes,flow_hash,ip_src,ip_dst,ip_proto,port_src,port_dst
0,5562,30.56,5779325,34.98,5346E247DCBCA579,10.85.73.237,202.122.145.141,6,49789,443
1,5043,27.71,5262864,31.86,359E06A6A0671690,10.85.73.237,202.122.145.141,6,40461,443
2,4328,23.78,4470177,27.06,D0D80A5F6A8B3F4F,10.85.73.237,202.122.145.141,6,50059,443
3,849,4.66,214889,1.30,13111CC415F92E54,10.85.73.237,58.71.141.53,6,49473,10000
4,209,1.15,46214,0.28,5255AFFF70F6BFB5,10.85.73.237,58.71.141.53,6,49407,10000
5,144,0.79,33478,0.20,DE86E0ED622568E9,10.85.73.237,208.67.76.95,6,49461,443
6,112,0.62,74928,0.45,EC297958A1CA3946,10.85.73.237,216.58.196.74,6,54805,443
7,101,0.55,28010,0.17,E26097004F952A54,10.85.73.237,208.67.76.95,6,49460,443
8,59,0.32,10551,0.06,D3CD1D4C2E38C2C2,10.85.73.237,121.123.204.19,6,49458,80
9,56,0.31,17508,0.11,54FD688F6BAAAF89,10.85.73.237,65.52.108.154,6,49453,443
Completed generation of 10 lines of traffic report.
```

From such output it's clear that the TCP connection (ip_proto=6) between IPs 10.85.73.237 and 202.122.145.141
on TCP ports 49789 and 443 (default HTTPs ports) is the connection which transported the highest number of 
packets (30.56%) and bytes (34.98%).

Note that, even if pkt/byte counts do not matter, the traffic reports are also an handy way to
count and dump all connections found inside a PCAP file.
