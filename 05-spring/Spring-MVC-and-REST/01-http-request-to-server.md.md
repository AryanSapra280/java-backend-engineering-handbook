# HTTP Request → Server

## Purpose

This document explains what happens to an HTTP request before it reaches
Spring MVC's `DispatcherServlet`.

The goal is to understand the request flow from:

Client → Network → Web Server / Servlet Container

This is the server-level part of the request lifecycle.

---

# 1. Big Picture

When a client calls a Spring Boot REST API:

```text
Client
   |
   | HTTP Request
   ↓
Network
   ↓
Server
   ↓
Tomcat
   ↓
DispatcherServlet