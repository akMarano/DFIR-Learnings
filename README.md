# TShark Command-Line Packet Analysis: Detailed Guide

TShark is the command-line version of Wireshark. It captures and analyses network packets, and it prints plain text, so its output can be piped into other tools (`grep`, `awk`, `sort`, `uniq`). That makes it well suited to automation, scripting, and carving specific data out of large captures.

**How each command is explained in this guide:**

- **Command**: the exact command to type
- **Flags**: what every part of the command means
- **What it does**: what actually happens when you run it
- **Purpose**: when and why an analyst would use it
- **Reading the output**: how to interpret the sample result

All examples use the sample capture `demo.pcapng`. Output is abbreviated where the original was.

## Table of Contents

1. [Helper tools used with TShark](#1-helper-tools-used-with-tshark)
2. [Basic parameters](#2-basic-parameters)
3. [Sniffing live traffic](#3-sniffing-live-traffic)
4. [Reading, limiting and writing capture files](#4-reading-limiting-and-writing-capture-files)
5. [Packet bytes and verbosity](#5-packet-bytes-and-verbosity)
6. [Capture conditions: autostop and ring buffer](#6-capture-conditions-autostop-and-ring-buffer)
7. [Capture filters vs display filters](#7-capture-filters-vs-display-filters)
8. [Statistics with `-z`](#8-statistics-with--z)
9. [Protocol-specific statistics](#9-protocol-specific-statistics)
10. [Advanced filtering: `contains`, `matches`, field extraction](#10-advanced-filtering-contains-matches-and-field-extraction)
11. [Analyst use cases](#11-analyst-use-cases)
12. [Quick reference and workflow](#12-quick-reference-and-workflow)

---

## 1. Helper tools used with TShark

TShark's strength is that its text output can be passed to other Linux tools. These are the ones used most often in packet analysis.

| Tool | What it does | Why it's useful with TShark |
|------|--------------|-----------------------------|
| `capinfos` | Prints metadata about a capture file | Gives you a summary of the file before you start investigating |
| `grep` | Searches text for a pattern | Quickly find a keyword (an IP, a domain) in TShark output |
| `cut` | Extracts chosen columns/parts of each line | Keep only the part of a line you need |
| `uniq` | Collapses repeated lines | Turns thousands of repeated values into a unique list (input must be sorted first) |
| `nl` | Numbers each line of output | Lets you count how many packets a filter matched |
| `sed` | Stream editor | Find-and-replace or reformat output |
| `awk` | Pattern-scanning and processing language | Filter or reshape output; for example, `awk NF` removes blank lines |

### `capinfos`: summarise a capture file

**Command**

```bash
capinfos demo.pcapng
```

**Flags**

- `demo.pcapng`: the capture file to describe. No flags are used; `capinfos` simply reads the file and reports on it.

**What it does**

It reads the capture file's headers and totals, without displaying any individual packets, and prints a summary report.

**Purpose**

Run this first on any capture. It tells you how big the capture is, how long it covers, and whether it is worth analysing in the way you planned. The hashes let you prove the file was not altered later, which matters in investigations.

**Output**

```
File name:           demo.pcapng
File type:           Wireshark/tcpdump/... - pcap
File encapsulation:  Ethernet
File timestamp precision:  microseconds (6)
Packet size limit:   file hdr: 65535 bytes
Number of packets:   4...
File size:           25 kB
Data size:           25 kB
Capture duration:    30.393704 seconds
First packet time:   2004-05-13 10:17:07.311224
Last packet time:    2004-05-13 10:17:37.704928
Data byte rate:      825 bytes/s
Data bit rate:       6604 bits/s
Average packet size: 583.51 bytes
Average packet rate: 1 packets/s
SHA256:              25a72bdf10339...
RIPEMD160:           6ef5f0c165a1d...
SHA1:                3aac91181c3b7...
Strict time order:   True
Number of interfaces in file: 1
Interface #0 info:
Encapsulation = Ethernet (1 - ether)
Capture length = 65535
Time precision = microseconds (6)
Time ticks per second = 1000000
Number of stat entries = 0
Number of packets = 4...
```

**Reading the output**

| Field | Meaning |
|-------|---------|
| `File encapsulation` | The link-layer type (here Ethernet), which tells you which protocols to expect |
| `Packet size limit` | The snap length: packets larger than this were truncated when captured |
| `Number of packets` | Total packets in the file |
| `Capture duration` / `First` / `Last packet time` | The time window the capture covers; useful for matching to an incident timeline |
| `Data byte rate`, `Average packet rate`, `Average packet size` | Quick sense of how busy or bulky the traffic was |
| `SHA256`, `SHA1`, `RIPEMD160` | Hashes of the file, used to verify integrity |
| `Strict time order` | `True` means packets are in chronological order |

---

## 2. Basic parameters

These are the starting-point flags: getting help, checking the version, and choosing where to capture.

| Flag | What it does | Purpose | Example |
|------|--------------|---------|---------|
| `-h` | Prints the help page with the most common options | Quick reminder of syntax without leaving the terminal | `tshark -h` |
| `-v` | Prints the TShark version | Confirm what version you are running (features and field names vary between versions) | `tshark -v` |
| `-D` | Lists the interfaces TShark can capture from | Find out which interface to sniff on | `tshark -D` |
| `-i` | Selects the interface to capture on, by number or name | Choose exactly which network card or pseudo-interface to monitor | `tshark -i 1` / `tshark -i ens55` |
| *(none)* | Starts capturing on the default interface and prints a line per packet, like `tcpdump` | Fast, no-setup sniffing | `tshark` |

> Capturing live traffic and listing interfaces requires superuser privileges (`sudo`). Reading an existing file does not.

### Version info

**Command**

```bash
tshark -v
```

**What it does / Purpose**

Prints the TShark build and licence information. Useful when a filter or field name from documentation doesn't work, because it may belong to a different version.

**Output**

```
TShark (Wireshark) 3 (Git v3. packaged as 3.)

Copyright 1998-2020 Gerald Combs and contributors. License GPLv2+: GNU GPL version 2 or later.
This is free software; see the source for copying conditions.
..
```

---

## 3. Sniffing live traffic

A computer can have several network interfaces, and each may be used for a different job. Choosing the correct interface decides which traffic you will see.

### List interfaces

**Command**

```bash
sudo tshark -D
```

**Flags**

- `sudo`: run as superuser, which TShark needs to access network interfaces.
- `-D`: **D**isplay the list of available capture interfaces.

**What it does**

Prints each interface with a number. That number is what you pass to `-i`.

**Purpose**

You must know which interface carries the traffic you care about before you can capture it.

**Output**

```
1. ens5
2. lo (Loopback)
3. any
4. bluetooth-monitor
5. nflog
..
```

**Reading the output**

| Interface | Meaning |
|-----------|---------|
| `ens5` | The main Ethernet network card; normal network traffic |
| `lo (Loopback)` | Internal traffic the machine sends to itself (`127.0.0.1`) |
| `any` | A pseudo-interface that captures on all interfaces at once |
| `bluetooth-monitor` | Bluetooth traffic |
| `nflog` | Packets logged by the Linux firewall (netfilter) |

### Sniff on the default interface

**Command**

```bash
tshark
```

**Flags**

- None. With no `-i`, TShark uses the first interface in the list. This is the same as running `tshark -i 1`.

**What it does**

Starts a live capture and prints one summary line per packet until you stop it (Ctrl+C) or a limit is reached. TShark prints `Capturing on '<interface>'` at the start so you know which interface it chose.

**Purpose**

The quickest way to see what is happening on the network right now.

**Output**

```
Capturing on 'ens5'
1   0.000000 aaa.aaa.aaa.aaa → bbb.bbb.bbb.bbb TCP 3372 → 80 [SYN] Seq=0 Win=8760 Len=0 MSS=1460 SACK_PERM=1
2   0.911310 aaa.aaa.aaa.aaa → bbb.bbb.bbb.bbb TCP 80 → 3372 [SYN, ACK] Seq=0 Ack=1 Win=5840 Len=0 MSS=1380 SACK_PERM=1
3   0.911310 aaa.aaa.aaa.aaa → bbb.bbb.bbb.bbb TCP 3372 → 80 [ACK] Seq=1 Ack=1 Win=9660 Len=0
...
100 packets captured
```

**Reading the output**

Each line is: packet number, seconds since capture start, source → destination, protocol, then protocol details. These three lines are a **TCP three-way handshake**:

1. `[SYN]`: the client (port 3372) asks to open a connection to port 80
2. `[SYN, ACK]`: the server agrees
3. `[ACK]`: the client confirms, and the connection is established

### Sniff on a chosen interface

**Command**

```bash
tshark -i 2
```

**Flags**

- `-i 2`: capture on interface number 2 from the `-D` list (the loopback `lo`). You can also use the name, for example `-i ens5`.

**What it does**

Same as above, but on the interface you picked.

**Purpose**

Monitor a specific interface, such as watching loopback traffic when debugging a service running on the same machine.

**Output**

```
Capturing on 'Loopback: lo'
...
```

---

## 4. Reading, limiting and writing capture files

| Flag | Stands for | What it does |
|------|-----------|--------------|
| `-r <file>` | **R**ead | Reads packets from a saved capture file instead of the network |
| `-c <n>` | **C**ount | Stops after *n* packets |
| `-w <file>` | **W**rite | Saves packets to a file instead of printing them |

### Read a capture file

**Command**

```bash
tshark -r demo.pcapng
```

**Flags**

- `-r demo.pcapng`: read packets from this file (offline analysis).

**What it does**

Replays the whole capture and prints one summary line per packet, exactly like live sniffing, but from a file.

**Purpose**

Analyse traffic captured earlier (from Wireshark, tcpdump or TShark). This is the foundation for nearly every analysis command in this guide. No `sudo` is needed, because nothing touches the network.

**Output**

```
1   0.000000 145.254.160.237 → 65.208.228.223 TCP 3372 → 80 [SYN] Seq=0 Win=8760 Len=0 MSS=1460 SACK_PERM=1
2   0.911310 65.208.228.223 → 145.254.160.237 TCP 80 → 3372 [SYN, ACK] Seq=0 Ack=1 Win=5840 Len=0 MSS=1380 SACK_PERM=1
3   0.911310 145.254.160.237 → 65.208.228.223 TCP 3372 → 80 [ACK] Seq=1 Ack=1 Win=9660 Len=0
..
```

### Limit the number of packets

**Command**

```bash
tshark -r demo.pcapng -c 2
```

**Flags**

- `-r demo.pcapng`: read from the file.
- `-c 2`: stop after 2 packets.

**What it does**

Reads only the first 2 packets and then stops. (On a live capture, `-c` ends the capture after that many packets.)

**Purpose**

Preview a large file without flooding the screen, or grab just the first few packets (for example, the start of a connection).

**Output**

```
1   0.000000 145.254.160.237 → 65.208.228.223 TCP 3372 → 80 [SYN] Seq=0 Win=8760 Len=0 MSS=1460 SACK_PERM=1
2   0.911310 65.208.228.223 → 145.254.160.237 TCP 80 → 3372 [SYN, ACK] Seq=0 Ack=1 Win=5840 Len=0 MSS=1380 SACK_PERM=1
```

### Write packets to a new file

**Command**

```bash
tshark -r demo.pcapng -c 1 -w write-demo.pcap
```

**Flags**

- `-r demo.pcapng`: read from the original capture.
- `-c 1`: process only 1 packet.
- `-w write-demo.pcap`: save the result to a new file.

**What it does**

Copies the selected packet(s) from the source file into a new, separate capture file. The original is untouched. The packets are written to the file instead of being printed.

**Purpose**

"Carving": isolating just the packets that matter (by count, or later by filter) so you can analyse them separately or hand a small, relevant file to a senior investigator instead of the whole capture.

**Verify it worked**

```bash
ls
```

```
demo.pcapng  write-demo.pcap
```

```bash
tshark -r write-demo.pcap
```

```
1   0.000000 145.254.160.237 → 65.208.228.223 TCP 3372 → 80 [SYN] Seq=0 Win=8760 Len=0 MSS=1460 SACK_PERM=1
```

The new file contains exactly one packet: the first SYN of the original capture.

---

## 5. Packet bytes and verbosity

By default TShark shows one summary line per packet. These flags show more.

| Flag | Stands for | What it does | Purpose |
|------|-----------|--------------|---------|
| `-x` | he**x** dump | Shows each packet's raw bytes in hex with an ASCII view | Inspect exact byte content: payloads, hidden strings, malformed data |
| `-V` | **V**erbose | Shows every decoded protocol layer and field for each packet (like Wireshark's Packet Details pane) | Understand exactly what each header field contains |
| `-q` | **Q**uiet | Suppresses the per-packet lines | Show only statistics or other requested output |

> Both `-x` and `-V` produce a lot of output per packet. Use them after reducing the capture to a few packets (with `-c` or a filter), not on a whole file.

### Hex and ASCII dump

**Command**

```bash
tshark -r write-demo.pcap -x
```

**Flags**

- `-r write-demo.pcap`: read the one-packet file created earlier.
- `-x`: print the raw packet bytes.

**What it does**

Prints the packet as a classic hex dump instead of the one-line summary.

**Purpose**

When the decoded view isn't enough, for example to look at raw payload bytes for text, signatures, or odd data.

**Output**

```
0000  fe ff 20 00 01 00 00 00 01 00 00 00 08 00 45 00   .. ...........E.
0010  00 30 0f 41 40 00 80 06 91 eb 91 fe a0 ed 41 d0   .0.A@.........A.
0020  e4 df 0d 2c 00 50 38 af fe 13 00 00 00 00 70 02   ...,.P8.......p.
0030  22 38 c3 0c 00 00 02 04 05 b4 01 01 04 02         "8............
```

**Reading the output**

Each row has three parts: the **offset** (position in the packet, left), the **hex bytes** (middle), and the **ASCII** rendering (right; unprintable bytes show as `.`). The `45 00` near the end of row one marks the start of the IPv4 header (version 4, header length 5 words).

### Verbose, full protocol details

**Command**

```bash
tshark -r demo.pcapng -c 1 -V
```

**Flags**

- `-r demo.pcapng`: read from the file.
- `-c 1`: only the first packet (important, because `-V` is very long).
- `-V`: show full details of every protocol layer.

**What it does**

Decodes the packet layer by layer (Frame → Ethernet → IP → TCP) and prints each field.

**Purpose**

Deep inspection of a specific packet, and discovering the exact **field names** you can later use in filters (`-Y`) and extractions (`-e`). Because every field is exposed, it also underpins scripting and correlation.

**Compare with the default view**

```bash
tshark -r demo.pcapng -c 1
```

```
1   0.000000 145.254.160.237 → 65.208.228.223 TCP 3372 → 80 [SYN] Seq=0 Win=8760 Len=0 MSS=1460 SACK_PERM=1
```

**Verbose output**

```
Frame 1: 62 bytes on wire (496 bits), 62 bytes captured (496 bits)
...
Ethernet II, Src: 00:00:01:00:00:00, Dst: fe:ff:20:00:01:00
...
Internet Protocol Version 4, Src: 145.254.160.237, Dst: 65.208.228.223
0100 .... = Version: 4
.... 0101 = Header Length: 20 bytes (5)
Total Length: 48
Identification: 0x0f41 (3905)
Flags: 0x4000, Don't fragment
Fragment offset: 0
Time to live: 128
Protocol: TCP (6)
Source: 145.254.160.237
Destination: 65.208.228.223
Transmission Control Protocol, Src Port: 3372, Dst Port: 80, Seq: 0, Len: 0
...
```

**Reading the output**

| Layer | What it shows |
|-------|---------------|
| `Frame` | Size of the packet as seen on the wire |
| `Ethernet II` | Source and destination **MAC addresses** (link layer) |
| `Internet Protocol Version 4` | Source and destination **IPs**, TTL (128 suggests a Windows host), `Don't fragment` flag, and the next protocol (`TCP (6)`) |
| `Transmission Control Protocol` | Source port 3372 → destination port 80, sequence numbers |

---

## 6. Capture conditions: autostop and ring buffer

Live captures can grow without limit. These options control when a capture stops and how files are split.

> These work **only in live capture mode**. If you combine them with `-r` you get an error. To pull packets out of an existing file, use read + write (`-r` and `-w`, optionally with a `-Y` filter) instead.

### `-a`: autostop (a single run that stops when a condition is met)

| Condition | Meaning | Example |
|-----------|---------|---------|
| `duration:X` | Stop after X seconds | `tshark -w test.pcap -a duration:1` |
| `filesize:X` | Stop when the file reaches X kilobytes | `tshark -w test.pcap -a filesize:10` |
| `files:X` | Stop after X output files have been created | `tshark -w test.pcap -a filesize:10 -a files:3` |

### `-b`: ring buffer (multiple runs; creates new files and loops)

| Condition | Meaning | Example |
|-----------|---------|---------|
| `duration:X` | After X seconds, close the file and start a new one | `tshark -w test.pcap -b duration:1` |
| `filesize:X` | After the file reaches X KB, start a new one | `tshark -w test.pcap -b filesize:10` |
| `files:X` | Keep at most X files; once reached, overwrite the oldest | `tshark -w test.pcap -b filesize:10 -b files:3` |

**Autostop vs ring buffer:** `-a` ends the capture; `-b` keeps capturing indefinitely, rotating files. A ring buffer is how you monitor continuously without filling the disk: only the most recent N files are kept.

> **Warning:** `-b` runs in an infinite loop. Use at least one `-a` condition to stop it, or stop it manually with Ctrl+C. The two can be combined.

### Example: bounded capture split across files

**Command**

```bash
tshark -w autostop-demo.pcap -a duration:2 -a filesize:5 -a files:5
```

**Flags**

- `-w autostop-demo.pcap`: save to files with this base name (TShark adds a number and timestamp to each file).
- `-a duration:2`: stop after 2 seconds.
- `-a filesize:5`: each file is limited to 5 KB.
- `-a files:5`: stop after 5 files.

**What it does**

Captures live traffic into files of up to 5 KB each, and stops as soon as **any** one condition is met (2 seconds elapsed, or 5 files created).

**Purpose**

Take short, size-controlled samples of traffic, for example small files that are easy to open, share or store.

**Output**

```
Capturing on 'ens5'
13
```

The number is the count of packets captured. Listing the folder shows the generated files:

```bash
ls
```

```
-rw------- 1 ubuntu ubuntu   autostop-demo_..1_2022.pcap
-rw------- 1 ubuntu ubuntu   autostop-demo_..2_2022.pcap
-rw------- 1 ubuntu ubuntu   autostop-demo_..3_2022.pcap
-rw------- 1 ubuntu ubuntu   autostop-demo_..4_2022.pcap
-rw------- 1 ubuntu ubuntu   autostop-demo_..5_2022.pcap
```

---

## 7. Capture filters vs display filters

TShark filters packets in two different ways at two different times.

| | **Capture filter** (`-f`) | **Display filter** (`-Y`) |
|---|---|---|
| **When it applies** | During the live capture, before packets are saved | After capture, while reading/analysing |
| **Syntax** | BPF (Berkeley Packet Filter) / Wireshark capture-filter syntax | Wireshark display-filter syntax |
| **Effect** | Packets that don't match are never recorded | Packets that don't match are only hidden; the file is not modified |
| **Changeable later?** | No; set before the capture starts | Yes; try as many filters as you like |
| **Power** | Limited: scope by host, network, port, protocol, direction | Very detailed: any decoded protocol field |
| **Purpose** | Keep capture files small and focused | Investigate in depth |

Use a capture filter when you already know what you want to record, and a display filter when you're exploring. Both support Boolean operators (`and`, `or`, `not`).

| Flag | Purpose |
|------|---------|
| `-f "<filter>"` | Apply a capture filter |
| `-Y '<filter>'` | Apply a display filter |

### 7.1 Capture filters (`-f`)

A capture filter is built from three kinds of qualifier.

**Type**: *what* to match

| Type | Matches | Example |
|------|---------|---------|
| `host` | A single IP or hostname | `tshark -f "host 10.10.10.10"` |
| `net` | A network range (CIDR) | `tshark -f "net 10.10.10.0/24"` |
| `port` | A single port | `tshark -f "port 80"` |
| `portrange` | A range of ports | `tshark -f "portrange 80-100"` |

If you give no type, `host` is assumed.

**Direction**: *which way* the traffic flows

| Direction | Matches | Example |
|-----------|---------|---------|
| `src` | Traffic **from** the address | `tshark -f "src host 10.10.10.10"` |
| `dst` | Traffic **to** the address | `tshark -f "dst host 10.10.10.10"` |

If you give no direction, both directions are matched.

**Protocol**: *which protocol*

| Protocol | Example |
|----------|---------|
| `arp`, `ether`, `icmp`, `ip`, `ip6`, `tcp`, `udp` | `tshark -f "tcp"` |
| MAC address (`ether host`) | `tshark -f "ether host F8:DB:C5:A2:5D:81"` |
| IANA protocol number (`ip proto`) | `tshark -f "ip proto 1"` (1 = ICMP) |

#### Demo: capture only one host's traffic

This uses `terminator` (a split-screen terminal). Terminal-1 sniffs; Terminal-2 generates traffic so there is something to see.

**Terminal-1: the sniffer**

```bash
tshark -f "host 10.10.10.10"
```

- `-f "host 10.10.10.10"`: capture only packets to or from 10.10.10.10. Everything else on the network is ignored.

**Terminal-2: the traffic generator**

```bash
curl -v 10.10.10.10
```

- `curl`: sends an HTTP request.
- `-v`: verbose, so you can see the request and response.

**Terminal-1 output**

```
Capturing on 'ens5'
1 0.000000000 YOUR-IP → 10.10.10.10  TCP 74 36150 → 80 [SYN] Seq=0 Win=62727 Len=0 MSS=8961 SACK_PERM=1 TSval=2045205701 TSecr=0 WS=128
2 0.003452830  10.10.10.10 → YOUR-IP TCP 74 80 → 36150 [SYN, ACK] Seq=0 Ack=1 Win=62643 Len=0 MSS=8961 SACK_PERM=1 TSval=744450747 TSecr=2045205701 WS=64
3 0.003487830 YOUR-IP → 10.10.10.10  TCP 66 36150 → 80 [ACK] Seq=1 Ack=1 Win=62848 Len=0 TSval=2045205704 TSecr=744450747
4 0.003610800 YOUR-IP → 10.10.10.10  HTTP 141 GET / HTTP/1.1
```

**Terminal-2 output**

```
*   Trying 10.10.10.10:80...
* TCP_NODELAY set
* Connected to 10.10.10.10 (10.10.10.10) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.10.10
> User-Agent: curl/7.68.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Content-Length: 1220
< Content-Type: text/html; charset=utf-8
```

**Reading the output**

Terminal-1 shows the handshake (packets 1-3) followed by the `GET /` request (packet 4). Terminal-2 shows the same exchange from `curl`'s side: the request it sent and the `200 OK` response it received. Only traffic involving 10.10.10.10 appears, which is the capture filter working.

#### Practice: capture filters with matching traffic generators

| Category | Generate traffic with | Capture filter | What it proves |
|----------|----------------------|----------------|----------------|
| Host (by name) | `curl tryhackme.com` | `tshark -f "host tryhackme.com"` | Filtering by hostname |
| IP address | `nc 10.10.10.10 4444 -vw 5` | `tshark -f "host 10.10.10.10"` | Filtering by IP |
| Port | `nc 10.10.10.10 4444 -vw 5` | `tshark -f "port 4444"` | Filtering by port |
| Protocol | `nc -u 10.10.10.10 4444 -vw 5` | `tshark -f "udp"` | Filtering by protocol |

Netcat flags: `-v` = verbose, `-w 5` = time out after 5 seconds, `-u` = use UDP instead of TCP.

### 7.2 Display filters (`-Y`)

Display filters work on **decoded fields**, so they are far more precise. Wrap them in **single quotes** to avoid problems with spaces and bash expansion. (The Wireshark Display Filter Reference lists the available fields.)

| Category | What it does | Command |
|----------|--------------|---------|
| **IP** | Matches an address as source **or** destination | `tshark -Y 'ip.addr == 10.10.10.10'` |
| | Matches a whole network range | `tshark -Y 'ip.addr == 10.10.10.0/24'` |
| | Matches only traffic **from** the address | `tshark -Y 'ip.src == 10.10.10.10'` |
| | Matches only traffic **to** the address | `tshark -Y 'ip.dst == 10.10.10.10'` |
| **TCP** | TCP port, either side | `tshark -Y 'tcp.port == 80'` |
| | TCP source port only | `tshark -Y 'tcp.srcport == 80'` |
| **HTTP** | All HTTP packets | `tshark -Y 'http'` |
| | Only HTTP responses with status 200 (OK) | `tshark -Y "http.response.code == 200"` |
| **DNS** | All DNS packets | `tshark -Y 'dns'` |
| | Only `A` record queries (name → IPv4 address) | `tshark -Y 'dns.qry.type == 1'` |

#### Example: filter by IP address

**Command**

```bash
tshark -r demo.pcapng -Y 'ip.addr == 145.253.2.203'
```

**Flags**

- `-r demo.pcapng`: read the file.
- `-Y 'ip.addr == 145.253.2.203'`: show only packets where this IP is the source or the destination.

**What it does / Purpose**

Hides everything except traffic involving one host, so you can follow what a single suspicious or interesting machine did.

**Output**

```
13 2.55 145.254.160.237 → 145.253.2.203 DNS Standard query 0x0023 A ..
17 2.91 145.253.2.203 → 145.254.160.237 DNS Standard query response 0x0023 A ..
```

**Reading the output**

Two packets matched: a DNS query going to 145.253.2.203 (a DNS server) and its response coming back.

#### Example: filter by protocol

**Command**

```bash
tshark -r demo.pcapng -Y 'http'
```

**Output**

```
4   0.911 145.254.160.237 → 65.208.228.223 HTTP GET /download.html HTTP/1.1
18   2.984 145.254.160.237 → 216.239.59.99 HTTP GET /pagead/ads?client...
27   3.955 216.239.59.99 → 145.254.160.237 HTTP HTTP/1.1 200 OK (text/html)
38   4.846 65.208.228.223 → 145.254.160.237 HTTP/XML HTTP/1.1 200 OK
```

**Reading the output**

The client requested a page (`GET /download.html`) and an ad (`/pagead/ads`), and each server answered `200 OK`.

#### Counting matches with `nl`

TShark does **not** renumber filtered packets. The first column is the packet's **original position in the capture** (4, 18, 27, 38 above, not 1-4). So you can't tell how many packets matched just by looking at that column.

**Command**

```bash
tshark -r demo.pcapng -Y 'http' | nl
```

**Flags**

- `|`: pipe TShark's output into the next command.
- `nl`: **n**umber **l**ines. It adds its own sequential count in front of every line.

**Output**

```
1    4  0.911 145.254.160.237 → 65.208.228.223 HTTP GET /download.html HTTP/1.1
2   18  2.984 145.254.160.237 → 216.239.59.99 HTTP GET /pagead/ads?client...
3   27   3.955 216.239.59.99 → 145.254.160.237 HTTP HTTP/1.1 200 OK (text/html)
4   38   4.846 65.208.228.223 → 145.254.160.237 HTTP/XML HTTP/1.1 200 OK
```

**Reading the output**

The leftmost column (1-4) is `nl`'s count, so **4 packets** matched. The next column (4, 18, 27, 38) is the original packet number in the file. This distinction matters when your filter returns more than a handful of packets.

---

## 8. Statistics with `-z`

TShark can reproduce many of Wireshark's statistics windows. General rules:

- Statistics apply to **all packets in scope** unless you add a display filter.
- TShark prints the name of the statistic (for example, "Protocol Hierarchy Statistics") at the top of the result.
- Packets are listed **first**, then the statistics. Add `-q` to hide the packets and see only the statistics.
- `tshark -z help` lists every available statistic.

| Flag | What it does |
|------|--------------|
| `-z <stat>` | Produces a statistics report |
| `-q` | Quiet: suppresses the packet list so only the report is shown |
| `--color` | Colourised, Wireshark-style output; helps you spot anomalies faster. Example: `tshark -r colour.pcap --color` |

### 8.1 Protocol hierarchy

**Command**

```bash
tshark -r demo.pcapng -z io,phs -q
```

**Flags**

- `-z io,phs`: the **I/O → protocol hierarchy statistics** report.
- `-q`: hide the packets, show only the report.

**What it does**

Builds a tree showing every protocol found in the capture, nested by layer, with the number of frames and bytes for each.

**Purpose**

The best first look at an unknown capture: it tells you what kinds of traffic are present and where to focus (for example, unexpected protocols).

**Output**

```
===================================================================
Protocol Hierarchy Statistics
Filter:

eth                                    frames:43 bytes:25091
  ip                                   frames:43 bytes:25091
    tcp                                frames:41 bytes:24814
      http                             frames:4 bytes:2000
        data-text-lines                frames:1 bytes:214
          tcp.segments                 frames:1 bytes:214
        xml                            frames:1 bytes:478
          tcp.segments                 frames:1 bytes:478
    udp                                frames:2 bytes:277
      dns                              frames:2 bytes:277
===================================================================
```

**Reading the output**

Indentation shows protocol layering: all 43 frames are Ethernet carrying IP; 41 of those are TCP (4 of them HTTP) and 2 are UDP (both DNS).

**Focus on one protocol:** add the protocol name to the statistic.

```bash
tshark -r demo.pcapng -z io,phs,udp -q
```

- `io,phs,udp`: the same report, filtered to UDP traffic only (note `Filter: udp` in the output).

```
===================================================================
Protocol Hierarchy Statistics
Filter: udp

eth                                    frames:2 bytes:277
  ip                                   frames:2 bytes:277
    udp                                frames:2 bytes:277
      dns                              frames:2 bytes:277
===================================================================
```

### 8.2 Packet lengths

**Command**

```bash
tshark -r demo.pcapng -z plen,tree -q
```

**Flags**

- `-z plen,tree`: the **packet lengths** report, displayed as a tree.
- `-q`: hide the packets.

**What it does**

Groups all packets into size ranges and reports the count, average, min, max and percentage for each range.

**Purpose**

Spot unusual packet sizes at a glance. Unexpected very large packets can indicate data transfer or exfiltration; floods of tiny packets can indicate scanning or control traffic.

**Output**

```
=========================================================================================================================
Packet Lengths:
Topic / Item       Count     Average       Min val       Max val       Rate (ms)     Percent       Burst rate    Burst start
-------------------------------------------------------------------------------------------------------------------------
Packet Lengths     43        583.51        54            1484        0.0014        100         0.0400        2.554
 0-19              0         -             -             -           0.0000        0.00        -             -
 20-39             0         -             -             -           0.0000        0.00        -             -
 40-79             22        54.73         54            62          0.0007        51.16       0.0200        0.911
 80-159            1         89.00         89            89          0.0000        2.33        0.0100        2.554
 160-319           2         201.00        188           214         0.0001        4.65        0.0100        2.914
 320-639           2         505.50        478           533         0.0001        4.65        0.0100        0.911
 640-1279          1         775.00        775           775         0.0000        2.33        0.0100        2.984
 1280-2559         15        1440.67       1434          1484        0.0005        34.88       0.0200        2.554
 2560-5119         0         -             -             -           0.0000        0.00        -             -
 5120 and greater  0         -             -             -           0.0000        0.00        -             -
-------------------------------------------------------------------------------------------------------------------------
```

**Reading the output**

Each row is a size bucket. Here 51% of packets are 40-79 bytes (handshakes and acknowledgements with no data), and 35% (15 packets) are 1280-2559 bytes (full-size data packets carrying the actual page content). Nothing is unusually large, so there is no sign of oversized transfers.

### 8.3 Endpoints

**Command**

```bash
tshark -r demo.pcapng -z endpoints,ip -q
```

**Flags**

- `-z endpoints,ip`: the **endpoints** report, using **IPv4** addresses.
- `-q`: hide the packets.

The second word selects the address type:

| Type | Lists |
|------|-------|
| `eth` | Ethernet (MAC) addresses |
| `ip` | IPv4 addresses |
| `ipv6` | IPv6 addresses |
| `tcp` | TCP addresses (works for IPv4 and IPv6) |
| `udp` | UDP addresses (works for IPv4 and IPv6) |
| `wlan` | IEEE 802.11 (Wi-Fi) addresses |

**What it does**

Lists every unique address that appears in the capture, with how many packets and bytes it sent (Tx) and received (Rx).

**Purpose**

Find out *who* is on the network in this capture and which hosts are the busiest or the most unusual.

**Output**

```
================================================================================
IPv4 Endpoints
Filter:
                       |  Packets  | |  Bytes  | | Tx Packets | | Tx Bytes | | Rx Packets | | Rx Bytes |
145.254.160.237               43         25091         20            2323          23           22768
65.208.228.223                34         20695         18           19344          16            1351
216.239.59.99                  7          4119          4            3236           3             883
145.253.2.203                  2           277          1             188           1              89
================================================================================
```

**Reading the output**

145.254.160.237 appears in all 43 packets, so it is the local client. 65.208.228.223 transmitted 19,344 bytes but received only 1,351, so it behaves like a server delivering content to the client.

### 8.4 Conversations

**Command**

```bash
tshark -r demo.pcapng -z conv,ip -q
```

**Flags**

- `-z conv,ip`: the **conversations** report for IPv4. It takes the same type options as `endpoints`.
- `-q`: hide the packets.

**What it does**

Lists every pair of hosts that exchanged traffic, with totals in each direction, when the conversation began, and how long it lasted.

**Purpose**

Endpoints tell you *who*; conversations tell you *who talked to whom*, how much, and when. This is the basis for spotting a host that talks to something it shouldn't, or a long-lived connection.

**Output**

```
================================================================================
IPv4 Conversations
Filter:
                                               |       <-      | |       ->      | |     Total     |    Relative    |   Duration
                                               | Frames  Bytes | | Frames  Bytes | | Frames  Bytes |      Start     |
65.208.228.223   <-> 145.254.160.237           16      1351      18     19344      34     20695     0.000000000        30.3937
145.254.160.237  <-> 216.239.59.99              4      3236       3       883       7      4119     2.984291000         1.7926
145.253.2.203    <-> 145.254.160.237            1        89       1       188       2       277     2.553672000         0.3605
================================================================================
```

**Reading the output**

| Column | Meaning |
|--------|---------|
| `<-` | Traffic from the **second** address to the **first** |
| `->` | Traffic from the **first** address to the **second** |
| `Total` | Both directions combined |
| `Relative Start` | Seconds after the capture began that the conversation started |
| `Duration` | How long the conversation lasted |

The first row is the main exchange: 34 frames over about 30 seconds, mostly flowing from 65.208.228.223 (19,344 bytes). The other two are short (under 2 seconds) exchanges with an ad server and a DNS server.

### 8.5 Expert info

**Command**

```bash
tshark -r demo.pcapng -z expert -q
```

**Flags**

- `-z expert`: the **expert information** report.
- `-q`: hide the packets.

**What it does**

Collects the automatic observations Wireshark's analysis engine makes about the traffic, grouped by severity (Errors, Warnings, Notes, Chats).

**Purpose**

A fast way to find network problems and notable events without reading every packet: retransmissions, duplicate ACKs, connection setup and teardown, and requests.

**Output**

```
Notes (3)
=============
   Frequency      Group           Protocol  Summary
           1   Sequence                TCP  This frame is a (suspected) spurious retransmission
           1   Sequence                TCP  This frame is a (suspected) retransmission
           1   Sequence                TCP  Duplicate ACK (#1)

Chats (8)
=============
   Frequency      Group           Protocol  Summary
           1   Sequence                TCP  Connection establish request (SYN): server port 80
           1   Sequence                TCP  Connection establish acknowledge (SYN+ACK): server port 80
           1   Sequence               HTTP  GET /download.html HTTP/1.1\r\n
           1   Sequence               HTTP  GET /pagead/ads?client=ca-pub-2309191948673629&random=1084443430285&lmt=1082467020
           2   Sequence               HTTP  HTTP/1.1 200 OK\r\n
           2   Sequence                TCP  Connection finish (FIN)
```

**Reading the output**

- **Notes** are unusual-but-minor events. Here they are retransmissions and a duplicate ACK, which can indicate small amounts of packet loss or delay.
- **Chats** are ordinary workflow events: a connection opening, HTTP requests and responses, and a connection closing.
- `Frequency` is how many times each event occurred.

---

## 9. Protocol-specific statistics

All of these follow the same pattern: `tshark -r <file> -z <stat> -q`. They summarise one protocol or address family so you can see distribution without reading packets.

| Goal | Statistic |
|------|-----------|
| IP protocol types (TCP/UDP/ICMP split) | `-z ptype,tree` |
| All IPv4 hosts | `-z ip_hosts,tree` |
| All IPv6 hosts | `-z ipv6_hosts,tree` |
| IPv4 sources and destinations | `-z ip_srcdst,tree` |
| IPv6 sources and destinations | `-z ipv6_srcdst,tree` |
| IPv4 destinations and ports | `-z dests,tree` |
| IPv6 destinations and ports | `-z ipv6_dests,tree` |
| DNS summary | `-z dns,tree` |
| HTTP packet and status counter | `-z http,tree` |
| HTTP/2 packet and status counter | `-z http2,tree` |
| HTTP load distribution | `-z http_srv,tree` |
| HTTP requests | `-z http_req,tree` |
| HTTP requests and responses | `-z http_seq,tree` |

### 9.1 IP protocol types

**Command**

```bash
tshark -r demo.pcapng -z ptype,tree -q
```

**What it does / Purpose**

Counts packets by the protocol carried inside IP (TCP, UDP, ICMP, ...). It gives a quick picture of the traffic mix, and an unexpected protocol (lots of ICMP, say) stands out immediately.

**Output**

```
==========================================================================================================================
IPv4 Statistics/IP Protocol Types:
Topic / Item       Count         Average       Min val       Max val Rate (ms)     Percent       Burst rate    Burst start
--------------------------------------------------------------------------------------------------------------------------
IP Protocol Types  43                                                0.0014        100          0.0400        2.554
 TCP               41                                                0.0013        95.35        0.0300        0.911
 UDP               2                                                 0.0001        4.65         0.0100        2.554
--------------------------------------------------------------------------------------------------------------------------
```

**Reading the output**

95.35% of the traffic is TCP and 4.65% is UDP. There is no ICMP or any other protocol.

### 9.2 All hosts

**Command**

```bash
tshark -r demo.pcapng -z ip_hosts,tree -q
```

(For IPv6, use `-z ipv6_hosts,tree -q`.)

**What it does / Purpose**

Lists every IP address with the number and percentage of packets it appears in. In large captures this helps you detect an anomalous host at a glance.

**Output**

```
===========================================================================================================================
IPv4 Statistics/All Addresses:
Topic / Item      Count         Average       Min val       Max val  Rate (ms)     Percent       Burst rate    Burst start
---------------------------------------------------------------------------------------------------------------------------
All Addresses     43                                                 0.0014        100          0.0400        2.554
 145.254.160.237  43                                                 0.0014        100.00       0.0400        2.554
 65.208.228.223   34                                                 0.0011        79.07        0.0300        0.911
---------------------------------------------------------------------------------------------------------------------------
```

**Reading the output**

145.254.160.237 appears in 100% of packets (the capturing host), and 65.208.228.223 appears in 79%.

### 9.3 Source and destination addresses

**Command**

```bash
tshark -r demo.pcapng -z ip_srcdst,tree -q
```

(For IPv6, use `-z ipv6_srcdst,tree -q`.)

**What it does / Purpose**

Separates the list into **source** addresses (who sent) and **destination** addresses (who received). Complex cases need correlation, and knowing who is sending versus receiving is the starting point.

**Output**

```
==========================================================================================================================
IPv4 Statistics/Source and Destination Addresses:
Topic / Item                     Count         Average       Min val       Max val  Rate (ms)     Percent       Burst rate    Burst start
--------------------------------------------------------------------------------------------------------------------------
Source IPv4 Addresses            43                                                 0.0014        100          0.0400
 145.254.160.237                 20                                                 0.0007        46.51        0.0200
 65.208.228.223                  18                                                 0.0006        41.86        0.0200
...
Destination IPv4 Addresses       43                                                 0.0014        100          0.0400
 145.254.160.237                 23                                                 0.0008        53.49        0.0200
 65.208.228.223                  16                                                 0.0005        37.21        0.0200
...
------------------------------------------------------------------------------------------------------------------------
```

**Reading the output**

145.254.160.237 sent 20 packets and received 23; 65.208.228.223 sent 18 and received 16.

### 9.4 Destinations and ports

**Command**

```bash
tshark -r demo.pcapng -z dests,tree -q
```

(For IPv6, use `-z ipv6_dests,tree -q`.)

**What it does / Purpose**

For each destination address, shows which protocol and **port** the traffic was sent to. This is how you identify which services were contacted (port 80 = HTTP, 53 = DNS, and so on) and spot odd ports.

**Output**

```
=============================================================================================================================
IPv4 Statistics/Destinations and Ports:
Topic / Item            Count         Average       Min val       Max val       Rate (ms)     Percent       Burst rate    Burst start
-----------------------------------------------------------------------------------------------------------------------------
Destinations and Ports  43                                                      0.0014        100          0.0400        2.554
 145.254.160.237        23                                                      0.0008        53.49        0.0200        2.554
  TCP                   22                                                      0.0007        95.65        0.0200        2.554
   3372                 18                                                      0.0006        81.82        0.0200        2.554
   3371                 4                                                       0.0001        18.18        0.0200        3.916
  UDP                   1                                                       0.0000        4.35         0.0100        2.914
   3009                 1                                                       0.0000        100.00       0.0100        2.914
 65.208.228.223         16                                                      0.0005        37.21        0.0200        0.911
...
-----------------------------------------------------------------------------------------------------------------------------
```

**Reading the output**

Under the destination 145.254.160.237, 22 packets arrived over TCP on ports 3372 (18) and 3371 (4), and 1 over UDP on port 3009. These are temporary client-side ports, which makes sense because this host is the client receiving replies.

### 9.5 DNS statistics

**Command**

```bash
tshark -r demo.pcapng -z dns,tree -q
```

**What it does / Purpose**

Summarises DNS activity: total packets, response codes (`rcode`), operation types (`opcodes`), and query types. Useful for spotting failed lookups, or unusual volumes and types of DNS queries.

**Output**

```
===========================================================================================================================
DNS:
Topic / Item                   Count         Average       Min val       Max val       Rate (ms)     Percent       Burst rate    Burst start
---------------------------------------------------------------------------------------------------------------------------
Total Packets                  2                                             0.0055        100          0.0100        2.554
 rcode                         2                                             0.0055        100.00       0.0100        2.554
  No error                     2                                             0.0055        100.00       0.0100        2.554
 opcodes                       2                                             0.0055        100.00       0.0100        2.554
  Standard query               2                                             0.0055        100.00       0.0100        2.554
...
-------------------------------------------------------------------------------------------------------------------------
```

**Reading the output**

Both DNS packets were standard queries/responses and both completed with `No error`. A high share of error codes (such as non-existent domain) would be worth investigating.

### 9.6 HTTP statistics

**Command**

```bash
tshark -r demo.pcapng -z http,tree -q
```

**What it does / Purpose**

Counts HTTP packets and breaks the responses down by status class (2xx success, 3xx redirection, 4xx client error, 5xx server error). A pile of 4xx/5xx responses can point to scanning, brute-forcing or a broken service.

**Output**

```
=============================================================================================================================
HTTP/Packet Counter:
Topic / Item            Count         Average       Min val       Max val       Rate (ms)     Percent     Burst rate  Burst start
----------------------------------------------------------------------------------------------------------------------------
Total HTTP Packets      4                                                       0.0010        100          0.0100     0.911
 HTTP Response Packets  2                                                       0.0005        50.00        0.0100     3.956
  2xx: Success          2                                                       0.0005        100.00       0.0100     3.956
   200 OK               2                                                       0.0005        100.00       0.0100     3.956
  ???: broken           0                                                       0.0000        0.00         -          -
  3xx: Redirection      0                                                       0.0000        0.00         -          -
...
-----------------------------------------------------------------------------------------------------------------------
```

**Reading the output**

4 HTTP packets in total: 2 requests and 2 responses. Both responses were `200 OK`; there were no redirects or errors.

**Other HTTP reports**

| Statistic | What it gives you |
|-----------|-------------------|
| `-z http2,tree` | Same packet/status counter for HTTP/2 |
| `-z http_srv,tree` | Load distribution: which servers handled the requests |
| `-z http_req,tree` | The requests that were made (hosts and URIs) |
| `-z http_seq,tree` | Requests paired with their responses |

---

## 10. Advanced filtering: `contains`, `matches` and field extraction

Sometimes the built-in filters aren't enough. Wireshark's comparison operators and TShark's field extraction handle those cases.

### 10.1 The operators

| Operator | What it does | Case | Notes |
|----------|--------------|------|-------|
| `contains` | True if a field includes the given text. Works like Wireshark's "Find", but limited to one field. | Case-**sensitive** | Not usable on integer fields |
| `matches` | True if a field fits a **regular expression** pattern | Case-**insensitive** | Not usable on integer fields; complex regex can give unreliable results |

**Tip:** hex and regex values usually match more reliably than plain ASCII text.

#### `contains`: find a specific server software

**Command**

```bash
tshark -r demo.pcapng -Y 'http.server contains "Apache"'
```

**Flags**

- `-Y '...'`: display filter.
- `http.server`: the HTTP `Server:` header field.
- `contains "Apache"`: keep only packets whose `Server` header includes the text `Apache`.

**What it does / Purpose**

Finds every HTTP response from an Apache web server. Useful for identifying server software, which helps in checking for known vulnerabilities or inventorying systems.

**Output**

```
38   4.846969 65.208.228.223 → 145.254.160.237 HTTP/XML HTTP/1.1 200 OK
```

**Add field extraction** to see the actual value instead of the whole packet line:

```bash
tshark -r demo.pcapng -Y 'http.server contains "Apache"' -T fields -e ip.src -e ip.dst -e http.server -E header=y
```

```
ip.src	ip.dst	http.server
65.208.228.223	145.254.160.237	Apache
```

The matching packet came from 65.208.228.223 and its `Server` header is `Apache`.

#### `matches`: find several values with one pattern

**Command**

```bash
tshark -r demo.pcapng -Y 'http.request.method matches "(GET|POST)"'
```

**Flags**

- `http.request.method`: the HTTP method field (GET, POST, ...).
- `matches "(GET|POST)"`: a regular expression meaning "GET **or** POST".

**What it does / Purpose**

Shows only requests that use either method. Regex lets you search for multiple alternatives in one filter, such as `.php` or `.html` pages, several methods, or several keywords.

**Output**

```
4   0.911310 145.254.160.237 → 65.208.228.223 HTTP GET /download.html HTTP/1.1
18   2.984291 145.254.160.237 → 216.239.59.99 HTTP GET /pagead/ads?client=ca-pub-2309191948673629&random=1084443430285&
```

**With field extraction:**

```bash
tshark -r demo.pcapng -Y 'http.request.method matches "(GET|POST)"' -T fields -e ip.src -e ip.dst -e http.request.method -E header=y
```

```
ip.src	ip.dst	http.request.method
145.254.160.237	65.208.228.223	GET
145.254.160.237	216.239.59.99	GET
```

### 10.2 Extract fields

| Flag | What it does |
|------|--------------|
| `-T fields` | Changes the output format: instead of packet summary lines, print only the fields you choose |
| `-e <field>` | Names a field to extract. Repeat `-e` once per field; they print in the order given |
| `-E header=y` | Prints a header row with the field names |

**Command**

```bash
tshark -r demo.pcapng -T fields -e ip.src -e ip.dst -E header=y -c 5
```

**Flags**

- `-T fields`: field output mode.
- `-e ip.src -e ip.dst`: extract the source IP and the destination IP.
- `-E header=y`: add the column header row.
- `-c 5`: first 5 packets only.

**What it does**

Prints a clean, tab-separated table of just the chosen fields for each packet.

**Purpose**

Collect and correlate specific values (IPs, hostnames, URLs, user agents) and feed them into other tools. You can find field names with `-V` output or Wireshark's Display Filter Reference.

**Output**

```
ip.src	ip.dst
145.254.160.237	65.208.228.223
65.208.228.223	145.254.160.237
145.254.160.237	65.208.228.223
145.254.160.237	65.208.228.223
65.208.228.223	145.254.160.237
```

---

## 11. Analyst use cases

After viewing the statistics and drafting an investigation plan, a security analyst typically extracts hostnames, DNS queries and user agents first to find "low-hanging fruit".

Raw `-T fields` output is messy: it has blank lines (packets without that field) and thousands of repeated values. The solution is the same cleanup pipeline every time.

### The cleanup pipeline

```bash
tshark -r FILE -T fields -e FIELD | awk NF | sort -r | uniq -c | sort -r
```

| Part | What it does | Why it's needed |
|------|--------------|-----------------|
| `tshark -r FILE -T fields -e FIELD` | Extracts one field from every packet | The main query, producing the raw values |
| `awk NF` | Keeps only lines that have at least one field (`NF` = number of fields) | Removes blank lines left by packets without the field |
| `sort -r` | Sorts the values | `uniq` only collapses **adjacent** duplicates, so identical values must be grouped first |
| `uniq -c` | Collapses duplicates and prefixes each unique value with its **count** | Turns thousands of lines into a short list with frequencies |
| `sort -r` | Sorts again, highest count first | Puts the most frequent values at the top |

### 11.1 Extract hostnames (DHCP)

**What it does / Purpose**

DHCP requests include the device's hostname. Extracting them gives you a list of the machines present on the network, which is useful for asset inventory and for spotting unknown or oddly-named devices.

**Raw output** (blank lines and repeats are hard to use):

```bash
tshark -r hostnames.pcapng -T fields -e dhcp.option.hostname
```

```
92-rkd
92-rkd
T3400

T3400

60-alfb-sec2
60-alfb-sec2

aminott
...
```

**Cleaned and counted:**

```bash
tshark -r hostnames.pcapng -T fields -e dhcp.option.hostname | awk NF | sort -r | uniq -c | sort -r
```

```
     26 202-ac
     18 92-rkd
     14 93-sts-sec
...
```

**Reading the output**

The number is how many DHCP packets each hostname produced. `202-ac` was seen 26 times, `92-rkd` 18 times, and so on. Very chatty or unfamiliar hostnames are the ones to look at first.

### 11.2 Extract DNS queries

**What it does / Purpose**

Lists every domain name the hosts asked DNS to resolve. This shows where machines are trying to connect. Unusual or long random-looking domains can indicate malware contacting its server or DNS tunnelling.

**Command**

```bash
tshark -r dns-queries.pcap -T fields -e dns.qry.name | awk NF | sort -r | uniq -c | sort -r
```

**Output**

```
     96 connectivity-check.ubuntu.com.rhodes.edu
     94 connectivity-check.ubuntu.com
      8 3.57.20.10.in-addr.arpa
      4 e.9.d.b.c.9.d.7.1.b.0.f.a.2.0.2.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa
      4 0.f.2.5.6.b.e.f.f.f.b.7.2.4.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa
      2 _ipps._tcp.local,_ipp._tcp.local
      2 84.170.224.35.in-addr.arpa
      2 22.2.10.10.in-addr.arpa
```

**Reading the output**

| Entry | Meaning |
|-------|---------|
| `connectivity-check.ubuntu.com` | Ubuntu's routine "am I online?" check (the most frequent, so benign noise) |
| `*.in-addr.arpa` | **Reverse lookups**: asking "what name belongs to this IPv4 address?" |
| `*.ip6.arpa` | Reverse lookups for IPv6 addresses |
| `_ipp._tcp.local` | Local printer discovery (mDNS) |

### 11.3 Extract HTTP user agents

**What it does / Purpose**

The `User-Agent` header identifies the client software making a web request. It reveals which browsers and operating systems are in use, and attack tools often announce themselves here.

**Command**

```bash
tshark -r user-agents.pcap -T fields -e http.user_agent | awk NF | sort -r | uniq -c | sort -r
```

**Output**

```
      6 Mozilla/5.0 (Windows; U; Windows NT 6.4; en-US) AppleWebKit/534.10 (KHTML, like Gecko) Chrome/8.0.552.237 Safari/534.10
      5 Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0
      5 Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.32 Safari/537.36
      4 sqlmap/1.4#stable (http://sqlmap.org)
      3 Wfuzz/2.7
      3 Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)
```

**Reading the output**

| User agent | What it is |
|------------|-----------|
| `Mozilla/5.0 ... Chrome` / `Firefox` | Ordinary browsers (the first line is an old Chrome on a Windows build, which is itself worth noting) |
| `sqlmap/1.4#stable` | An automated **SQL-injection** testing tool |
| `Wfuzz/2.7` | A web **fuzzer**, which tries many inputs to discover hidden pages or weaknesses |
| `Nmap Scripting Engine` | **Nmap** scripts probing the web server during a scan |

The last three are security and scanning tools, not normal browsing. Seeing them suggests someone was testing or attacking the web server, so the next step would be to filter on those user agents and see which IPs sent them and what they requested.

---

## 12. Quick reference and workflow

### Flags

| Flag | Meaning |
|------|---------|
| `-h` | Help |
| `-v` | Version |
| `-D` | List interfaces |
| `-i <iface>` | Capture on this interface |
| `-r <file>` | Read a capture file |
| `-w <file>` | Write packets to a file |
| `-c <n>` | Stop after *n* packets |
| `-V` | Verbose: full decoded details |
| `-x` | Hex and ASCII dump |
| `-q` | Quiet: hide packet lines |
| `-a <cond>` | Autostop (`duration`, `filesize`, `files`) |
| `-b <cond>` | Ring buffer (`duration`, `filesize`, `files`) |
| `-f "<bpf>"` | Capture filter (live) |
| `-Y '<filter>'` | Display filter (analysis) |
| `-z <stat>` | Statistics report |
| `-T fields` | Field output mode |
| `-e <field>` | Field to extract (repeatable) |
| `-E header=y` | Print column header |
| `--color` | Colourised output |

### Which statistic answers which question

| Question | Command |
|----------|---------|
| What protocols are in this capture? | `-z io,phs -q` |
| Are there unusual packet sizes? | `-z plen,tree -q` |
| Who is on the network? | `-z endpoints,ip -q` |
| Who talked to whom, and for how long? | `-z conv,ip -q` |
| Any network problems or notable events? | `-z expert -q` |
| What is the TCP/UDP split? | `-z ptype,tree -q` |
| Which IPs appear, and how often? | `-z ip_hosts,tree -q` |
| Who sends vs receives? | `-z ip_srcdst,tree -q` |
| Which services/ports were contacted? | `-z dests,tree -q` |
| DNS summary | `-z dns,tree -q` |
| HTTP summary | `-z http,tree -q` |

### Suggested investigation workflow

1. `capinfos file.pcapng`: understand the size, time window and integrity of the file.
2. `tshark -r file.pcapng -z io,phs -q`: see which protocols are present and where to focus.
3. `-z conv,ip -q` and `-z endpoints,ip -q`: identify the busiest or unexpected hosts and conversations.
4. `-Y '<display filter>'`: narrow down to the suspicious traffic.
5. `-T fields -e ... | awk NF | sort -r | uniq -c | sort -r`: extract and rank hostnames, DNS queries and user agents.
6. `-V` or `-x` with `-c`: inspect individual interesting packets in depth.
7. `-w suspicious.pcap`: save the relevant packets for further analysis or hand-off.
