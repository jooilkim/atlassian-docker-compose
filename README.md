# atlassian-docker-compose

docker-compose.yml
```
version: '3.7'

services:

  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - dhparam:/etc/nginx/dhparam
      - certs:/etc/nginx/certs:ro
      - /var/run/docker.sock:/tmp/docker.sock:ro
    networks:
      devtool:
        ipv4_address: 172.20.1.2

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: nginx-proxy-le
    restart: always
    environment:
      - DEFAULT_EMAIL=dev@example.com
    volumes_from:
      - nginx-proxy
    volumes:
      - certs:/etc/nginx/certs:rw
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      devtool:
        ipv4_address: 172.20.1.3

  postgresql:
    image: postgres:11.10
    container_name: postgresql
    restart: always
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${PG_PASSWORD}
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8
      - TZ=Asia/Seoul
    networks:
      devtool:
        ipv4_address: 172.20.1.4

  jira:
    image: atlassian/jira-software
    container_name: jira
    restart: always
    environment:
      - VIRTUAL_HOST=jira.example.com
      - VIRTUAL_PORT=8080
      - atl_proxy_name=jira.example.com
      - atl_proxy_port=443
      - atl_tomcat_scheme=https
      - atl_tomcat_secure=true
      - LETSENCRYPT_HOST=jira.example.com
      - LETSENCRYPT_EMAIL=dev@example.com
    volumes:
      - jiraVolume:/var/atlassian/application-data/jira
      - jiraInstallVolume:/opt/atlassian/
    depends_on:
      - postgresql
    links:
      - postgresql
    networks:
      devtool:
        ipv4_address: 172.20.1.5

  confluence:
    image: atlassian/confluence-server
    container_name: confluence
    restart: always
    environment:
      - VIRTUAL_HOST=wiki.example.com
      - VIRTUAL_PORT=8090
      - atl_proxy_name=wiki.example.com
      - atl_proxy_port=443
      - atl_tomcat_scheme=https
      - atl_tomcat_secure=true
      - LETSENCRYPT_HOST=wiki.example.com
      - LETSENCRYPT_EMAIL=dev@example.com
    volumes:
      - confluenceVolume:/var/atlassian/application-data/confluence
      - confluenceInstallVolume:/opt/atlassian/
    depends_on:
      - postgresql
    links:
      - postgresql
    networks:
      devtool:
        ipv4_address: 172.20.1.6

  bitbucket:
    image: atlassian/bitbucket-server
    container_name: bitbucket
    restart: always
    environment:
      - VIRTUAL_HOST=git.example.com
      - VIRTUAL_PORT=7990
      - atl_proxy_name=git.example.com
      - atl_proxy_port=443
      - atl_tomcat_scheme=https
      - atl_tomcat_secure=true
      - LETSENCRYPT_HOST=git.example.com
      - LETSENCRYPT_EMAIL=dev@example.com
    volumes:
      - bitbucketVolume:/var/atlassian/application-data/bitbucket
      - bitbucketInstallVolume:/opt/atlassian/
    depends_on:
      - postgresql
    links:
      - postgresql
    networks:
      devtool:
        ipv4_address: 172.20.1.7

  bamboo:
    image: atlassian/bamboo-server
    container_name: bamboo
    restart: always
    environment:
      - VIRTUAL_HOST=ci.example.com
      - VIRTUAL_PORT=8085
      - atl_proxy_name=ci.example.com
      - atl_proxy_port=443
      - atl_tomcat_scheme=https
      - atl_tomcat_secure=true
      - LETSENCRYPT_HOST=ci.example.com
      - LETSENCRYPT_EMAIL=dev@example.com
    volumes:
      - bambooVolume:/var/atlassian/application-data/bamboo
      - bambooInstallVolume:/opt/atlassian/
    depends_on:
      - postgresql
    links:
      - postgresql
    networks:
      devtool:
        ipv4_address: 172.20.1.8

  crowd:
    image: atlassian/crowd
    container_name: crowd
    restart: always
    environment:
      - VIRTUAL_HOST=crowd.example.com
      - VIRTUAL_PORT=8095
      - atl_proxy_name=crowd.example.com
      - atl_proxy_port=443
      - atl_tomcat_scheme=https
      - atl_tomcat_secure=true
      - LETSENCRYPT_HOST=crowd.example.com
      - LETSENCRYPT_EMAIL=dev@example.com
    volumes:
      - crowdVolume:/var/atlassian/application-data/crowd
      - crowdInstallVolume:/opt/atlassian/
    depends_on:
      - postgresql
    links:
      - postgresql
    networks:
      devtool:
        ipv4_address: 172.20.1.9

networks:
  devtool:
    driver: bridge
    ipam:
      driver: default
      config:
       - subnet: 172.20.1.0/24
         gateway: 172.20.1.1
volumes:
  conf:
  vhost:
  html:
  dhparam:
  certs:
  pgdata:
  jiraVolume:
  jiraInstallVolume:
  confluenceVolume:
  confluenceInstallVolume:
  bitbucketVolume:
  bitbucketInstallVolume:
  bambooVolume:
  bambooInstallVolume:
  crowdVolume:
  crowdInstallVolume:
```
