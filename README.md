# Managing SSL Certificates & Reverse Proxy Using Traefik

### What is an SSL Certificate?

SSL stands for **Secure Sockets Layer**.

An SSL certificate is used to **verify the identity of a website**.

When a computer connects to a website using SSL, the web browser asks the website to identify itself. The web server then sends a copy of its **SSL certificate**, which is a small digital certificate used to authenticate the website’s identity and confirm it is trustworthy.

The browser checks whether it trusts the certificate. If it does, it sends a message back to the server. The server responds with an acknowledgment.

Once this process is complete, a secure SSL session is established, and **encrypted data** can be exchanged safely between the browser and the web server.


---
### What is a Reverse Proxy?

A reverse proxy is used to **manage and regulate incoming traffic** to a server or service.

It sits in front of your backend servers and acts as an intermediary between clients and those servers.

A reverse proxy also **enhances security** by hiding the server’s IP address, blocking malicious traffic, and can perform **load balancing** to distribute traffic evenly across multiple servers.

---
###   Why SSL Certificates and a Reverse Proxy are Needed in a Home Lab?

In a home lab, services are usually accessed using IP addresses and ports, which is not secure and hard to remember.

#### Simple Example

Without:
```
http://192.168.1.10:8080
```

With:
```
https://app.yourdomain.com
```

#### SSL Certificates

- Encrypt data between your browser and services
- Keep passwords and data safe
- Enable secure **HTTPS access**
- Remove “Not Secure” warning in browsers

#### Reverse Proxy

- Use simple domain names instead of IP:PORT
- Route traffic to the correct service
- Hide backend servers for better security
- Manage all services from one place

#### Why Both?

- Secure access (**HTTPS**)
- Clean and easy URLs
- Better control and management


---
### What is Traefik?

Traefik is a **modern reverse proxy and load balancer** designed for microservices and containers.

It automatically:

- Routes traffic to the correct service
- Generates and manages **SSL certificates (Let’s Encrypt)**
- Detects new services (Docker, Kubernetes, etc.)

**Result:** Automatic, secure, and simple service exposure


Traefik = Reverse Proxy + Auto SSL + Smart Routing

---
### Prerequisites
* [[Self_Hosted_services/Docker_Installation|Docker]] installed system.
* [[/Self_Hosted_Services/DNS_Stack/DNS_With_Pihole_&_Unbound|PiHole]]
* Availed domain from domain provider (cloudflare) 
----
### Download docker-compose.yml

```
git clone 
```

----
### Update permissions on `acme.json`

```
cd traefik/config
chmod 600 acme.json
```

`acme.json` is used by Traefik to store and manage SSL certificates securely.

----
### Update docker-compose.yml



----
