---
title: "TCP & UDP reverse proxy With NGINX"
description: "NGINX를 이용해 TCP/UDP reverse proxy 서버 구축하기"
date: 2021-09-09T15:34:23+09:00
categories: ["dev"]
keywords: ["NGINX TCP reverse proxy", "NGINX UDP reverse proxy", "Minecraft", "Minecraft Server"]
tags: ["NGINX", "TCP reverse proxy", "UDP reverse proxy", "Minecraft Server"]
draft: false
---

클라우드플레어로 마인크래프트 서버 컴퓨터 IP를 완전히 숨기지 못하여 각종 공격에 취약해집니다. Origin Server IP를 숨기기 위해 NGINX로 구축한 proxy server를 경유하도록 하여 Origin Server의 IP를 노출하지 않도록 구축하였습니다.

## 구성

 **우분투 20.04 환경을 전제로 설명합니다.** \
 **클라우드플레어 스펙트럼이 아닌 SRV 레코드를 사용하는 환경입니다.**

`Origin Server <--> Proxy Server <--> Client` \
ex) `121.xxx.xxx.xxx <--> 151.xxx.xxx.xxx / test-server.example.com <--> Client`

## NGINX 설정

```nginx
# /etc/nginx.conf

stream {
        ## Minecraft JE 서버의 경우
        upstream minecraft_tcp {
                server [server_ip]:[port];
        }
        
        ## Minecraft BE 서버의 경우
        # upstream minecraft_udp {
        #         server [server_ip]:[port];
        # }

        ## TCP
        server {
                listen [port];
                proxy_pass minecraft_tcp;
                proxy_connect_timeout 1s;
        }

        ## UDP
        # server {
        #         listen [port] udp;
        #         proxy_pass minecraft_udp;
        #         proxy_connect_timeout 1s;
        # }       
}
```

**upstream**의 [server_ip]와 [port]에 Origin Server IP와 PORT로 변경하고 server의 listen [port]를 원하는 포트나 또는 origin server 포트로 변경해 주고 다음 명령어를 입력하여 설정을 적용합니다.

```bash
sudo service nginx restart
```

NGINX 재실행 후 프록시 서버 IP 또는 프록시 서버에 설정된 도메인으로 접속이 가능합니다.

순서대로 리버스 프록시 적용 / 미적용
![리버스 프록시 적용](https://media.discordapp.net/attachments/670229327564242944/884294649861251112/unknown.png)
![리버스 프록시 미적용](https://media.discordapp.net/attachments/633971402550280192/885433320173867028/unknown.png)

origin server ip 대역은 21x.xx.xx.xx 이지만 리버스 프록시를 적용하면 origin 서버의 IP 대신 프록시 서버 IP만 표시됩니다.
![리버스 프록시 IP 표시함](https://media.discordapp.net/attachments/633971402550280192/885431811063308298/unknown.png)
