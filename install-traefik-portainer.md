## Configurando o Portainer e Traefik

*Autor: Gilberto Toledo*<br/>
Tutorial completo: [Youtube](https://www.youtube.com/@gilbertotoledo)

### 1. Criando uma rede pública no Docker
```
docker network create web
```

### 2. Criando uma senha para o usuário do Traefik
```
sudo apt-get install apache2-utils
```

```
htpasswd -nb admin SUA_SENHA
```

### 3. Criando os arquivos necessários

##### docker-compose.yml
```
services:  
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    networks:
      - web
    ports:
      - 80:80
      - 443:443
    volumes:
      - /etc/localtime:/etc/localtime
      - /var/run/docker.sock:/var/run/docker.sock
      - /home/docker/traefik/config/traefik.toml:/traefik.toml
      - /home/docker/traefik/config/traefik_dynamic.toml:/traefik_dynamic.toml
      - /home/docker/traefik/config/acme.json:/acme.json
    logging:
      options:
        max-size: "10m"
        max-file: "3"

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /home/docker/portainer/data:/data
    ports:
      - 8000:8000
      - 9000:9000
      - 9443:9443
    networks:
      - web
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.SEU_DOMINIO`)"
      - "traefik.http.routers.portainer.tls=true"
      - "traefik.http.routers.portainer.tls.certresolver=lets-encrypt"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
      - "traefik.docker.network=web"
    logging:
      options:
        max-size: "10m"
        max-file: "3"

networks:
  web:
    external: true
```

##### traefik/traefik.toml
```
[entryPoints]
  [entryPoints.web]
    address = ":80"
    
    [entryPoints.web.http]
      [entryPoints.web.http.redirections]
        [entryPoints.web.http.redirections.entryPoint]
          to = "websecure"
          scheme = "https"

  [entryPoints.websecure]
    address = ":443"

[log]
  level = "WARN"

[accessLog]

[metrics]
  [metrics.prometheus]
    addEntryPointsLabels = true
    addServicesLabels = true
    addRoutersLabels = true

[api]
  dashboard = true

[certificatesResolvers.lets-encrypt.acme]
  email = "SEUEMAIL@gmail.com"
  storage = "acme.json"
  [certificatesResolvers.lets-encrypt.acme.tlsChallenge]

[providers.docker]
  watch = true
  network = "web"

[providers.file]
  filename = "traefik_dynamic.toml"

```

##### traefik/traefik_dynamic.toml
```
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "SENHA_CRIPTOGRAFADA_GERADA_COM_HTPASSWD"
  ]

# Use with traefik.http.routers.myRouter.middlewares: "redirect-www-to-main@file"
[http.middlewares]
  [http.middlewares.redirect-www-to-main.redirectregex]
      permanent = true
      regex = "^https?://www\\.(.+)"
      replacement = "https://${1}"

[http.routers.api]
  rule = "Host(`traefik.SEU_DOMINIO`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```

##### acme.json (arquivo vazio) e configurar permissões
```
touch acme.json && chmod 600 acme.json
```

```
services:
  hello-world-webapp:
    image: crccheck/hello-world
    networks:
     - web
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.hello_world.rule=Host(`hello.gilberto.cloud`)"
      - "traefik.http.routers.hello_world.tls=true"
      - "traefik.http.routers.hello_world.tls.certresolver=lets-encrypt"
      - "traefik.http.services.hello_world.loadbalancer.server.port=8000"
      - "traefik.docker.network=web"

networks:
  web:
    external: true
```
