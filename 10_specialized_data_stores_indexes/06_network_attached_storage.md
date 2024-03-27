# Network Attached Storage

**Network Attached Storage (NAS)** is a centralized file storage device for storing data on a network. They provide fast, secure, and reliable storage services to private networks.

## Why use NAS?

NAS file servers are cost effective and scalable. They provide fast data access and are easy to configure and manage. They can support various business applications, including private email systems, accounting databases, payroll, video recording and editing, data logging, and business analytics.

NAS is typically used by small to medium sized business or within individual home networks. Their cost effectiveness and ease of use come at the cost of resiliency - the NAS device itself is a single point of failure.

## How NAS works

Typically, a NAS will be a server with multiple hard drives configured with RAID (described in more detail below), and a network interface card that will be directly attached to a switch or router so that it can access data on the network. Once on the network, data can be accessed through a shared drive via other devices connected to the network.

## RAID

**Redundant Array of Independent Disks** or **RAID** for short, is a technology that combines multiple physical disk drive components into one or more logical units for the purposes of data redundancy. Data is distributed across hard drives in one of several ways, which are referred to as RAID levels, depending on the required level of redundancy and performance. Each distribution layout is denoted by the word "RAID", followed by a number, like RAID 0, RAID 1, etc.

The 4 most common levels of RAID are 0, 1, 5, and 10.

### RAID 0

RAID 0 is technically not RAID since it's not redundant or fault tolerant. In RAID 0, data is striped across multiple disks. This actually increases the likelihood of data loss, since if a single disk fails, all the data is lost. Therefore, the only reason for using RAID 0 is speed.

![raid-0](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/raid0.png?alt=media&token=0708d894-9500-4b8f-9772-303f8aabdaa8)

### RAID 1

RAID 1 is fault tolerant. In this setup, data is copied across two disks, removing the single point of failure. However, less data can be stored since it needs to be duplicated across two disks.

![raid1](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/raid1.png?alt=media&token=cb57d075-2571-4ff4-a463-9628d0f67a37)

### RAID 5

RAID 5 is the most common setup, as it provides fault tolerance along with speed. Instead of duplicating data across multiple disks, data is striped across 3 or more disks alongside a piece of metadata known as _parity_. Parity data is used to reconstruct data in the event of a disk failure. This of course means that less data can be stored across the entire array, since an entire disks' worth of space will be dedicated towards storing parity.

![raid5](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/raid5.png?alt=media&token=d56f53cf-fadb-4efb-9020-dad054580ab8)

### RAID 10

RAID 10 is a combination of RAID 0 and RAID 1 and requires a minimum of 4 disks. In this configuration, data is striped _and_ duplicated across disks. Thus, RAID 10 benefits from the fault tolerance of RAID 1 and the speed of RAID 0. However, you will need to reserve 50% of your total available storage capacity for duplication.

![raid10](https://firebasestorage.googleapis.com/v0/b/system-design-daily.appspot.com/o/raid10.png?alt=media&token=438745c4-3fb4-49bb-9648-b11f6b006241)

# Storage Area Network

A **Storage Area Network**, or **SAN**, is a special high speed network that stores and provides access to large amounts of data. It consists of multiple disk arrays, switches and servers. Servers and other devices connected to the network recognize SANs as a shared local hard drive.

Since it consists of multiple storage devices with data shared among several disk arrays, a SAN is more fault tolerant than a NAS. It is also more scalable, as more devices can be easily and continually added to the network to provide more storage. All devices on the SAN are interconnected using fiber optic cables (known as Fiber Channel), which provide high speed communication (up to 128 Gbps) across the network. Fiber Channel is fairly expensive, so some SANs may opt for Internet Small Computer System Interface (iSCSI) instead, which is a cheaper alternative for providing network connectivity. Additionally, a SAN is resistant to bottlenecks in a local area network since it is a self contained network that is unaffected by external traffic.

SANS are very expensive and are primarily utilized by large companies who require fault tolerance, high speed, and scalability
