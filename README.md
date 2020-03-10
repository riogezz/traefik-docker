# Traefik on Docker
##### HTTP and HTTPS example with Let's Encrypt certificates served by DNS01-Challenge on AWS Route53 and HTTP-to-HTTPS redirect

___

##### __|__ Please check [traefik 2.1 docs](https://docs.traefik.io/) for more.


### .env file variables  

| Name | Example value value | Description |
|:----|:--------------|:----------- |
|- __*Global*__|
`TZ`| Europe/Rome| container timezone |
`RESTART`| unless-stopped| container restart policy |
`COMPOSE_PROJECT_NAME`| traefik_router| project name used as prepend string |
|- __*Traefik specific*__|
`VERSION`| latest| traefik version |
`LOG`| INFO| traefik minimum logging |
|- __*ACME DNS-01 Challenge*__|
|`PROVIDER`|route53|check [provider list](https://docs.traefik.io/https/acme/#providers) |
|`RESOLVER`|1.1.1.1:53|public DNS server to be used for acme TXT fields checks
|`EMAIL`|xyz@domain.ltd|your email |
|- __*ACME AWS Route53 example*__|
|`AWS_ACCESS_KEY_ID`|ABCXYZ|IAM username |
|`AWS_SECRET_ACCESS_KEY`|123890|IAM secret |
|`AWS_REGION`|us-east-1|AWS Route53 service is region independent |

__|__ for AWS Route53 provider configuration head to [AWS docs](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html#setup-credentials-setting) about AWS IAM credentials and [policy document example](https://go-acme.github.io/lego/dns/route53/#policy) or [Let's Encrypt GO client route53 provider](https://go-acme.github.io/lego/dns/route53/) docs. 


### docker-compose explanation

_docker-compose.yml_ conatins a _whoami_ test instance with http-to-https redirect features

```
    labels:
      - "traefik.enable=true"
      # default route
      - "traefik.http.routers.whoami.rule=Host(`whoami.domain.tld`)"
      - "traefik.http.routers.whoami.entrypoints=https"
      - "traefik.http.routers.whoami.tls.certresolver=${PROVIDER}"
      # HTTP to HTTPS
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
      - "traefik.http.routers.whoami-redirs.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.whoami-redirs.entrypoints=http"
      - "traefik.http.routers.whoami-redirs.middlewares=redirect-to-https"

```
#### sections explanation:  

- enable traefik configuration on this container
```
      - "traefik.enable=true"
```

- configure router to serve "whoami.domain.tld" FQDN over https entrypoint and generate SSL certificate using provider defined inside .env (eg: _route53_ )
```
      - "traefik.http.routers.whoami.rule=Host(`whoami.domain.tld`)"
      - "traefik.http.routers.whoami.entrypoints=https"
      - "traefik.http.routers.whoami.tls.certresolver=${PROVIDER}"
```

- configure http-to-https redirect scheme
```
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
```

- apply redirect scheme to every request directed to host over http ([link](https://community.containo.us/t/global-http-to-https-redirect-in-v2/1658/3))
```
      - "traefik.http.routers.whoami-redirs.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.whoami-redirs.entrypoints=http"
      - "traefik.http.routers.whoami-redirs.middlewares=redirect-to-https"
```


Traefik dashboard will run on exposed [TCP/8080](http://localhost:8080) and should be like this

![traefik dashboard image](https://docs.traefik.io/assets/img/webui-dashboard.png)