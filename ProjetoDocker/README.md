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
### Criação do RDS e configuração
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

Após a criação do banco de dados, espere o status ficar como "Available", e clique no nome do seu BD.

Em "Connectivity e security", copie o "Endpoint" pois ele será usado para acessar o BD pelo MySQL Workbench.

Abra o MySQL Workbench, e onde está escrito "MySQL Connections" clique no sinal de adição ao lado.

Em seguida irá abrir uma janela, e em "Connection Name" crie um nome para conexão.

Em "Hostname" cole o Endpoint copiado do BD, em "Username" coloque o login ID criado, e clique em "OK" abaixo, irá pedir a senha que você criou anteriormente. 

![Image](https://github.com/user-attachments/assets/8d0fb345-9eb5-4bd9-9d89-4bdffa48c10a)

Estando conectado ao BD, digite "CREATE DATABASE wordpress;" e depois clique no primeiro simbolo de raio.

![Image](https://github.com/user-attachments/assets/2cae46ff-ca04-4338-b772-368518a3cf3a)

Feito isto o BD estará pronto para ser usado no Wordpress.

### Criação do EFS
Na AWS pesquise por EFS e clique na primeira opção.

Na página do EFS clique no botão amarelo "Create file system" no canto superior direito.

Em "Create file system" digite um nome para o EFS e selecione a VPC criada anteriormente e depois clique no botão amarelo "Create file system".
### Criação da instância EC2
Dentro da AWS pesquise por EC2 e entre na página, na página da EC2 clique no botão amarelo escrito "Launch instance".

Dentro da página de criação de instância,em "Application and OS Images (Amazon Machine Image)" selecione a imagem "Amazon Linux 2023 AMI".

Na opção "Key pair (login)" crie uma chave .pem ou use uma já criada.

Em "Network settings" clique no botão "Edit", use a VPC criada anteriormente, em "Auto-assign public IP" deixe como "Enable".

No "Firewall (security groups)" selecione o security group criado anteriormente.

Role para baixo e clique na opção "Advanced details" e em seguida procure a opção "User data - optional".

no campo de texto de User data, para criar uma instância com docker instalado insira:
```
#!/bin/bash

# Atualiza o sistema
sudo dnf update -y

# Instala o Docker
sudo dnf install -y docker

#Efs
sudo dnf install -y amazon-efs-utils nfs-utils
sudo mkdir -p /mnt/efs/wordpress/wp-content
sudo mount -t efs fs-0be7ffff46e2b08b1:/ /mnt/efs/wordpress/wp-content
echo "fs-0be7ffff46e2b08b1:/ /mnt/efs/wordpress/wp-content efs _netdev,tls,noresvport 0 0" | sudo tee -a /etc/fstab


# Inicia o Docker
sudo systemctl enable docker
sudo systemctl start docker

# Adiciona o usuário ec2-user ao grupo Docker
sudo usermod -aG docker ec2-user

# Cria um diretório para o WordPress
mkdir -p /home/ec2-user/wordpress
cd /home/ec2-user/wordpress

# Cria o docker-compose.yml
cat <<EOF > docker-compose.yml
version: '3'
services:
  wordpress:
    image: wordpress:latest
    user: "33:33"
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: database-1.czk86sq22bgw.us-east-1.rds.amazonaws.com  # Substitua pelo endpoint do RDS
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: Teste123  # Substitua pela senha do RDS
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - /mnt/efs/wordpress/wp-content:/var/www/html/wp-content
EOF

#Instala docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#dá permissão ao Docker-composse
sudo chmod +x /usr/local/bin/docker-compose

sudo chown -R 33:33 /mnt/efs/wordpress/wp-content  # 33:33 é o UID/GID do usuário www-data no container
sudo chmod -R 755 /mnt/efs/wordpress/wp-content

# Inicia o WordPress com Docker Compose
docker-compose up -d
```
Após a criar a instância você pode acessá-la e usar um comando docker para confirmar que ele está instalado corretamente.
