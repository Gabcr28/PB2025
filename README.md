# Projeto: Configuração de Servidor Web com Monitoramento
Objetivo: Desenvolver e testar habilidades em Linux, AWS e automação de processos através da configuração de um ambiente de servidor web monitorado.

## Etapa 1: Configuração do Ambiente Tarefas:
### Criar uma VPC na AWS:
Entre na sua conta AWS e pesquise o serviço VPC, entrando na página irá aparecer um botão amarelo escrito "Create VPC" clique nele.

Na tela de criação da VPC na opção "Resources to create" selecione a opção "VPC and more".

Na opção "Name tag auto-generation" deixe a opção "Auto-generate" e preencha o campo com o nome da sua VPC para nomear automaticamente com o mesmo nome a VPC, Subnets, Route tables e Network connections.

Para este projeto deixe o restante das configurações como padrão, e clique em "Create VPC" e a VPC estará pronta para ser usada.

### Criar uma instância EC2 na AWS:
Dentro da AWS pesquise pelo serviço EC2, entrando na página clique na parte esquerda da tela a opção "Instances" e carregando a página clique no botão amarelo escrito "Launch instances" no canto superior direito.

Após nomear sua instância, na parte de "Application and OS Images (Amazon Machine Image)" selecione o Amazon Linux 2023 AMI que será o utilizado neste projeto.

Em "Instance type" será utilizado "t2.micro".

Em "key pair(login)" selecione uma chave .pem ou crie uma caso não tenha uma já criada.

Em "Network settings" selecione a VPC criada e uma subnet pública da VPC criada, deixe "Enable" a opção "Auto-assign public IP" para ser possível acessar a instância via SSH e selecione um Security Group Com as regras de Inbound com HTTP na porta 80 para Anywhere-IPv4 e SSH na porta 22 para Anywhere-IPv4

