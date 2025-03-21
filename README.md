## Configuração inicial de VPS Ubuntu 24.04 com Docker

*Autor: Gilberto Toledo*<br/>
Tutorial completo: [Youtube](https://youtu.be/RxTyDLsnmAs)

### Configurando o VPS

1. Acesso remoto com SSH
```
ssh root@[SEU_IP]
```

2. Criar usuário local

```
adduser [SEU_NOME_DE_USUARIO]
```

3. Conceder prilégios de administrador
```
usermod -aG sudo [SEU_NOME_DE_USUARIO]
```

4. Configurando o Firewall
```
ufw app list
```
```
ufw allow OpenSSH
```
```
ufw status
```
<br/>

### Instalando a Engine do Docker
[Docker Docs](https://docs.docker.com/engine/install/ubuntu/)

1. Adicionar o repositório do Docker
```
sudo apt-get update && sudo apt-get install ca-certificates curl -y
```
```
sudo install -m 0755 -d /etc/apt/keyrings
```
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```
```
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
sudo apt-get update
```
<br/>

2. Instalar o pacote do Docker
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verificar status do serviço
```
sudo systemctl status docker
```
<br/>

4. (Opcional) Adicionar seu usuário ao grupo `docker`
Útil para executar comandos docker sem o prefixo `sudo`

```
sudo usermod -aG docker ${USER}
```

```
su - ${USER}
```
<br/>

5. Executar sua primeira aplicação
```
sudo docker run hello-world
```

## Comandos úteis do Docker

Listar imagens
```
docker images
```

Listar containers
```
docker ps
docker ps -a
```

Manipular um container
```
docker start [CONTAINER_ID]
docker stop [CONTAINER_ID]
docker restart [CONTAINER_ID]
```

Acessar o conteúdo de um container
```
docker exec -it [CONTAINER_ID] /bin/bash
```

Executar uma imagem
```
docker run -p [PORTA_EXTERNA]:[PORTA_INTERNA] -v [VOLUME_EXTERNO]:[VOLUME_INTERNO] -e [VARIAVEL_DE_AMBIENTE]:[VALOR] [NOME_DA_IMAGEM]
```
