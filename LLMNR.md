# LLMNR
### Link-Local Multicast Name Resolution

It is a protocol that allows computers to resolve the hostnames of neighboring devices on the same local network when a standard DNS server is unavailable or fails to respond. 

While convenient for ad-hoc networks or home use, LLMNR lacks authentication. Because of this, it is widely considered a significant security risk in corporate environments and is frequently disabled to prevent spoofing attacks.

---

## 1. How LLMNR is Configured

LLMNR is a "zero-configuration" protocol. It is typically enabled by default on Windows operating systems and some Linux distributions.

### Windows Settings (Group Policy)
In corporate environments, LLMNR is managed via the Local Group Policy Editor (`gpedit.msc`). The setting is found under:
`Computer Configuration > Administrative Templates > Network > DNS Client > Turn off multicast name resolution`

*   **Enabled:** Turns LLMNR *off* (Recommended for security).
*   **Disabled / Not Configured:** Leaves LLMNR *on* (Default Windows behavior).

### Linux Settings (`systemd-resolved`)
On Linux systems using `systemd-resolved`, LLMNR is configured in the `/etc/systemd/resolved.conf` file using the `LLMNR=` tag:
*   `LLMNR=yes`: Fully enabled (both resolves local names and answers incoming queries).
*   `LLMNR=resolve`: The system will use LLMNR to resolve names but will not answer queries from other machines.
*   `LLMNR=no`: Completely disabled.

---

## 2. How LLMNR Resolves a Name

LLMNR operates strictly on the local subnet (it does not cross routers). When a user or application tries to connect to a local device (e.g., typing `\\fileserver` into Windows Explorer), the resolution process flows like this:

1.  **Primary DNS Check:** The computer first asks the standard configured DNS server for the IP address of `fileserver`.
2.  **The Multicast Query:** If DNS fails or says the name doesn't exist, LLMNR steps in as the fallback. The computer sends a multicast query across the local network (via IPv4 at `224.0.0.252` or IPv6 at `FF02::1:3`, using UDP port 5355) asking everyone: *"Who has the IP address for 'fileserver'?"*
3.  **The Evaluation:** Every device on the local network running LLMNR receives this message and checks if the requested hostname matches its own.
4.  **The Response:** The device named `fileserver` sends a direct, unicast response back to the requester containing its IP address. 
5.  **The Connection:** The requesting computer receives the IP address and initiates the connection.

*Note on Security:* Because LLMNR does not validate or authenticate the responder, any malicious machine on the network can answer the multicast query, falsely claiming to be the requested server. This makes the protocol highly vulnerable to Man-in-the-Middle (MitM) and credential theft attacks.

---
## Sources :
- [IT-Connect](https://www.it-connect.fr/active-directory-comment-et-pourquoi-desactiver-les-llmnr-et-netbios/)
- [RFC4795](https://datatracker.ietf.org/doc/html/rfc4795)
