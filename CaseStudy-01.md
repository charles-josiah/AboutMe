# Case Study: Resolving Port Conflicts and Optimizing Docker Deployments

## Overview  
A client was struggling with a Dockerized Nginx application that failed to start on a Linux production server. The project required deep-dive diagnostics into network bindings, configuration fixes, and implementing best practices for container stability.

---

## 1. The Problem: Port Contention & Silent Failures  

The container was stuck in an `Exited (1)` state. Upon inspection of the container logs, I identified a critical binding error:

```
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
```

**Impact:**  
- The web application was inaccessible  
- Service downtime affecting availability  

---

## 2. The Solution: Technical Diagnosis & Reconfiguration  

I followed a structured troubleshooting workflow to resolve the conflict without disrupting other host services:

### Network Auditing  
Used system tools to identify which process was using port 80:

```bash
ss -tulnp | grep :80
```
Output:
```bash
tcp   LISTEN 0      128    0.0.0.0:80      0.0.0.0:*    users:(("nginx",pid=1234,fd=6))
```
### Root Cause Analysis

A local Nginx process was already running on the host and actively listening on port 80.
This caused a port binding conflict, preventing the Docker container from starting.

### Decision & Fix

Instead of stopping the existing service (which could impact other applications), I opted for a safer and non-disruptive approach:

Kept the host service running on port 80
Remapped the container to use an alternative host port

### Deployment Command  

```bash
docker run -d -p 8080:80 --name web_app --restart always nginx
```

*Note: The `--restart always` policy was added to ensure high availability and automatic recovery after failures.*

---

## 3. The Result: A Stable, Production-Ready Environment  

### Successful Deployment  
The container transitioned to a healthy running state:

```
0.0.0.0:8080->80/tcp
```

### Verified Accessibility  
Confirmed service availability using:

```bash
curl http://localhost:8080
```

Received a valid `200 OK` response with application content.

### Reliability Improvements  
- Automatic restart policies implemented  
- Stable container lifecycle  
- No further port conflicts  

---

## Key Skills Applied  

- **Docker & Containerization:** Configuration, lifecycle management, logging  
- **Linux Administration:** Network troubleshooting (`ss`, `netstat`), process analysis  
- **DevOps Best Practices:** Port mapping, high availability, production stability  

---

## Conclusion  

This project demonstrates the importance of structured troubleshooting and pragmatic decision-making in production environments. Instead of applying disruptive fixes, the solution ensured stability, continuity, and long-term reliability.
