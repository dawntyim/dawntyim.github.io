---
title: "Http Connection Management"
date: 2023-01-09T23:00:54-08:00
draft: true
author: "Dawn Yim"
summary: "HTTP Connections and management tips"
tags: ""
hideMeta: false
searchHidden: false
ShowBreadCrumbs: false
---

# HTTP connection management

Tags: .NET, HTTP

## HTTP Connection Management

### HTTP/1.0

**Short-lived connection** is the HTTP/1.0 default connection. Each HTTP request entails opening and closing up a connection between a client and a server. HTTP depends on TCP protocol and opening up a TCP connection is resource consuming operation because it requires handshake process every time. 

### HTTP/1.1

The new protocol introduces 2 connection types, persistent connection and HTTP Pipelining. In persistent connection, a server keeps TCP connections open for some time after sending response. Then new requests can be made without opening up new connections. HTTP pipelining further develop the idea by sending multiple requests before receiving any response. This enables skipping waiting for RTT (round-trip time).

## Ports, Sockets

### Ports

The reason ports are used is to tell one stream of data from another. For example, having one port for file transfer and another port for HTTP coming from the same endpoint makes it easier for the computer to process the data respectively. 

### Socket

A socket is just a combination of port and IP address. 

### Ephemeral Ports Consumption

One thing you need to keep in mind when building a large-sclae application is to manage ports. Let’s say there’s a feature where you need to send HTTP requests to the same end point multiple times in a short period of time, 

If you create an HTTP client every time you send a HTTP request to an endpoint, it will open up an outbound port by default. To save up outbound ports as much as possible, you need to use the same HTTP client across requests that share the same endpoint unless it’s restricted for some reason. 

Managing HTTP connections is one of the major responsibilities of Apache or Nginx web servers are doing. 

### C# Implementation

Most of the programming languages have libraries to open and manage HTTP clients which send and receive HTTP requests. For example, in .NET,
