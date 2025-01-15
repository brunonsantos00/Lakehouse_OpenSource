# Projeto Lakehouse OpenSource

## Documentação da Configuração do Docker Compose

Este projeto configura uma arquitetura **Lakehouse** com MinIO, PostgreSQL, Airflow, Trino e TensorFlow. Cada serviço está configurado com volumes persistentes para armazenamento de dados e conectividade em uma rede dedicada.

---

## 1. MinIO

O MinIO é uma solução de armazenamento de objetos, usada como a camada de armazenamento principal da arquitetura.

### Configuração:
- **Imagem**: `bitnami/minio:latest`
- **Container Name**: `minio`
- **Portas**: 
  - `9000:9000` (API S3)
  - `9001:9001` (console web)
- **Volumes**:
  - `./volumes/minio/data:/data`
  - `./volumes/minio/config:/root/.minio`
- **Variáveis de Ambiente**:
  - `MINIO_ROOT_USER=admin`
  - `MINIO_ROOT_PASSWORD=password123`

---

## 2. PostgreSQL

O PostgreSQL serve como o banco de dados para os metadados do Airflow e outras aplicações.

### Configuração:
- **Imagem**: `bitnami/postgresql:latest`
- **Container Name**: `postgresql`
- **Portas**: `5432:5432`
- **Volumes**:
  - `./volumes/postgresql/data:/bitnami/postgresql`
- **Variáveis de Ambiente**:
  - `POSTGRESQL_USERNAME=postgresuser`
  - `POSTGRESQL_PASSWORD=postgrespass`
  - `POSTGRESQL_DATABASE=postgresdb`

---

## 3. Airflow

O Airflow é usado para orquestrar pipelines de dados. Está configurado para usar o PostgreSQL como banco de dados de metadados.

### Configuração:
- **Imagem**: `bitnami/airflow:latest`
- **Container Name**: `airflow`
- **Portas**: `8080:8080` (interface web)
- **Volumes**:
  - `./volumes/airflow/dags:/opt/airflow/dags`
  - `./volumes/airflow/logs:/opt/airflow/logs`
  - `./volumes/airflow/plugins:/opt/airflow/plugins`
- **Variáveis de Ambiente**:
  - `AIRFLOW__CORE__EXECUTOR=LocalExecutor`
  - `AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://postgresuser:postgrespass@postgresql:5432/postgresdb`
  - `AIRFLOW__CORE__FERNET_KEY=xCm03biQbfFdTvcgFu-cB7xndtx-uUJC4lUnLRcO_3Q=`

---

## 4. Trino

O Trino é um mecanismo de consulta SQL distribuído usado para executar consultas analíticas em grandes volumes de dados.

### Configuração:
- **Imagem**: `trinodb/trino:latest`
- **Container Name**: `trino`
- **Portas**: `8081:8080`
- **Volumes**:
  - `./volumes/trino/etc:/etc/trino`

### Arquivos necessários no diretório `./volumes/trino/etc`:
1. `jvm.config`
    ```text
    -Xmx2G
    ```
2. `node.properties`
    ```text
    node.environment=production
    node.id=1
    node.data-dir=/etc/trino/data
    ```
3. `config.properties`
    ```text
    coordinator=true
    http-server.http.port=8080
    discovery.uri=http://localhost:8080
    ```

---

## 5. TensorFlow

O TensorFlow com Jupyter Notebook permite o desenvolvimento de aprendizado de máquina e experimentação.

### Configuração:
- **Imagem**: `tensorflow/tensorflow:nightly-gpu-jupyter`
- **Container Name**: `tensorflow`
- **Portas**: `8888:8888` (Jupyter Notebook)
- **Volumes**:
  - `./volumes/tensorflow/notebooks:/tf`

---

# Arquitetura do Projeto Lakehouse: Integração e Relações entre Tecnologias

A arquitetura **Lakehouse** combina as funcionalidades de um Data Lake com as vantagens de um Data Warehouse. Cada tecnologia na arquitetura tem um papel crucial e interage de maneira sinérgica para garantir armazenamento, processamento, consulta e aprendizado de máquina eficientes. O **Delta Lake** é a tecnologia que unifica essas interações ao oferecer transações ACID, versionamento de dados e integração com formatos como o **Apache Parquet**.

---

## Relação e Funcionalidade de Cada Tecnologia

### **PostgreSQL**
- **Função:** 
  - Atua como o backend relacional para gerenciar metadados e persistência em diferentes serviços.
- **Interações:**
  - **Airflow:** Armazena metadados dos workflows, como DAGs, estados de tarefas e histórico de execuções.
  - **Trino:** Serve como o Metastore, armazenando informações sobre tabelas, partições e schemas. Isso possibilita que o Trino entenda a estrutura do Data Lake e realize consultas eficientes.
  - **Delta Lake:** Rastreia transações e alterações de dados, garantindo integridade e consistência nos dados do Delta Lake.
- **Importância:**
  - Centraliza metadados para facilitar a coordenação entre diferentes tecnologias.
  - Garante consistência e integridade no acesso a informações críticas.

---

### **Airflow**
- **Função:**
  - Ferramenta de orquestração para gerenciar pipelines de dados (ETL/ELT) e automação de tarefas.
- **Interações:**
  - **PostgreSQL:** Utiliza o banco de dados para armazenar informações sobre tarefas agendadas e suas execuções.
  - **MinIO:** Orquestra a ingestão de dados brutos no Data Lake, aplicando transformações e salvando os resultados em tabelas Delta.
  - **Trino:** Automatiza consultas SQL, verificando a qualidade dos dados ou gerando insights antes de carregá-los em modelos de aprendizado.
  - **TensorFlow:** Automatiza o treinamento e a inferência de modelos de aprendizado de máquina, fornecendo dados prontos para análise ou aprendizado.
- **O que é uma DAG?**
  - DAG (Directed Acyclic Graph) é uma estrutura que define a sequência e a dependência entre tarefas no Airflow. Ela garante que tarefas sejam executadas na ordem correta e evita loops cíclicos.
- **Importância:**
  - Coordena o fluxo de dados entre as tecnologias.
  - Garante execução eficiente e ordenada de tarefas complexas.

---

### **MinIO**
- **Função:**
  - Sistema de armazenamento de objetos compatível com S3, centralizando o armazenamento de dados no Data Lake.
- **Interações:**
  - **Airflow:** Recebe dados brutos e processados por meio de pipelines orquestrados pelo Airflow.
  - **Trino:** Funciona como a fonte primária de dados para o Trino realizar consultas SQL. Permite que dados estruturados e semiestruturados sejam analisados diretamente.
  - **TensorFlow:** Armazena conjuntos de dados para treinamento e inferência de modelos de aprendizado de máquina.
  - **Delta Lake:** Implementa a camada física de tabelas Delta, permitindo transações ACID e versionamento de dados.
- **Tipos de Arquivos Compatíveis:**
  - Arquivos CSV, JSON, XML.
  - Arquivos binários e imagens, como PNG, JPEG, e MP4.
  - Formatos de dados analíticos, como Apache Parquet e ORC.
  - Arquivos comprimidos, como ZIP e GZIP.
- **Importância:**
  - Fornece escalabilidade no armazenamento de grandes volumes de dados.
  - Garante compatibilidade com ferramentas que usam o protocolo S3.

---

### **Trino**
- **Função:**
  - Motor SQL distribuído para consultas analíticas em grandes volumes de dados.
- **Interações:**
  - **PostgreSQL:** Consulta e armazena metadados sobre tabelas e schemas por meio do Metastore.
  - **MinIO:** Realiza consultas SQL diretamente nos dados armazenados em tabelas Delta, incluindo suporte ao formato Parquet.
  - **Delta Lake:** Habilita consultas eficientes em dados versionados e transacionais.
  - **TensorFlow:** Analisa dados antes de serem usados para treinamento, fornecendo amostras ou insights específicos.
- **Importância:**
  - Unifica o acesso a diferentes fontes de dados.
  - Permite análises rápidas e escaláveis diretamente no Data Lake.

---

### **TensorFlow com Jupyter Notebook**
- **Função:**
  - Framework para aprendizado de máquina usado para criar, treinar e validar modelos de IA.
- **Interações:**
  - **Airflow:** Recebe pipelines automáticos para realizar treinamentos em batch ou inferências periódicas.
  - **MinIO:** Consome dados armazenados para treinar modelos e salvar os resultados das inferências.
  - **Trino:** Utiliza amostras processadas ou resultados de consultas SQL para ajustar e otimizar modelos.
  - **Delta Lake:** Lê e grava dados em tabelas Delta para garantir que os modelos sejam treinados com dados consistentes e atualizados.
- **Importância:**
  - Central para implementar soluções baseadas em IA e aprendizado profundo.
  - Garante consistência e qualidade de dados para experimentação e produção.

---

## **Delta Lake e sua Relação com Apache Parquet**

- **Delta Lake:**
  - Atua como uma camada de armazenamento transacional sobre o Data Lake (MinIO), garantindo integridade de dados com suporte a ACID.
  - Permite rastreamento de versões de dados, possibilitando reproduções de estados anteriores e correções de inconsistências.
  - Habilita atualizações incrementais, otimizando ingestão e processamento de dados.
- **Apache Parquet:**
  - É o formato subjacente usado pelo Delta Lake para armazenar dados. Ele é eficiente para leitura, compressão e análise de grandes volumes de informações.
  - Parquet reduz significativamente o tamanho dos dados e melhora a velocidade de consulta.

---

### **Vantagens do Delta Lake e Parquet na Arquitetura**

A tabela abaixo exemplifica as melhorias de eficiência e custo proporcionadas pelo uso do formato Parquet com tabelas Delta:

| **Dataset**                      | **Size on Storage**   | **Query Run Time** | **Data Scanned**  | **Cost**   |
|-----------------------------------|-----------------------|--------------------|-------------------|------------|
| **Data stored as CSV files**      | 1 TB                 | 236 seconds        | 1.15 TB           | $5.75      |
| **Data stored in Apache Parquet** | 130 GB               | 6.78 seconds       | 2.51 GB           | $0.01      |
| **Savings**                       | 87% menos espaço     | 34x mais rápido    | 99% menos dados   | 99.7% menos custo |

---

## **Delta Lake, TensorFlow e o Desenvolvimento de IA**

- **Consistência de Dados:** 
  - O Delta Lake garante que TensorFlow receba dados confiáveis para treinamento, evitando inconsistências que poderiam comprometer a qualidade dos modelos.
- **Versionamento:**
  - Permite rastrear mudanças nos dados usados em experimentos, essencial para reproduzir resultados ou melhorar modelos.
- **Parquet e Performance:**
  - Ao armazenar dados em formato Parquet, TensorFlow pode consumir grandes conjuntos de dados de maneira eficiente, acelerando o treinamento.
- **Ingestão Incremental:**
  - Com atualizações incrementais do Delta Lake, novos dados podem ser integrados ao pipeline de IA sem a necessidade de reprocessar dados antigos.

---

## **Relação Geral entre as Tecnologias**

1. **PostgreSQL**:
   - Centraliza metadados, facilitando a coordenação entre Airflow, Trino e Delta Lake.
2. **Airflow**:
   - Orquestra todo o fluxo de dados, conectando ingestão, transformação, análise e aprendizado de máquina.
3. **MinIO**:
   - Serve como o repositório central para armazenamento de dados no formato Delta.
4. **Trino**:
   - Garante acesso rápido e escalável aos dados armazenados no MinIO, promovendo análises avançadas.
5. **TensorFlow**:
   - Consome dados processados para criar modelos de IA avançados.

Essa arquitetura aproveita o melhor de cada tecnologia, promovendo um ambiente escalável, eficiente e integrado para análise de dados e aprendizado de máquina.
