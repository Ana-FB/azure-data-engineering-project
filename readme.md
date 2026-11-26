# 🚀 End-to-End Azure Data Engineering Project | Medallion Architecture & BI

Proyecto integral de **Ingeniería de Datos en Microsoft Azure** que implementa una arquitectura **Medallion (Bronze, Silver, Gold)** para la ingesta multiorigen, control de calidad automatizado, optimización de cómputo en Spark, despliegue continuo (CI/CD) y modelado dimensional para **Power BI**.

---

## 🏗️ Arquitectura y Stack Tecnológico

* **Orquestación e Ingesta:** Azure Data Factory con pipelines dinámicos guiados por metadatos y desencadenador programado.
* **Data Lake (Almacenamiento):** Azure Data Lake Storage Gen2 bajo formato transaccional **Delta Lake**.
* **Procesamiento Distribuido:** Azure Synapse Analytics (Apache Spark Pools + PySpark).
* **Capa Analítica (Serving Layer):** Synapse Serverless SQL Pool (`OPENROWSET` dinámico sobre Delta Lake).
* **Visualización y BI:** Power BI Desktop (Modelo Estrella + Medidas DAX).
* **DevOps / DataOps (CI/CD):** GitHub & GitHub Actions con separación de entornos y validación automatizada de artefactos.

---

## 🔄 Flujo de Ingesta y Orquestación (ADF + Synapse)

### 1. Ingesta Multiorigen (`PL_Ingesta_Staging_Bronze`)
Extracción parametrizada desde **Azure SQL Database** (iteración dinámica con `Lookup` + `ForEach`), consumo de **API REST** (clientes) y archivos **CSV** (productos), con transformación a formato Delta en la capa Bronze y limpieza de staging temporal:

<img width="1345" height="475" alt="image" src="https://github.com/user-attachments/assets/74098c0b-ba6e-4fdc-87d9-83a179336514" />


### 2. Orquestación End-to-End y Optimización Spark (`PL_Orquestacion_EndToEnd`)
Ejecución secuencial consolidada en **`NB_Orquestador_Spark`** mediante `mssparkutils.notebook.run`, evitando múltiples arranques en frío del clúster Spark y reduciendo el tiempo de procesamiento en **más del 60 % (de ~21 min a ~8 min)**:

<img width="879" height="553" alt="image" src="https://github.com/user-attachments/assets/7d1318ee-57db-499e-b7e3-c7a84dbdbf26" />


---

## 🗄️ Organización del Data Lake (ADLS Gen2)

Estructura de almacenamiento en el contenedor `datos` siguiendo el patrón Medallion (`bronze`, `silver`, `gold`):

<img width="1256" height="619" alt="image" src="https://github.com/user-attachments/assets/fd2dee0f-b2bb-4e68-a293-1eec87c367ae" />


---

## 🧩 Modelado de Datos (Silver ➡️ Gold)

### 1. Capa Silver: Entidades Limpias, Normalizadas y Cuarentena
* **`NB_Bronze_a_Silver`:** Limpieza, estandarización de tipos de datos y deduplicación.
* **`NB_Silver_Cuarentena_DataQuality`:** Aislamiento de pedidos sin correspondencia de cliente para auditoría.
* **`NB_Test_Silver`:** Pruebas automatizadas de integridad referencial antes de promover a Gold.

<img width="860" height="607" alt="image" src="https://github.com/user-attachments/assets/dadf4596-d44e-487d-9a12-b6261184e314" />


### 2. Capa Gold: Modelo Dimensional (Esquema Estrella)
* **`NB_Silver_a_Gold`:** Construcción de tabla de hechos y dimensiones con claves subrogadas (`_key`), expuestas mediante vistas SQL (`OPENROWSET`) en **Synapse Serverless SQL Pool** (`db_gold_powerbi`):
  * **`vw_fact_ventas`:** Granularidad a nivel de línea de detalle de pedido (`cantidad`, `precio_unitario`, `importe`).
  * **`vw_dim_cliente`**, **`vw_dim_producto`**, **`vw_dim_ubicacion`**, **`vw_dim_fecha`**.

<img width="860" height="607" alt="image" src="https://github.com/user-attachments/assets/d0284d0a-badc-427b-8dab-98c57253d32b" />

<img width="1275" height="714" alt="image" src="https://github.com/user-attachments/assets/647b256f-91f6-4b60-8487-3c967af38065" />

Tablas Delta materializadas en la carpeta `/gold/` listas para consumo en BI:

<img width="860" height="579" alt="image" src="https://github.com/user-attachments/assets/4814543e-8281-4c59-8ed2-588184024ee2" />


---

## ⚙️ Estrategia de CI/CD y Gobernanza de Ramas

El repositorio implementa buenas prácticas de **DataOps** con separación de carpetas raíz (`/adf` y `/synapse`):

* **`develop`:** Rama de desarrollo donde se construyen y validan pipelines, notebooks y scripts SQL.
* **`main`:** Rama principal con los artefactos validados.
* **`adf_publish` / `workspace_publish`:** Ramas gestionadas por Azure que almacenan las plantillas ARM compiladas tras cada publicación.
* **GitHub Actions (`.github/workflows/ci-cd-azure.yml`):** Pipeline automatizado que valida la sintaxis de todos los archivos JSON de ADF y notebooks `.ipynb` de Synapse en cada *push* o *Pull Request*, dejando preparado el despliegue continuo por ARM Templates.

---

## 📂 Estructura del Repositorio

```text
├── .github/
│   └── workflows/
│       └── ci-cd-azure.yml          # Pipeline de validación CI/CD en GitHub Actions
├── adf/
│   ├── dataflow/                    # Mapping Data Flows de transformación (Staging -> Bronze)
│   ├── dataset/                     # Datasets parametrizados (Azure SQL, API REST, CSV, ADLS Gen2)
│   ├── factory/                     # Configuración del recurso Azure Data Factory (df-ana)
│   ├── integrationRuntime/          # Entorno de ejecución (IRAzureBrazilSouth)
│   ├── linkedService/               # Conexiones a orígenes de datos, Data Lake y Synapse
│   ├── pipeline/                    # Pipelines de ingesta (PL_Ingesta_Staging_Bronze) y orquestación (PL_Orquestacion_EndToEnd)
│   ├── trigger/                     # Desencadenador programado diario (TR_Diario_Orquestacion_EndToEnd)
│   └── publish_config.json          # Configuración de publicación de ARM Templates
├── synapse/
│   ├── notebook/                    # Notebooks PySpark (Bronze -> Silver -> Quality -> Test -> Gold + Orquestador)
│   └── sqlscript/                   # Scripts SQL Serverless (Base de datos y vistas OPENROWSET para Power BI)
└── readme.md                        # Documentación completa del proyecto
```
---

## 📊 Dashboard Ejecutivo en Power BI

<img width="906" height="750" alt="image" src="https://github.com/user-attachments/assets/e621aec7-c173-4cd2-b4df-35009ff8a40e" />

---

👩‍💻 Autora

Ana Ferreira — Data Engineer | Business Intelligence Analyst

Este proyecto fue desarrollado como una demostración práctica de una arquitectura moderna de datos de extremo a extremo (End-to-End), utilizando Microsoft Azure y Power BI.

![LinkedIn](https://img.shields.io/badge/LinkedIn-Perfil-blue?logo=linkedin)

![Email](https://img.shields.io/badge/Email-Contacto-red?logo=gmail)


