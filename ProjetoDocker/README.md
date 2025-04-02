# Projeto Docker
## Tecnologias utilizadas
AWS EC2

EC2 Amazon Linux 2023 AMI

AWS Cloudwatch

AWS EFS

AWS Aurora and RDS

MySQL Workbench 8.0.41 Community

## 1. Instalação e configuração do DOCKER com Wordpress no host EC2
### Criação da VPC
Dentro da AWS pesquise por VPC e clique na primeira opção, dentro da página clique no botão amarelo "Create VPC".

Na página de criação, na opção "Resources to create" marque a opção "VPC and more"

Na opção "Name tag auto-generation" escolhe um nome para VPC e deixe a opção "Auto-generate" marcada.

No restante das opções deixe padrão, com 2 zonas de disponibilidade, 2 subnet públicas e 2 privadas.

Imagem de como deve ficar o "Preview"

![Image](https://github.com/user-attachments/assets/6b9fdaf1-ebf8-43ff-8e64-f9d6024e1a9b)

---

### Criação do Security group
Dentro da AWS pesquise por EC2 e entre na página, dentro da página na esquerda procure por "Security Groups" e clique.

Clique no botão amarelo escirto "Create security group", na página de criação digite um nome para o SG e selecione a VPC criada com configurações padrão.

Na parte de "Inbound rules" clique em add rules e adicione estas regras:
```
Type: SSH  Port:22  Source type: My IP  #Para acessar a instância via SSH
Type: HTTP  Port:80  Source type: SG criado #Para o ALB
Type: HTTP  Port:80  Source type: My IP #Para acessar o Wordpress
Type: MySQL/Aurora  Port:3306 Source type: SG criado #Conexão Instância com banco de dados
Type: MySQL/Aurora  Port:3306 Source type: My IP #Conexão banco de dados
Type: NFS  Port:2049  Source type: SG criado #Para montagem do EFS
Type: HTTPS  Port:443  Source type:My IP #Caso necessário
```
Na parte de "Outbound rules" clique em add rules e adicione estas regras:
```
Type: All traffic  Port:All  Source type: Anywhere-IPv4 
```
Clique no botão amarelo "Create security group".

---

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

Em "Connectivity e security", copie o "Endpoint" pois ele será usado para acessar o BD pelo MySQL Workbench. Imagem da localização do endpoint abaixo.

![Image](https://github.com/user-attachments/assets/02ddb9f7-1b4b-43e6-9000-7a3abef3f9f4)

Abra o MySQL Workbench, e onde está escrito "MySQL Connections" clique no sinal de adição ao lado.

Em seguida irá abrir uma janela, e em "Connection Name" crie um nome para conexão.

Em "Hostname" cole o Endpoint copiado do BD, em "Username" coloque o login ID criado, e clique em "OK" abaixo, irá pedir a senha que você criou anteriormente. 

![Image](https://github.com/user-attachments/assets/8d0fb345-9eb5-4bd9-9d89-4bdffa48c10a)

Estando conectado ao BD, digite "CREATE DATABASE wordpress;" e depois clique no primeiro simbolo de raio.

![Image](https://github.com/user-attachments/assets/2cae46ff-ca04-4338-b772-368518a3cf3a)

Feito isto o BD estará pronto para ser usado no Wordpress.

---

### Criação do EFS
Na AWS pesquise por EFS e clique na primeira opção.

Na página do EFS clique no botão amarelo "Create file system" no canto superior direito.

Em "Create file system" digite um nome para o EFS e selecione a VPC criada anteriormente e depois clique no botão amarelo "Create file system".

Após criado salve o "File system ID", pois ele será usado no userdata da EC2. Print do file system ID abaixo.

![Image](https://github.com/user-attachments/assets/2e5ca25f-90e4-4254-9eda-7ac86d5a0b81)

---

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
sudo mount -t efs fs-0be7ffff46e2b08b1:/ /mnt/efs/wordpress/wp-content #Subustitua o fs-... pelo seu File System ID
echo "fs-0be7ffff46e2b08b1:/ /mnt/efs/wordpress/wp-content efs _netdev,tls,noresvport 0 0" | sudo tee -a /etc/fstab #Subustitua o fs-... pelo seu File System ID


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
      WORDPRESS_DB_USER: admin
      WORDPRESS_DB_PASSWORD: Teste123  # Substitua pela senha do RDS
      WORDPRESS_DB_NAME: wordpress #Nome da database criada no MySQL Workbench
    volumes:
      - /mnt/efs/wordpress/wp-content:/var/www/html/wp-content
EOF

#Instala docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#dá permissão ao Docker-composse
sudo chmod +x /usr/local/bin/docker-compose

sudo chown -R 33:33 /mnt/efs/wordpress/wp-content  # 33:33 é o UID/GID do usuário www-data no container
sudo chmod -R 755 /mnt/efs/wordpress/wp-content
sudo chown -R www-data:www-data /var/www/html/wp-content

# Inicia o WordPress com Docker Compose
sudo docker-compose up -d
```
Após a criar a instância, selecione a isntância e copie o "Public IPv4 adress" e cole na barra de pesquisa de seu navegador.

Se tudo estiver correto irá aparecer esta página de instalação do Wordpress.
![Image](https://github.com/user-attachments/assets/d3199f4a-bb50-49d1-a65c-8c3beff0abd7)

Não clique em nada nesta página. Exclua a instância para não gerar custos.

## 2. Configuração do Auto Scaling e Load Balancer 
### Load Balancer
Na página EC2 da AWS, clique em "Load Balancers" do lado esquerdo.

Clique no botão amarelo "Create load balancer".

Nesta tela clique aqui onde está apontado na imagem.
![Image](https://github.com/user-attachments/assets/ba342df5-09b3-4ba9-b062-d4d79d2aec27)

E na opção "Classic Load Balancer" clique em "Create".

Escolha um nome para o Load Balancer. E em "Network mapping" selecione a VPC criada anteriormente.

E na opção "Availability Zones and subnets" selecione as 2 subnets públicas disponíveis.

Em "Security groups" selecione o security group criado anteriormente.

Em "Health checks", na opção "Ping path" escreva "/wp-admin/install.php" que é o caminho para a página de instalação do Wordpress.

O restante das configurações deixe como padrão e clique no botão amarelo "Create load balancer" abaixo.

---

### Launch Template
Na página EC2 da AWS, clique em "Launch Templates" do lado esquerdo.

Em "Launch template name and description" coloque o nome do template e a versão pode deixar como "1".

Em "Application and OS Images" clique em "Click Start" e selecione a AMI Amazon Linux 2023.

Em "Instance type" selecione "t2.micro".

Em "Network settings", na opção "Security groups" selecione o security group criado anteriormente.

Abaixo clique em "Advanced details", e na opção "User data" coloque novamente o mesmo userdata usado para cirar a instância.

Deixe o restante das configurações como padrão e clique no botão amarelo "Create launch template" do lado esquerdo.

---

### Auto Scaling
Na página EC2 da AWS, clique em "Auto Scaling Groups" do lado esquerdo.

Clique no botão amarelo "Create Auto Scaling group".

Em "Choose launch template", escolha o nome do ASG e na opção "Launch template" selecione o template criado anteriormente. e Clique no botão amarelo "Next" abaixo.

Em "Network" na opção "VPC" selecione a VPC criada anteriormente.

Em "Availability Zones and subnets" selecione as 2 subnets públicas.

Em "Availability Zone distribution" selecione a opção "Balanced best effort".

Clique no botão amarelo "Next" abaixo.

Em "Load balancing" selecione a opção "Attach to an existing load balancer".

Em "Attach to an existing load balancer" selecione a opção "Choose from Classic Load Balancers" e depois selecione o Load Balancer criado anteriormente.

Deixe o restante como padrão e clique no botão amarelo "Next" abaixo.

Em "Group size" na opção "Desired capacity" coloque "2".

Em "Scaling" na opção "Min desired capacity" coloque "2" para definir o número mínimo de instâncias do ASG.

Em "Scaling" na opção "Max desired capacity" coloque "3" para definir o número máximo de instâncias do ASG.

Em "Automatic scaling" selecione a opção "Target tracking scaling policy".

Na opção "Metric type" deixe como "Average CPU utilization" para aumentar o número de instâncias baseado na utilização da CPU.

Na opção "Target value" deixe como "80" para aumentar o número de instâncias baseado nesta métrica criado de utilização de CPU a 80%.

Em "Additional settings", marque a opção "Enable group metrics collection within CloudWatch" para monitoramento cloudwatch.

Em "Additional settings", marque a opção "Enable default instance warmup" e deixe o valor como "300".

Deixe o restante como padrão e clique no botão amarelo "Next" abaixo.

Clique no botão amarelo "Next" novamente.

Clique no botão amarelo "Next" novamente. E clique no botão amarelo "Create Auto Scaling group" abaixo.

---

### Acesso ao Wordpress
Na página EC2 da AWS, clique em "Load Balancers" do lado esquerdo.

Na página do Load Balancer, copie o "DNS name" e cole no navegador para acessar o Wordpress.

Dentro do Wordpress irá aparecer a página de instalação novamente.

![Image](https://github.com/user-attachments/assets/d3199f4a-bb50-49d1-a65c-8c3beff0abd7)

Coloque seu idioma de preferência e os dados pedidos, e após isto você conseguirá usar normalmente. 

Após clicar em "Acessar" irá aparecer esta tela:

![Image](https://github.com/user-attachments/assets/743e8b9c-b810-4f3d-831d-0e9d75427b3e)

Após entrar com seu login irá aparecer está página:

![Image](https://github.com/user-attachments/assets/4e19e512-f651-4d48-bf08-7cd1a6c77b80)

Nesta interface você pode criar uma página clicando em "Adicionar nova página". Ao entrar novamente pelo DNS irá aparecer que você tenha criado.

![Image](https://github.com/user-attachments/assets/ae8796d4-5fef-491d-8f98-ac175b4487c4)

Sempre ao entrar no DNS do CLB será possível apenas visualizar as páginas criadas, para entrar no seu login criado para criar página adicione no final do DNS do CLB "/wp-admin" na barra de pesquisa.

---

### Alarme Cloudwatch
Na AWS pesquise por "Amazon Simple Notification Service" e clique na opção de "Services".

Dentro da página clique em "Topics" do lado esquerdo.

Clique em "aws-controltower-SecurityNotifications".

Clique no botão amarelo "Create subscription".

Na opção "Protocol" selecione o meio de receber o alerta, e logo depois em "Endpoint" selecione o destino do alerta. Depois clique no botão amarelo "Create subscription" abaixo.

Após isto pesquise na AWS por "Cloudwatch" e clique na primeira opção.

Dentro da página do Cloudwatch, clique na opção "In Alarm" do lado esquerdo.

Clique no botão amarelo "Create alarm".

Clique em "Select metric".

Selecione "EC2", depois "By Auto Scaling Group" e selecione o ASG criado com o "Metric name" "CPUUtilization", esta será a metrica usada nesta exemplo, você pode usar mais de uma. Depois clique no botão amarelo "Select metric" abaixo.

O padrão vai estar selecionado as opções "Static" e "Greater" deixe desta forma mesmo, e no valor a ser digitado digite 80, isto fará o alarme ser enviado quando a Utilização da CPU passar de 80%.

Clique no botão amarelo "Next" abaixo.

Nesta página irá estar marcada as opções "In alarm" e "Select an existing SNS topic", deixe desta forma.

Em "Send a notification to..." selecione a primeira opção disponível que deve ser "aws-controltower-SecurityNotifications" ou outra que você tenha criado por conta.

Clique no botão amarelo "Next" abaixo.

Escolha um nome para o alarme e digite o texto que irá aparecer na mensagem que você irá receber do alarme.

Clique no botão amarelo "Next" abaixo.

E clique no botão amarelo "Create alarm" abaixo.

## Conclusão
Este projeto aborda desde a criação de uma VPC e a configuração de um Security Group até a implantação de um EFS, um Banco de dados RDS, também aborda a criação de um Classic Load Balancer e um Auto Scaling Group rodando um container Docker com Wordpress. A implementação de um AS para lidar com uma maior demanda se necessário e alta disponibilidade do serviço.

Por fim, ao final deste projeto de forma prática consegui compreender melhor alguns conceitos da AWS, complementando os conhecimentos adquiridos durante o ProjetoLinux feito anteriormente e presente neste repositório.
