---
layout: page
title: Feature Overview
permalink: /docs/features
nav_order: 3
---

# Feature Overview
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Packet capture

Packet capture (A.K.A packet sniffing or network tapping) is the process of intercepting and logging traffic that passes over a digital network or part of a network (taken from [Wikipedia](https://en.wikipedia.org/wiki/Packet_analyzer)). It is one of the most important and popular features of PcapPlusPlus and it is what PcapPlusPlus is basically all about.

There are multiple packet capture engines out there, the most popular are [libpcap](https://www.tcpdump.org/), [WinPcap](https://www.winpcap.org/) (which is libpcap for Windows), [Npcap](https://nmap.org/npcap/) (WinPcap's successor), [Intel DPDK](https://www.dpdk.org/), [ntop's PF_RING](https://www.ntop.org/products/packet-capture/pf_ring/) and [raw sockets](https://en.wikipedia.org/wiki/Network_socket#Raw_socket). Each engine has different strengths, purposes and features, works on different platforms and obviously has different APIs and setup process. Most of them are written in C (to achieve the best performance) and don't expose a C++ interface.

PcapPlusPlus aims to create a consolidated and easy-to-use C++ API for all of these engines which simplifies their complexity and provides a common infrastructure for capturing, processing, analyzing and forging of network packets.

Here is a list of of the packet capture engines currently supported:

- [libpcap](https://www.tcpdump.org/) live capture (on Linux and MacOS)
- [WinPcap](https://www.winpcap.org/)/[Npcap](https://nmap.org/npcap/) live capture (on Windows)
- [Intel DPDK](https://www.dpdk.org/) (on Linux)
- [ntop's Vanilla PF_RING](https://www.ntop.org/products/packet-capture/pf_ring/) (on Linux)
- [WinPcap Remote capture](https://www.winpcap.org/docs/docs_412/html/group__remote.html) (on Windows)

The main features provided for each one are:

- Browse all interfaces available on the machine
- Capture network traffic on a specific interface
- Send packets to the network
- Filter packets

### What's next?
{: .no_toc }

You can find out more information in the [API documentation](), [tutorial pages](/docs/tutorials) and browse through the code of the [example apps](/docs/examples).

## Packet parsing and crafting

PcapPlusPlus provides advanced capabilities for packet analysis. These include:

- Packet parsing, which is a detailed analysis of the protocols and layers a packet is built from
- Packet crafting which is manually generating and editing of network packets

A large variety of network protocols are supported, see the full list [here](#supported-network-protocols)).

Packets can be analyzed from any source including packets captured from the network, packets stored in PCAP/PCAPNG files, etc.
The design of PcapPlusPlus allows similar analysis capabilities for each packet, regardless of where it came from. For example, you can write the same code for parsing packets regardless of whether they come from DPDK, libpcap, WinPcap, raw sockets or read from a PCAP/PCAPNG file. Same goes for packet crafting.

Consider this simple code snippet that shows how to read a packet from a PCAP file, parse it, and if it's an IPv4 packet extract and print the source and dest IP addresses:

```cpp
#include "IPv4Layer.h"
#include "Packet.h"
#include "PcapFileDevice.h"

int main(int argc, char* argv[])
{
    // open a pcap file for reading
    pcpp::PcapFileReaderDevice reader("1_packet.pcap");
    if (!reader.open())
    {
        printf("Error opening the pcap file\n");
        return 1;
    }

    // read the first (and only) packet from the file
    pcpp::RawPacket rawPacket;
    if (!reader.getNextPacket(rawPacket))
    {
        printf("Couldn't read the first packet in the file\n");
        return 1;
    }

    // parse the raw packet into a parsed packet
    pcpp::Packet parsedPacket(&rawPacket);

    // verify the packet is IPv4
    if (parsedPacket.isPacketOfType(pcpp::IPv4))
    {
        // extract source and dest IPs
        pcpp::IPv4Address srcIP = parsedPacket.getLayerOfType()->getSrcIpAddress();
        pcpp::IPv4Address destIP = parsedPacket.getLayerOfType()->getDstIpAddress();

        // print source and dest IPs
        printf("Source IP is '%s'; Dest IP is '%s'\n", srcIP.toString().c_str(), destIP.toString().c_str());
    }

    // close the file
    reader.close();

    return 0;
}
```

## Read and write packets from/to files

Network packets can be stored in files, usually for offline analysis. [PCAP](https://wiki.wireshark.org/Development/LibpcapFileFormat) and [PCAPNG](https://github.com/pcapng/pcapng) are the two most popular file formats and both are supported in PcapPlusPlus.

The main feature include:

- Read packets from PCAP/PCAPNG files
- Use the [packet filtering mechanism](#packet-filtering) to read only packets that match the filter
- Create new PCAP/PCAPNG files and write packets to them
- Append packets to existing PCAP/PCAPNG files

Consider this simple code snippet that shows how to open a PCAP file for reading and another PCAPNG file for writing, and then read all packets from the PCAP file and write them to the PCAPNG file:

```cpp
// create a pcap file reader
pcpp::PcapFileReaderDevice pcapReader("input.pcap");
pcapReader.open();

// create a pcapng file writer
pcpp::PcapNgFileWriterDevice pcapNgWriter("output.pcapng");
pcapNgWriter.open();

// raw packet object
pcpp::RawPacket rawPacket;

// read packets from pcap reader and write pcapng writer
while (pcapReader->getNextPacket(rawPacket)) {
  pcapNgWriter.writePacket(rawPacket);
}
```

For more information please refer to the [Read/Write Pcap Files tutorial](/docs/tutorials/read-write-pcap), look at the [API documentation]() or browse through the code of the [example apps](/docs/examples).

## DPDK support

The Data Plane Development Kit (DPDK) is a set of data plane libraries and network interface controller drivers for fast packet processing, currently managed as an open-source project under the Linux Foundation. The DPDK provides a programming framework for x86, ARM, and PowerPC processors and enables faster development of high speed data packet networking applications (taken from [Wikipedia](https://en.wikipedia.org/wiki/Data_Plane_Development_Kit)).

DPDK provides packet processing in line rate using kernel bypass for a large range of network interface cards. Notice that not every NIC supports DPDK as the NIC needs to support the kernel bypass feature. You can read more about DPDK in [DPDK's web-site](https://www.dpdk.org/) and get a list of supported NICs [here](http://core.dpdk.org/supported/).

As DPDK API is written in C, PcapPlusPlus wraps the main functionality in a C++ easy-to-use classes which should have minimum affect on performance and packet processing rate. In addition it brings DPDK to the PcapPlusPlus framework and API so you can use DPDK together with other PcapPlusPlus features such as packet parsing and editing, etc.

You can find more information about setting up DPDK and the API in [DPDK support page](/docs/dpdk) and also in [DPDK tutorial](/docs/tutorials/dpdk).

## PF_RING support

PF_RING™ is a new type of network socket that dramatically improves the packet capture speed. It's providing high-speed packet capture, filtering and analysis (taken from [ntop's website](https://www.ntop.org/products/packet-capture/pf_ring/)).

PcapPlusPlus provides a clean and simple C++ wrapper API for [Vanilla PF_RING](https://www.ntop.org/products/packet-capture/pf_ring/). Currently only Vanilla PF_RING is supported which provides significant performance improvement in comparison to libpcap or Linux kernel, but [PF_RING Zero Copy](https://www.ntop.org/products/packet-capture/pf_ring/pf_ring-zc-zero-copy/) (which allows kernel bypass and zero-copy of packets from NIC to user-space) is not supported.

In order to compile PcapPlusPlus with PF_RING you need to:

- Download PF_RING from [ntop's web-site](https://www.ntop.org/get-started/download/#PF_RING)
- Note that PcapPlusPlus was tested with PF_RING versions 6.2.0 - 6.4.1. Later versions aren't guaranteed to work. Please report if you're seeing issues with later versions
- Once PF_RING is compiled successfully, you need to run PcapPlusPlus's `configure-linux.sh` script and type "y" in "Compile PcapPlusPlus with PF_RING?"
- Then you can compile PcapPlusPlus as usual
- Before you activate any PcapPlusPlus application that uses PF_RING, don't forget to enable PF_RING kernel module. If you forget to do that, PcapPlusPlus will output an - appropriate error on startup which will remind you:

```shell
sudo insmod <PF_RING_LOCATION>/kernel/pf_ring.ko
```

## Packet reassembly

Network protocols often need to transport large chunks of data which are complete in themselves, e.g. when transferring a file. The underlying protocol might not be able to handle that chunk size (e.g. limitation of the network packet size), or is stream-based like TCP, which doesn’t know data chunks at all.

In that case the network protocol has to handle the chunk boundaries itself and (if required) spread the data over multiple packets. It obviously also needs a mechanism to determine the chunk boundaries on the receiving side.

This mechanism is called reassembly, although a specific protocol specification might use a different term for this (e.g. desegmentation, defragmentation, etc). 

(this description is taken from [Wireshark documentation](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvReassemblySection.html)).

PcapPlusPlus currently supports two types of packets reassembly:

- IPv4 and IPv6 defragmentation which is a Layer 3 (Network later) packet reassembly. You can read about IP fragmentation [here](https://en.wikipedia.org/wiki/IP_fragmentation). To get more information on how it works and the API to use it please refer to the [API documentation]() and browse through the code of the [IPDefragUtil](/docs/examples#ipdefragutil) and [IPFragUtil](/docs/examples#ipfragutil) example apps
- TCP reassembly which is a Layer 4 (Transport layer) packet reassembly. To get more information on how it works and the API to use it please refer to the [API documentation]() and browse through the code of the [TcpReassembly](/docs/examples#tcpreassembly) example app

## Packet filtering

Most packet capture engines contain packet filtering capabilities. In order to set the filters there should be a known syntax user can use. The most popular syntax is [Berkeley Packet Filter (BPF)](http://en.wikipedia.org/wiki/Berkeley_Packet_Filter). Detailed explanation of the syntax can be found here: http://www.tcpdump.org/manpages/pcap-filter.7.html.

The problem with BPF is that, for my opinion, the syntax is too complicated and too poorly documented. In addition the BPF filter compilers may output syntax errors that are hard to understand. My experience with BPF was not good, so I decided to include in PcapPlusPlus a filter mechanism which is more structured, easier to understand and less error-prone by creating classes that represent filters. Each possible filter phrase is represented by a class. The filter, in the end, is that class.

Consider the following code snippet for creating the filter `src ip 1.1.1.1 and dst port 80` and setting it up on a packet capture device:

```cpp
// create a filter instance to capture only traffic on port 80
pcpp::PortFilter portFilter(80, pcpp::DST);

// create a filter instance to capture only TCP traffic
pcpp::IPFilter ipFilter("1.1.1.1", pcpp::SRC);

// create an AND filter to combine both filters - capture only TCP traffic on port 80
pcpp::AndFilter andFilter;
andFilter.addFilter(&portFilter);
andFilter.addFilter(&ipFilter);

// set the filter on the device
dev->setFilter(andFilter);
```

You can read more in the `PcapFilter.h` API documentation and in the exampl

## Supported network protocols

PcapPlusPlus currently supports parsing, editing and creation of packets of the following protocols:

1. Ethernet
2. SLL (Linux cooked capture)
3. Null/Loopback
4. Raw IP (IPv4 & IPv6)
5. IPv4
6. IPv6
7. ARP
8. VLAN
9. VXLAN
10. MPLS
11. PPPoE
12. GRE
13. TCP
14. UDP
15. ICMP
16. IGMP (IGMPv1, IGMPv2 and IGMPv3 are supported)
17. SIP
18. SDP
19. Radius
20. DNS
21. DHCP
22. HTTP headers (request & response)
23. SSL/TLS - parsing only (no editing capabilities)
24. Packet trailer (a.k.a footer or padding)
25. Generic payload

## License

PcapPlusPlus is released under the [Unlicense license](https://unlicense.org/).