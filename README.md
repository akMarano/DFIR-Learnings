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

### RegRipper

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

## Evidence of File Usage

### Recent Files

Windows keep track of recently opened files and this includes the time of when these files were last used. The list of these recently opened files can be found at `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`.

Registry Explorer allows us to sort this data using the given tabs. For example the Recent Documents tab shows the most recently used files at the top of the list. What's also interesting is that different file extensions have different keys. So for example if we were looking for recently used `.exe` files we can look at the registry key `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.pdf`

### Office Recent Files

Microsoft Office also keeps a list of recently opened documents. This can be found at `NTUSER.DAT\Software\Microsoft\Office\<version> `, the registry key is different for every version and application. For example Microsoft Word would use a key like so `NTUSER.DAT\Software\Microsoft\Office\15.0\Word`. In this case this refers to Version 15.0 which is Office 2013.

Starting from Office 365, Microsoft now uses the location tied to the user's live ID which is a Hexadecimal string of your Microsoft Account. This is located at the key `HKEY_CURRENT_USER\Software\Microsoft\Office\<version>\<Office App>\User MRU\LiveId_<Hexadecimal String>\`

### ShellBags

When opening a folder, it opens a specific layout, different folders have different layouts. This layout can be changed by the user according to their preferences. This information is called ShellBags and it can identify the most recently used files and folders. Since this information is unique for each user, it is found in the user hives. They are at the following keys.
- `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags`
- `USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU`
- `NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU`
- `NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags` 

Registry Explorer doesn't give much information about ShellBags, but another tool called ShellBag Explorer shows us the information in a friendly format. We just have to point to the hive we extracted and it'll read the data and show us the results.

### Recent Dialog MRUs (Most Recently Used)

A dialog box appears when we open or save a file, Windows remembers that location, meaning we can find recently used files using this information. These are found at the following keys

- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU `
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU`

### Windows Explorer Address and Search Bars

We can also identify the user's recent activity by looking at the paths typed in the Windows Explorer address bar or search bar. This is found at the keys
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths`
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery`

## Evidence of Execution

### UserAssist

Windows keeps track of applications used by the user, including what program was launched, when it was launched and how many times it was launched. However this doesn't include applications launched in the Command Line. The UserAssit information is stored in the User Assist Registry keys at the NTUSER hive, mapped to the user's GUID (Globally Unique Identifier). The key is at `NTUSER.DAT\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist\{GUID}\Count`

## ShimCache

ShimCache is also known as the Application Compatibility Cache (AppCompatCache), which keeps track of the application compatibility with the OS, and it keeps track of all applications launched, but its main purpose is to ensure backwards compatibility of apps. This is stored at the SYSTEM hive at key `SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache `. It stores the file name, file size, and last modified time of executables.

Registry Explorer doesn't have ShimCache data in readable format, so we use another tool called AppCompatCache Parser. It takes SYSTEM hive as input, reads all the data, and then outputs a CSV of the data, which we can view using EZviewer. We can use the command below to run the AppCompatCache Parser.
```
AppCompatCacheParser.exe --csv <path to save output> -f <path to SYSTEM hive for data parsing> -c <control set to parse>
```

## AmCache

AmCache hive is related to the ShimCache where it also stores data related to app launches, but it adds the execution path, installation, execution and deletion times, and SHA1(Secure Hash Algorithm 1) Hashes of the executed programs. This hive is located at `C:\Windows\appcompat\Programs\Amcache.hve`, and the information about the recently launched apps are found at `Amcache.hve\Root\File\{Volume GUID}\`

### BAM/DAM

Background Activity Monitor (BAM) keeps track of the background apps activity. Similarly, Desktop Activity Moderator (DAM), optimizes the power consumption of the device. Both also keep track of the fullpath of the launched apps, and both are part of the Modern Standby System in Microsoft Windows.

We can find these two at the following locations.
- `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}`
- `SYSTEM\CurrentControlSet\Services\dam\UserSettings\{SID}`

## Evidence in External Devices

When often need to check if there were any Removable Drives attached the the machine, as the information related to those devices are important.

### Device Identification

We can see the USB Keys plugged into the system along with their vendor id, product id, and version of the USB plugged in, which we can use the identify the devices in the following locations `SYSTEM\CurrentControlSet\Enum\USBSTOR`, and `SYSTEM\CurrentControlSet\Enum\USB`.

### First and Last Connection

We can also find when the device was the first and last time the device was connected into the system. This can be found at `SYSTEM\CurrentControlSet\Enum\USBSTOR\Ven_Prod_Version\<USB Serial Num>\Properties\{83da6326-97a6-4088-9453-a19231573b29}\####`

The `####` is replaced based on the information you want.

- **0064** = First Connection Time
- **0066** = Last Connection Time
- **0067** = Last Removal Time

### USB Device Volume Name

The name of the devices connected can be found at `SOFTWARE\Microsoft\Windows Portable Devices\Devices`. We can connect the GUID we see in the registry key and compare it with the Disk ID in the Device Identification to correlate the names with the unique devices.




