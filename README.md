# Proxychains-setup

## Objective
A hands-on guide and configuration examples for using ProxyChains on Linux. This repo demonstrates how to route network traffic through multiple proxies, configure SOCKS5/HTTP proxies, and integrate ProxyChains with tools like PhoneInfoga and Nmap for ethical pentesting and OSINT.

### Skills Learned
- Network traffic routing and proxy chaining
- Anonymity and DNS leak prevention
- Linux system administration and configuration
- Integration with pentesting and reconnaissance tools

### Tools Used
- ProxyChains – For routing traffic through multiple proxies.
- Tor – Provides a reliable SOCKS5 proxy for anonymity.
- Linux Terminal / Bash – For configuration, file management, and running commands.
- Nano – For editing configuration files like proxychains.conf.

ProxyChains is a Linux tool that forces any network connection made by a program to go through one or more proxy servers without that program needing to support proxies itself.

It works by intercepting network traffic (via LD_PRELOAD) and redirecting it through proxies such as:

- SOCKS4 / SOCKS5 proxies
- HTTP or HTTPS proxies
- Tor (often used to anonymize traffic)

Main uses:

- Hide your IP address when connecting to websites or services.
- Chain multiple proxies together for added anonymity (e.g., Tor → SOCKS proxy → HTTP proxy).
- Bypass firewalls or geo-restrictions.
- Test how your application behaves when routed through proxies.

For Pentesting, the ideal order is;  
**VPN → ProxyChains → Target**

- Your ISP: Only sees encrypted traffic going to your VPN.
- VPN provider: Sees you connecting to proxies, not the final target.
- Target: Sees the last proxy in your ProxyChains chain.

Benefit:

- Your real IP is hidden from both the proxies and the target.
- If a proxy in your chain is compromised or logged, it only sees your VPN IP, not your real one.
- Easier to keep your identity hidden during recon/scanning.

### Configuring Proxychains
In Kali Linux, ProxyChains comes pre-installed. To locate its configuration file, run:  
`locate proxychains`  
The first file listed in the output is the configuration file.  
<img width="1004" height="294" alt="1" src="https://github.com/user-attachments/assets/7e539cef-5386-4289-ad23-8c5eaec1912c" />

Navigate into the file using `sudo nano /etc/proxychains4.conf` 
<img width="1003" height="1012" alt="2" src="https://github.com/user-attachments/assets/830a993e-bfc1-4196-bc34-0c63f9992679" />

In the config file we can find the following details;  
Chain Types (choose one)

- <ins>dynamic_chain</ins> – Uses proxies in the listed order, skips dead ones, needs at least one working proxy.
- <ins>strict_chain (active)</ins> – Uses proxies in listed order, fails if any are down.
- <ins>round_robin_chain</ins> – Cycles through proxies, starting where it left off, skips dead ones.
- <ins>random_chain</ins> – Picks random proxies or random sets (chain_len can limit set size).

DNS Handling

- <ins>proxy_dns (active)</ins> – Proxies DNS lookups to avoid leaks.
- <ins>proxy_dns_old</ins> – Slower, uses proxyresolv and dig. No .onion support.
- <ins>proxy_dns_daemon</ins> – Needs a running daemon, more stable for some software.
- <ins>remote_dns_subnet 224</ins> – Reserved subnet used to “fake” DNS IPs inside ProxyChains.

Timeouts

- tcp_read_time_out 15000 – Max wait for data from proxy: 15s.
- tcp_connect_time_out 8000 – Max wait to connect to proxy: 8s.

Localnet (bypass rules)

- You can exclude certain IP ranges or ports from being proxied.
- Useful for direct access to local or internal networks. (Currently off)

DNAT (Destination NAT)

- Lets you rewrite the destination IP/port before proxying.
- Good for testing or rerouting blocked addresses. (Currently off)

Proxy List  
Format: `type IP_address port \[username\] \[password\]`  
Default file: `socks4 127.0.0.1 9050`  
This means: use a SOCKS4 proxy on localhost port 9050 (Tor default).

We shall select `dynamic_chain` by uncommenting it and commenting out `strict_chain`
<img width="995" height="546" alt="3" src="https://github.com/user-attachments/assets/1bad0cc5-8eed-4850-9c87-d9b5586f1a17" />

Having the proxy_dns option active ensures that our DNS requests are routed through the proxies for additional caution
<img width="619" height="173" alt="4" src="https://github.com/user-attachments/assets/ee0b4a0c-86aa-413b-9dd7-0e42ca3aba54" />

Next we look at out available proxy servers
<img width="767" height="398" alt="5" src="https://github.com/user-attachments/assets/7969e4ce-cfdf-4c4a-9fc7-92108d994932" />

Below are the differences between SOCKS5 AND SOCKS4 where SOCKS5 is usually preffered
| Feature                | SOCKS4                              | SOCKS5                                         |
|------------------------|-------------------------------------|-----------------------------------------------|
| Protocol support       | TCP only                            | TCP and UDP                                   |
| DNS handling           | No DNS proxy (possible DNS leaks)   | Can proxy DNS requests (prevents DNS leaks)   |
| Authentication         | None                                | Supports username/password authentication     |
| IPv6 support           | No                                  | Yes                                           |
| Performance            | Slightly faster (less overhead)     | Slightly slower (more features)               |
| Security/anonymity     | Lower (DNS can leak)                | Higher (full traffic + DNS through proxy)     |
| Typical use cases      | Simple TCP tunneling                | Secure web browsing, VoIP, torrents, pentests |

Comment out the default SOCKS4 proxy and enter in a new proxy but first ensure Tor is installed and updated
<img width="1006" height="925" alt="6" src="https://github.com/user-attachments/assets/c76b9b3d-172f-4d99-938f-1b6842e291e2" />

Start the Tor service

<img width="1005" height="202" alt="7" src="https://github.com/user-attachments/assets/553b3e92-6809-49b4-a2db-6409727f01bd" />

Back in our proxy list we create a new SOCKS5 proxy
<img width="494" height="106" alt="8" src="https://github.com/user-attachments/assets/423a8bd8-8bb5-466d-b02c-e1d2dd8b29e8" />
Since we are using nano, we `Crtl + O` , press `Enter` and `Ctrl + X` to exit

To use proxychains we append the proxychains command before any command as shown in the example below
<img width="1004" height="1006" alt="9" src="https://github.com/user-attachments/assets/72ae73e0-5a58-4e59-ae09-ca0bac4ab773" />
A recommended 3-5 proxies can be chained for effective security 
