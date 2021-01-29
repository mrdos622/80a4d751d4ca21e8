# 80a4d751d4ca21e8

Pre-requisites: Use Debian Buster

1. Install Docker and Compose
user@server:/$ sudo apt update && sudo apt install docker.io docker-compose

2. Create a script for the docker image configuration
user@server:/$ sudo mkdir /root/test
user@server:~/$ sudo nano /root/configure

3. Create local volume for the database files 
user@server:/$ sudo docker volume create dbase

4. Build the docker image:
user@server:/$ sudo docker build -t test:latest -f- . <<EOF
FROM debian:stable
LABEL maintainer="test@example.com"
COPY ./configure /tmp
RUN /bin/bash /tmp/configure
EXPOSE 80
CMD /usr/bin/pg_ctlcluster 11 main start && /usr/sbin/nginx && tail -f /dev/null 
EOF

5. Create docker-compose file
user@server:/$ sudo nano /root/test/docker-compose.yml
version: "3"
services:
  test:
    image: "test:latest"
    volumes:
      - "dbase:/var/lib/postgresql/11/main"
    ports:
      - "8080:80"
    restart: unless-stopped
volumes:
  dbase:
    external: true

6. As root run container with Compose
root@server:~/test# docker-compose up -d 

7. Test if it works
user@server:/$ sleep 60 #wait for the docker image boots up
user@server:/$ curl -X GET http://localhost:8080
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>

user@server:/$ curl -X POST -d "arg1=Hello&arg2=Sweet&arg3=Home" http://localhost:8080
               curl -X POST -d "arg1=Welcome&arg2=Aboard&arg3=Admin" http://localhost:8080

user@server:/$ sudo docker exec -ti `docker ps | grep "test:latest" | cut -d " " -f 1` /bin/bash
  root@93ce0ace374a:/# psql -h localhost -U dbuser dbase
  Password for user dbuser: 
  psql (11.9 (Debian 11.9-0+deb10u1))
  SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
  Type "help" for help.
  
  dbase=> select * from args;
   id |  arg1   |  arg2  | arg3  
  ----+---------+--------+-------
    1 | Hello   | Sweet  | Home
    2 | Welcome | Aboard | Admin
  (2 rows)

  dbase=> \q
  root@93ce0ace374a:/# exit
  exit

