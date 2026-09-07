# DFIR Learnings 

Digital Forensics and Incident Response (DFIR) is the process of gathering digital evidence left after an attack and, detecting and responding to an in-progress cyberattack. These two disciplines come together to stop threats and analyze evidence to further improve security.

Digital Forensics are the ones responsible for identifying forensic artifacts or evidence of human activity in digital devices. Incident Response are the ones that use that forensic information to identify the activity of interest and take appropriate action.

## Artifacts

Artifacts are the pieces of evidence left by activities performed on a device. We collect artifacts when performing DFIR to figure out the attackers activity. For example, if we suspect that an attacker used Windows Registry keys to maintain persistence (When malware keeps running on a system even after restart) on a system, we can analyze and collect artifacts from the registry keys to support this. Artifacts can be collected from the Endpoint or Server's file system, memory, or network activity. 

Most enterprise environments mainly consist of Windows and Linux Operating Systems. Windows systems are primarily used for endpoints and server use cases. Enterprises primarily use Linux systems for server hosting like web servers or database servers.

## Evidence Preservation

In DFIR, it is important to maintain the integrity of evidence when we are collecting it, there are established best practices in the industry to do so. Note that any forensic analysis contaminates the evidence, therefore, the evidence must first be collected and write protected so it cannot be modified. A copy of the evidence is then used for analysis, this is done to ensure the integrity of the original evidence.

## Order of Volatility

Digital evidence is often volatile, meaning it can be easily lost if not captured in time. For example, data in the systems memory (RAM) will be lost once the computer is shut down since this data is only kept while the computer is powered on. Some sources are more volatile than others, for example, hard drives are persistent storage devices, so the data is kept even when the computer is powered off, therefore it is less volatile than ram. It is important to understand the order of volatility of different evidence types in order not to lose any artifacts when investigating.

## Timeline Creation

This is the process of putting together the collected artifacts in order to create a timeline of the activities in chronological order for efficient and accurate analysis as it helps create a story of how things happened.









