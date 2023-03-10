<h1 align="center">Olá! <img src="https://raw.githubusercontent.com/kaueMarques/kaueMarques/master/hi.gif" width="30px"></h1>
<h3 align="center">Indice</h3>

1. [WSL2](#1-wsl2)
   - 1.1 [Limitando o uso de recursos do WSL na sua máquina](#11-limitando-o-uso-de-recursos-do-wsl-na-sua-máquina)
2. [Instalando o DOCKER](#2-instalando-o-docker)
    - 2.1 [Preparando os repositórios](#21-preparando-os-repositórios)
    - 2.2 [Instalando o Docker Engine](#22-instalando-o-docker-engine)
    - 2.3 [Verificando se a instalação teve sucesso](#23-verificando-se-a-instalação-teve-sucesso)
     - 2.3.1 [Deve retornar "Starting Docker: docker"](#231-deve-retornar-starting-docker-docker)
3. [Criando um container .NET com OH-my-ZSH](#3-criando-um-container-net-com-oh-my-zsh)
4. [Docker File & Docker Compose](#4-docker-file--docker-compose)
5. [Comandos úteis](#5-comandos-%C3%BAteis)
   - 5.1 [Linux](#51-linux)
   - 5.2 [Docker](#52-docker)
6. [Connection String](#6-connection-string)

<hr>
Este repositório é voltado para a instalação de um ambiente de desenvolvimento Windows/Linux com .NET 7, 6 e/ou 5, usando docker e WSL2.

Para a instalação desse ambiente de desenvolvimento, você precisa ter instalado o Windows Terminal, o Visual Studio Code e o WSL2 com alguma distro Linux da sua preferência. Aqui usarei o Ubuntu, mas você pode ficar livre para escolher qual desejar. Lembrando que irá precisar adaptar a instalação das ferramentas para a sua distro.

# 1. WSL2
## 1.1 Limitando o uso de recursos do WSL na sua máquina
Caso o WSL esteja consumindo muito recurso da sua máquina, você pode limitar isso criando um arquivo com o nome ".wslconfig" na pasta de usuários no seu computador "c:\users\\\<seuUsuario>".
Crie o aquivo e adicione a seguinte configuração:

```
[wsl2]
memory=8GB
processors=4
swap=2GB
```

# 2. Instalando o DOCKER
## 2.1 Preparando os repositórios

Se você preferir, pode acompanhar a instalação direto no site do Docker clicando aqui [Instalação do Docker](https://docs.docker.com/engine/install/ubuntu/)

Para fazer a preparação para a instalação do Docker na sua máquina, é necessário executar os comandos abaixo no seu terminal. 

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```bash
sudo mkdir -p /etc/apt/keyrings
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 2.2 Instalando o Docker Engine

```bash
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 2.3 Verificando se a instalação teve sucesso

### 2.3.1 Deve retornar "Starting Docker: docker"
```bash
sudo service docker start
```

```bash
sudo docker run hello-world
```

Adicionando o seu usuário atual ao grupo do Docker. Isso vai fazer com que você consiga executar os comandos do Docker sem a necessidade de permissão de super usuário, "sudo".
```bash
sudo usermod -aG docker $USER
```

# 3. Criando um container .NET com OH-my-ZSH

Este passo é apenas para teste. Não é necessário que você execute esse comando. Se desejar você pode ir para [Docker File & Docker Compose](#4-docker-file--docker-compose)

```bash
docker run -dit --name restapinet6 -p 5000:5000 -p 5001:5001 -v $(pwd):/app/ mcr.microsoft.com/dotnet/sdk:6.0 /bin/bash -c "apt-get update && apt-get install -y zsh && wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh && echo | sh install.sh && chsh -s $(which zsh) | echo && wget https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && dpkg -i packages-microsoft-prod.deb && rm packages-microsoft-prod.deb && apt-get update && apt-get install -y dotnet-sdk-5.0 && useradd -r -u 1000 -d /app dotnet && zsh"
```

Caso você deseje usar esse container, é necessário que gere uma nova imagem para aí sim, criar e usar o container. Tudo isso baseado no código docker acima. 
Após a execução do comando acima, execute o comando de commit do docker passando o Id do container que foi criado e o nome da imagem da sua escolha. 

Verificando o id do container

```bash
docker ps -a
```
Criando a nova imagem (remova os caracteres <>)
```bash
docker commit <id do container> <novo nome da imagem>
```

Para verificar as imagem que você criou
```bash
docker images
```

Rodando um novo container
```bash
docker run -dit --name <novo nome do container> <nome da imagem que você criou no passo anterior> zsh
```

Agora é possível entrar no container pelo comando exec.

```bash
docker exec -it <nome do container> zsh
```

# 4. Docker File & Docker Compose 
Aqui começa a parte mais interessante, pois trabalhar com o docker apenas na linha de comando pode se tornar algo bem cansativo com o passar do tempo. O uso do docker compose junto com o Dockerfile irá trazer uma grande liberdade e clareza na criação de imagens e containers. 
Para que possamos começar a usar os dois, crie um arquivo com o seguinte nome "docker-compose.yml" e outro arquivo com o nome "Dockerfile", você pode acompanhar todo o código clicando [Dockerfile](https://github.com/allysonreeis/dotnet-wsl-dev/blob/main/Dockerfile) e no [Docker Compose](https://github.com/allysonreeis/dotnet-wsl-dev/blob/main/docker-compose.yml).


# 5. Comandos úteis

## 5.1 Linux

```bash
#Informa se você está em um terminal dentro ou fora de um container
grep -q docker /proc/self/cgroup && echo "in Docker container" || echo "not in Docker container”

#Mostra os usuários da máquina
cat /etc/passwd

#Mostra o usuário atual da máquina
whoami
```

## 5.2 Docker

```bash
#Inicia ou para o serviço do docker
sudo service docker start
sudo service docker stop

#Mostra os container que estão rodando
docker ps

#Mostra todos os containers que estão rodando e parados
docker ps -a

#apaga um container
# pode-se adicinar o parâmetro -f para forçar a exclusão
docker container rm <nome_do_container>

#apaga uma imagem
# pode-se adicinar o parâmetro -f para forçar a exclusão
docker rmi <nome_da_imagem>

#Mostra as imagens criadas
docker images

#Mostra os logs do container
docker logs <nome_do_container>

#Para um container
docker stop <nome_do_container>

#Cria um novo container
# parâmetros
# -d -> detached, roda um container em segundo plano
# -i -> ativa o modo interativo do container
# -t -> ativa o TTY. Vincula o console atual com o do container
# --name -> especifica um nome pro container
# -v -> cria um volume
# -p -> expõe uma porta e mapeia pro host
docker run 

#Exibe as informações do container
docker container inspect <nome_do_container>

#Sobe todos os containers definidos no arquivo docker compose. --build é interessante para quando há alguma alteração no Dockerfile
docker compose up -d --build

# Para todos os container e deleta
docker compose down

#Para todos os container sem deletar
docker compose stop

#Inicia todos os containers
docker compose start
```

# 6. Connection String

```bash
"Server=<nome_do_container>,1433;Database=<nome_do_banco>;User=sa;Password=yourStrong(!)Password;TrustServerCertificate=True;"
```
