# Projeto Docker
## Tecnologias utilizadas
AWS EC2 Amazon Linux 2023 AMI

AWS EFS

AWS Aurora and RDS

Visual Studio Code 1.97.2

MySQL Workbench 8.0.41 Community

## 1. Instalação e configuração do DOCKER no host EC2
### Criação do Security group
Dentro da AWS pesquise por EC2 e entre na página, dentro da página na esquerda procure por "Security Groups" e clique.

Clique no botão amarelo escirto "Create security group", na página de criação selecione uma VPC criada com configurações padrão.

Na parte de "Inbound rules" clique em add rules e adicione estas regras:
```
Type: SSH  Port:22  Source type: Anywhere-IPv4 #Para acessar a instância via SSH
Type: HTTP  Port:80  Source type: Anywhere-IPv4 #Para acessar o Wordpress
Type: MySQL/Aurora  Port:3306 Source type: Anywhere-IPv4 #Conexão Instância com banco de dados
Type: NFS  Port:2049  Source type: Anywhere-IPv4 #Para montagem do EFS
```
Na parte de "Outbound rules" clique em add rules e adicione estas regras:
```
Type: All traffic  Port:All  Source type: Anywhere-IPv4 
```
Clique no botão amarelo "Create security group".
### Criação da instância EC2
Dentro da AWS pesquise por EC2 e entre na página, na página da EC2 clique no botão amarelo escrito "Launch instance".

Dentro da página de criação de instância,em "Application and OS Images (Amazon Machine Image)" selecione a imagem "Amazon Linux 2023 AMI"

Na opção "Key pair (login)" crie uma chave .pem ou use uma já criada.

Em "Network settings" clique no botão "Edit", use uma VPC com configuração padrão, em "Auto-assign public IP" deixe como "Enable"

No "Firewall (security groups)" selecione o security group criado anteriormente.

Role para baixo e clique na opção "Advanced details" e em seguida procure a opção "User data - optional".

no campo de texto de User data insira:
```
#!/bin/bash
# User Data para WordPress + Docker

# Atualiza o sistema
sudo dnf update -y

# Instala o Docker
sudo dnf install -y docker

# Inicia o Docker
sudo systemctl enable docker
sudo systemctl start docker

# Adiciona o usuário ec2-user ao grupo Docker
sudo usermod -aG docker ec2-user

#Instala docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#Dá permissão de execução ao Docker-composse
sudo chmod +x /usr/local/bin/docker-compose

# Inicia o WordPress com Docker Compose
docker-compose up -d

```
