# Windows Forensics

Computer Forensics involves gathering evidence of activities performed on computers, It is part of the wide Digital Forensics Field that deals with forensic analysis of all types of  digital devices including recovering, examining, and analyzing data.

Microsoft Windows is the most used Desktop Operating System right now. Private users and Enterprises prefer it, with it holding 80% of the Desktop Market. This means it is important to know how to perform forensic analysis on Windows Systems.

## Windows Tracking

Windows keeps track of a lot of activity performed by the user, this is primarily for following the user's preferences to save it and make your computer feel more personalized based on the preferences. While it is not there to spy on us, the same information can be used by forensic investigators when performing forensic analysis as windows stores these artifacts in different locations throughout the file system like the registry, user profile directory, and application specific files.

## Windows Registry

Windows Registry is the collection of databases that contains the system's configuration data, this can be about the hardware, software or user information. This includes the data about recently used files, programs, or devices connected to the system. We can view the registry using the `regedit.exe` which is a built in Windows utility to view and edit the registry. We can run this by searching it in the windows search bar, or by pressing `[WINDOWS KEY]` + `[R KEY]` and inputting `regedit.exe`. You will then be greeted the by registry GUI.

The Windows registry consists of Keys and Values, when opening the registry, the folders we see are the Registry Keys, the Registry Values are stored in them. A Registry Hive is a group of Keys, subkeys, and values stored in a single file.


### Structure of the Registry

The registry has the following 5 root keys:
1. **HKEY_CURRENT_USER** = HKCU
2. **HKEY_USERS** = HKU
3. **HKEY_LOCAL_MACHINE** = HKLM
4. **HKEY_CLASSES_ROOT** = HKCR
5. **HKEY_CURRENT_CONFIG** = HKCC

Below are the details and information regarding each Registry Key.

| Folder | Description |
| ----------- | ----------- |
| **HKEY_CURRENT_USER** | Contains the Root information of the logged in user. The user's profile information such as, folders, screen colors, and control panel settings, are stored here |
| **HKEY_USERS** | Contains all the user profiles on the computer, **HKEY_CURRENT_USER** is a subkey of **HKEY_USERS**| 
| HKEY_LOCAL_MACHINE | Contains the configuration information of the computer itself |
| HKEY_CLASSES_ROOT | A subkey of **HKEY_LOCAL_MACHINE\Software**, contains the information to ensure the correct program opens when opening a file. Starting with Windows 2000, this information is stored in **HKEY_LOCAL_MACHINE\Software\Classes** which contains default settings for all users and **HKEY_CURRENT_USER\Software\Classes** which override the default settings and apply only to the specific user. **HKEY_CLASSES_ROOT** merges these two registries. To make changes to the settings of the user then **HKEY_CURRENT_USER\Software\Classes** must be edited, to change the default settings then **HKEY_LOCAL_MACHINE\Software\Classes** must be edited, any changes made to **HKEY_CLASSES_ROOT** will be stored in **HKEY_CURRENT_USER\Software\Classes** if the key already exists there, otherwise it will be stored in **HKEY_LOCAL_MACHINE\Software\Classes** | 
| HKEY_CURRENT_CONFIG | Contains information about the hardware profile used by the computer system at startup |

### Accessing Registry Hives Offline

We are able to access the Registry using `regedit.exe` however in a case where we only have a disk image, we must know how to access the different registry hives on the disk. Majority of these hives are located in the `C:\Windows\System32\Config` directory.

1. DEFAULT (mounted on HKEY_USERS\DEFAULT)
2. SAM (mounted on HKEY_LOCAL_MACHINE\SAM)
3. SECURITY (mounted on HKEY_LOCAL_MACHINE\Security)
4. SOFTWARE (mounted on HKEY_LOCAL_MACHINE\Software)
5. SYSTEM (mounted on HKEY_LOCAL_MACHINE\System)

Apart from those hives, two other hives containing the user information are found in the User profile directory. For Windows 7 and above, the profile directory is at `C:\Users\<username>\`.
1. NTUSER.DAT (mounted on HKEY_CURRENT_USER when a user logs in, located at `\Users\<username>\AppData\Local\Microsoft\Windows`)
2. USRCLASS.DAT (mounted on HKEY_CURRENT_USER\Software\CLASSES, located at `C:\Users\<username>\`)

Apart from these files, there is also the **AmCache hive**, located in `C:\Windows\AppCompat\Programs\Amcache.hve`, which holds the information of the programs recently ran on the system.

Lastly, registry transaction logs ans backups are also vital sources of data. The transactions log located at `C:\Windows\System32\Config ` contains the changelog of the registry hive, Windows often uses transaction logs when writing to the registry hives, so these logs can often have the latest changes to the registry data that hasn't been added to the Registry Hive itself. It has the same name as the Registry hive followed by the extension `.LOG`. Registry Backups located at `C:\Windows\System32\Config`, and they copy the `\Windows\System32\Config\RegBack` every 10 days. This is the place to look if you suspect that some registry keys have been changed recently.

## Data Acquisition

When performing forensics, we will face either a live system or an image of the system, it is recommended to practice on an image of the system or a copy of the data. This process is called data acquisition. 

While we can view the registry through the registry editor (`regedit.exe`), the best method is to create a copy of the data and perform analysis on the copy. However, if we try to navigate to where the hives are stored `C:\Windows\System32\Config`, then we'd see that it is a restricted file. 

To acquire the files we can either use **KAPE** which is primarily CLI live data acquisition and analysis tool but it does have a GUI, or **Autopsy** which allows us to extract the data from both live systems or a disk image, or **FTK Imager** which is similar to Autopsy where it allows us to extract the data from a live system or a disk image.

## Viewing Registry Hives

After acquiring the Registry Hives, we need to view the files as we would on the registry editor, the registry editor only works on live system files and not on imported hives, to do so we need another tool such as the following.

### Registry Viewer

Registry Viewer uses a similar interface to the Registry editor, however it can only load one have at a time, and it can't add transaction logs

### Registry Explorer

Registry Explorer has a more complex layout but it can load multiple hives at once and it can add transaction logs into the hive. It also has a Bookmarks option that contains the important registry keys in a forensic investigation. 

### RegRippper

RegRippper is a tool that takes the registry hive as input and outputs a report that extracts the data of some of the important registry keys, the output is placed in a text file and shows all the results in sequential order. This tool works in both CLI and GUI. Similarly to Registry Viewer however, it does not add transaction logs to the hives.

## Analyzing System Information with Registry Hives

Since we now know how to acquire and view the Registry Hives, We can now explore the Registry Hive to perform our forensic analysis to find information regarding the machine's System Information and Account Information.

### OS Version

The first step is finding out the Systems OS Version, We can find this in the `SOFTWARE\Microsoft\Windows NT\CurrentVersion`, Here we can find all the information about the Systems OS.

### Control Sets

The next step is for the Control Sets, which are the machine's configuration data used for controlling system start up. Here we commonly see two Control Sets, ControlSet001 and ControlSet002 located at `SYSTEM\ControlSet001` and `SYSTEM\ControlSet002`. In most cases, ControlSet001 will point to the Control Set the machine booted, while ControlSet002 will be pointed to the last known good configuration.

Windows creates a Control Set when the machine is live called the CurrentControlSet located at `HKLM\SYSTEM\CurrentControlSet `. We explore this hive for the most accurate system information. We can find out the current Control Set used by looking at the registry value of 
`SYSTEM\Select\Current`, we can also view the last known good configuration by looking at `SYSTEM\Select\LastKnownGood`

### Computer Name

Next up is the Computer Name which is located at `SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName`

### Time Zone Information

Next is the Time Zone information which is important for accuracy to establish what time zone the computer is located in as it will help us understand the chronology of the events that happened as some data will have their timestamps in UTC/GMT or in local timezones. This information is located at `SYSTEM\CurrentControlSet\Control\TimeZoneInformation`

### Network Interfaces and Past Networks

Next is the Network Interfaces which is found at `SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces`. Each interface has a unique identifier (GUID) which contains values relating to the interfaces TCP/IP configuration. This will give us the information regarding its IP address, DHCP IP address, Subnet Mask, DNS Servers, and etc. This information helps us be assured that we are performing forensics on the right machine.

Following that are the Past Networks found at either `SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged` or `SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed`. These Keys contain the information regarding the past networks and the last time they were connected. The last write time of the registry indicates when these networks were last connected.

### Autostart Programs

Next up are the Autostart Programs, which are the programs or commands that run when a user logs on. These can be found in the following directories.
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\RunOnce`
- `SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce`
- `SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer\Run`
- `SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

Meanwhile the following registry key holds the information about the services.
`SYSTEM\CurrentControlSet\Services`

In the Registry Keys Field, if start is 0x02 or 2, that means that the service will run.

### Secure Account Management (SAM) and User Information

Lastly is the Secure Account Management (SAM) Hive which is located at `SAM\Domains\Account\Users`. The information here contains the relative identifier (RID) of the user, how many times the user logged in, last login failed, last password change, password expiry, password policy and password hint, and any groups that the user is a part of. 
