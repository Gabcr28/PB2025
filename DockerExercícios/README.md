# Atividades Docker

## 1. Rodando um container básico
Instruções:

- Execute um container usando a imagem do Nginx e acesse a página padrão no navegador

- Exemplo de aplicação: Use a landing.page do TailwindCSS como site estático dentro do container.

- Resolução:

1.1 - Fiz o downlaod de um template de um página estática.
  
1.2 - Arquivo Dockerfile na mesma pasta do template
  ```Dockerfile
FROM nginx:latest

COPY . /usr/share/nginx/html

EXPOSE 80  
  ```
1.3 - dentro do terminal do Docker Desktop dentro do diretório do Dockerfile
```
docker build -t atnginx /diretorio/do/dockerfile

docker run -d -p 8080:80 atnginx
```
1.4 - Acessar a página pelo navegador
```
http://localhost:8080
```

## 2. Criando e rodando um container interativo
Instruções:
- Inicie um container Ubuntu e interaja com o terminal dele
- Exemplo de aplicação: Teste um script Bash que imprime logs do sistema ou instala pacotes de forma interativa

2.1 - Criar um diretório com um arquivo Dockerfile com o conteúdo:
```Dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y nano
```
2.2 - No terminal do Docker Desktop ir até o terminal e ir até o diretório criado com o arquivo Dockerfile.
```
docker build -t atv02 .

docker run -it atv02
```
2.3 - Dentro do container criar o script bash de verificação de logs
```bash
nano meuscript.sh
```
```bash
#!/bin/bash

# Script simples para mostrar o conteúdo dos arquivos em /var/log
echo "Mostrando o conteúdo dos arquivos em /var/log:"
for file in /var/log/*; do
    if [ -f "$file" ]; then
        echo "=== Conteúdo de $file ==="
        cat "$file"
        echo -e "\n"  # Adiciona uma linha em branco entre os arquivos
    fi
done
```
2.4 - Permissão de execução
```
chmod +x meuscript.sh
```
2.5 - Rodar script
```
./meuscript.sh
```

## 3. Listando e removendo containers
Instruções:
- Liste todos os containers em execução e parados, pare um container em execução e remova um container específico.
- Exemplo de aplicação: Gerenciar containers de testes criados para verificar configurações e dependências

3.1 - Listar no terminal todos os containers em execução e parados
```
docker ps -a
```

3.2 - Parar um container
```
docker stop id_container
```
3.3 - Remover um container parado
```
docker rm id_container
```
3.4 - Inspecionar configurações do container
```
docker inspect id_container
```

## 4. Criando um Dockerfile para uma aplicação simples em Python
Instruções:
- Crie um Dockerfile para uma aplicação Flask que retorna uma mensagem ao acessar um endpoint
- Exemplo de aplicação: Use a API de exemplo Flask Restful API Starter para criar um endpoint de teste

4.1 - Criar um diretório e dentro dele criei um arquivo chamado app.py com o conteúdo:
```
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {'message': 'Docker OK!'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')

```

4.2 - No mesmo diretório criei o arquivo requirements.txt com o conteúdo:
```
Flask
Flask-RESTful
```
4.3 - Na mesma pasta criar arquivo Dockerfile com o conteúdo:
```Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

4.4 - No Docker Desktop no diretório do Dockerfile:
```
docker build -t flaskapp .
docker run -p 5000:5000 flaskapp
```
4.5 - Acessar pelo navegador:
```
http://localhost:5000/
```

## 5. Criando e utilizando volumes para persistência de dados
Instruções:
- Execute um container MySQL e configure um volume para armazenar os dados do banco de forma persistente.
- Exemplo de aplicação: Use o sistema de login e cadastro do Laravel Breeze, que usa MySQL.

5.1 - Baixar imagem MySQL no terminal Docker Desktop:
```
docker pull mysql
```

5.2 - Criar volume no terminal do Docker Desktop:
```
docker volume create mysql-data
```

5.3 - Iniciar container MySQL com o volume criado:
```
docker run -d --name mysql-ct -e MYSQL_ROOT_PASSWORD=senha -v mysql-data:/var/lib/mysql mysql
```

5.4 - Verificar quais containers estão usando o volume criado:
```
docker ps -a --filter volume=mysql-data
```

OBS: Não consegui realizar o exemplo de aplicação

## 6. Criando e rodando um container multi-stage
Instruções:
- Utilize um multi-stage build para otimizar uma aplicação Go, reduzindo o tamanho da imagem final.
- Exemplo de aplicação: Compile e rode a API do Go Fiber Example dentro do container.

6.1 - No terminal Docker Desktop baixar imagens
```
docker pull golang
docker pull alpine
```

6.2 - Criar uma pasta e dentro dela criar um arquivo app.go com o conteúdo:
```Go
package main
import (
    "fmt"
)

func main() {
  fmt.Println("Qual é o seu nome:? ")
  var name string
  fmt.Scanln(&name)
  fmt.Printf("Oi, %s! Eu sou a linguagem Go! ", name)
}
```

6.3 - Na pasta do app.go criar um arquivo Dockerfile com o conteúdo:
```Dockerfile
FROM golang as exec

COPY app.go /go/src/app/

ENV GO111MODULE=auto

WORKDIR /go/src/app

RUN go build -o app.go .

FROM alpine

WORKDIR /appexec
COPY --from=exec /go/src/app/ /appexec
RUN chmod -R 755 /appexec
ENTRYPOINT ./app.go
```

6.4 - Dentro do terminal no diretório do Dockerfile:
```
docker image build -t appgo .
docker run -ti --name meugo appgo
```

OBS: o link do repositório Go Fiber Exemple está dando a mensagem "404 - page not found"

## 7. Construindo uma rede Docker para comunicação entre containers
Instruções:
- Crie uma rede Docker personalizada e faça dois containers, um Node.js e um MongoDB, se comunicarem
- Exemplo de aplicação: utilize o projeto MEAN Todos para criar um app de tarefas usando Node.js + MongoDB

7.1 - Criar rede no terminal Docker Desktop
```
docker network create minha-rede
```

7.2 - Crie uma pasta e dentro dela crie o arquivo Dockerfile:
```Dockerfile
FROM node:14

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

7.3 - Criar arquivo docker-compose.yml dentro do diretório do Dockerfile com o conteúdo:
```YAML
version: '3'
services:
  node-app:
    build: .
    ports:
      - "3000:3000"
    networks:
      - minha-rede
    depends_on:
      - mongo

  mongo:
    image: mongo
    ports:
      - "27017:27017"
    networks:
      - minha-rede

networks:
  minha-rede:
    external: true

#dentro do diretorio cirar app.js
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const port = 3000;

// Conectar ao MongoDB
mongoose.connect('mongodb://mongo:27017/todos', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

const db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', () => {
  console.log('Connected to MongoDB');
});

// Definir um modelo simples para as tarefas
const todoSchema = new mongoose.Schema({
  task: String,
  completed: Boolean
});

const Todo = mongoose.model('Todo', todoSchema);

// Middleware para parsear JSON
app.use(express.json());

// Rota para adicionar uma nova tarefa
app.post('/todos', async (req, res) => {
  const todo = new Todo(req.body);
  try {
    await todo.save();
    res.status(201).send(todo);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Rota para listar todas as tarefas
app.get('/todos', async (req, res) => {
  try {
    const todos = await Todo.find();
    res.status(200).send(todos);
  } catch (error) {
    res.status(500).send(error);
  }
});

// Iniciar o servidor
app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

7.4 - Criar arquivo package.json dentro do diretório do Dockerfile com o conteúdo:
```JSON
{
  "name": "mean-todos",
  "version": "1.0.0",
  "description": "A simple MEAN stack todo app",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "express": "^4.17.1",
    "mongoose": "^6.0.12"
  }
}
```

7.5 - Cronstruir e iniciar conatiner
```
docker-compose up --build
```

7.6 - Acessar no navegador:
```
http://localhost:3000/todos
```

7.7 - Enviar requisição para ser salva no banco de dados e exibida acessando o link acima:
```
curl -X POST http://localhost:3000/todos -H "Content-Type: application/json" -d "{\"task\": \"Comprar leite\", \"completed\": false}"
```

OBS: Link do MEAN Todos não encontra repositório

## 8. Criando um compose file para rodar uma aplicação com banco de dados
Instruções:
- Utilize Docker Compose para configurar uma aplicação Django com um banco de dados PostgreSQL.
- Exemplo de aplicação: Use o projeto Django Polls App para criar uma pesquisa de opinião integrada ao banco.

8.1 - Clonei o repositório Git pelo terminal:
```
git clone https://github.com/devmahmud/Django-Poll-App.git
```

8.2 - Dentro da pasta do repositório Git criar arquivo 
