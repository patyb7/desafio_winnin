# Documento do Pipeline de Dados

## 1. Introdução

Este documento foi criado durante o processo de execução do desafio da Winnin no qual detalha o design e a implementação de um pipeline de dados robusto e escalável, projetado para coletar, processar e analisar dados da fonte YouTube. O pipeline segue a arquitetura de camadas Bronze, Silver e Gold, utilizando tecnologias como Databricks, Apache Spark e Delta Lake para garantir eficiência, confiabilidade e facilidade de manutenção. O objetivo principal é transformar dados brutos em insights acionáveis, suportando dashboards, relatórios e aplicações de negócio.

O pipeline é projetado para ingerir dados diariamente, enriquecê-los, limpá-los e, por fim, agregá-los para análise.  Cada camada é projetada com um objetivo específico, o que permite manter as responsabilidades distintas e a qualidade dos dados em cada fase do processamento.

## 2.  Estrutura do Pipeline
 
O pipeline de dados é estruturado segundo a arquitetura de padrão em medalhão (Medallion Architecture), que divide os dados em três camadas: Bronze, Silver e Gold.  Esse modelo, que é bastante utilizado em contextos de data lakehouse, fornece uma maneira organizada de lidar com os dados, garantindo qualidade, governança e eficiência para diversos casos de uso [1]. 
### 2.1.  Camada Bronze (Raw Data Ingestion)
 
A camada Bronze é onde todo o dado cru entra no pipeline pela primeira vez.  O foco é integrar dados de várias fontes externas com o menor nível de transformação possível.  Os dados são mantidos em seu formato original, garantindo que a integridade das fontes seja respeitada.   Isso possibilita que os dados sejam reproduzidos e, caso seja necessário, reprocessados a partir da fonte original.  Nesta camada, os dados geralmente ficam armazenados no formato Delta Lake, que traz funcionalidades como transações ACID, versionamento e imposição de esquema, mesmo para dados brutos [2].

**Processo de Ingestão:**

*   **Coleta de Criadores, Coleta de Posts** Um único job de ingestão diária se conecta à API do YouTube para buscar canais relevantes e os posts (vídeos) mais recentes de cada criador identificado. A lista inicial de canais pode ser gerenciada em uma tabela separada ou configurada. O foco é coletar metadados básicos dos canais.Isso inclui informações como título, visualizações, curtidas e data de publicação.
*   **Armazenamento:** Os dados brutos são salvos como tabelas Delta na camada Bronze, usando o modo overwrite para atualizações completas.

**Tabelas da Camada Bronze:**

*   `bronze.youtube_channels`: Contém dados brutos de criadores do YouTube.
*   `bronze.youtube_posts`: Contém dados brutos de posts (vídeos) do YouTube.

### 2.2. Camada Silver (Enriquecimento e Limpeza)

A camada Silver atua como uma área de preparação onde os dados brutos da camada Bronze são limpos, transformados e enriquecidos. O objetivo é refinar os dados para torná-los mais consistentes, estruturados e úteis para análises posteriores. Nesta camada, aplicam-se regras de negócio, remoção de duplicatas, padronização de formatos e e enriquecidos com novas colunas (ex: likes_per_view). O resultado é salvo em novas tabelas Delta na camada Silver.

**Processo de Enriquecimento:**

*   **Seleção e Limpeza:** Um job de transformação lê os dados da camada Bronze, seleciona as colunas relevantes (e.g., `creator_id`, `views`, `likes`, `published_at`) e remove dados duplicados com base em chaves primárias.
*   **Armazenamento:** Os dados limpos e enriquecidos são salvos em novas tabelas Delta Lake, mantendo um histórico de alterações e permitindo consultas eficientes.

**Tabelas da Camada Silver:**

*   `silver.posts_enriched`: Posts limpos e com informações relevantes.
*   `silver.creators_enriched`: Criadores com o `user_id` da Wikipedia e outros metadados enriquecidos.

### 2.3. Camada Gold (Análise e Agregação)

A camada Gold é a camada de consumo final, onde os dados são agregados e otimizados para casos de uso específicos, como dashboards, relatórios de BI e aplicações de negócio. Os dados nesta camada são altamente sumarizados, desnormalizados e prontos para serem consumidos por usuários finais, sem a necessidade de complexas transformações adicionais [4].

**Processo de Agregação:**

*   **Junção de Dados:** As tabelas Silver são unidas para criar uma visão única alimentando assim a camada Gold que é a tabela final. Os dados são agregados por canal, calculando métricas-chave como total_views e avg_likes_per_view. O resultado é salvo na camada Gold, pronto para análise.
*   **Análise e Agregação:** São executadas análises específicas, como a identificação dos top posts por criador, contagem de posts por mês, e outras métricas de desempenho.
*   **Armazenamento:** O resultado final dessas agregações é salvo em tabelas Gold, otimizadas para leitura e consulta rápida.


## 3. Ferramentas e Tecnologias

O pipeline de dados é construído sobre um conjunto de ferramentas e tecnologias modernas, escolhidas por sua escalabilidade, flexibilidade e integração em ambientes de big data.

*   **Orquestração:** [Databricks Workflows](https://docs.databricks.com/workflows/index.html) - Utilizado para agendar e gerenciar a execução dos jobs do pipeline, garantindo a automação e a resiliência.
*   **Dados:** [Delta Lake](https://delta.io/) - Formato de armazenamento de código aberto que traz confiabilidade e desempenho de data warehouse para data lakes, com suporte a transações ACID, schema enforcement e versionamento de dados.
*   **Processamento:** [Apache Spark](https://spark.apache.org/) (no Databricks) - Motor de processamento de dados distribuído em larga escala, ideal para transformações complexas e análises de grandes volumes de dados.
*   **Coleta de Dados:** APIs do YouTube - Interfaces de programação de aplicações utilizadas para extrair dados diretamente das plataformas.

## 4. Fluxo de Dados


%% Diagrama do Fluxo de Dados do Pipeline de YouTube

graph TD
    subgraph "Camada Bronze (Ingestão)"
        A[API do YouTube] --> B(Coleta de Dados Brutos)
        B --> C{Tabelas Delta - Bruto}
        C --> D[canais_youtube]
        C --> E[posts_youtube]
    end

    subgraph "Camada Silver (Limpeza & Enriquecimento)"
        F[Leitura da Bronze] --> G(Processamento de Dados)
        G --> H{Tabelas Delta - Limpo}
        H --> I[canais_silver]
        H --> J[posts_silver]
    end

    subgraph "Camada Gold (Análise & Agregação)"
        K[Leitura da Silver] --> L(Junção e Agregação)
        L --> M{Tabela Delta - Analítico}
        M --> N[youtube_analytics]
    end

    subgraph "Orquestração"
        O(Job Agendado do Databricks)
        O --> P(Executar Pipeline)
        P --> Q[Bronze]
        P --> R[Silver]
        P --> S[Gold]
    end

    %% Conexões do Fluxo Principal
    D --> F
    E --> F
    I --> K
    J --> K
    N --> T[BI & Análise de Negócio]

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#ccf,stroke:#333,stroke-width:2px
    style D fill:#ccf,stroke:#333,stroke-width:2px
    style E fill:#ccf,stroke:#333,stroke-width:2px

    style F fill:#bbf,stroke:#333,stroke-width:2px
    style G fill:#bbf,stroke:#333,stroke-width:2px
    style H fill:#f9f,stroke:#333,stroke-width:2px
    style I fill:#f9f,stroke:#333,stroke-width:2px
    style J fill:#f9f,stroke:#333,stroke-width:2px

    style K fill:#bbf,stroke:#333,stroke-width:2px
    style L fill:#bbf,stroke:#333,stroke-width:2px
    style M fill:#ccf,stroke:#333,stroke-width:2px
    style N fill:#ccf,stroke:#333,stroke-width:2px
    style T fill:#9ff,stroke:#333,stroke-width:2px

    linkStyle 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19 stroke-width:2px,fill:none,stroke:#005
## 5. Schemas Detalhados das Tabelas

Esta seção apresenta os schemas detalhados para cada tabela nas camadas Bronze, Silver e Gold, incluindo o nome da coluna, tipo de dados e uma breve descrição. A definição clara dos schemas é crucial para a governança de dados, a qualidade dos dados e a facilidade de consumo por aplicações downstream.

### 5.1. Camada Bronze

A camada Bronze armazena os dados brutos, conforme coletados diretamente das APIs do YouTube. Os dados são mantidos em seu formato original (JSON) e enriquecidos com um timestamp de ingestão.

#### `bronze.canais_youtube`

| Coluna              | Tipo de Dados | Descrição                                        |
| :------------------ | :------------ | :----------------------------------------------- |
| `channel_id`        | STRING        | ID único do canal do YouTube.(Chave Primária)    |
| `channel_name`      | STRING        | Nome do canal do YouTube.                        |
| `subscribers`       | LONG          | Número de inscritos do canal.                    |
| `api_response_raw`  | STRING        | Resposta JSON bruta da API do YouTube para o canal. |
| `ingestion_timestamp` | TIMESTAMP     | Timestamp de quando o registro foi ingerido na camada Bronze. |

#### `bronze.posts_youtube`

| Coluna              | Tipo de Dados | Descrição                                        |
| :------------------ | :------------ | :----------------------------------------------- |
| `post_id`           | STRING        | ID único do (vídeo) postado no YouTube           |
| `channel_id`        | STRING        | ID do canal ao qual o post pertence              |
| `title`             | STRING        | Título do post/vídeo                             |
| `views`             | LONG          | Número de visualizações do video                 |
| `likes`             | LONG          | Número de curtidas/linkes  do video              |
| `published_at`      | STRING        | Data de publicação do post (formato ISO 8601).   |
| `api_response_raw`  | STRING        | Resposta JSON bruta da API do YouTube            |
| `ingestion_timestamp` | TIMESTAMP     | Timestamp de quando o registro foi ingerido na camada Bronze |

### 5.2. Camada Silver

A camada Silver contém dados limpos, transformados e enriquecidos. Os dados brutos da camada Bronze são processados para remover duplicatas, selecionar colunas relevantes e enriquecer com informações adicionais.

#### `posts_silver`

| Coluna                          | Tipo de Dados | Descrição                                        |
| :------------------------------ | :------------ | :----------------------------------------------- |
| `post_id`                       | STRING        | ID único do post (vídeo) do YouTube(Chave Primária)             |
| `channel_id`                    | STRING        | ID do canal ao qual o video pertence(Chave Estrangeira)         |
| `title`                         | STRING        | Título do post/vídeo.                            |
| `views`                         | LONG          | Número de visualizações, com tratamento para valores negativos  |
| `likes`                         | LONG          | Número de curtidas com tratamento de negativos                  |
| `published_at`                  | STRING        | Data de publicação do post (formato ISO 8601)    |
| `ingestion_timestamp`           | TIMESTAMP     | Timestamp de quando o registro foi ingerido na camada Bronze    |
| `likes_per_view`                | DOUBLE        | Nova coluna calculada: likes dividido por views, arredondado para 4 casas decimais |

#### `canais_silver`

| Coluna                          | Tipo de Dados | Descrição                                        |
| :------------------------------ | :------------ | :--------------------------------------------------------------------- |
| `channel_id`                    | STRING        | ID único do canal, sem duplicatas (Chave Primária)                     |
| `channel_name`                  | STRING        | Nome do canal do YouTube                                               |
| `subscribers`                   | LONG          | Número de inscritos do canal com tratamento para valores negativos     |
| `ingestion_timestamp`           | TIMESTAMP     | data/hora da ingestão                                                  |

### 5.3. Camada Gold

A camada Gold contém dados agregados e otimizados para consumo por dashboards, relatórios e aplicações de negócio. Os dados são resultantes de junções e análises das tabelas da camada Silver.

#### `youtube_analytics`

| Coluna                      | Tipo de Dados | Descrição                                        |
| :-------------------------- | :------------ | :----------------------------------------------- |
| `channel_id`              | STRING          | ID do canal. (Chave Primária)                    |
| `channel_name`              | STRING        | Nome do canal do YouTube                         |
| `subscribers`               | INT           | Número de inscritos do canal                     |
| `total_views`               | BIGINT        | Soma total das visualizações de todos os vídeos do canal   |
| `total_likes`               | BIGINT        | Soma total de likes de todos os vídeos do canal            |
| `total_videos`              | BIGINT        | Contagem total de vídeos coletados do cana                 |
| `rank`                      | INT           | Classificação do post dentro do canal (por views e likes). |
| `avg_likes_per_view`        | DOUBLE        | Média da taxa de likes por visualização de todos os vídeos do canal |


## 6. Código Python/Spark

Esta seção apresenta o script Python/Spark unificado desenvolvido para o pipeline de dados. Este script é projetado para ser executado como um job no ambiente Databricks, utilizando Apache Spark e Delta Lake para processamento e armazenamento de dados, cobrindo as camadas Bronze, Silver e Gold em uma única execução.

### 6.1. Script Unificado: `youtube_data_pipeline.py`

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import current_timestamp, col, when, round, sum, avg, count
from googleapiclient.discovery import build
import json

# ==================================================================================== #
# Pipeline com as etapas de coleta de dados do youtube,ingestão na camada bronze,      #
#  limpeza e enriquecimento na Silver e gregação na Golden                             #
#  1. INICIALIZAÇÃO E CONFIGURAÇÃO                                                     #
#                                                                                      #
# ==================================================================================== #
spark = SparkSession.builder \
    .appName("YoutubeDataPipeline") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .config("spark.connect.enabled", "false") \
    .getOrCreate()

# Caminhos dos volumes no Unity Catalog
path_bronze_channels = "/Volumes/workspace/desafio_dataeng/desafio_youtube/canais_youtube"
path_bronze_posts = "/Volumes/workspace/desafio_dataeng/desafio_youtube/posts_youtube"
path_silver_channels = "/Volumes/workspace/desafio_dataeng/desafio_youtube/canais_silver"
path_silver_posts = "/Volumes/workspace/desafio_dataeng/desafio_youtube/posts_silver"
path_gold = "/Volumes/workspace/desafio_dataeng/desafio_youtube/youtube_analytics"

# Credencial da API do YouTube
YOUTUBE_API_KEY = "Gerar a chave"
youtube = build("youtube", "v3", developerKey=YOUTUBE_API_KEY)

# Lista de canais para ingestão
channel_ids_to_ingest = ["UC_x5XG1OV2P6uZZ5FSM9Ttw", "UCBR8-60-B28hp2BmDPdntcQ"]

# ======================================================================
# 2. FUNÇÕES DE INGESTÃO
# Mantendo as funções de ingestão da API
# ======================================================================
def get_channel_info(channel_id):
    request = youtube.channels().list(part="snippet,statistics", id=channel_id)
    response = request.execute()
    if response["items"]:
        item = response["items"][0]
        return {
            "channel_id": item["id"],
            "channel_name": item["snippet"]["title"],
            "subscribers": int(item["statistics"].get("subscriberCount", 0)),
            "api_response_raw": json.dumps(item)
        }
    return None

def get_channel_videos(channel_id, max_results=50):
    videos = []
    channel_request = youtube.channels().list(part="contentDetails", id=channel_id)
    channel_response = channel_request.execute()
    if not channel_response.get("items"):
        return videos
    uploads_playlist_id = channel_response["items"][0]["contentDetails"]["relatedPlaylists"]["uploads"]
    playlist_items_request = youtube.playlistItems().list(part="snippet,contentDetails", playlistId=uploads_playlist_id, maxResults=max_results)
    playlist_items_response = playlist_items_request.execute()
    for item in playlist_items_response["items"]:
        video_id = item["contentDetails"]["videoId"]
        video_request = youtube.videos().list(part="statistics,snippet", id=video_id)
        video_response = video_request.execute()
        if video_response.get("items"):
            video_item = video_response["items"][0]
            videos.append({
                "post_id": video_item["id"],
                "channel_id": channel_id,
                "title": video_item["snippet"]["title"],
                "views": int(video_item["statistics"].get("viewCount", 0)),
                "likes": int(video_item["statistics"].get("likeCount", 0)),
                "published_at": video_item["snippet"]["publishedAt"],
                "api_response_raw": json.dumps(video_item)
            })
    return videos

# ======================================================================
# 3. CAMADA BRONZE: INGESTÃO DE DADOS
# ======================================================================
all_channels_data = [get_channel_info(cid) for cid in channel_ids_to_ingest if get_channel_info(cid)]
all_posts_data = [video for cid in channel_ids_to_ingest for video in get_channel_videos(cid)]

df_channels_bronze = spark.createDataFrame(all_channels_data).withColumn("ingestion_timestamp", current_timestamp())
df_posts_bronze = spark.createDataFrame(all_posts_data).withColumn("ingestion_timestamp", current_timestamp())

# Salva os dados na camada Bronze
df_channels_bronze.write.format("delta").mode("overwrite").save(path_bronze_channels)
df_posts_bronze.write.format("delta").mode("overwrite").save(path_bronze_posts)

print(f"Dados de canais salvos em: {path_bronze_channels}")
print(f"Dados de posts salvos em: {path_bronze_posts}")

# ======================================================================
# 4. CAMADA SILVER: LIMPEZA E ENRIQUECIMENTO
# ======================================================================
df_channels_silver = df_channels_bronze.dropDuplicates(["channel_id"]) \
    .withColumn("subscribers", when(col("subscribers") < 0, 0).otherwise(col("subscribers")))

df_posts_silver = df_posts_bronze.dropDuplicates(["post_id"]) \
    .withColumn("views", when(col("views") < 0, 0).otherwise(col("views"))) \
    .withColumn("likes", when(col("likes") < 0, 0).otherwise(col("likes"))) \
    .withColumn("likes_per_view", round(col("likes") / col("views"), 4))

# Salva os dados limpos na camada Silver
df_channels_silver.write.format("delta").mode("overwrite").save(path_silver_channels)
df_posts_silver.write.format("delta").mode("overwrite").save(path_silver_posts)

print(f"Dados de canais limpos salvos em: {path_silver_channels}")
print(f"Dados de posts limpos salvos em: {path_silver_posts}")

# ======================================================================
# 5. CAMADA GOLD: AGREGAÇÃO PARA ANÁLISE
# ======================================================================
df_gold = df_posts_silver.join(df_channels_silver, on="channel_id", how="left")

df_gold_agg = df_gold.groupBy("channel_id", "channel_name", "subscribers").agg(
    sum("views").alias("total_views"),
    sum("likes").alias("total_likes"),
    count("post_id").alias("total_videos"),
    avg("likes_per_view").alias("avg_likes_per_view")
).withColumn("avg_likes_per_view", when(col("avg_likes_per_view").isNull(), 0).otherwise(col("avg_likes_per_view")))

# Salva o DataFrame final na camada Gold
df_gold_agg.write.format("delta").mode("overwrite").save(path_gold)

print(f"Dados agregados salvos com sucesso em: {path_gold}")

# ======================================================================
# 6. VALIDAÇÃO E EXIBIÇÃO
# ======================================================================
df_gold_final = spark.read.format("delta").load(path_gold)
display(df_gold_final)

spark.stop()
```

## 7. Jobs e Frequência

O pipeline será orquestrado usando Databricks Jobs, com a seguinte configuração:

*   **Job:** `youtube_data_pipeline_job`
*   **Script:** `youtube_data_pipeline.py`
*   **Frequência:** Diária (executado uma vez por dia)
*   **Cluster:** Cluster de jobs configurado para auto-scaling, otimizando custos e desempenho.

## 8. Conclusão

Este documento de design fornece uma visão abrangente do pipeline de dados, desde a ingestão até a análise. A arquitetura de medalhão, combinada com as tecnologias Databricks, Spark e Delta Lake, garante um pipeline robusto, escalável e de fácil manutenção. O script unificado simplifica a orquestração do job no Databricks, e a documentação detalhada serve como um guia para desenvolvimento, manutenção e futuras melhorias.

