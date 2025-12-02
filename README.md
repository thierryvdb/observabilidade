# Stack de Observabilidade e Automação

Este projeto implementa uma stack completa de observabilidade para Oracle Database usando Prometheus, Grafana e n8n para automação e integração com IA.

## Arquitetura

1.  **Bancos de dados**:
    - **Oracle Database (`oracle-db`)**
    - **MySQL Database (`mysql-db`)**
    - **PostgreSQL Database (`postgres-db`)**
2.  **Exporters**:
    - **OracleDB Exporter (`oracledb-exporter`)**: Porta `9161`.
    - **MySQL Exporter (`mysqld-exporter`)**: Porta `9104`.
    - **Postgres Exporter (`postgres-exporter`)**: Porta `9187`.
3.  **Prometheus (`prometheus`)**: Coleta métricas de todos os exporters a cada 15 segundos.
4.  **Grafana (`grafana`)**: Visualiza métricas. Pré-configurado com Prometheus como fonte de dados.
5.  **n8n (`n8n`)**: Ferramenta de automação de workflows para processar alertas e integrar com LLMs.

## Pré-requisitos

- Docker
- Docker Compose

## Primeiros Passos

1.  **Suba a stack**:
    ```bash
    docker-compose up -d
    ```

2.  **Acesse os serviços**:
    - **Grafana**: [http://localhost:3000](http://localhost:3000) (Usuário: `admin`, Senha: `admin`)
    - **Prometheus**: [http://localhost:9090](http://localhost:9090)
    - **n8n**: [http://localhost:5678](http://localhost:5678) (Usuário: `admin`, Senha: `admin`)
    - **Oracle DB**: `localhost:1521` (System/Senha: `system`/`oracle`)
    - **MySQL DB**: `localhost:3306` (Usuário/Senha: `appuser`/`apppassword`, Senha do root: `root`)
    - **Postgres DB**: `localhost:5432` (Usuário/Senha: `postgres`/`postgres`)

## Detalhes de Configuração

### 1. Dashboards do Grafana
- Faça login no Grafana.
- Vá em **Dashboards** > **Import**.
- Importe um dashboard padrão de Oracle (por exemplo, ID `3333` ou `12463` do Grafana Labs) ou crie o seu usando o datasource Prometheus.

### 2. Configurando a Integração com LLM no n8n
Para atingir o objetivo de usar IA para prever erros:

1.  **Crie um workflow no n8n**:
    - Abra o n8n em [http://localhost:5678](http://localhost:5678).
    - Crie um novo workflow.
2.  **Adicione um gatilho**:
    - Use um nó **Webhook** (para alertas do Grafana) ou um nó **Schedule** (para rodar periodicamente, por exemplo, a cada hora).
3.  **Busque métricas**:
    - Use um nó **HTTP Request** para consultar a API do Prometheus (`http://prometheus:9090/api/v1/query`) buscando métricas-chave.
    - Exemplos:
        - Oracle: `oracle_session_count`
        - MySQL: `mysql_global_status_threads_connected`
        - Postgres: `pg_stat_activity_count`
4.  **Analise com LLM**:
    - Adicione um nó **AI/LLM Chain** (se disponível na sua versão do n8n) ou um nó **HTTP Request** genérico para chamar a API da OpenAI/Anthropic.
    - **Exemplo de prompt**:
      > "Aqui estão as métricas atuais do Oracle DB: {json_metrics}. Analise-as em busca de possíveis gargalos de performance ou problemas de espaço. Forneça um resumo e ações recomendadas."
5.  **Ação**:
    - Use um nó **Slack**, **Email** ou **Discord** para enviar a análise do LLM para a equipe de operações.

### 3. Alertas no Grafana
- Configure o Grafana Alerting para enviar um webhook para a URL do seu Webhook no n8n quando limiares específicos forem atingidos (por exemplo, Tablespace > 90%).

## Solução de Problemas
- **Conexão com Oracle**: Se o exporter não conseguir se conectar, confirme se o container `oracle-db` está saudável (`docker ps`).
- **Targets do Prometheus**: Acesse [http://localhost:9090/targets](http://localhost:9090/targets) para verificar se `oracle-db` está UP.
