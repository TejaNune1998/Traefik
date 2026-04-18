# Managing SSL Certificates & Reverse Proxy Using Traefik

### What is an SSL Certificate?

SSL stands for **Secure Sockets Layer**, but today websites actually use **TLS (Transport Layer Security)**, which is its modern and secure version.

An SSL certificate helps **prove that a website is genuine** and enables secure communication between your browser and the server.

When you visit a website using HTTPS, the server shows its certificate. Your browser checks if it’s valid and trusted.

If everything looks good, a secure connection is created, and all data exchanged (like passwords or personal data) is **encrypted and protected**.

---
### What is a Reverse Proxy?

A reverse proxy sits in front of your applications and **handles all incoming requests**.

Instead of users connecting directly to your services, they connect to the reverse proxy, which then forwards the request to the correct backend service.

It helps to:

- Route traffic to the right service
- Hide internal services from direct access
- Balance traffic across multiple servers
- Improve security and control

---
### Why SSL Certificates and a Reverse Proxy are Needed in a Home Lab?

In a home lab, services are usually accessed using IP addresses and ports, which is inconvenient and often less secure.

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
- Redirect HTTP traffic to HTTPS, ensuring all connections are secure.
- Generates and manages TLS certificates using providers like Let’s Encrypt
- Detects new services (Docker, Kubernetes, etc.)

**Result:** Automatic, secure, and simple service exposure

Traefik = Reverse Proxy + Automatic TLS + Dynamic Routing

---
### Prerequisites
* A system with Docker installed.
* Pi-Hole
* A registered domain from a provider (e.g., Cloudflare)
----
### Download docker-compose.yml

```
git clone https://github.com/TejaNune1998/Traefik.git
```

----------------
### Directory Structure 

```
/opt/docker_files/traefik
├── cf_api_token.txt
├── config
│   ├── acme.json
│   ├── config.yml
│   └── traefik.yml
└── docker-compose.yml
```

----
### Create Docker Network for Traefik

```
docker network create proxy
```

---
### Update permissions on `acme.json`

```
cd traefik/config
chmod 600 acme.json
```

`acme.json` is used by Traefik to store and manage SSL certificates securely.

----
### Update `docker-compose.yml` file

#### DNS

```
# Update DNS servers according to your network configuration
dns:
- 10.1.0.16
- 10.1.0.17
```

#### Labels

```
- "traefik.http.routers.traefik.rule=Host(`traefik.yourdomain.com`)"
- "traefik.http.routers.traefik-secure.tls.domains[0].main=yourdomain.com"
- "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.yourdomain.com"
- "traefik.http.routers.traefik-secure.tls.domains[1].main=traefik.miniverse.com.co"  
```

----

### Update `.env` file

Update Traefik Dashboard Admin Password in .env file  

* Install apache2-utils and Encrypt the password

```
sudo apt install apache2-utils -y
htpasswd -nb admin your_password
```

--------------------------
### Update `cf_api_token.txt` file

Get api-token from your Domain provider. In my case **Cloudflare**

1. Login to `Cloudflare` dashboard

```
https://dash.cloudflare.com/
```

2. Navigate to your `Profile` on the top right corner.
3.  Select `API Tokens` from `My Profile` menu.
4.  Click on `Create Token` 
5. Find `Create Custom Token` and click `Get Started`
6. Give the token a name: `Docker Traefik`
7.  Configure permissions for the token as follows:
	* Drop down and select `Zone` --> `Zone` and `Read`
	* Click on `Add More`
	* Drop down and select `Zone` --> `Zone` and `Edit`
8.  Mention `Zone Resource`.
	* Drop down and select `Include` and `Specific Zone` and select `yourdomain`
9.  Apply **Client IP Address Filtering** `(optional)` only if you have **public static IP**
	 * Operator: Drop down and select `Is in`
	 * Value: Click `Use my IP` to get you *public static IP*
10.  Click `Continue to summary` and validate the summary 
11.  Click `Create Token` and Copy the `token` and paste it into the `cf_api_token.txt` file

-----
### Update `config/traefik.yml` file 

```
certificatesResolvers:
	cloudflare:
		acme:
			email: youremail@id.com # email used for your Cloudflare account
			storage: acme.json
			#caServer: https://acme-v02.api.letsencrypt.org/directory # prod (default)
			caServer: https://acme-staging-v02.api.letsencrypt.org/directory # staging
```

###### Note: 
Initially, use the staging environment to request certificates from Let's Encrypt. Move to production once staging is successful. As Let’s Encrypt has **rate limits** (e.g., 50 certificates per domain per week)

------------
### Update *config/config.yml* file for *External Applications*

Update IP addresses for applications according to your network
##### Note: 
`config/config.yml` is for **External Applications** i.e. apps hosted on different hosts.
e.g., Proxmox, TrueNAS, Sophos, etc.

----------------
### Labels For Internal Applications

##### Note :
*  Internal applications are those running on the same Docker host.
* These applications should be using same docker network as Traefik i.e. `proxy`

Below are example `Traefik` Labels which can be used in that respective app `docker-compose.yml` file

```
labels:
	- "traefik.enable=true"
	- "traefik.http.routers.app.entrypoints=http"
	- "traefik.http.routers.app.rule=Host(`app.yourdomain.com`)"
	- "traefik.http.middlewares.app-https-redirect.redirectscheme.scheme=https"
	- "traefik.http.routers.app.middlewares=app-https-redirect"
	- "traefik.http.routers.app-secure.entrypoints=https"
	- "traefik.http.routers.app-secure.rule=Host(`app.yourdomain.com`)"
	- "traefik.http.routers.app-secure.tls=true"
	- "traefik.http.routers.app-secure.service=app"
	- "traefik.http.services.app.loadbalancer.server.port=app_webui_port"
	- "traefik.docker.network=proxy"
```

-----------
### Create a CNAME record  for Traefik in Pi-Hole (DNS)

1. Login to Pi-hole dashboard
```
https://pi-hole.yourdomain.com/admin
```
2. Navigate to **Settings → Local DNS Settings  → Local CNAME records**
3. Create CNAME record as below and click "+" to add

| Domain                 | Target                     |
| ---------------------- | -------------------------- |
| traefik.yourdomain.com | docker_host.yourdomain.com |
###### Note:
`docker_host.yourdomain.com` is the Docker host where Traefik is running.

-----------------

### Create a CNAME record  for Applications in Pi-Hole (DNS)

1. Login to Pi-hole dashboard
```
https://pi-hole.yourdomain.com/admin
```
2. Navigate to **Settings → Local DNS Settings  → Local CNAME records**
3. Create CNAME record as below and click "+" to add

| Domain             | Target                     |
| ------------------ | -------------------------- |
| app.yourdomain.com | docker_host.yourdomain.com |
###### Note:
`docker_host.yourdomain.com` is the Docker host where Traefik is running.

-----------------
### Start our compose stack as a daemon.

```
docker compose up -d
```

----------
### Verify the Traefik Dashboard and its Certificate

* Enter Traefik dashboard URL in browser
```
traefik.yourdomain.com
```

* You may see a `Not Secure` warning. So check the Certificate details by clicking on 
  *not secure* --> *Certificate details*
 
If the issuer is `[STAGING] Let's Encrypt` that means *traefik* is able to pull certificate from `Let's Encrypt`. So update `caServer` in `config/traefik.yml` to pull *Prod* certificate.

```
certificatesResolvers:
	cloudflare:
		caServer: https://acme-v02.api.letsencrypt.org/directory
		#caServer: https://acme-staging-v02.api.letsencrypt.org/directory
```

-------
### Troubleshooting Traefik
 
 If the certificate issuer shows `Traefik Default Certificate`, it means Traefik failed to obtain a valid certificate.
 
 Check the following:
1.  Check `traefik` logs

```
docker logs -f traefik
```

2.  Check permission of `config/acme.json` file

```
ls -lrt config/acme.json
```

Expected:
```
-rw-------+ 1 USER USER 218605 Apr 18 10:26 config/acme.json
```

If not update
```
chmod 600 config/acme.json
```

3. Ensure the Traefik container has internet access

-------
