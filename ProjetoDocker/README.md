# Projeto Docker
## Tecnologias utilizadas
AWS EC2 Amazon Linux 2023 AMI

AWS EFS

AWS Aurora and RDS

Visual Studio Code 1.97.2

MySQL Workbench 8.0.41 Community

## 1. Instalação e configuração do DOCKER no host EC2
## Criação do Security group
```
Type: SSH  Port:22  Source type: Anywhere #Para acessar a instância via SSH
Type: HTTP  Port:80  Source type: Anywhere #Para acessar o Wordpress
Type: NFS  Port:2049  Source type: Anywhere #Conexão Instância com banco de dados
Type: SSH  Port:22  Source type: Anywhere
```
## Criação da instância EC2
Dentro da AWS pesquise por EC2 e entre na página, na página da EC2 clique no botão amarelo escrito "Launch instance".

Dentro da página de criação de instância,em "Application and OS Images (Amazon Machine Image)" selecione a imagem "Amazon Linux 2023 AMI"

Na opção "Key pair (login)" crie uma chave .pem ou use uma já criada

Em "Network settings" clique no botão "Edit", use uma VPC com configuração padrão, em "Auto-assign public IP" deixe como "Enable"

No "Firewall (security groups)" crie um security group com as regras Inbound:
