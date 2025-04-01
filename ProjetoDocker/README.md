# Projeto Docker
## Tecnologias utilizadas
AWS EC2 Amazon Linux 2023 AMI

AWS Cloudwatch

AWS EFS

AWS Aurora and RDS

Visual Studio Code 1.97.2

MySQL Workbench 8.0.41 Community

## 1. Instalação e configuração do DOCKER com Wordpress no host EC2
### Criação da VPC
Dentro da AWS pesquise por VPC e clique na primeira opção, dentro da página clique no botão amarelo "Create VPC".

Na página de criação, na opção "Resources to create" marque a opção "VPC and more"

Na opção "Name tag auto-generation" escolhe um nome para VPC e deixe a opção "Auto-generate" marcada.

No restante das opções deixe padrão, com 2 zonas de disponibilidade, 2 subnet públicas e 2 privadas.

### Criação do Security group
Dentro da AWS pesquise por EC2 e entre na página, dentro da página na esquerda procure por "Security Groups" e clique.

Clique no botão amarelo escirto "Create security group", na página de criação digite um nome para o SG e selecione a VPC criada com configurações padrão.

Na parte de "Inbound rules" clique em add rules e adicione estas regras:
```
Type: SSH  Port:22  Source type: My IP  #Para acessar a instância via SSH
Type: HTTP  Port:80  Source type: Anywhere-IPv4 #Para o ALB
Type: HTTP  Port:80  Source type: My IP #Para acessar o Wordpress
Type: MySQL/Aurora  Port:3306 Source type: SG das Instâncias #Conexão Instância com banco de dados
Type: MySQL/Aurora  Port:3306 Source type: My IP #Conexão banco de dados
Type: NFS  Port:2049  Source type: Anywhere-IPv4 #Para montagem do EFS
```
Na parte de "Outbound rules" clique em add rules e adicione estas regras:
```
Type: All traffic  Port:All  Source type: Anywhere-IPv4 
```
Clique no botão amarelo "Create security group".
### Criação do RDS
Pesquise por RDS na AWS e clique na opção "Aurora and RDS".

Dentro da página do RDS clique no botão amarelo "Create database".

Na página de criação na opção "Choose a database creation method" selecione "Standard create".

Na opção "Engine options" selecione a opção "MySQL".

Na opção "Templates" usarei "Free tier".

Na opção "Credentials Settings" crie um login ID ou use o padrão, e selecione a opção "Self managed" e crie uma senha para poder ter acesso ao banco de dados.

Em "Instance Configuration" selecione a opção "db.t3.micro".

Em "Connectivity" na opção "Virtual private cloud (VPC)" selecione a VPC criada anteriormente, e na opção "Public Acess" deixe como "Yes" para se conectar ao DB pelo MySQL Workbench.

Na opção "VPC security group (firewall)" selecione a opção "Choose existing" e abaixo selecione o security group criado anteriormente.

O restante deixe como padrão e ao final da página clique no botão amarelo "Create database".
### Criação da instância EC2
Dentro da AWS pesquise por EC2 e entre na página, na página da EC2 clique no botão amarelo escrito "Launch instance".

Dentro da página de criação de instância,em "Application and OS Images (Amazon Machine Image)" selecione a imagem "Amazon Linux 2023 AMI".

Na opção "Key pair (login)" crie uma chave .pem ou use uma já criada.

Em "Network settings" clique no botão "Edit", use uma VPC com configuração padrão, em "Auto-assign public IP" deixe como "Enable".

No "Firewall (security groups)" selecione o security group criado anteriormente.

Role para baixo e clique na opção "Advanced details" e em seguida procure a opção "User data - optional".

no campo de texto de User data, para criar uma instância com docker instalado insira:
```
#!/bin/bash

# Atualiza o sistema
sudo dnf update -y

# Instala o Docker
sudo dnf install -y docker

# Inicia o Docker
sudo systemctl enable docker
sudo systemctl start docker

# Adiciona o usuário ec2-user ao grupo Docker
sudo usermod -aG docker ec2-user

# Instala docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Dá permissão de execução ao Docker-composse
sudo chmod +x /usr/local/bin/docker-compose

# Inicia o WordPress com Docker Compose
docker-compose up -d
```
Após a criar a instância você pode acessá-la e usar um comando docker para confirmar que ele está instalado corretamente.
