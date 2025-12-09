# Pipeline de Deploy Aplicação Web Estática:

## Visão Geral e Objetivo

O objetivo principal foi aplicar conhecimentos básicos de DevOps para automatizar o deploy de uma aplicação web estática contida em um Container Docker para uma VM no AWS EC2, orquestrado pela Action appleboy/ssh-action@master.

### Conceitos e Ferramentas Exploradas

A realização deste projeto permitiu o contato prático com os seguintes conceitos e ferramentas:

- **Empacotamento:** Docker
- **Cloud e Infraestrutura:** AWS EC2, Security Group
- **Sistema Operacional:** Linux, Shell Script
- **Automação:** CI/CD, GitHub Actions
- **Redes:** Configuração de Portas e Acesso SSH

### Arquitetura do Projeto

Abaixo está o diagrama que ilustra o fluxo da aplicação e do deploy:

![Diagrama da arquitetura de deploy: GitHub Actions envia o container Docker para o AWS EC2 via SSH.](/imagens/image.png)

---

## Detalhamento das Etapas Técnicas

### 1. Construção da Aplicação e Imagem Docker

#### 1.1 Aplicação Web

O projeto utiliza um `index.html` simples com um título e a imagem de arquitetura. O arquivo fica em um diretório local, junto aos demais arquivos de configuração.

#### 1.2 Dockerfile e Empacotamento

- O **`Dockerfile`** utiliza a imagem oficial `httpd:2.4` (Apache HTTP Server).
- Esta imagem já inicializa o Apache como um serviço em _foreground_ e serve arquivos estáticos a partir do diretório `/usr/local/apache2/htdocs/`.
- O `Dockerfile` é responsável por **copiar** o `index.html` e a pasta `/imagens` para o local correto dentro do container.
- O arquivo **`.dockerignore`** foi usado para garantir que arquivos desnecessários (`README.md`, `.gitignore`, etc.) não fossem copiados, mantendo a imagem **leve** e focada no essencial.

#### 1.3 Teste Local

Para validação inicial, a imagem foi construída (`docker build`) e o container foi executado (`docker run`), mapeando a porta local (`8080`) para a porta do container, permitindo o teste via `localhost:8080`.

### 2. Configuração na AWS

- Uma **VM (EC2)** na AWS foi criada.
- No **Security Group** da instância, a porta **8080** foi liberada (Inbound) para permitir o acesso externo à aplicação.
- Inicialmente, o repositório foi clonado na VM e os comandos de `docker build` e `docker run` foram executados manualmente para garantir que a aplicação funcionava corretamente em nuvem.

### 3. Automação do Deploy (CI/CD com GitHub Actions)

Para eliminar a necessidade de acesso manual à VM a cada atualização, foi criado um _workflow_ no arquivo `.github/workflows/deploy.yml`:

- O fluxo é ativado a cada **commit** para a _branch_ `main`.
- A Action **`appleboy/ssh-action@master`** é utilizada para estabelecer uma conexão SSH segura com a EC2.
- Parâmetros de conexão (**HOST**, **USER** e **SSH Private Key**) foram armazenados como **Secrets** no GitHub para segurança.
- **Pares de chave SSH** foram gerados, com a chave pública na VM e a chave privada no GitHub Secrets.

#### Sequência de Comandos Executados na EC2

Após o login via SSH, o fluxo executa a seguinte sequência de comandos para o deploy:

1.  **`git pull origin main`**: Atualiza o código na VM.
2.  **`docker stop [nome_container]`**: Para o container atualmente em execução.
3.  **`docker rm [nome_container]`**: Remove o container antigo.
4.  **`docker build -t [nome_imagem] .`**: Constrói a nova imagem com o código atualizado.
5.  **`docker run ...`**: Executa o novo container, mapeando as portas.

Este fluxo garante que o deploy seja **totalmente automatizado** após cada _push_ no GitHub.

---

## Como Testar Localmente

1.  Clone este repositório.
2.  Execute o comando para construir a imagem Docker:
    ```bash
    docker build -t minha-web-app .
    ```
3.  Execute o comando para rodar o container, mapeando a porta 8080:
    ```bash
    docker run -d -p 8080:80 minha-web-app
    ```
4.  Acesse a aplicação em `http://localhost:8080`.
