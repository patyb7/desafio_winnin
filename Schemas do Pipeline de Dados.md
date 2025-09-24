# Schemas do Pipeline de Dados

Este documento detalha os schemas das tabelas em cada camada do pipeline de dados (Bronze, Silver e Gold).

## Camada Bronze

A camada Bronze armazena os dados brutos, conforme coletados diretamente da API do YouTube. Os dados são mantidos em seu formato original (JSON) e enriquecidos com um timestamp de ingestão.

### `canais_youtube`

| Coluna              | Tipo de Dados | Descrição                                        |
| :------------------ | :------------ | :----------------------------------------------- |
| `channel_id`        | STRING        | ID único do canal. (Chave Primária)              |
| `channel_name`      | STRING        | Nome do canal do YouTube                         |
| `subscribers`       | LONG          | Número de inscritos do canal                     |
| `api_response_raw`  | STRING        | Resposta JSON bruta da API do YouTube para o canal |
| `ingestion_timestamp` | TIMESTAMP     | Timestamp de quando o registro foi ingerido na camada Bronze. |

### `posts_youtube`

| Coluna              | Tipo de Dados | Descrição                                        |
| :------------------ | :------------ | :----------------------------------------------- |
| `post_id`           | STRING        | ID único do post (vídeo) do YouTube (Chave Primária)             |
| `channel_id`        | STRING        | ID do canal ao qual o post pertence (Chave Estrangeira)          |
| `title`             | STRING        | Título do post/vídeo.                            |
| `views`             | LONG          | Número de visualizações do post.                 |
| `likes`             | LONG          | Número de curtidas do post.                      |
| `published_at`      | STRING        | Data de publicação do post (formato ISO 8601).   |
| `api_response_raw`  | STRING        | Resposta JSON bruta da API do YouTube            |
| `ingestion_timestamp` | TIMESTAMP   | Timestamp de quando o registro foi ingerido na camada Bronze. |


## Camada Silver

A camada Silver contém dados limpos, transformados e enriquecidos. Os dados brutos da camada Bronze são processados para remover duplicatas, selecionar colunas relevantes e enriquecer com informações adicionais, como o `user_id` da Wikipedia.

### `posts_silver`

| Coluna                          | Tipo de Dados | Descrição                                        |
| :------------------------------ | :------------ | :----------------------------------------------- |
| `post_id`                       | STRING        | ID único do post (vídeo) do YouTube  sem duplicatas            |
| `channel_id`                    | STRING        | ID do canal ao qual o post pertence.             |
| `title`                         | STRING        | Título do post/vídeo                             |
| `views`                         | LONG          | Número de visualizações do vídeo, com tratamento para valores negativos |
| `likes`                         | LONG          | Número de curtidas do post, com tratamento para valores negativos       |
| `published_at`                  | STRING        | Data de publicação do post (formato ISO 8601)   |
| `ingestion_timestamp`           | TIMESTAMP     | Timestamp de quando o registro foi ingerido na camada Bronze |
| `silver_processing_timestamp`   | TIMESTAMP     | Timestamp de quando o registro foi processado na camada Silver |

### `canais_silver`

| Coluna                          | Tipo de Dados | Descrição                                        |
| :------------------------------ | :------------ | :----------------------------------------------- |
| `channel_id`                    | STRING        | ID único do canal do YouTube, sem duplicatas.                    |
| `channel_name`                  | STRING        | Nome do canal do YouTube.                        |
| `subscribers`                   | LONG          | Número de inscritos do canal, com tratamento para valores negativos |
| `ingestion_timestamp`           | TIMESTAMP     | Timestamp de quando o registro foi ingerido na camada Bronze. |




## Camada Gold

A camada Gold contém dados agregados e otimizados para consumo por dashboards, relatórios e aplicações de negócio. Os dados são resultantes de junções e análises das tabelas da camada Silver.

### `youtube_analytics`

| Coluna                      | Tipo de Dados | Descrição                                        |
| :-------------------------- | :------------ | :----------------------------------------------- |
| `channel_id  `              | STRING        | D do canal do Youtube                            |
| `channel_name`              | STRING        | Nome do canal do YouTube.                        |
| `subscribers`               | LONG          | Número de inscritos do canal, com tratamento para valores negativos |
| `post_title`                | STRING        | Título do post/vídeo.                            |
| `total_views`               | LONG          | Soma total das visualizações de todos os vídeos do canal                 |
| `total_likes`               | LONG          | Soma total de likes de todos os vídeos do canal  |
| `avg_likes_per_view`        | DOUBLE        | Média da taxa de likes por visualização de todos os vídeos do canal   |


