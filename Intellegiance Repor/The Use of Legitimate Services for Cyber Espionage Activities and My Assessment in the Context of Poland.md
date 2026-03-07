 While working on this TryHackMe room, one detail caught my attention. Normally there are many records to follow, but in this room there were only two logs that needed analysis.
 What was suspicious here was not the website the log connected to, but how it connected. 
The connection had a User-Agent set to bitsadmin.
 Bitsadmin is a legitimate tool, but with a bit of research, it is easy to see how this legitimate tool can provide stealth for threat actors. 

What I observed in the logs was:
A file downloaded over HTTPS using a legitimate Windows binary (bitsadmin)

The User-Agent is not a browser, but a system utility
A legitimate platform such as Pastebin is used as C2

Open-source threat intelligence shows that this technique has been repeatedly used by state-sponsored actors.

The common problems of state-sponsored actors are:
They need to stay in the network for a long time
Noisy tools (custom malware downloaders) get detected early
Their own C2 infrastructure is easy to track

At this point, whether the actor is from China, Russia, or Iran does not really matter. This is an operational need. Because of this, in systems that contain sensitive data, legitimate tools that are able to communicate externally should be monitored using behavior-focused detection. This is especially important for government institutions, energy, finance, military organizations, and archives.
China-linked APT groups (especially groups like APT41) are generally:
Patient
Focused on long-term access
Aiming for both data theft and positional advantage

Because of this, they prefer:
“Download fast and execute”
Instead of “blend in with normal user behavior”

bitsadmin + pastebin fits this profile very well.

In the Context of Poland:
When it comes to espionage activities and state-sponsored threats, Poland’s position makes these kinds of low-profile cyber espionage techniques more relevant. Being a NATO member, located on Europe’s eastern border, and having strategic importance in military, logistics, and diplomatic processes increases the value of its information assets.
In this context, the risk is not destructive attacks, but long-term data collection activities that remain unnoticed. The threat here involves:
Systems that can communicate with external networks

Activity that appears legitimate and does not attract attention

Abuse of common and legitimate tools

In particular, defense supply chains, energy infrastructure (which has previously been targeted by both cyber and physical attacks), digital archives of public institutions, and systems related to international logistics processes are more sensitive to these types of threats.
As a result, systems that contain strategic information and communicate with external networks should be monitored using behavior-based analysis to detect out-of-context use of legitimate tools.
