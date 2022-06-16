# Custom HTTP tunnel setup for on-permise PAM

This page contains setup info on how to configure HTTP tunnel using https://github.com/mmatczuk/go-http-tunnel for on-premise PAM

## Architecture

![image](https://user-images.githubusercontent.com/4685314/174011178-0780d6e4-a820-4ffd-b0e7-4fc0f6f9d5e4.png)


## Setup

### Install tunneld & nginx proxy manager on AWS EC2
1. Create a AWS EC2 instance (Amazon Linux 2) & allow inbound ports: `22, 81, 443, 5223`
2. Install `docker` & `docker-compose`
3. Download tunnel client for Linux from https://github.com/mmatczuk/go-http-tunnel/releases
4. Prepare server key & cert files
```
openssl req -x509 -nodes -newkey rsa:2048 -sha256 -keyout server.key -out server.crt
```
5. Prepare startup script, and update the path if needed
```
#!/bin/bash
rm -f nohup.out
nohup sudo /home/ec2-user/go-http-tunnel/tunneld \
  -httpsAddr ":10443" \
  -httpAddr ":10080" \
  -tlsCrt .tunneld/server.crt \
  -tlsKey .tunneld/server.key &
```
6. Start `tunneld` by executing the startup script
7. Review `nohup.out` to make sure `tunneld` is up & running
8. Follow the steps from https://nginxproxymanager.com/setup/ to install nginx proxy manager 
8. Create a proxy host with valid SSL certificate (Let's Encrypt) with valid FQDN that points to EC2 IP with TCP port `10443` (tunneld).  
9. (optional) create a proxy host with valid SSL certificate (Let's Encrypt) that points to local port `81`.   Check and block inbound port `81`.

### Install tunnel client on PVWA

1. Download tunnel client for Windows from https://github.com/mmatczuk/go-http-tunnel/releases
2. Prepare client key & cert
```
openssl req -x509 -nodes -newkey rsa:2048 -sha256 -keyout client.key -out client.crt
```
3. Prepare `tunnel.yml` file using valid FQDN, for example:
```
server_addr: server.customer.quincycheng.com:5223
tunnels:
  webui:
    proto: http
    addr: https://comp01.cybr.com
    host: tunnel.customer.quincycheng.com
```
4. Start tunnel client by executing
```
tunnel -config tunnel.yml start-all
```

### Create CEM user on PAM and confgure CEM
Please refer to https://docs.cyberark.com/Product-Doc/OnlineHelp/CEM/Latest/en/Content/CloudAdmin/kd_PAS-PrivCloud.htm for details
The URL will be the one from nginx proxy manager with valid FQDN & SSL certificate (e.g. `https://tunnel.customer.quincycheng.com`)
