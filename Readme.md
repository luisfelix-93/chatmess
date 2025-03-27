# Documentação Técnica da Aplicação

## Visão Geral
Este arquivo `docker-compose.yaml` define um ambiente de conteinerização utilizando Docker Swarm para um sistema composto por:
- Um balanceador de carga Nginx
- Um banco de dados MongoDB
- Um servidor Redis
- Dois microsserviços: uma API e um servidor WebSocket
- Uma rede personalizada do tipo `overlay`
- Volumes para persistência de dados do MongoDB e Redis

## Serviços

### 1. **nginx_load_balancer**
- **Imagem:** `nginx:latest`
- **Portas:** Mapeia a porta `50` do host para a porta `80` do container
- **Réplicas:** 1 (o Nginx é stateless, não necessário mais de um container)
- **Volumes:** Mapeia o arquivo de configuração `nginx.conf` para dentro do container (somente leitura)
- **Rede:** `custom_network`

### 2. **mongo**
- **Imagem:** `mongo:latest`
- **Portas:** Mapeia `27017` do host para `27017` do container
- **Réplicas:** 1 (MongoDB é stateful, necessário rodar apenas um container)
- **Placement:** Roda somente em um nó do tipo `manager`
- **Volumes:**
  - `mongo_data`: Persistência de dados
  - `mongo_logs`: Armazena logs do MongoDB
- **Comando:** Define o caminho do arquivo de logs do MongoDB
- **Rede:** `custom_network`

### 3. **redis**
- **Imagem:** `redis:latest`
- **Portas:** Mapeia `6379` do host para `6379` do container
- **Réplicas:** 1 (Redis é stateful, necessário rodar apenas um container)
- **Volumes:**
  - `redis_data`: Persistência de dados
- **Rede:** `custom_network`

### 4. **chatmess-api**
- **Imagem:** `luisffilho/chatmessapi:latest`
- **Portas:** Mapeia `8080` do host para `8080` do container
- **Réplicas:** 2 (escalabilidade horizontal para atender mais requisições)
- **Configuração de Update:**
  - Atualiza os containers 1 por vez
  - Delay de 10s entre as atualizações
- **Variáveis de ambiente:**
  - Conexão com MongoDB
  - Nome do banco de dados e coleções
- **Rede:** `custom_network`

### 5. **chatmess-ws**
- **Imagem:** `luisffilho/chatmessws:latest`
- **Portas:** Mapeia `3001` do host para `3001` do container
- **Réplicas:** 1 (evita concorrência desnecessária no WebSocket)
- **Variáveis de ambiente:**
  - URL da API para mensagens
  - Configuração do Redis para sincronização
- **Rede:** `custom_network`

## Redes
- **custom_network**: Rede do tipo `overlay` para intercomunicação entre os containers

## Volumes
- **mongo_data**: Armazena os dados do MongoDB
- **mongo_logs**: Armazena os logs do MongoDB
- **redis_data**: Armazena os dados do Redis


## Deploy

- Para fazer o deploy da aplicação, seguir as instruções abaixo:

```bash
$ docker swarm init
$ docker stack deploy -c docker-compose.yaml chatmess-stack
```