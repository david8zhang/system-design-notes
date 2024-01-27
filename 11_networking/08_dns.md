# DNS: The Domain Name System

The Domain Name System is responsible for turning domain names (like www.google.com) into IP addresses that computers can understand. It has two main components: DNS resolvers and DNS name servers, which work in a hierarchical, distributed fashion to resolve queries in a robust and performant manner.

Let's take a deeper look:

## DNS Resolvers and Nameservers

The **DNS resolver** is a server that's responsible for sending and coordinating further downstream requests to DNS nameservers. Resolvers are maintained by ISPs and popular DNS providers like Google (8.8.8.8) and CloudFlare (1.1.1.1).

**DNS Nameservers** are servers that are responsible for locating the domain's corresponding IP in a hierarchical fashion. There are 3 types of nameservers

1. Root Nameservers: the first server that the DNS resolver reaches out to, which mainly just stores the IP addresses of the TLD nameservers.

2. TLD (Top Level Domain) Nameserver: the next level of our nameserver hierarchy, which typically hosts the last portion of a given domain (.com, .net, .org) and stores the IP addresses of the respective authoritative nameservers.

3. Authoritative Nameserver: the final level of the hierarchy which provides an authoritative answer for a given domain query. This is where the full domain name mapping record is stored (e.g. google.com, amazon.com).

## How A Domain Gets Resolved

Let's take a look at an example where a user types "www.google.com" into their browser.

1. First, the browser checks its cache. If the result isn't there, it makes an operating system call which checks _its_ cache and reaches out to the DNS resolver.

2. If the result isn't cached at the DNS resolver level, the resolver reaches out to the Root Nameserver, which responds with a list of ".com" TLD nameserver IP addresses.

3. The DNS Resolver then reaches out to one of these ".com" TLD nameservers, which returns the IP address for the "google.com" authoritative nameserver.

4. Finally, the DNS Resolver gets the IP address for "google.com" from the authoritative nameserver and returns the result to the OS, which can then return that result to the browser.

## DNS Propagation

When we register a new domain or change a domain record, we need to wait for it to propagate across all the DNS nameservers. This can be slow since nameservers cache domain record information for a certain amount of time (TTL) before refreshing. Since these TTLs are configured on the records themselves, one way we can speed up DNS propagation is to decrease the TTL value one day before implementing a change.

There are various online tools for checking if DNS record changes have propagated globally, such as [Global DNS Checker](https://www.gdnspc.com/) or Google's [DNS Checker](https://dns.google/).
