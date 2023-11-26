---
weight: 4
title: "Building a Virtual Machine Lab Part 1: Introduction"
description: "Introduction to a blog series about building a reversing lab with VMWare"
date: "2023-11-26"
draft: false
author: "philmish"
tags: ["Practical Malware Analysis", "Writeup", "Reverse Engineering", "Learning"]
categories: ["Learning"]
lightgallery: true
---
## Motivation

Not to long ago I started to work on the training material provided by  [Practical Malware Analysis](https://nostarch.com/malware) using a pretty basic VM setup by assigning the Reversing Windows VM no network adapter after installing all the tools I wanted and called it a day.

This would be sufficient to avoid unwanted network communication from the machine. Additional I made sure there is no mounted file shares to avoid accidentally installing malicious/unwanted stuff to my actual machine.

Even though this would be enough to do the static analysis parts of the book without the danger of installing unwanted stuff to the host, it would not suffice for any dynamic analysis. While researching sources to figure out how to setup a VM lab environment which would make sure there is as little as possible chance that something could escape from my reversing machine to keep my host save. I know, as these are pretty old samples the book is working with there is little chance that something actually bad happens, but eh I am paranoid and ready to learn.

## Planning

After some research I found [Building Virtual Machine Labs: A Hands-on Guide (Second Edition)](https://leanpub.com/avatar2).
This book provides a Hands-On Guide for setting up virtual machine labs for 3 different hypervisors. I can recommend it to anyone who wants to get into building a little bit more involved VM Labs than what I described above. It goes into detail about setting up some basic networking and describes the specific setups required for building a lab used for learning offensive Cyber Security with Boot To Root machines.

I will use the scenario described in the book as basis for building my reversing lab.

As a tl;dr for what the book describes, we will setup 3 Network segments to manage the virtual machines. At the core of the setup is a `pfSense` firewall VM, which will be responsible to for managing the access of the different machines to other networks. 
Outbound communication to the internet will be provided through the  `pfSense` VM, which will be connected to a virtual bridge network. All access will be managed through firewall settings.
## The Host

As a rough outline for what I am working with:

Host:
	- Windows 10
	- 32gb of RAM
	- 500 GB SSD
	- AMD Ryzen 3600 6-Core

I chose to use `VMWare Workstation 17`, as it was available to me


## Network Segments

There will exist 3 network segments providing a different level of access to resources in the lab, like other segments/networks and hosts.

- Management, this network will be used for interfaces which should have access to outbound https traffic, it will use the [Host Only Network](https://docs.vmware.com/en/VMware-Workstation-Player-for-Windows/17.0/com.vmware.player.win.using.doc/GUID-93BDF7F1-D2E4-42CE-80EA-4E305337D2FC.html)
- IPS, this network will be used for interfaces which should have a more restricted and/or proxied access to other networks, it will be a virtual network
- Lab, this network will be used by interfaces which should be isolated from all other networks and the host itself, the reversing and monitoring VMs interfaces will be connected to this virtual network, this will be a segment of the IPS Network

The `IPS` and `Lab` segments are both part of the same virtual Network. 


## The Virtual Machines

### pfSense

- OpenBSD
- [Official Community Edition ISO](https://www.pfsense.org/download/)
- manages network communication and DHCP for the virtual networks
- available to the Host machine through the IPS Management network
- configured over the web interface
- 1 Core, 1024 Mb RAM
- 8 GB Storage
- Network Interfaces: 
	- WAN (bridged) Local Network DHCP
	- LAN (Management Network) Host Only 172.16.1.1/24
	- OPT1 (IPS Network) Virtual Network 172.16.2.1/24

### IPS Machine

- Ubuntu Server 22.04
- running [Snort](https://www.snort.org/) with community rules
- 1 Core, 4GB RAM
- 55 GB Storage
- will be used occasionally for log collection
- Network interfaces:
	- eth0 (Management Network) 172.16.1.4/24
	- eth1,2 used with AFPACKET to bridge communication between IPS and Lab segments (will not have an IP)

### SIEM

- Ubuntu Server 22.04
- running [Elasticsearch](https://github.com/elastic/elasticsearch) and [Kibana](https://www.elastic.co/kibana)
- 2 Core, 8 GB RAM
- 75 GB Storage
- will be used occasionally for log parsing/visualizing
- spoiler: this machine should be assigned a lot more resources, as soon as I can upgrade RAM on the host this machine will be used more
- Network Interfaces:
	- eth0 (Management Network) 172.16.1.2/24

### Remnux

- [Remnux ISO](https://remnux.org/)
- used to provide tools for analysis on a Linux Platform
- will be used to provide local analysis tool for static analysis
- will be used for dynamic analysis with [Inetsim](https://www.inetsim.org/)
- 1 Core 8 GB RAM
- 60 GB Storage
- Network Interfaces
	- eth0 (Lab Network) 172.16.2.2/24

### Reversing Machine

- Windows 10
- used for isolated analysis of samples
- put under complete lock down by the pfSense VM
- will use Remnux as a DNS to pass all traffic to `Inetsim`
- 2 Cores, 8 GB RAM
- 90 GB Storage
- Network Interfaces
	- eth0 (Lab Network) 172.16.2.3/24


## The Goal

My main goal for this project is to get the network and virtual machines setup, set up the firewall configuration to enable restricted communication for the VMs, set up `Inetsim` on the `Remnux` VM and test tracking the network communication of the Reversing Machine.


![Network](images/network_vmware_lab.png "Network Plan for the Lab")


This wraps up the introduction to what I am going to document in the following parts of this blog series.
In the next part I will configure VMWare and the host and set up the pfSense Firewall
