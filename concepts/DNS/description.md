# Domain Name System (DNS)

The Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities. Most prominently, it translates more readily memorized domain names to the numerical IP addresses needed for locating and identifying computer services and devices with the underlying network protocols. By providing a worldwide, distributed directory service, the Domain Name System has been an essential component of the functionality of the Internet since 1985.

## Hierarchical Structure

1. Root Level (.)

    - Top of the DNS hierarchy.
    - Managed by IANA (Internet Assigned Numbers Authority).
    - Contains pointers to the Top-Level Domain (TLD) name servers.

2. Top-Level Domain (TLD)

    - Examples: .com, .org, .net, .gov, .edu, country codes like .uk, .jp
    - Managed by registries.
    - Contains pointers to the Second-Level Domain (SLD) authoritative name servers.

3. Second-Level Domain (SLD)

    - Examples: example.com, google.com
    - Owned by individuals or organizations.
    - Contains pointers to the subdomains and/or specific DNS records for the domain.

4. Subdomains

    - Examples: www.example.com, mail.example.com
    - Can be created under any SLD.
    - Points to specific services or machines within the domain.

5. DNS Records

    - A Record : Maps a domain name to an IPv4 address.
    - AAAA Record : Maps a domain name to an IPv6 address.
    - CNAME Record: Maps a domain name to another domain name (aliasing).
    - MX Record : Specifies mail servers for email handling.
    - NS Record : Specifies authoritative name servers for a domain.
    - PTR Record : Maps an IP address back to a domain name (reverse DNS lookup).
    - TXT Record : Contains text information, often used for verification and security purposes.

6. DNS Resolvers

    - DNS clients that initiate queries.
    - Typically provided by ISPs or configured on individual machines.
    - Caches DNS queries to speed up future requests.

7. Root Name Servers

    - There are 13 root name servers worldwide. Managed by various organizations, including ICANN and Verisign. Named with letters from A to M.

        | Letter | IPv4 Address      | IPv6 Address        | domain             | AS-number                                                     | Old Name         | Operator                           | Operator Origin | Location & No. of Sites (Global/Local) | Software               |
        | ------ | ----------------- | ------------------- | ------------------ | ------------------------------------------------------------- | ---------------- | ---------------------------------- | --------------- | -------------------------------------- | ---------------------- |
        | A      | 198.41.0.4        | 2001:503:ba3e::2:30 | a.root-servers.org | AS19836, AS36619, AS36620, AS36622, AS36625, AS36631, AS64820 | ns.internic.net  | Verisign                           | United States   | Distributed using anycast 14/2         | NSD and Verisign ATLAS |
        | B      | 170.247.170.2[12] | 2801:1b8:10::b[12]  | b.root-servers.org | AS394353                                                      | ns1.isi.edu      | USC-ISI                            | United States   | Distributed using anycast 6/0          | BIND and Knot DNS      |
        | C      | 192.33.4.12       | 2001:500:2::c       | c.root-servers.org | AS2149                                                        | c.psi.net        | Cogent Communications              | United States   | Distributed using anycast 10/0         | BIND                   |
        | D      | 199.7.91.13       | 2001:500:2d::d      | d.root-servers.org | AS10886                                                       | terp.umd.edu     | University of Maryland             | United States   | Distributed using anycast 22/127       | NSD                    |
        | E      | 192.203.230.10    | 2001:500:a8::e      | e.root-servers.org | AS21556                                                       | ns.nasa.gov      | NASA Ames Research Center          | United States   | Distributed using anycast 117/137      | BIND and NSD           |
        | F      | 192.5.5.241       | 2001:500:2f::f      | f.root-servers.org | AS3557                                                        | ns.isc.org       | Internet Systems Consortium        | United States   | Distributed using anycast 119/119      | BIND                   |
        | G      | 192.112.36.4      | 2001:500:12::d0d    | g.root-servers.org | AS5927                                                        | ns.nic.ddn.mil   | Defense Information Systems Agency | United States   | Distributed using anycast 6/0          | BIND                   |
        | H      | 198.97.190.53     | 2001:500:1::53      | h.root-servers.org | AS1508                                                        | aos.arl.army.mil | U.S. Army Research Lab             | United States   | Distributed using anycast 8/0          | NSD                    |
        | I      | 192.36.148.17     | 2001:7fe::53        | i.root-servers.org | AS29216                                                       | nic.nordu.net    | Netnod                             | Sweden          | Distributed using anycast 63/2         | BIND                   |
        | J      | 192.58.128.30     | 2001:503:c27::2:30  | j.root-servers.org | AS26415, AS36626, AS36628, AS36632                            | —                | Verisign                           | United States   | Distributed using anycast 63/55        | NSD and Verisign ATLAS |
        | K      | 193.0.14.129      | 2001:7fd::1         | k.root-servers.org | AS25152                                                       | —                | RIPE NCC                           | Netherlands     | Distributed using anycast 70/3         | BIND, NSD and Knot DNS |
        | L      | 199.7.83.42       | 2001:500:9f::42     | l.root-servers.org | AS20144                                                       | —                | ICANN                              | United States   | Distributed using anycast 165/0        | NSD and Knot DNS       |
        | M      | 202.12.27.33      | 2001:dc3::35        | m.root-servers.org | AS7500                                                        | —                | WIDE Project                       | Japan           | Distributed using anycast 4/1          | BIND                   |

8. TLD Name Servers

    - Manage the top-level domains and direct queries to the appropriate SLD authoritative name servers.

9. Authoritative Name Servers

    - Provide the definitive answer for DNS queries about a specific domain name.

10. Recursive vs. Iterative Queries

    - Recursive Query: The DNS resolver queries other servers on behalf of the client until it finds the IP address.
    - Iterative Query: The DNS resolver returns the best answer it can, directing the client to another DNS server if necessary.

11. Caching

    - DNS responses are cached to improve performance.
    - Time-To-Live (TTL) determines how long a DNS record is cached.

12. Anycast Routing

    - Multiple servers share the same IP address, with traffic routed to the nearest server.
    - Enhances redundancy and reduces latency.

13. DNS Load Balancing

    - Distributes traffic across multiple servers by returning different IP addresses for the same domain in different responses.

14. DNSSEC (DNS Security Extensions)

    - Adds security to DNS, enabling DNS responses to be verified using digital signatures.

15. Common DNS Tools:

    - nslookup:

        - A command-line tool for querying DNS servers.
        - Deprecated in favor of `dig`.
        - Still available on many systems.
        - Example: `nslookup google.com`

    - dig:

        - Detailed DNS queries and troubleshooting.
        - More powerful than `nslookup`.
        - Example: `dig google.com`

    - host:

        - Simple DNS lookups.
        - Example: `host google.com`

    - whois:

        - Retrieves domain registration details.
        - Example: `whois google.com`

    - ping:

        - Tests network connectivity and latency.
        - Example: `ping google.com`

    - traceroute:

        - Shows the path taken by packets to reach a host.
        - Example: `traceroute google.com`

    - ipconfig / ifconfig:

        - Displays network configurations.
        - Example: `ipconfig` on Windows or `ifconfig` on Linux.

    - resolvectl:

        - Interacts with systemd-resolved service on Linux.
        - Example: `resolvectl query google.com`

## DNS Resolution Process

1. User types a domain name (e.g., www.google.com) into a web browser.

2. Browser checks its cache for the corresponding IP address.

    - If found, the browser sends an HTTP request to the web server.
    - If not found, the browser queries the operating system.

3. The operating system checks its cache and the local hosts file.

    - If the IP address is found, the operating system returns it to the browser.
    - If the IP address is not found, then the operating system queries the DNS resolver.

4. The DNS resolver checks its cache for the IP address.

    - If found, it returns the IP address to the operating system.
    - If not found, the DNS resolver performs the following steps:
        - Randomly selects a root name server from the list of 13 root name servers.
        - Anycast routing is used to direct the query to the nearest root name server.
        - Asks the root name server for the TLD name server that manages the top-level domain of the domain name.
        - The root name server responds with the IP address of the TLD name server.

5. The root name servers direct the DNS resolver to the TLD name servers.

    - The DNS resolver queries the TLD name server for the authoritative name server of the domain name.
    - The TLD name server responds with the IP address of the authoritative name server.

6. The TLD name servers direct the DNS resolver to the authoritative name servers.

    - The DNS resolver queries the authoritative name server for the IP address of the domain name.
    - The authoritative name server responds with the IP address.

7. The authoritative name servers provide the IP address to the DNS resolver.

8. The DNS resolver caches the IP address and returns it to the operating system.

9. The operating system caches the IP address and returns it to the browser.

10. The browser caches the IP address and sends an HTTP request to the web server.

11. The web server processes the request and sends an HTTP response to the browser.
