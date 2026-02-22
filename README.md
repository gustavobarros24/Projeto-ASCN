# Execução dos playbooks

Para executar os playbooks do Ansible, é necessário indicar o caminho para o inventory e a senha do vault.

```sh
ansible-playbook <playbook> -i inventory --ask-vault-pass
```

# 1. Instalacao manual no node1

primeiro tentei instalar manualmente a aplicacao no node1

## requirements

1. Software
   1.1. Docker
   1.2. Docker Compose
2. Hardware
   2.1. 2 CPU cores
   2.2. 2 GB RAM
   2.3. 10 GB disk space

## Manual instalation

follow this guide https://airtrail.johan.ohly.dk/docs/install/manual

### requirements

- git
- node.js
- Bun
- PostgresSQL

### passo a passo

**instalar ferramentas**

1. Nodejs e npm `sudo apt install nodejs npm`
2. Bun `curl -fsSL https://bun.com/install | bash`
   2.1 deu erro: `error: unzip is required to install bun`
   2.2 `sudo apt install unzip`
   2.3 retry installation

**Instalar o docker**
Comecei por instalar normalmente o docker seguindo a documentação
https://docs.docker.com/engine/install/ubuntu/
https://docs.docker.com/engine/install/linux-postinstall/

**Criar base de dados**

1. `docker image pull postgres:latest`
2. `docker network create airtrail-network`
3. Executar o container da base de dados postgres

```
docker run --name airtrail-pg --net airtrail-network -p 5432:5432 -d \
-e POSTGRES_USER=airtrail -e POSTGRES_PASSWORD=password \
-e POSTGRES_DB=airtrail \
-v pgdata:/var/lib/postgresql \
postgres:latest
```

**Instalar aplicação web**

1. clonar o repositório `git clone https://github.com/JohanOhly/AirTrail.git`
2. `cd AirTrail`
3. `bun install`
4. `cp .env.example .env`
5. Alterar a DB_URL para localhost: `DB_URL=postgres://airtrail:password@localhost:5432/airtrail`
6. `bun run build`
   5.1 Por algum motivo, o build dá erro por falta de espaço na HEAP: `FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory`
   5.2 A solução encontrada foi: `NODE_OPTIONS="--max-old-space-size=4096" bun run build`

# Instalação com Docker

O bun possui uma imagem oficial. A utilizada foi a `oven/bun:1`

## Passo a passo

**Criar base de dados**

1. `docker image pull postgres:latest`
2. `docker network create airtrail-network`
3. Executar o container da base de dados postgres

```
docker run --name airtrail-pg --net airtrail-network -p 5432:5432 -d \
-e POSTGRES_USER=airtrail -e POSTGRES_PASSWORD=password \
-e POSTGRES_DB=airtrail \
-v pgdata:/var/lib/postgresql \
postgres:latest
```

**Criar container da aplicação web**

1. Copia o `Dockerfile` para dentro da VM
2. Criar imagem `docker build -t airtrail-web .`
3. Executar container
   💡 Atenção à variável `ORIGIN`, altere conforme o IP da máquina virtual utilizada

```sh
docker run --net airtrail-network -p 3000:3000 -d -e DB_URL=postgres://airtrail:password@airtrail-pg:5432/airtrail -e ORIGIN=http://192.168.56.103:3000 --name airtrail-web airtrail-web
```

## dificuldades encontradas nesta fase

A única dificuldade encontrada neste momento foi a configuração da URL da base de dados e a URL `ORIGIN`.

Para resolver o primeiro, percebi que o DATABASE_HOST se encontra no meio da URL, que antes era apenas `db`, mas alterei para `airtrail-pg`, que é o nome do container postgres na mesma network.

O segundo problema relacionado à `ORIGIN` foi identificado ao tentar efetuar o registo na aplicação, já que havia um erro "Cross-site POST form submissions are forbidden". Esse erro foi causado porque a variável ambiente `ORIGIN` estava mal configurada. Ao alterar esta variável ambiente para o IP da máquina virtual que está a executar aplicação, o erro foi corrigido.




# <p style="text-align: center;">To Do</p>

## Configuração
 - [x] Deve ser possível parar a aplicação e voltar a executar a mesma (p.ex., devido a uma tarefa de manutenção programada) sem que sejam perdidos dados críticos (p.ex., dados de utilizadores).

## Replicação

 - [x] Fazer replicação e distribuição da base de dados.

## Benchmarking

 - [x] Usar a ferramenta de monitorização do Google Cloud para obter métricas.
 - [x] Desenvolver testes experimentais para avaliar funcionalidades e componentes da aplicação.

### Relatório

 - [x] Responder às questões 1 e 2.
 - [x] Explicar o porquê de ser escolhida a base de dados para ser efetuada a replicação e porquê este método.
 - [x] Descrição breve da arquitetura, API e principais componentes da app.
 - [x] Identificar as ferramentas e abordagens utilizadas para instalação e configuração automática da app.
 - [x] Ferramentas de monitorização, métricas e visualizações escolhidas, justificando a sua escolha. (Provavelmente vai ser o disponibilizado pelo GoogleCloud).
 - [x] Análise de resultados.
 - [x] Reflexão final discutindo pontos fortes e pontos a melhorar, e ainda acrescentar comparação com o relatório inicial.
