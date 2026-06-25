# Guia de Configuração: Ubuntu Server + Docker

Este guia contém todos os comandos necessários para instalar o SSH, configurar o ambiente Docker e colocar o seu primeiro projeto estruturado para rodar.

---

## 1. Configurar Conexão SSH

O Ubuntu Server geralmente traz o SSH instalado. Caso precise instalar ou ativar manualmente, utilize os comandos abaixo:

```bash
# Atualiza a lista de pacotes do sistema
sudo apt update

# Instala o servidor SSH
sudo apt install openssh-server -y

# Ativa e inicia o serviço do SSH para rodar junto com o sistema
sudo systemctl enable --now ssh

# Verifica se o SSH está rodando corretamente
sudo systemctl status ssh

# Libera o SSH no firewall do Ubuntu (UFW)
sudo ufw allow ssh

# Ativa o firewall (confirme com 'y' se ele avisar sobre a conexão atual)
sudo ufw enable
```

---

## 2. Configurar o Ambiente para Docker

Comandos para configurar o repositório oficial e instalar a versão estável mais recente do Docker:

```bash
# Instala dependências necessárias do sistema
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

# Cria o diretório para chaves de segurança (caso não exista)
sudo mkdir -p /etc/apt/keyrings

# Baixa e adiciona a chave GPG oficial do Docker
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ajusta as permissões de leitura da chave
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Adiciona o repositório oficial do Docker às fontes do APT
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


# Atualiza a lista de pacotes com o novo repositório integrado
sudo apt update

# Instala o Docker Engine, CLI, Containerd e o plugin do Docker Compose
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Adiciona seu usuário atual ao grupo do Docker (evita a necessidade de usar 'sudo' antes de cada comando do Docker)
sudo usermod -aG docker $USER

# Aplica a mudança de grupo imediatamente na sessão atual
newgrp docker
```

---

## 3. Comandos Principais do Docker

Guia rápido de gerenciamento para quem está iniciando com contêineres:

```bash
# Verificar status do Docker
sudo systemctl status docker

# Baixar uma imagem do Docker Hub sem iniciar o contêiner
docker pull ubuntu

# Listar todas as imagens baixadas no servidor
docker images

# Criar e iniciar um contêiner (Exemplo com um servidor Web Nginx)
# (-d = segundo plano | -p = mapeia porta externa:interna | --name = nomeia o contêiner)
docker run -d -p 80:80 --name meu-servidor nginx

# Listar apenas os contêineres que estão ATIVOS no momento
docker ps

# Listar TODOS os contêineres (tanto em execução quanto parados)
docker ps -a

# Parar um contêiner em execução
docker stop meu-servidor

# Iniciar um contêiner que já existe mas estava parado
docker start meu-servidor

# Remover permanentemente um contêiner (ele precisa ser parado antes)
docker rm meu-servidor

# Remover uma imagem do servidor (liberar espaço)
docker rmi nginx
```

---

## 4. Deploy de Projeto Automatizado (Docker Compose)

O Docker Compose permite gerenciar múltiplos serviços de um projeto usando apenas um arquivo de configuração.

### Passo 1: Criar a pasta do seu projeto
```bash
mkdir meu-projeto && cd meu-projeto
```

### Passo 2: Criar o arquivo de configuração
Crie um arquivo chamado `docker-compose.yml`:
```bash
nano docker-compose.yml
```

### Passo 3: Adicionar a estrutura do ambiente
Cole o conteúdo abaixo dentro do arquivo (Este exemplo cria uma aplicação WordPress integrada com um banco de dados MySQL):

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: senha_secreta_do_banco
      MYSQL_DATABASE: wordpress

  web:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: senha_secreta_do_banco
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db
```
*(Para salvar e sair do editor Nano: Pressione `Ctrl + O`, depois `Enter`, e finalize com `Ctrl + X`).*

### Passo 4: Subir a aplicação
Execute o comando abaixo na mesma pasta onde o arquivo `docker-compose.yml` foi salvo:
```bash
docker compose up -d
```

### Passo 5: Testar o acesso
Abra o navegador e digite o endereço IP do seu servidor seguido da porta configurada:
```text
http://IP_DO_SEU_SERVIDOR:8080
```

### Passo 6: Parar o projeto
Para encerrar todos os contêineres e redes criadas pelo projeto de uma só vez:
```bash
docker compose down
```
