# Lakehouse_OpenSource_IA_Ativos


# Documentação da Configuração do Docker Compose

Este documento contém os detalhes da configuração de um arquivo Docker Compose utilizado para uma arquitetura **Lakehouse**. Os serviços incluídos são PostgreSQL, Airflow, MinIO, Apache Hive, Apache Spark e TensorFlow com suporte para Delta Lake. Abaixo está a documentação detalhada para cada serviço.

---

## 1. PostgreSQL

O serviço PostgreSQL atua como o banco de dados para os metadados do Airflow e do Metastore do Hive. Ele está configurado com variáveis de ambiente para definir o nome de usuário, senha e nome do banco de dados.

### Configuração:
- **Imagem**: `bitnami/postgresql:latest`
- **Nome do Container**: `PostAirTeste`
- **Portas**: `5432:5432` (porta padrão do PostgreSQL)
- **Volumes**: `./crypto_opensouce_lakehouse_data_container/postegres/data:/bitnami/postgresql`
- **Variáveis de Ambiente**:
  - `POSTGRESQL_USERNAME=bruno`
  - `POSTGRESQL_PASSWORD=123`
  - `POSTGRESQL_DATABASE=postegresAirflow`
  - `POSTGRESQL_POSTGRES_PASSWORD=123`
- **Rede**: `lakehouse-net`

---

## 2. Airflow

O serviço Airflow é utilizado para orquestrar pipelines de dados. Ele está configurado para usar o banco de dados PostgreSQL para armazenamento de metadados e está definido para ser executado utilizando o LocalExecutor.

### Configuração:
- **Imagem**: `bitnami/airflow:latest`
- **Nome do Container**: `airflow-container`
- **Portas**: `8080:8080` (interface web do Airflow)
- **Volumes**: `./crypto_opensouce_lakehouse_data_container/airflow/logs:/bitnami/airflow`
- **Depende de**: `postgresql`
- **Variáveis de Ambiente**:
  - `AIRFLOW__CORE__EXECUTOR=LocalExecutor`
  - `AIRFLOW__CORE__SQL_ALCHEMY_CONN=postgresql+psycopg2://bruno:123@postgresql:5432/postegresAirflow`
  - `AIRFLOW__CORE__FERNET_KEY=xCm03biQbfFdTvcgFu-cB7xndtx-uUJC4lUnLRcO_3Q=`
- **Rede**: `lakehouse-net`

---

## 3. MinIO

O MinIO é uma solução de armazenamento de objetos que atua como a camada de armazenamento para a arquitetura Lakehouse. Ele é utilizado para armazenar grandes volumes de dados e suporta APIs compatíveis com o S3.

### Configuração:
- **Imagem**: `bitnami/minio:latest`
- **Nome do Container**: `minio-lakehouse`
- **Portas**: 
  - `9000:9000` (API S3)
  - `9001:9001` (console web do MinIO)
- **Volumes**:
  - `./crypto_opensouce_lakehouse_data_container/data_persist/data:/data`
  - `./crypto_opensouce_lakehouse_data_container/data_persist/config:/opt/bitnami/minio/.minio`
- **Variáveis de Ambiente**:
  - `MINIO_ROOT_USER=bruno`
  - `MINIO_ROOT_PASSWORD=minioadmin`
- **Rede**: `lakehouse-net`

---

## 4. Apache Hive

O Apache Hive é utilizado como solução de data warehouse, fornecendo consultas SQL sobre grandes volumes de dados. Ele está configurado para armazenar seus metadados no banco de dados PostgreSQL.

### Configuração:
- **Imagem**: `apache/hive:4.0.0`
- **Nome do Container**: `hive-container`
- **Portas**:
  - `10000:10000` (servidor Thrift)
  - `10002:10002` (WebHCat)
  - `9083:9083` (Metastore do Hive)
- **Variáveis de Ambiente**:
  - `JAVA_HOME=/usr/local/openjdk-8`
  - `HADOOP_HOME=/opt/hadoop`
  - `HIVE_HOME=/opt/hive`
- **Rede**: `lakehouse-net`

---

## 5. Apache Spark (Master)

O Apache Spark é uma estrutura de computação distribuída usada para processamento de dados em grande escala. O serviço Spark Master está configurado para orquestrar tarefas submetidas ao cluster Spark.

### Configuração:
- **Imagem**: `bitnami/spark:latest`
- **Nome do Container**: `spark-lakehouse`
- **Portas**:
  - `7077:7077` (RPC do Spark Master)
  - `8081:8081` (interface web do Spark)
- **Volumes**: `./crypto_opensouce_lakehouse_data_container/spark:/opt/spark/work-dir`
- **Variáveis de Ambiente**:
  - `SPARK_MODE=master`
  - `HIVE_METASTORE_URI=thrift://hive-container:9083`
- **Rede**: `lakehouse-net`

---

## 6. Apache Spark (Worker)

O serviço Spark Worker é responsável por executar tarefas como parte do cluster distribuído do Spark. Ele se conecta ao Spark Master e processa as tarefas.

### Configuração:
- **Imagem**: `bitnami/spark:latest`
- **Nome do Container**: `spark-worker-lakehouse`
- **Portas**: `8082:8082` (interface web do Spark Worker)
- **Depende de**: `spark-lakehouse (Master)`
- **Variáveis de Ambiente**:
  - `SPARK_MODE=worker`
  - `SPARK_MASTER_URL=spark://spark-lakehouse:7077`
  - `SPARK_WORKER_MEMORY=2G`
  - `SPARK_WORKER_CORES=1`
- **Rede**: `lakehouse-net`

---

## 7. TensorFlow com Jupyter Notebook

O serviço TensorFlow com Jupyter Notebook é utilizado para desenvolvimento de aprendizado de máquina. Ele está configurado para executar um Jupyter Notebook acessível a partir da máquina host.

### Configuração:
- **Imagem**: `tensorflow/tensorflow:nightly-gpu-jupyter`
- **Nome do Container**: `crypto-ia-project`
- **Portas**: `8888:8888` (Jupyter Notebook)
- **Variáveis de Ambiente**:
  - `NVIDIA_VISIBLE_DEVICES=all`
  - `NVIDIA_DRIVER_CAPABILITIES=compute,utility`
- **Rede**: `lakehouse-net`

---

## Redes

Os serviços se comunicam por meio de uma rede bridge personalizada chamada `lakehouse-net`. Isso permite que os containers resolvam uns aos outros pelo nome e garante que estejam isolados da rede do host.

### Configuração:
- **Nome da Rede**: `lakehouse-net`
- **Driver**: `bridge`
