### **Solution**

#### **Overview**
The task is to troubleshoot and resolve an issue where a virtual machine (VM) running Ubuntu 24.04 with 64GB of storage is consistently using 99% of its memory. The VM is responsible for running a single service: an NGINX load balancer that routes traffic to upstream services. This document outlines the steps to identify the root cause, analyze potential scenarios, and propose recovery actions.

---

#### **Troubleshooting Steps**

##### **Step 1: Verify the Issue**
Before proceeding with root cause analysis, confirm the reported issue:
1. **Check Memory Usage**:
   Use tools such as `top`, `htop`, or `free -m` to verify memory consumption.
   ```bash
   free -m
   top
   ```

2. **Inspect System Logs**:
   Examine system logs (`/var/log/syslog`) and NGINX logs (`/var/log/nginx/access.log` and `/var/log/nginx/error.log`) for anomalies.
   ```bash
   sudo tail -n 100 /var/log/syslog
   sudo tail -n 100 /var/log/nginx/error.log
   ```

---

##### **Step 2: Analyze Potential Causes**
Based on the scenario, the following are potential causes of high memory usage:

###### **1. Misconfigured NGINX**
- **Cause**: Incorrect settings in NGINX configuration (e.g., `worker_processes`, `worker_connections`) can lead to excessive resource consumption.
- **Impact**: Reduced performance, increased latency, or service unavailability.
- **Recovery Steps**:
  1. Validate NGINX configuration:
     ```bash
     sudo nginx -t
     ```
  2. Adjust `worker_processes` and `worker_connections` in `/etc/nginx/nginx.conf`:
     ```nginx
     worker_processes auto;
     events {
         worker_connections 1024;
     }
     ```
  3. Restart NGINX:
     ```bash
     sudo systemctl restart nginx
     ```

###### **2. DDoS Attack**
- **Cause**: A Distributed Denial of Service (DDoS) attack floods the server with malicious requests, exhausting memory resources.
- **Impact**: Complete service downtime or degraded performance for legitimate users.
- **Recovery Steps**:
  1. Identify suspicious IPs using `netstat` or `ss`:
     ```bash
     sudo netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr
     ```
  2. Block malicious IPs using `iptables` or `ufw`:
     ```bash
     sudo ufw deny from <malicious-ip>
     ```
  3. Enable rate limiting in NGINX:
     ```nginx
     limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
     server {
         location / {
             limit_req zone=one burst=5 nodelay;
         }
     }
     ```
  4. Contact your cloud provider for additional DDoS protection.

###### **3. Memory Leak**
- **Cause**: A bug in NGINX or a related module may cause memory to be consumed without being released.
- **Impact**: Gradual increase in memory usage until the system becomes unresponsive.
- **Recovery Steps**:
  1. Check the NGINX version:
     ```bash
     nginx -v
     ```
  2. Update NGINX to the latest stable version if a memory leak is reported in the current version:
     ```bash
     sudo apt update
     sudo apt install nginx
     ```
  3. Monitor memory usage of processes using `top` or `htop` to identify leaks:
     ```bash
     top
     ```
  4. Restart NGINX if necessary:
     ```bash
     sudo systemctl restart nginx
     ```

###### **4. Traffic Spike**
- **Cause**: A sudden increase in legitimate traffic due to marketing campaigns or unexpected user demand.
- **Impact**: Overloaded server, leading to slow responses or timeouts.
- **Recovery Steps**:
  1. Scale up the VM by increasing its memory allocation (if supported by your cloud provider).
  2. Implement caching in NGINX to reduce load:
     ```nginx
     proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m use_temp_path=off;
     server {
         location / {
             proxy_cache my_cache;
             proxy_pass http://upstream;
         }
     }
     ```
  3. Add more NGINX instances behind a load balancer to distribute traffic.

---

#### **Conclusion**
High memory usage on the NGINX load balancer VM can stem from various causes, including misconfigurations, DDoS attacks, memory leaks, or traffic spikes. By systematically verifying the issue, analyzing logs, and applying targeted recovery steps, the problem can be resolved effectively. Additionally, implementing proactive measures such as rate limiting, caching, and monitoring will help prevent similar issues in the future.

---

#### **Final Notes**
- Always back up configurations before making changes.
- Test changes in a staging environment if possible before applying them to production.
- Regularly monitor system performance to detect anomalies early.