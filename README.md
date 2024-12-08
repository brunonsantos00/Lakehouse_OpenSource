# Lakehouse_OpenSource_IA_Ativos

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

## Estrutura de Pastas

Certifique-se de criar as seguintes pastas antes de iniciar o ambiente:

```bash
mkdir -p ./volumes/minio/data
mkdir -p ./volumes/minio/config
mkdir -p ./volumes/postgresql/data
mkdir -p ./volumes/airflow/dags
mkdir -p ./volumes/airflow/logs
mkdir -p ./volumes/airflow/plugins
mkdir -p ./volumes/trino/etc
mkdir -p ./volumes/tensorflow/notebooks
