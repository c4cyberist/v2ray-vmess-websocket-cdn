# VMESS(v2ray) over WebSocket + TLS behind CDN  via Tor with docker-compose
By using this guide you can setup your own server to fight with internet censorship.
The solution is a combination of a CDN network and a VM outside of the censored network that hosts the VMESS server.
Users will use VMESS protocol over the WebSocket to connect to the CDN network and the CDN network will proxy the requests to your VM outside of the censored network.

Requirements:
- A Linux server outside of censored network
- A domain name
- An account in a CDN provider

## Server Configuration
- Install docker: [docker engine installation guide on Linux servers](https://docs.docker.com/engine/install/#server "Install docker engine on Linux servers")
- Install docker-compose: [docker-compose installation guide on Linux servers](https://docs.docker.com/compose/install/other/#on-linux "Install docker-compose on Linux servers")
- Clone this repository in your server:
```shell
git clone https://github.com/wc4680/v2ray-vmess-websocket-cdn.git
cd v2ray-vmess-websocket-cdn
```
- Apply kernel configuration:
```shell
cp 10-v2ray.conf /etc/sysctl.d/
sysctl -p /etc/sysctl.d/10-v2ray.conf
```
- Change directory to `vmess` or `vmess-tor`:
	- If you trust users and their activity and want to have high-speed connectivity, you can provide high-speed internet directly to users, use `vmess` directory.
	- If you don't trust users and their activity and you have anonymity concerns about your server, you can route the traffic to the Tor network by using `vmess-tor` directory.
- Generate a random UUID:
```shell
cat /proc/sys/kernel/random/uuid
```
- Edit `v2ray/config.json` file and fill `id` with the generated UUID above:
```json
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 10000,
      "listen": "0.0.0.0",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "00000000-0000-0000-0000-000000000000",   <== Fill with the generated UUID
            "alterId": 64
          }
        ]
      },
```
- Generate a self-signed certificate:
```shell
openssl req -x509 -newkey rsa:4096 -keyout nginx/server.key -out nginx/server.crt -sha256 -days 365 -nodes
```
- Run docker-compose:
```shell
docker-compose up -d
```

## CDN Configuration
All CDN providers which offer websocket feature are suitable for this use case. You can configure the CDN easily:
- Register your domain in the CDN provider panel
- Create a subdomain (A record) in the DNS management section and assign the server IP to it
- Activate CDN on the subdomain to send traffic to port 443 with the https protocol
- Activate free SSL for your domain

## Client Configuration
Different clients for different platforms already exist for you and your users to use.
You can see a list of clients in this link: [v2ray clients](https://www.v2ray.com/en/awesome/tools.html "v2ray clients")
In all of these clients you should configure these parameters:
- Protocol: VMESS
- Address: The subdomain that you have created in CDN
- Port: 443
- ID: The same id parameter (UUID) in v2ray config.json file
- AlterID: 64
- Network: ws (or WebSocket)
- Security: Don't select "none"! Anything else is ok.
- Request Host: The subdomain that you have created in CDN
- SNI: The subdomain that you have created in CDN
- Path: /graphql
- TLS: tls

### Domain Fronting
If your CDN provider supports the "Domain Fronting" feature, you can change "Address" and "SNI" (if your client has it) to a domain that is served by the same CDN provider. In this case, DNS resolution and TLS hello messages will not reveal anything about your server. So it will be very difficult for internet censorship equipment to detect your traffic.

For example:
Assume that your subdomain is `myserver.mydomain.com` and `popularsite.com` is also hosted in the same CDN.
You can check the domain fronting feature by running this command:
```shell
curl -H 'Host: myserver.mydomain.com' https://popularsite.com/test.html
```
If you see this message in the output, domain fronting is supported on your CDN:
```
Domain fronting is supported!
```
So you can change client configuration as follow:
- Address: popularsite.com
- SNI: popularsite.com
- Request Host: myserver.mydomain.com
