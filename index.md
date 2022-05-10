Google Cloud CLI Cheatsheet (en español)
========================================

Variables de entorno
====================

`DEVSHELL_PROJECT_ID`
  * Variable de entorno creada automáticamente en la terminal cloud. Contiene el id del proyecto creado por Qwiklabs para el lab.


`export PROJECT_ID=$(gcloud info --format='value(config.project)')`
  * Crea una variable de entorno con el id del proyecto seleccionado

`export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32`
  * Crea una variable con la IP del Cloud Shell

`MYSQLIP=$(gcloud sql instances describe my_db --format="value(ipAddresses.ipAddress)")`
  * Crea una variable con la IP de una instancia Cloud SQL



Comandos
========

## gcloud CLI

`gcloud auth list`
  * Lista las cuentas de GCP activas en la CLI

`gcloud projects list`
  * Lista los proyectos disponibles para la cuenta activa

`gcloud components list`
  * Lista los componentes disponibles en la CLI

`gcloud config list --all`
  * Muestra la configuración del entorno (CLI)

`gcloud config list project`
  * Obtiene el ID del proyecto seleccionado en la CLI

`gcloud config get-value $PARAMETER`
  * Obtiene el valor configurado para el parámetro indicado

`gcloud config set account $ACCOUNT`
  * Establece la cuenta activa para trabajar en la CLI

`gcloud config set compute/region $REGION`
  * Establece la region por defecto para la CLI

`gcloud config set compute/zone $ZONE`
  * Establece la zona por defecto para la CLI

`gcloud beta interactive`
  * Inicia el modo interactivo de la CLI. Este posee autocompletado de comandos y nombres de recursos.


## APIs

`gcloud services enable dataflow.googleapis.com`
  * Habilita una API en la cuenta de usuario


## IAM

```
gcloud iam service-accounts create my-account --display-name my-account
gcloud projects add-iam-policy-binding $PROJECT --member=serviceAccount:my-account@$PROJECT.iam.gserviceaccount.com --role=roles/bigquery.admin
gcloud iam service-accounts keys create key.json --iam-account=my-account@$PROJECT.iam.gserviceaccount.com
export GOOGLE_APPLICATION_CREDENTIALS=key.json
```
  * Crea una service account

`gcloud projects get-iam-policy $PROJECT --format='table(bindings.role)' --flatten="bindings[].members" --filter="bindings.members:$USER_EMAIL"`
  * Lista los roles IAM asociados a la cuenta (email)

`gcloud projects add-iam-policy-binding $PROJECT --member=user:$USER_EMAIL --role=roles/dataflow.admin`
  * Agrega un rol a la cuenta de usuario

`gcloud projects add-iam-policy-binding $PROJECT --member=user:$USER_EMAIL --role=roles/compute.networkAdmin`
  * Asigna el rol Compute Network Admin para poder trabajar con PGA


## Compute Engine

`gcloud compute project-info describe --project $PROJECT_ID`
  * Muestra la configuración del proyecto para gestionar instancias de Compute Engine

`gcloud compute instances describe $INSTANCE_NAME`
  * Muestra la configuración de una VM

`gcloud compute instances create $INSTANCENAME --machine-type n1-standard-2 --zone $ZONE`
  * Crea una VM del tipo n1-standard-2

`gcloud compute ssh $INSTANCE_NAME --zone $ZONE`
  * Inicia una conexión por SSH hacia una VM en Compute Engine

`gcloud compute networks subnets update default --region=$REGION --enable-private-ip-google-access`
  * Habilita el uso de Private Google Access (PGA) para las VM en la subnet. Esto se usa en caso de que se desactiven las IP publicas para los worker nodes.



## Cloud Storage

`gsutil mb -p <PROJECT_ID> -c <TYPE> -l <REGION> gs://mybucket`
  * Crea un bucket en cloud storage

`gsutil -m cp -r gs://cloud-training/automl-lab-clouds/* gs://$DEVSHELL_PROJECT_ID-vcm/`
  * Copia archivos desde un bucket hacia otro. (r) para copia recursiva.

`gsutil ls gs://$DEVSHELL_PROJECT_ID-vcm/`
  * Lista los archivos en un bucket. Agrega (*) al final para mostrarlos de forma recursiva.


## Cloud SQL

`gcloud sql instances create my_db --tier=db-n1-standard-1 --activation-policy=ALWAYS`
  * Crea una instancia en Cloud SQL

`gcloud sql users set-password root --host % --instance my_db --password PASSWORD`
  * Establece la contraseña para el usuario root

`gcloud sql instances patch my_db --authorized-networks $ADDRESS`
  * Otorga acceso a una red IP hacia una instancia SQL

`mysqlimport --local --host=$MYSQLIP --user=root --password --ignore-lines=1 --fields-terminated-by=',' my_db my_file.csv-*`
  * Importa archivos CSV a una instancia Cloud SQL

`CREATE ROW ACCESS POLICY apac_filter ON mydataset.mytable GRANT TO ("group:sales-apac@example.com") FILTER USING (Region="APAC");`
  * SQL: Crea una política de control de acceso a nivel de filas, usando un grupo y region especificos

```
CREATE OR REPLACE TABLE my_dataset.restored_table AS
SELECT 
	* 
FROM 
	my_project.my_dataset.my_table 
FOR SYSTEM_TIME AS OF 
	TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 24 HOUR);
```
  * SQL: Restaura una copia de una tabla usando un punto en el tiempo (snapshot) hasta 7 días.

```
NOW=$(date +%s)
SNAPSHOT=$(echo "($NOW - 120)*1000" | bc)
bq --location=US cp mydataset.mytable@$SNAPSHOT mydataset.restored_table
```
  * Restaura una copia de hace 2 minutos de una tabla usando la línea de comandos


## BigQuery

`bq ls`
  * Lista los datasets del proyecto

`bq ls mydataset`
  * Lista las tablas del dataset

`bq show --schema --format=prettyjson logs.logs`
  * Muestra el schema de una tabla

`bq show --schema --format=prettyjson logs.logs | sed '1s/^/{"BigQuery Schema":/' | sed '$s/$/}/' > schema.json`
  * Guarda el schema de una tabla en formato JSON (para lanzar pipelines Dataflow template)

`bq mk taxirides`
  * Crea un dataset dentro del proyecto actual

```
bq mk --time_partitioning_field timestamp \
--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,timestamp:timestamp \
-t taxirides.realtime
```
  * Crea una tabla vacía con definición de schema y particionada por una columna

`bq load --source_format=CSV --autodetect --noreplace nyctaxi.2018trips gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv`
  * Carga datos en BigQuery desde un CSV almacenado en Cloud Storage

Cuando trabajes con arrays:
  * Crear array con ARRAY_AGG(<column>)
  * Contar el numero de elementos con ARRAY_LENGTH(<array>)
  * Contar los elementos distintos con ARRAY_AGG(DISTINCT <field>)
  * Ordenar los elementos con ARRAY_AGG(<field> ORDER BY <field>)
  * Limitar la cantidad elementos con ARRAY_AGG(<field> LIMIT 5)
  * Separar los elementos en filas distintas con UNNEST(<array>)

Cuando trabajes con structs:
  * Para consultar dentro del STRUCT, recuerda usar un CROSS JOIN (o coma) en el FROM

`SELECT APPROX_QUANTILES(mycolumn, 10)[OFFSET (5)] FROM mydataset.mytable`
  * Calcula la mediana de un set de datos (divide por 10 y toma el número en la mitad)



## Pub/Sub

`gcloud pubsub topics create $TOPIC`
  * Crea un topic

`gcloud pubsub topics publish $TOPIC --message "hello"`
  * Publica un mensaje en un topic


## Cloud Natural Language API

`curl "https://language.googleapis.com/v1/documents:classifyText?key=${API_KEY}" -s -X POST -H "Content-Type: application/json" --data-binary @request.json`
  * Usa cURL para enviar un documento a la API, la cual retorna las categorías a las cuales pertenece el texto.


## Dataflow

```
gcloud dataflow jobs run job1 \
--gcs-location gs://dataflow-templates-us-central1/latest/Word_Count \
--region $REGION \
--staging-location gs://$PROJECT/tmp \
--parameters inputFile=gs://dataflow-samples/shakespeare/kinglear.txt,output=gs://$PROJECT/results/outputs 
```
  * Lanza un pipeline usando un template

`Parametro: --disable-public-ips`
  * Deshabilita el uso de IP publicas en los workers (los desconecta de internet)
