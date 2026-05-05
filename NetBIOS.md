# NetBIOS
### Network Basic Input/Output System

**NetBIOS**  is a legacy networking API created in the 1980s that allows applications on separate computers to communicate over a local area network (LAN). In modern environments, it operates over TCP/IP—a combination known as **NBT** (NetBIOS over TCP/IP).

A core component of NBT is the **NetBIOS Name Service (NBT-NS)**. Much like LLMNR, NBT-NS is a fallback name resolution protocol used to find devices on a local network when standard DNS fails. Because it was designed in an era before network security was a primary concern, it completely lacks authentication, making it a major vulnerability in modern corporate environments.

---

## 1. How NetBIOS is Configured

Despite its age, NetBIOS is often still enabled by default on Windows operating systems to ensure backward compatibility with older hardware and legacy software. 

### Windows Settings (Network Adapter)
NetBIOS is typically managed per network interface rather than via a global Group Policy. To configure it:
1. Open the **Network Connections** panel and open the properties of your network adapter.
2. Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
3. Click **Advanced**, then navigate to the **WINS** tab.

Under the NetBIOS setting, you will find three options:
*   **Default:** Uses the NetBIOS setting provided by the DHCP server.
*   **Enable NetBIOS over TCP/IP:** Forces the protocol on.
*   **Disable NetBIOS over TCP/IP:** Turns the protocol off (Highly recommended for modern, secure networks).

### DHCP Server Settings
Network administrators can also push a configuration via the corporate DHCP server (DHCP Option 43) to disable NetBIOS globally across all endpoints.

---

## 2. How NetBIOS Resolves a Name

When a user tries to access a local network resource (like a shared folder or printer), Windows follows a specific resolution order. If the standard DNS query fails, and LLMNR (if enabled) also fails, the system falls back to NBT-NS. 

Here is how the NetBIOS name resolution process works:

1.  **The Local Cache:** The computer first checks its internal NetBIOS name cache and the local `LMHOSTS` file to see if it already knows the IP address.
2.  **The Broadcast Query:** If the name isn't cached, the computer sends an NBT-NS broadcast message across the local subnet (using UDP port 137). It essentially shouts: *"Is anyone here named 'FILESERVER'?"*
3.  **The Evaluation:** Every device on the local network receives this broadcast and checks its own NetBIOS name.
4.  **The Response:** If a device matches the requested name, it sends a unicast reply back to the requester with its IP address, allowing the connection to happen.

---

## 3. The Security Risk: NBT-NS Poisoning Attack Scenario

Because NetBIOS broadcasts are unauthenticated, any device on the network can answer the query. This makes NBT-NS susceptible to the exact same **Poisoning/Spoofing attacks** as LLMNR. In fact, attackers usually exploit both protocols simultaneously using tools like `Responder`.

Here is how an attacker exploits NetBIOS:

1.  **The Typo:** A user intends to access a server named `\\BACKUP` but types `\\BAKCUP`.
2.  **The Fallback:** The DNS server cannot resolve the misspelled name. The machine broadcasts an NBT-NS query to the entire local network asking for the IP of `BAKCUP`.
3.  **The Spoofed Response:** The attacker’s machine, listening for these broadcasts, instantly replies: *"I am 'BAKCUP', connect to me."*
4.  **The Credential Theft:** The victim's computer blindly trusts the response and tries to authenticate with the attacker's machine using Windows Single Sign-On. It hands over the user's NTLMv2 password hash.
5.  **The Exploit:** The attacker captures the hash and can either crack it offline to reveal the plaintext password or use an "NTLM Relay" attack to pass the hash to a legitimate server, granting the attacker instant access as the victimized user.
