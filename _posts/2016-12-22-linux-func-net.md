---
layout: post
title: "Linux 网络编程常用函数"
subtitle: "网络编程常用函数"
date: 2016-12-22 09:50:00
author: "seventhking"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - 网络编程
---

# Host Address Functions
These additional functions for manipulating Internet addresses are declared in the header file arpa/inet.h. They represent Internet addresses in network byte order, and network numbers and local-address-within-network numbers in host byte order. See Byte Order, for an explanation of network and host byte order.

## int inet_aton (const char *name, struct in_addr *addr)
Preliminary: | MT-Safe locale | AS-Safe | AC-Safe | See POSIX Safety Concepts.

This function converts the IPv4 Internet host address name from the standard numbers-and-dots notation into binary data and stores it in the struct in_addr that addr points to. inet_aton returns nonzero if the address is valid, zero if not.

## uint32_t inet_addr (const char *name)
Preliminary: | MT-Safe locale | AS-Safe | AC-Safe | See POSIX Safety Concepts.

**过时了，被inet_aton取代**

## char * inet_ntoa (struct in_addr addr)
Preliminary: | MT-Safe locale | AS-Unsafe race | AC-Safe | See POSIX Safety Concepts.

This function converts the IPv4 Internet host address addr to a string in the standard numbers-and-dots notation. The return value is a pointer into a statically-allocated buffer. Subsequent calls will overwrite the same buffer, so you should copy the string if you need to save it.

In multi-threaded programs each thread has an own statically-allocated buffer. But still subsequent calls of inet_ntoa in the same thread will overwrite the result of the last call.

**过时，被inet_ntop取代**

## int inet_pton (int af, const char *cp, void *buf)
Preliminary: | MT-Safe locale | AS-Safe | AC-Safe | See POSIX Safety Concepts.

This function converts an Internet address (either IPv4 or IPv6) from presentation (textual) to network (binary) format. af should be either AF_INET or AF_INET6, as appropriate for the type of address being converted. cp is a pointer to the input string, and buf is a pointer to a buffer for the result. It is the caller’s responsibility to make sure the buffer is large enough.

## const char * inet_ntop (int af, const void *cp, char *buf, socklen_t len)
Preliminary: | MT-Safe locale | AS-Safe | AC-Safe | See POSIX Safety Concepts.

This function converts an Internet address (either IPv4 or IPv6) from network (binary) to presentation (textual) form. af should be either AF_INET or AF_INET6, as appropriate. cp is a pointer to the address to be converted. buf should be a pointer to a buffer to hold the result, and len is the length of this buffer. The return value from the function will be this buffer address.


# Host Names
~~~
Data type: struct hostent

This data type is used to represent an entry in the hosts database. It has the following members:

char *h_name
    This is the “official” name of the host.

char **h_aliases
    These are alternative names for the host, represented as a null-terminated vector of strings.

int h_addrtype
    This is the host address type; in practice, its value is always either AF_INET or AF_INET6, 
    with the latter being used for IPv6 hosts. In principle other kinds of addresses could be 
    represented in the database as well as Internet addresses; if this were done, you might 
    find a value in this field other than AF_INET or AF_INET6. See Socket Addresses.

int h_length
    This is the length, in bytes, of each address.

char **h_addr_list
    This is the vector of addresses for the host. (Recall that the host might be connected to
    multiple networks and have different addresses on each one.) The vector is terminated by
    a null pointer.

char *h_addr
    This is a synonym for h_addr_list[0]; in other words, it is the first host address.
~~~

## struct hostent * gethostbyname (const char *name)
Preliminary: | MT-Unsafe race:hostbyname env locale | AS-Unsafe dlopen plugin corrupt heap lock | AC-Unsafe lock corrupt mem fd | See POSIX Safety Concepts.

The gethostbyname function returns information about the host named name. If the lookup fails, it returns a null pointer.

## struct hostent * gethostbyname2 (const char *name, int af)
Preliminary: | MT-Unsafe race:hostbyname2 env locale | AS-Unsafe dlopen plugin corrupt heap lock | AC-Unsafe lock corrupt mem fd | See POSIX Safety Concepts.

The gethostbyname2 function is like gethostbyname, but allows the caller to specify the desired address family (e.g. AF_INET or AF_INET6) of the result.

## struct hostent * gethostbyaddr (const void *addr, socklen_t length, int format)
Preliminary: | MT-Unsafe race:hostbyaddr env locale | AS-Unsafe dlopen plugin corrupt heap lock | AC-Unsafe lock corrupt mem fd | See POSIX Safety Concepts.

The gethostbyaddr function returns information about the host with Internet address addr. The parameter addr is not really a pointer to char - it can be a pointer to an IPv4 or an IPv6 address. The length argument is the size (in bytes) of the address at addr. format specifies the address format; for an IPv4 Internet address, specify a value of AF_INET; for an IPv6 Internet address, use AF_INET6.

If the lookup fails, gethostbyaddr returns a null pointer.

# Byte Irder Conversion

## uint16_t htons (uint16_t hostshort)
Preliminary: | MT-Safe | AS-Safe | AC-Safe | See POSIX Safety Concepts.

This function converts the uint16_t integer hostshort from host byte order to network byte order.

## uint16_t ntohs (uint16_t netshort)
Preliminary: | MT-Safe | AS-Safe | AC-Safe | See POSIX Safety Concepts.

This function converts the uint16_t integer netshort from network byte order to host byte order.

## uint32_t htonl (uint32_t hostlong)
Preliminary: | MT-Safe | AS-Safe | AC-Safe | See POSIX Safety Concepts.

This function converts the uint32_t integer hostlong from host byte order to network byte order.

This is used for IPv4 Internet addresses.

## uint32_t ntohl (uint32_t netlong)
Preliminary: | MT-Safe | AS-Safe | AC-Safe | See POSIX Safety Concepts.

This function converts the uint32_t integer netlong from network byte order to host byte order.

This is used for IPv4 Internet addresses.
