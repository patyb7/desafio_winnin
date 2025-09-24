## Requisitos da Arquitetura:

### Orquestrador: Qual orquestrador você utilizaria e por quê?
- Usei o Databricks Jobs por ser uma funcionalidade nativa do ambiente usado que no caso é o databricks reduzindo a complexidade da integração. Acho de grande relevancia o gerenciamento do ciclo de vida do cluster e auto-scaling otimizando os custos e desempenho

### Modelagem de Dados: Diagrama de relacionamento e documentação das tabelas (incluindo creators, posts, etc.)
Encontra-se na documentação Readme.md
erDiagram
    CREATORS ||--o{ POSTS : "têm"

    CREATORS {
        string  channel_id PK
        string  channel_name
        int     subscribers
        timestamp ingestion_timestamp
    }

    POSTS {
        string post_id PK
        string channel_id FK
        string title
        int    views
        int    likes
        string published_at
        timestamp ingestion_timestamp
    }
![Diagrama de relacionamento](image.png)
### Extração de Dados: Como você faria a extração inicial e as atualizações desses dados (APIs, web scraping)?
- A coleta de dados na API do YoutubeData v2 foi realizada de forma completa iterando sobre a lista de canais coletando o máximo de histórico disponivel e para atualização continua o pipe foi agendado paa rodar periodicamente fazendo a coleta de forma incremental.

### Etapas do Pipeline: Descreva o fluxo de dados (extração, transformação, carga, etc.).
- Esse Fluxo já consta na documentação Readme.md

### Monitoramento e Qualidade: Como você monitoraria o pipeline e avaliaria a qualidade dos dados finais?
- O Jobs já fornece um painel de monitoramento exibindo o status de cada execução e alertas por e-mail.
- Como qualidade é usada a validação de esquema do delta para verificar o formato dos dados de entrada correspondem ao esquema da Tabela
- Criar e adicionar alguns testes automatizados para garantir que valores de views e likes não sejam negativos

### Boas Práticas: Qual fluxo e boas práticas de engenharia de software (Gitflow, princípios, etc.) seriam importantes?

- Código modular e controle de versão para um fluxo mais colaborativo e de fácil rastreabilidade das mudanças.
- Reaproveitamento de código como fiz durante o projeto que foi possível reaproveitar algumas coisas do desafio 1