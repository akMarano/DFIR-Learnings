# DFIR Learnings 

Digital Forensics and Incident Response (DFIR) is the process of gathering digital evidence left after an attack and, detecting and responding to an in-progress cyberattack. These two disciplines come together to stop threats and analyze evidence to further improve security.

## Woke Files
### Hidden Files
Implemented for protection and discretion. In Linux it is usually files that start with a `.`, for Windows and Mac, it is metadata for a file that can be toggled on or off.

Usually hidden when viewing files using a GUI file explorer. Hidden files can be found by navigating to the folder using the shell or by toggling an option to show hidden files in the GUI file explorer.

### Metadata

Data about the data or when talking about files, it is the data beyond the actual file, such as who created the file, what the used to create the file, and other information.

In Linux files we understand metadata to be the Owner, Group, Permissions, and Timestamps. In Windows we understand metadata to be the files hidden property and Alternate Data Streams which is basically storing a file in a file.

We can view this metadata either using the GUI by opening the files properties or using the shell.  

We can sort metadata into two kinds.

**Filesystem metadata** - Managed by the file system and is separate from the file's data
**Filetype Metadata** - Managed by the application that creates the file and is within the file's data

In Linux we can use the `Exiftool` tool which allows us to view and edit the metadata of a file, to gather the metadata of a file.
```
exiftool "Test File" 
ExifTool Version Number         : 13.55
File Name                       : Bandit Passwords
Directory                       : .
File Size                       : 1778 bytes
File Modification Date/Time     : 2026:08:10 18:37:41+08:00
File Access Date/Time           : 2026:08:20 20:09:35+08:00
File Inode Change Date/Time     : 2026:08:10 18:37:41+08:00
File Permissions                : -rw-rw-r--
File Type                       : TXT
File Type Extension             : txt
MIME Type                       : text/plain
MIME Encoding                   : us-ascii
Newlines                        : Unix LF
Line Count                      : 60
Word Count                      : 245
```
As you can see we are able to list down the metadata of the file.

### Files as hex
When we open files, by default we interpret the data contained in the file to present it to us, such as in an image file, the data contains the pixel count, RGB values and etc. when we open the file, it interprets all this and displays us the actual image. If we want to open a file without interpretation, we refer to this as a Raw file, we open it using a hex editor. 

We can do this using the `xxd` tool which either allows us to create a hex dump of a file, or revert a hex dump into a file.
```
xxd Hello.txt   
00000000: 4865 6c6c 6f20 576f 726c 640a            Hello World.
```
The first column is the offset in the file, the next columns are the actual hex dump of the file, and the last column is the ASCII interpretation of the file data.

### Magic bytes
Magic bytes are the unique bytes in a files hex dump that allow us to identify what kind of file it is. We can access these magic bytes using the link provided.
```
https://filesig.search.org/
```

## Getting Flags with Metadata
When finding a suspicious string of characters in a file's details it is important to consider decrypting it. Such as for "Information" Challenge in PicoCTF. Opening the files details with `exiftool` gave us this output.
```
exiftool "cat.jpg"                                          
ExifTool Version Number         : 13.55
File Name                       : cat.jpg
Directory                       : .
File Size                       : 878 kB
File Modification Date/Time     : 2026:08:20 20:57:24+08:00
File Access Date/Time           : 2026:08:20 20:59:35+08:00
File Inode Change Date/Time     : 2026:08:20 20:57:24+08:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.02
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Current IPTC Digest             : 7a78f3d9cfb1ce42ab5a3aa30573d617
Copyright Notice                : PicoCTF
Application Record Version      : 4
XMP Toolkit                     : Image::ExifTool 10.80
License                         : cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9
Rights                          : PicoCTF
Image Width                     : 2560
Image Height                    : 1598
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 2560x1598
Megapixels                      : 4.1
```
My preliminary scan made me suspicious of these two lines as the were a strange series of characters.
```
Current IPTC Digest             : 7a78f3d9cfb1ce42ab5a3aa30573d617
License                         : cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9
```
I first tried decoding both with common encoding formats, first was a Caesar cipher since it was alphanumeric but it lead nowhere, next was base32 but that didn't work, and then I tried base64 which didn't result with anything for the IPTC Digest, but it did give me the flag when I inputted the License contents.

Next was the "Glory of the Garden" level where I utilized the `xxd` command to analyze the hex dump of the file and in the last sections of the ASCII interpretation of the file was the flag.
```
00230500: d9f9 9f63 4b2b c1e5 daf2 7b59 db49 4ba3  ...cK+....{Y.IK.
00230510: f43e b881 5e30 1060 8030 47d6 bacb 58cb  .>..^0.`.0G...X.
00230520: 1046 07b5 7216 df7e 5ff7 c576 363f ebab  .F..r..~_..v6?..
00230530: b70d 18ce 3ccd 6a7e 6b8d af56 b579 39ca  ....<.j~k..V.y9.
00230540: eeef 53ae 8620 31b8 751f 9514 f7fb cff5  ..S.. 1.u.......
00230550: a2bb bdac 9687 98e4 d3b2 e87f ffd9 4865  ..............He
00230560: 7265 2069 7320 6120 666c 6167 3a20 7069  re is a flag: pi
00230570: 636f 4354 467b 6d6f 7265 5f74 6861 6e5f  coCTF{more_than_
00230580: 6d33 3374 735f 7468 655f 3379 3361 3633  m33ts_the_3y3a63
00230590: 6235 6232 377d 0a                        b5b27}.
```
Next was the "Enchanced!" level where I used an new command `xmllint` since when I checked the metadata details of the file it output this.
```
exiftool drawing.flag.svg         
ExifTool Version Number         : 13.55
File Name                       : drawing.flag.svg
Directory                       : .
File Size                       : 4.1 kB
File Modification Date/Time     : 2026:08:20 21:33:48+08:00
File Access Date/Time           : 2026:08:20 21:34:06+08:00
File Inode Change Date/Time     : 2026:08:20 21:33:49+08:00
File Permissions                : -rw-rw-r--
File Type                       : SVG
File Type Extension             : svg
MIME Type                       : image/svg+xml
Xmlns                           : http://www.w3.org/2000/svg
Image Width                     : 210mm
Image Height                    : 297mm
View Box                        : 0 0 210 297
SVG Version                     : 1.1
ID                              : svg8
Version                         : 0.92.5 (2060ec1f9f, 2020-04-08)
Docname                         : drawing.svg
Metadata ID                     : metadata5
Work Format                     : image/svg+xml
Work Type                       : http://purl.org/dc/dcmitype/StillImage
Work Title                      : 
```
As we can see it has xml properties in the type and format. This led to me research how to open or analyze XML files which led me to the '`xmllint`  command which I used like so.
```
xmllint --format  drawing.flag.svg 
---<cutoff>---
1;stroke:none;stroke-width:0.26458332;" x="107.43014" y="132.08501" id="text3723">
      <tspan sodipodi:role="line" x="107.43014" y="132.08501" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3748">p </tspan>
      <tspan sodipodi:role="line" x="107.43014" y="132.08942" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3754">i </tspan>
      <tspan sodipodi:role="line" x="107.43014" y="132.09383" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3756">c </tspan>
      <tspan sodipodi:role="line" x="107.43014" y="132.09824" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3758">o </tspan>
      <tspan sodipodi:role="line" x="107.43014" y="132.10265" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3760">C </tspan>
      <tspan sodipodi:role="line" x="107.43014" y="132.10706" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3762">T </tspan>
      <tspan sodipodi:role="line" x="107.43014" y="132.11147" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3764">F { 3 n h 4 n </tspan>
      <tspan sodipodi:role="line" x="107.43014" y="132.11588" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3752">c 3 d _ d 0 a 7 5 7 b f }</tspan>
    </text>
  </g>
</svg>
```
I used the option `--format` since without it the output is too messy to recognize the flag. From the output above, we can see the flag appearing.

## Disk Analysis
This is the process of analyzing a disks storage space in order to locate evidence or traces left by cyberattacks to reconstruct timelines, recover deleted files, and find malicious activity. A rule of thumb is to work from a copy in order to preserve the original state and avoid tampering the evidence. 

We can use the tool `dd` to either clone a partition on the disk or clone the entire disk itself. 
`if` = [input file] to copy
`of` = [output file] name of copied file
`bs` = how man bytes to read at a time, use 32M for faster processing

Example of copying a file:
```
dd if=hello.txt of=hello.txt2
0+1 records in
0+1 records out
12 bytes copied, 0.0014034 s, 8.6 kB/s
```
We can check the disks on our system using the `lsblk` command
```
lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   25G  0 disk 
├─sda1   8:1    0 23.7G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0  1.3G  0 part [SWAP]
sr0     11:0    1 1024M  0 rom
```
As we can see, we have 1 disk in our system named `sda` as indicated by the type `disk`, the lines below that is the partitions within the disk as indicated by the type `part`.

Example of copying a disk image
```
dd if=/dev/sda | gzip -c image.dd.gz
```
Our disks and the partitions are located at the `/dev` directory, so we use that file path when copying a disk or partition. I unfortunately did not run this command as it could take up my entire disk space, however that is also connected to why we combine the command with `gzip` so that we are able to compress the file for better storage.

## Layers
Disk images are comprised of different layers.
- Multimedia Layer
- Block Layer
- Journal Layer
- Metadata Layer
- Filename Layer

**Multimedia Layer**

This Layer holds the partitions of the disk, usually the starting point when analyzing.

**Block Layer**

This layer holds the actual data of the disks in equally sized parts.

**Journal Layer**

This layer is used for incremental backups of the disk and its used for recovery when crashes happen.

**Metadata Layer**

In Linux this holds the inodes which is what holds the metadata of the filesystem which are the permissions, ownership, timestamp, and blocks.

**Filename Layer**

Similar to the normal shell experience of navigating through files, it is also the highest level of a disk image.

## Sleuthkit
This is a tool we can use to navigate the different layers of a disk image, as stated before, it starts us at the multimedia layer.

The sleuthkit primarily has `stat`. `ls` and `cat` commands for all layers, they simply have different prefixes,

| Layer | stat | ls | cat | Extra Commands |
| ----------- | ----------- |  ----------- |  ----------- |   ----------- |
| Multimedia | mmstat | mmls | mmcat | --- |
| Block | blkstat | blkls | blkcat | blkcalc |
| Journal | jstat | --- | jcat | --- |
| Metadata | istat | ils | icat | ifind |
| File System | --- | fls | fcat | ffind |

We can check the partitions in a image using the `mmls` command
```
 mmls disk.img             
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0000204799   0000202752   Linux (0x83)
```
Now we are primarily interested in the `Linux` partition, so we will use the offset `2048` indicated by the option `-o` to explore that partition with our commands.

With that, let's check the filesystem of the disk image.
```
fls -o 2048 disk.img
d/d 15617:      home
d/d 11: lost+found
r/r 12: .dockerenv
d/d 21473:      bin
d/d 1953:       boot
d/d 13665:      dev
d/d 17569:      etc
d/d 3905:       lib
d/d 15618:      media
d/d 13669:      mnt
d/d 13670:      opt
d/d 13671:      proc
d/d 15622:      root
d/d 13672:      run
d/d 15623:      sbin
d/d 13673:      srv
d/d 15700:      sys
d/d 13674:      tmp
d/d 15701:      usr
d/d 13675:      var
V/V 25377:      $OrphanFiles
```
This command shows us the top level directories of the disk, we can add the `-r` option to display all the files within the directories. It is recommended to pipe the output into the `less` command for easier viewing.

```
fls -ro 2048 disk.img | less
d/d 15617:      home
d/d 11: lost+found
r/r 12: .dockerenv
d/d 21473:      bin
+ -/- * 2049:   ^
+ l/l 21476:    base64
+ l/l 21477:    bbconfig
+ r/r 21478:    busybox
+ l/l 21479:    cat
+ l/l 21480:    chgrp
+ l/l 21481:    chmod
+ l/l 21482:    chown
+ l/l 21483:    conspy
+ l/l 21484:    cp
+ l/l 21485:    date
+ l/l 21486:    dd
+ l/l 21487:    df
+ l/l 21488:    dmesg
+ l/l 21489:    dnsdomainname
+ l/l 21490:    dumpkmap
+ l/l 21491:    echo
+ l/l 21492:    ed
+ l/l 21493:    egrep
+ l/l 21494:    false
+ l/l 21495:    fatattr
+ l/l 21496:    fdflush
+ l/l 21497:    fgrep
-----<cutoff>-----
:
```
From here we can input strings in the `:` space to locate files we are looking for.

Now beside the file name is actually the inode number of the file which we can use with the command `istat` to view more details about the file. For this example let us use the inode for the `echo` file (`21491`).

```
istat -o 2048 disk.img 21491
inode: 21491
Allocated
Group: 11
Generation Id: 601166711
symbolic link to: /bin/busybox
uid / gid: 0 / 0
mode: lrwxrwxrwx
size: 12
num of links: 1

Inode Times:
Accessed:       2021-09-22 03:34:53 (PST)
File Modified:  2021-09-22 03:34:53 (PST)
Inode Modified: 2021-09-22 03:34:53 (PST)

Direct Blocks:
0 
```
Above are the details of the file, keep in mind the same syntax can be used to run any command that is part of the metadata layer.

For further information regarding the sleuthkit commands, we can visit the URL below for a cheat sheet of the commands and what they can do.

```
https://github.com/sleuthkit/sleuthkit/wiki/The_Sleuth_Kit_commands
```

## Autopsy
Autopsy is the Graphical User Interface (GUI) of the sleuthkit. It utilizes a Web UI and it is better for exploration due to it simply being point and click.

We can start Autopsy using the command below
```
sudo autopsy

============================================================================

                       Autopsy Forensic Browser 
                  http://www.sleuthkit.org/autopsy/
                             ver 2.24 

============================================================================
Evidence Locker: /var/lib/autopsy
Start Time: Sun Aug 23 03:17:15 2026
Remote Host: localhost
Local Port: 9999

Open an HTML browser on the remote host and paste this URL in it:

    http://localhost:9999/autopsy

Keep this process running and use <ctrl-c> to exit
Can't open log: autopsy.log at /usr/share/autopsy/lib/Print.pm line 383.
```
As you can see it gave us a localhost url we can visit to open autopsy.

From here we can add our image and explore, we can select details for a disk, and then proceed to file system to analyze the metadata of the disk, we can then proceed to file analysis to explore the files within the disk.

Regarding CTF Challenges with Disk Exploration, ensure to maximize search features and open common directories where flags may be stored and look for common file names and keywords where the flag may be contained.

## Steganography
### Steganography vs Cryptography

**Steganography**

From the greek word "Steganos" meaning concealed or covered and "Graphy" meaning writing.

**Cryptography**

From the greek word "Kryptos" meaning secret or hidden and "Graphy" meaning writing. A popular example is the Caesar Cypher which shifts the letters of the alphabet based.

Steganography is more focused on hiding data from plain view whereas Cryptography is more focused on protecting the data from being accessed. For now let us focus on Digital Steganography.

There are many forms of Digital Steganography some are:
- File Format
- Least Significant Bit
- Encrypted Embed

### File Format
We can hide information in a files hex dump, now we'd usually view the hex dump with `xxd`, but to edit the hex dump and insert information, we would use the `bvi` command.
```
bvi cat.jpg          
000d65b0: bfe9 1cfe d9d2 3ff8 cffe 52ff 00a4 4937  ......?...R...I7
000d65c0: 938a 2213 2622 0e31 a45f ff00 19ff 00ca  ..".&".1._......
000d65d0: 5ff4 84ff 00b6 348f fe33 ff00 94bf e912  _.....4..3......
000d65e0: 4de4 d5c4 26a3 113f db0a 47ff 0019 ff00  M...&..?..G.....
000d65f0: ca5f f482 9c5f 493f fbdf ff00 297f d224  ._..._I?....)..$
000d6600: 9bc9 3260 aa31 1871 652b ff00 8aff 00e5  ..2`.1.qe+......
000d6610: affa 410e 2ca5 ff00 f13f fcb5 ff00 4892  ..A.,....?....H.
000d6620: 6f24 8982 dcc4 61c5 74bf fe27 ff00 96af  o$....a.t..'....
000d6630: e91c 38a6 99ff 00c4 ff00                 ..8........
```
From here we can navigate to the bottom of the file by pressing `[SHIFT+G]`, we can then enter insert mode by pressing `:`, and then running `set memmove`, after we've written our changes, we can press `[ESC]` to exit and then press `:` and run `w` to save and `g` to exit. Let's check our changes using the `xxd` command.
```
xxd cat.jpg | tail             
000d65b0: bfe9 1cfe d9d2 3ff8 cffe 52ff 00a4 4937  ......?...R...I7
000d65c0: 938a 2213 2622 0e31 a45f ff00 19ff 00ca  ..".&".1._......
000d65d0: 5ff4 84ff 00b6 348f fe33 ff00 94bf e912  _.....4..3......
000d65e0: 4de4 d5c4 26a3 113f db0a 47ff 0019 ff00  M...&..?..G.....
000d65f0: ca5f f482 9c5f 493f fbdf ff00 297f d224  ._..._I?....)..$
000d6600: 9bc9 3260 aa31 1871 652b ff00 8aff 00e5  ..2`.1.qe+......
000d6610: affa 410e 2ca5 ff00 f13f fcb5 ff00 4892  ..A.,....?....H.
000d6620: 6f24 8982 dcc4 61c5 74bf fe27 ff00 96af  o$....a.t..'....
000d6630: e91c 38a6 99ff 00c4 ff00 f254 6869 7320  ..8........This 
000d6640: 6461 7461 2069 7320 6869 6464 656e 6e    data is hidden.                                                                   
```
As you can see we've inserted the string `This data is hidden` in the images hex dump. This string is hidden even if we view the file's metadata.

```
exiftool "cat.jpg"                  
ExifTool Version Number         : 13.55
File Name                       : cat.jpg
Directory                       : .
File Size                       : 878 kB
File Modification Date/Time     : 2026:08:23 17:24:03+08:00
File Access Date/Time           : 2026:08:23 17:24:20+08:00
File Inode Change Date/Time     : 2026:08:23 17:24:03+08:00
File Permissions                : -rw-rw-r--
Warning                         : Processing JPEG-like data after unknown 7-byte header
JFIF Version                    : 1.02
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Current IPTC Digest             : 7a78f3d9cfb1ce42ab5a3aa30573d617
Copyright Notice                : PicoCTF
Application Record Version      : 4
Xmpmeta Xmptk                   : Image::ExifTool 10.80
Xmpmeta License                 : cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9
Xmpmeta Rights                  : PicoCTF
Image Width                     : 2560
Image Height                    : 1598
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 2560x1598
Megapixels                      : 4.1
```

It is however visible if we run the `strings` command with the file.
```
strings cat.jpg
;4--
8|"2
feRXVt
Hm\O>
\pkLS)S
ou9N
KMI//
Ok8k
F5/?
UA]L
POjXt
WX/yWX
L\-2
This data is hidden.
```

### Least Common Bit (LSB)
Pixels are Tuples of Red, Green, Blue, and some have Alpha which is Opacity.
Examples:
- (100%, 0%, 0%) = Red
- (100%, 100%, 0%) = Yellow
- (100%, 100%, 100%) = White

Color Depth allows us to decide how many colors we decide to represent. 
Commonly Expressed as 24 bits; Red: 8 bit, Green: 8 bit, Blue: 8 bit.

We can view this in image files with hex dump so their color pixels are displayed in hexadecimal.

**What is Bit Significance?**

The most significant bit shows the most change when changed.
| Bits | Binary | Hex | Decimal |
| -------- | -------- | -------- | -------- |
| No bits set | 00000000 | 03 | 0 |
| LSB set | 00000001 | 01 | 1 |
| MSB set | 10000000 | 80 | 128 |
As you can see when we flipped a no bit to LSB or the right-most bit, it changed a little compared to when flipping to MSB or the left-most bit where it changed a lot.

**Tweaking the LSB**
| Bits | Binary | Hex | Decimal |
| -------- | -------- | -------- | -------- |
| LSB on | 00000001 | 01| 1 |
| LSB off | 00000000 | 00 | 0 |
| MSB on | 11111111 | FF | 255 |
| MSB off | 11111110 | FE | 256 |

Thanks to this system, we can have a covert channel that can carry information and is undetectable to the eye. We can automate this process with the a python file found below.
```
[stego.py](https://github.com/djrobin17/image-stego-tool/blob/master/stego.py)
```

We can run it using the `ipython3` command.
```
--Welcome to $t3g0--
1: Encode
2: Decode
```
We can use this code to encode and decode hidden messages within our files.

**Steganalysis**
This is the process of extracting the hidden information within the file. This process is VERY hard, it is useful to know when steganography is present within the file and to have the original copy of the file without a hidden message in it. We can utilize the tool `zsteg` and `steghide` for steganalysis.

These tools allow us to extract and detect the hidden information in png and jpg images.

## Packet Analysis
A packet is a single unit of network communication composed of layers. A packet capture is a collection of many packets taken from a network interface. Packet Analysis is the process of inspecting and studying a packet capture.

### Packet Layers

**OSI Model**

1. Physical Layer
2. Data Link Layer
3. Network Layer
4. Transport Layer
5. Session Layer
6. Presentation Layer
7. Application Layer

**Wireshark Layers**

Similar to OSI Model but less layers

1. Physical Layer
2. Data Link Layer
3. Network Layer
4. Transport Layer
5. Application Layer

**Physical Layer**

Contains the raw bits of information. Transmitted through wires with an electric current, or radio signals through wireless networks.

**Data Link Layer**

Connection with the built-in addresses of the hardwares within the local network. Only directly connected nodes, including WiFi nodes with radio connection.

**Network Layer**

Connection with assigned address, how the packets know how to get from one network to another, and what connects to the internet.

**Transport Layer**

Multiplexing within host via ports and sometimes checking for connection reliability.

**Application Layer**

Structured Data for user programs like browsers, torrents, and emails.

**Example Packet**

```
0000  00 11 22 33 44 55 66 77 88 99 aa bb 08 00 45 00  ..".DUfw.....E.
0010  00 3c ab cd 40 00 40 06 12 34 c0 a8 01 0a c0 a8  .<..@.@..4......
0020  01 01 d3 48 00 50 12 34 56 78 ab cd ef 01 80 18  ...H.P.4Vx......
0030  00 e5 8e 35 00 00 01 01 08 0a 00 23 45 67 89 ab  ...5.......#Eg..
0040  cd ef 68 65 6c 6c 6f                         ..hello
```
In this Packet, the entire Packet is the Physical Layer as it contains all the bits of information.

The first line contains the information about the Data Link Layer

The second line contains the information about the Network Layer

The third and fourth line contains the information about the Transport Layer

The rest of the lines contains the information about the Application Layer

### Wireshark
A tool used for Packet Analysis

Some Challenges encountered during Packet Analysis are:
- Segmentation = Packets are sent as segments you have to assemble
- Volume = Packet captures return a LOT of packets.

**Segmentation**

Stream is the back and forth between devices. Follow TCP streams for better view, export objects.

**Volume**

Filters autocompletes all possible outcomes based on layers. We can also apply data as filters.

- Physical Layer = frame
- Data Link Layer = eth
- Network Layer = ip
- Transport Layer = tcp/udp
- Application Layer = http, etc.

Type filters in the filters box to sort the data within the packet

For Caesar Cipher Codes we can use this website
```
https://cryptii.com/pipes/caesar-cipher/
```

Or we can also run this command 
```
echo "<Insert Text Here>" | tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
```
Notes: Use filters as much as possible to isolate interesting packets in the packet capture. Decode any interesting strings, and extract objects like files that were included in the packet capture.

**Example with Trivial Flag Transfer Protocol:**

First step was analyzing the packets and then using the filter `tftp.type` to isolate the packets that have a string message or assignment. Next after applying this filter was to extract the TFTP objects as files and analyze these files. We then analyze any contents in these files like strings and then use that to find the flag. In this case one file indicated to use the Debian Program which was steghide to decrypt the information, so I used that and utilized the password encrypted in the plan file.



