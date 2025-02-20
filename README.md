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

Em "key pair(login)" selecione uma chave .pem ou crie uma caso não tenha uma já criada e faça o download da chave .pem em sua máquina.

Em "Network settings" selecione a VPC criada e uma subnet pública da VPC criada, deixe "Enable" a opção "Auto-assign public IP" para ser possível acessar a instância via SSH e selecione um Security Group Com as regras de Inbound e Outbound com HTTP na porta 80 para Anywhere-IPv4 e SSH na porta 22 para Anywhere-IPv4.

### Acessar a instância via SSH:
Para acessar a instância via SSH primeiro se deve alterar as permissões da chave .pem para somente leitura e na página da AWS "Instances" clique no botão "Connect" que fica na parte de cima, e clique na opção "SSH client" e copie o "Example" para colar posteriormente no VS Code.

No Visual Studio Code instale a extensão "Remote - SSH" disponibilizado pela Microsoft, e abra um novo terminal e colo o código copiado. Exemplo:
```
ssh -i "caminho/para/sua/chave.pem" ec2-user@ec2-44-199-191-13.compute-1.amazonaws.com
```
Após colar no VS Code com o caminho da sua chave.pem e ela estando com a permisão de somente leitura, irá aparecer a uma opção para você digitar "yes" e depois você estará conectado.

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
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meu site com Nginx</title>
</head>
<body>
    <h1>Olá, mundo!</h1>
    <p>Este é um site servido pelo Nginx.</p>
</body>
</html>
```
Após salvar seu arquvio HTML, use este comando para reiniciar o Nginx para atualizar sua página modificada:
```
sudo systemctl restart nginx
```
Agora dentro da AWS na página "Instances" clique na caixa de seleção da sua Instância e copie o "Public IPv4 Address" e cole na barra de pesquisa do seu navegador e deverá abrir a página HTML que você colocou no diretório /usr/share/nginx/html/index.html. Print da localização do "Public IPv4 Address"
![Print](https://github.com/Gabcr28/ProjetoLinuxPB/blob/main/Print/40.png)
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
```bash
#!/bin/bash

# Configuração
URL="http://localhost"  # Altere para a URL do seu site

# Verifica se o site responde corretamente
if curl -s --head --request GET $URL | grep "200 OK" > /dev/null
else
    sudo systemctl restart nginx
fi
```
E para fazer o arquvio criada para rodar a cada minuto, digite:
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
## Etapa 3: Monitoramento e Notificações Tarefas:
### Criar um script em Bash para monitorar a disponibilidade do site:
Para criar um script de monitoramento podemos apenas modificar o arquivo reinicio.sh:
```
sudo nano /usr/local/bin/renicio.sh
```
```bash
#!/bin/bash

# Configuração
URL="http://localhost"  # Altere para a URL do seu site
LOG_FILE="/var/log/monitoramento.log"

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

No terminal edite o arquivo reinicio.sh
```
sudo nano /usr/local/bin/renicio.sh
```
```bash
#!/bin/bash

# Configuração
URL="http://localhost"  # Altere para a URL do seu site
LOG_FILE="/var/log/monitoramento.log"
DISCORD_WEBHOOK="https://discord.com/api/webhooks/SEU_WEBHOOK"

# Função para enviar alerta para o Discord
enviar_notificacao() {
    mensagem="$1"
    curl -H "Content-Type: application/json" -X POST -d "{\"content\": \"$mensagem\"}" $DISCORD_WEBHOOK
}

# Verifica se o site responde corretamente
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
![Print](https://github.com/Gabcr28/ProjetoLinuxPB/blob/main/Print/41.png)
