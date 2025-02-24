# Projeto: Configuração de Servidor Web com Monitoramento
Objetivo: Desenvolver e testar habilidades em Linux, AWS e automação de processos através da configuração de um ambiente de servidor web monitorado.

## Tecnologias Utilizadas
AWS EC2 Amazon Linux 2023 AMI

Visual Studio Code 1.97.2

Serviços Nginx 1.26.2 e Cron na instância EC2

## Etapa 1: Configuração do Ambiente Tarefas:
### Criar uma VPC na AWS:
Entre na sua conta AWS e pesquise o serviço VPC, entrando na página irá aparecer um botão amarelo escrito "Create VPC" clique nele.

Na tela de criação da VPC na opção "Resources to create" selecione a opção "VPC and more".

Na opção "Name tag auto-generation" deixe a opção "Auto-generate" e preencha o campo com o nome da sua VPC para nomear automaticamente com o mesmo nome a VPC, Subnets, Route tables e Network connections.

Para este projeto deixe o restante das configurações como padrão, e clique em "Create VPC" e a VPC estará pronta para ser usada.

### Criar um Security Group na AWS:
Dentro da AWS pesquise pelo serviço EC2, entrando na página clique na parte esquerda da tela no tópico "Network Settings" na opção "Security Group".

Clique no botão amarelo no canto superior direito da tela escrito "Create security group". Entrando na página de criação do Security Group, para ser possível acessar a instância via SSH e selecione um Security Group Com as regras de Inbound e Outbound com HTTP na porta 80 para Anywhere-IPv4 e SSH na porta 22 para Anywhere-IPv4. 

### Criar uma instância EC2 na AWS:
Dentro da AWS pesquise pelo serviço EC2, entrando na página clique na parte esquerda da tela na opção "Instances" e carregando a página clique no botão amarelo escrito "Launch instances" no canto superior direito.

Após nomear sua instância, na parte de "Application and OS Images (Amazon Machine Image)" selecione o Amazon Linux 2023 AMI que será o utilizado neste projeto.

Em "Instance type" será utilizado "t2.micro".

Em "key pair(login)" selecione uma chave .pem ou crie uma caso não tenha uma já criada e faça o download da chave .pem em sua máquina.

Em "Network settings" selecione a VPC criada e uma subnet pública da VPC criada, deixe "Enable" a opção "Auto-assign public IP" para ser possível acessar a instância via SSH e selecione a Security Group criada anteriormente com as regras de Inbound e Outbound com HTTP na porta 80 para Anywhere-IPv4 e SSH na porta 22 para Anywhere-IPv4. 


Obs: Caso você queria usar a implementação de UserData na criação da instância e já instalar com Nginx, Cron, HTML simples e script de monitoramento vá até o tópico "Bônus", adiante estas configurações serão instaladas manualmente

E após isto clique em "Launch Instance"

E após lançar a instância, para acessar a página da sua instância copie este IP e jogue na barra de pesquisa do seu navegador, no momento não vai aparecer nada pois o Nginx ainda não foi instalado, mas será usado posteriormente para verificar a página HTML criada.
![Image](https://github.com/user-attachments/assets/1dd7a089-91ca-4864-8a85-adc7ddef2ca6)


### Acessar a instância via SSH:
Para acessar a instância via SSH primeiro se deve alterar as permissões da chave .pem para somente leitura e na página da AWS "Instances" clique no botão "Connect" que fica na parte de cima, e clique na opção "SSH client" e copie o "Example" para colar posteriormente no VS Code.

No Visual Studio Code instale a extensão "Remote - SSH" disponibilizado pela Microsoft, e abra um novo terminal e cole o código copiado. Exemplo:
```
ssh -i "caminho/para/sua/chave.pem" ec2-user@ec2-44-199-191-13.compute-1.amazonaws.com
```
Após colar no VS Code com o caminho da sua chave.pem e ela estando com a permisão de somente leitura, irá aparecer a uma opção para você digitar "yes" e depois você estará conectado. Print:

![Image](https://github.com/user-attachments/assets/649870fa-f122-4224-80d0-56cea7037596)


## Etapa 2: Configuração do Servidor Web:
### Instalar o servidor Nginx na EC2:
Estando conectado na Instância primeiro é preciso atualizar os pacotes do sistema, utilize o seguinte comando no terminal:
```
sudo yum update -y
```
Após utilizar este comando já é possível instalar o Nginx, utilize este comando no terminal:
```
sudo yum install nginx -y
```
Após instalar o Nginx é necessário inicia-lo com o comando:
```
sudo systemctl start nginx
```
Para habilitar o Nginx para iniciar com o sistema:
```
sudo systemctl enable nginx
```
E para verificar onde devemos inserir o arquivo da página HTML podemos ver as configurações do Nginx para ver o diretório correto, para ver o diretório do arquivo das configurações do Nginx use o comando:
```
sudo nginx -t
```
Entrando no arquivo de configurações do Nginx localize esta parte:
```
 server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html; server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;
```
E no exemplo acima o caminho correto para salvar o arquivo HTML é /usr/share/nginx/html e dentro do diretório irá ter um exemplo de página HTML do próprio Nginx, você pode modificar o arquivo com sua página ou exclui-lo e adicionar sua próprio arquivo HTML.

Neste exemplo será modificado o arquvio index.html que vem por padrão do Nginx:
```
sudo nano /usr/share/nginx/html/index.html
```
```hmtl
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Servidor Nginx na Amazon EC2</title>
</head>
<body>
    <h1>Sucesso! Nginx está funcionando na Amazon EC2!</h1>
</body>
</html>
```
Após salvar seu arquvio HTML, use este comando para reiniciar o Nginx para atualizar sua página modificada:
```
sudo systemctl restart nginx
```
Agora dentro da AWS na página "Instances" clique na caixa de seleção da sua Instância e copie o "Public IPv4 Address" igual mostrado em um print anteriormente e cole na barra de pesquisa do seu navegador e deverá abrir a página HTML que você colocou no diretório /usr/share/nginx/html/index.html. Print da localização do "Public IPv4 Address"

Após colar o IPv4 da sua Instância na barra de pesquisa do navegador, a página irá abrir desta forma:
![Image](https://github.com/user-attachments/assets/69905e7b-3d73-4de1-ac5e-da7d21d2f81c)
Após isto vamos configurar o Nginx para reiniciar automaticamente a cada minuto. Primeiro instale o cronie:
```
sudo yum install cronie -y
```
Depois habilite o cronie:
```
sudo systemctl enable crond
sudo systemctl start crond
```
Após habilitar o crond iremos criar um arquivo .sh para rodar a cada minuto no crontab, exemplo:
```
sudo nano /usr/local/bin/renicio.sh
```
Conteúdo a ser inserido no arquivo "reinicio.sh"
```bash
#!/bin/bash

# Configuração
URL="http://localhost"  # Altere para a URL do seu site

# Verifica se o site responde corretamente e reinicia o nginx caso necessário
if curl -s --head --request GET $URL | grep "200 OK" > /dev/null
else
    sudo systemctl restart nginx
fi
```
E para fazer o arquvio criado rodar a cada minuto, digite:
```
crontab -e
```
E no conteúdo do contrab -e para reiniciar a cada minuto, coloque:
```
* * * * * /usr/local/bin/renicio.sh
```
E de permissão para execução:
```
sudo chmod +x /usr/local/bin/reinicio.sh
```
E para testar se está funcionando, dê o comando:
```
sudo systemctl stop nginx
```
E verifique se o nginx reinicia automaticamente.
## Etapa 3: Monitoramento e Notificações:
### Criar um script em Bash para monitorar a disponibilidade do site:
Para criar um script de monitoramento podemos apenas modificar o arquivo reinicio.sh:
```
sudo nano /usr/local/bin/renicio.sh
```
Contéudo do arquivo "reinicio.sh" após modificação:
```bash
#!/bin/bash

# Configuração
URL="http://localhost"  # Altere para a URL do seu site
LOG_FILE="/var/log/monitoramento.log" # Caminho para o script de monitoramento

# Verifica se o site responde corretamente
if curl -s --head --request GET $URL | grep "200 OK" > /dev/null
then
    echo "$(date) - O site está online." >> $LOG_FILE
else
    echo "$(date) - O site está fora do ar! Reiniciando Nginx..." >> $LOG_FILE
    sudo systemctl restart nginx
fi
```
E verifique se o nginx reinicia automaticamente e você também pode verificar o caminho do arquivo log que criamos no caminho /var/log/monitoramento.log e caso não esteja criado crie e deixe com permissão:
```
sudo chmod 666 /var/log/monitoramento.log
```
### Enviar uma notificação via Discord se detectar indisponibilidade.
Para adicionarmos essa funcionalidade primeiro abra seu discord, clique com o botão esquerdo do mouse no seu servidor, vá em configurações do servidor e depois clique em integrações. Depois clique em Webhooks, crie um Webhook caso não tenha e selecione o canal que ele irá enviar as notificações e copie o URL.

No terminal edite novamente o arquivo reinicio.sh
```
sudo nano /usr/local/bin/renicio.sh
```
Arquivo "reinicio.sh" modificado:
```bash
#!/bin/bash

# Configuração
URL="http://localhost"  # Altere para a URL do seu site
LOG_FILE="/var/log/monitoramento.log" # Caminho para o script de monitoramento
DISCORD_WEBHOOK="https://discord.com/api/webhooks/SEU_WEBHOOK" # Altere para o URL do seu Discord Webhook

# Função para enviar alerta para o Discord
enviar_notificacao() {
    mensagem="$1"
    curl -H "Content-Type: application/json" -X POST -d "{\"content\": \"$mensagem\"}" $DISCORD_WEBHOOK
}

# Verifica se o site responde corretamente e reinicia o nginx caso necessário
if curl -s --head --request GET $URL | grep "200 OK" > /dev/null
then
    echo "$(date) - O site está online." >> $LOG_FILE
else
    echo "$(date) - O site está fora do ar! Reiniciando Nginx..." >> $LOG_FILE
    sudo systemctl restart nginx
    enviar_notificacao "❌ Alerta: O site não está respondendo! Nginx foi reiniciado."
fi
```
Após este passo para testar o Webhook dê o camando para parar o Nginx:
```
sudo systemctl stop nginx
```
E verifique no Discord se o Webhook está respondendo no canal do seu servidor escolhido, Exemplo:
![Image](https://github.com/user-attachments/assets/67884914-3ce5-45c9-b1bf-a1f18363bc3e)

## Bônus (Opcional):
### Automação com User Data:
Configurar a EC2 para já iniciar com Nginx, HTML e script de monitoramento via User Data.
Na tela de criação de instância após fazer o restante das configurações no tópico de criação de instância, clique em "Advanced details"
![Image](https://github.com/user-attachments/assets/ae4cd0f1-6d70-4279-a40e-954fbbd41984)

e nesta caixa de texto coloque este script bash
```bash
#!/bin/bash
# Atualizar o sistema
yum update -y

# Instalar Nginx
amazon-linux-extras enable nginx1 -y
yum install nginx -y

# Iniciar e habilitar o serviço Nginx
systemctl enable nginx
systemctl start nginx

#Instalar cronie
yum install cronie -y

# Iniciar e habilitar o serviço Cronie
systemctl enable cronie
systemctl start cronie

# Criar uma página HTML simples
cat <<EOF > /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Servidor Nginx na Amazon EC2</title>
</head>
<body>
    <h1>Sucesso! Nginx está funcionando na Amazon EC2!</h1>
</body>
</html>
EOF

# Garantir permissão de leitura no arquivo HTML
chmod 644 /usr/share/nginx/html/index.html

# Criar script de monitoramento
cat <<EOF > /usr/local/bin/monitor_nginx.sh
#!/bin/bash
# Verifica se o Nginx está rodando e reinicia se necessário
if systemctl is-active --quiet nginx; then
    echo "\$(date): Nginx está funcionando." >> /var/log/monitor_nginx.log
else
    echo "\$(date): Nginx caiu, reiniciando..." >> /var/log/monitor_nginx.log
    sudo systemctl restart nginx
fi
EOF

# Tornar o script executável
chmod +x /usr/local/bin/monitor_nginx.sh

# Adicionar o script ao cron para rodar a cada 1 minuto
(crontab -l 2>/dev/null; echo "* * * * * /usr/local/bin/monitor_nginx.sh") | crontab -

# Reiniciar o serviço cron para aplicar a nova configuração
systemctl restart crond
```

## Conclusão:
Neste projeto foi guiada a criação de uma VPC simples, um Security Group e uma EC2 dentro da AWS e configurado dentro da EC2 um servidor Nginx com script de monitoramento e webhook via Discord e mostrando o passo a passo da criação e configuração da instância e com opção da implementação do User Data.
