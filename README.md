Yes 👍 You need the **basic Linux/network troubleshooting commands** like `curl`, `nslookup`, `ping`, `telnet`, `ss`, etc., especially for checking **EC2 → ALB → Fargate → Flask**.

## 1. `curl` — Check HTTP application

```bash
curl http://example.com
```

Show detailed connection:

```bash
curl -v http://example.com
```

Check only HTTP headers:

```bash
curl -I http://example.com
```

Test your ALB:

```bash
curl -v http://fargat-1719301384.us-east-1.elb.amazonaws.com
```

Expected:

```
HTTP/1.1 200 OK
Hello from AWS EC2
```

**Use:** Is the web application responding?

---

## 2. `nslookup` — Check DNS

```bash
nslookup google.com
```

Your ALB:

```bash
nslookup fargat-1719301384.us-east-1.elb.amazonaws.com
```

Example:

```
Name:    fargat-1719301384.us-east-1.elb.amazonaws.com
Address: 44.216.205.215
Address: 100.57.52.142
```

**Use:** Does the domain name resolve to an IP?

---

## 3. `dig` — Detailed DNS check

If installed:

```bash
dig google.com
```

Short output:

```bash
dig +short google.com
```

ALB:

```bash
dig +short fargat-1719301384.us-east-1.elb.amazonaws.com
```

**Use:** More detailed DNS troubleshooting than `nslookup`.

---

## 4. `ping` — Basic network reachability

```bash
ping google.com
```

or:

```bash
ping 8.8.8.8
```

**Important:** Ping uses **ICMP**, not HTTP. AWS Security Groups may block ICMP, so a failed `ping` does **not necessarily mean the web server is down**.

---

## 5. `nc` / Netcat — Check a TCP port

Very useful for AWS troubleshooting.

```bash
nc -vz google.com 80
```

Check your ALB:

```bash
nc -vz fargat-1719301384.us-east-1.elb.amazonaws.com 80
```

Check Flask:

```bash
nc -vz 172.31.45.107 5000
```

Successful:

```
Connection to ... 80 port [tcp/http] succeeded!
```

**Use:** Is the TCP port reachable?

---

## 6. `telnet` — Check TCP port

```bash
telnet google.com 80
```

ALB:

```bash
telnet fargat-1719301384.us-east-1.elb.amazonaws.com 80
```

If `telnet` isn't installed, use `nc`.

---

## 7. `ss` — Check listening ports

On EC2:

```bash
ss -tulnp
```

Specific port:

```bash
ss -tulnp | grep 5000
```

Example:

```
LISTEN 0 128 0.0.0.0:5000
```

**Use:** Is an application actually listening on the port?

---

## 8. `netstat` — Older alternative

```bash
netstat -tulnp
```

If command doesn't exist, use:

```bash
ss -tulnp
```

---

## 9. `traceroute` — See network path

```bash
traceroute google.com
```

Sometimes you need:

```bash
tracepath google.com
```

**Use:** See the path packets take toward the destination.

---

## 10. `hostname` — Check machine name

```bash
hostname
```

Full information:

```bash
hostnamectl
```

---

## 11. `ip` — Check IP/network

Show IP addresses:

```bash
ip addr
```

Short:

```bash
ip -br addr
```

Show routes:

```bash
ip route
```

Example:

```
default via 172.31.0.1
```

**Use:** Check EC2 network configuration.

---

## 12. `curl` with different ports

This is especially useful for your Docker/Fargate learning.

```bash
curl http://localhost:5000
```

```bash
curl http://localhost:8080
```

```bash
curl http://ALB-DNS
```

With port:

```bash
curl http://ALB-DNS:801
```

Remember:

```
curl localhost:5000
       ↓
your local machine/application

curl ALB-DNS:80
       ↓
Load Balancer

curl ALB-DNS:801
       ↓
Load Balancer listener 801
```

---

# Most important troubleshooting flow

For your **ALB → Fargate → Flask** setup, remember this order:

```
1. DNS
   ↓
nslookup ALB-DNS

2. TCP
   ↓
nc -vz ALB-DNS 80

3. HTTP
   ↓
curl -v http://ALB-DNS

4. ALB
   ↓
Target health = healthy

5. Fargate
   ↓
Task = RUNNING

6. Container
   ↓
Port = 5000

7. Flask
   ↓
0.0.0.0:5000
```

### Quick cheat sheet

| Command | What it checks |
| --- | --- |
| `curl` | HTTP/application |
| `curl -v` | HTTP + connection details |
| `curl -I` | HTTP headers |
| `nslookup` | DNS |
| `dig` | Detailed DNS |
| `ping` | ICMP/network reachability |
| `nc -vz` | TCP port |
| `telnet` | TCP port |
| `ss -tulnp` | Listening ports |
| `ip addr` | IP addresses |
| `ip route` | Routing |
| `traceroute` | Network path |
| `hostname` | Hostname |

For DevOps, the **four I would memorize first** are:

```bash
nslookup
curl -v
nc -vz
ss -tulnp
```

They cover **DNS → HTTP → TCP → local listening port**, which is exactly the kind of troubleshooting you just did with your Fargate load balancer.
