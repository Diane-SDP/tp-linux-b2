# TP1

## I. Init

### 3. sudo c pa bo

🌞 Ajouter votre utilisateur au groupe docker

```powershell
[diane@localhost ~]$ sudo usermod -aG docker ${USER}
[sudo] password for diane:
[diane@localhost ~]$ su - diane
Password:
Last login: Wed Dec 11 10:25:33 CET 2024 from 10.2.2.1 on pts/0
[diane@localhost ~]$ id -nG
diane wheel docker
```

### 4. Un premier conteneur en vif

🌞 Lancer un conteneur NGINX

```powershell
[diane@localhost ~]$ docker run -d -p 9999:80 nginx
```

🌞 Visitons

```powershell
[diane@localhost ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                     NAMES
e616eb26f94d   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp   wizardly_noyce

[diane@localhost ~]$ docker logs wizardly_noyce
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2024/12/11 09:29:33 [notice] 1#1: using the "epoll" event method
2024/12/11 09:29:33 [notice] 1#1: nginx/1.27.3
2024/12/11 09:29:33 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2024/12/11 09:29:33 [notice] 1#1: OS: Linux 5.14.0-284.11.1.el9_2.x86_64
2024/12/11 09:29:33 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
2024/12/11 09:29:33 [notice] 1#1: start worker processes
2024/12/11 09:29:33 [notice] 1#1: start worker process 28

[diane@localhost ~]$ docker inspect wizardly_noyce
# là y'a pleins d'infos flm de copier

[diane@localhost ~]$ sudo ss -lnpt | grep docker
LISTEN 0      4096         0.0.0.0:9999      0.0.0.0:*    users:(("docker-proxy",pid=3827,fd=4))
LISTEN 0      4096            [::]:9999         [::]:*    users:(("docker-proxy",pid=3832,fd=4))

[diane@localhost ~]$ sudo firewall-cmd --add-port=9999/tcp
success
[diane@localhost ~]$ sudo firewall-cmd --reload
success
```

🌞 On va ajouter un site Web au conteneur NGINX

```powershell
[diane@localhost nginx]$ nano index.html
[diane@localhost nginx]$ nano site_nul.conf
[diane@localhost nginx]$ docker run -d -p 9999:8080 -v /home/diane/nginx/index.html:/var/www/html/index.html -v /home/diane/nginx/site_nul.conf:/etc/nginx/conf.d/site_nul.conf nginx
8566682b469c5dc84a2d42581467f39f99e596273ac58373957cc265e0065aa1
```

🌞 Visitons

```powershell
[diane@localhost nginx]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                                 NAMES
8566682b469c   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp, 0.0.0.0:9999->8080/tcp, [::]:9999->8080/tcp   practical_panini
```

### 5. Un deuxième conteneur en vif

🌞 Lance un conteneur Python, avec un shell

```powershell
[diane@localhost nginx]$ docker run -it python bash
Unable to find image 'python:latest' locally
latest: Pulling from library/python
fdf894e782a2: Pull complete
5bd71677db44: Pull complete
551df7f94f9c: Pull complete
ce82e98d553d: Pull complete
5f0e19c475d6: Pull complete
abab87fa45d0: Pull complete
2ac2596c631f: Pull complete
Digest: sha256:220d07595f288567bbf07883576f6591dad77d824dce74f0c73850e129fa1f46
Status: Downloaded newer image for python:latest
root@39baf2e522e1:/#
```

🌞 Installe des libs Python

```powershell
root@39baf2e522e1:/# pip install aioconsole
root@39baf2e522e1:/# pip install aiohttp

root@39baf2e522e1:/# python
Python 3.13.1 (main, Dec  4 2024, 20:40:27) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import aiohttp
```

## II. Images

### 1. Images publiques

🌞 Récupérez des images

```powershell
[diane@localhost nginx]$ docker pull python:3.11
[diane@localhost nginx]$ docker pull mysql:5.7
[diane@localhost nginx]$ docker pull wordpress:latest
[diane@localhost nginx]$ docker pull linuxserver/wikijs:latest
```

🌞 Lancez un conteneur à partir de l'image Python

```powershell
[diane@localhost nginx]$ docker run -it python:3.11 bash
root@5d0261ca9dc2:/# python --version
Python 3.11.11
```

### 2. Construire une image

🌞 Ecrire un Dockerfile pour une image qui héberge une application Python

```powershell
[diane@localhost python_app_build]$ sudo nano Dockerfile
FROM debian

RUN apt update -y
RUN apt install -y python3
RUN apt install python3-emoji

COPY ./app.py /home/diane/python_app_build/app.py

ENTRYPOINT ["python3", "/home/diane/python_app_build/app.py"]
```

🌞 Build l'image

```powershell
[diane@localhost python_app_build]$ docker build . -t python_app:version_de_ouf
[+] Building 11.2s (11/11) FINISHED 
[diane@localhost python_app_build]$ docker image ls
REPOSITORY           TAG              IMAGE ID       CREATED         SIZE
python_app           version_de_ouf   ed6996cd6183   2 minutes ago   637MB  
```

🌞 Lancer l'image

```powershell
[diane@localhost python_app_build]$ docker run python_app:version_de_ouf
Cet exemple d'application est vraiment naze 👎
```

## III. Docker compose

🌞 Créez un fichier docker-compose.yml

```powershell
[diane@localhost compose_test]$ cat docker-compose.yml
version: "3"

services:
  conteneur_nul:
    image: debian
    entrypoint: sleep 9999
  conteneur_flopesque:
    image: debian
    entrypoint: sleep 9999
```

🌞 Lancez les deux conteneurs avec docker compose

```powershell
[diane@localhost compose_test]$ docker compose up -d
WARN[0000] /home/diane/compose_test/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 3/3
 ✔ conteneur_nul Pulled                                                                                                                                3.2s
 ✔ conteneur_flopesque Pulled                                                                                                                          2.9s
   ✔ fdf894e782a2 Already exists                                                                                                                       0.0s
[+] Running 3/3
 ✔ Network compose_test_default                  Created                                                                                               0.8s
 ✔ Container compose_test-conteneur_nul-1        Started                                                                                               0.6s
 ✔ Container compose_test-conteneur_flopesque-1  Started   
```

🌞 Vérifier que les deux conteneurs tournent

```powershell
[diane@localhost compose_test]$ docker ps
CONTAINER ID   IMAGE     COMMAND        CREATED          STATUS          PORTS     NAMES
90315c5dc9d5   debian    "sleep 9999"   45 seconds ago   Up 44 seconds             compose_test-conteneur_nul-1
aa6a888ac795   debian    "sleep 9999"   45 seconds ago   Up 44 seconds             compose_test-conteneur_flopesque-1
```

🌞 Pop un shell dans le conteneur conteneur_nul

```powershell
[diane@localhost compose_test]$ docker exec -it compose_test-conteneur_nul-1 /bin/bash
root@90315c5dc9d5:/# apt-get update -y
root@90315c5dc9d5:/# apt-get install -y iputils-ping
root@90315c5dc9d5:/# ping conteneur_flopesque
PING conteneur_flopesque (172.18.0.2) 56(84) bytes of data.
64 bytes from compose_test-conteneur_flopesque-1.compose_test_default (172.18.0.2): icmp_seq=1 ttl=64 time=0.142 ms
64 bytes from compose_test-conteneur_flopesque-1.compose_test_default (172.18.0.2): icmp_seq=2 ttl=64 time=0.077 ms
^C
--- conteneur_flopesque ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1004ms
rtt min/avg/max/mdev = 0.077/0.109/0.142/0.032 ms
```
