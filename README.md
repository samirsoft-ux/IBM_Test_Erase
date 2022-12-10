# Implementación de Cloud Object Storage sobre un cluster de Openshift

Se recomienda hacer esta implementación sobre un SO Linux para agilizar la instalación de herramientas y ejecución de comandos. Estas implementaciones se realizan suponiendo que se tiene un SO Linux.

## Tabla de contenido 📑
1. Instalación del plugin de Cloud Object Storage sobre el clúster utilizando Helm
2. Almacenamiento de la información del Cloud Object Storage en el Cluster
3. Asoicación de un Bucket con el cluster

## Pre-Requisitos :pencil:
* La cuenta tiene una instancia en plan Standard de Cloud Object Storage <a href="https://cloud.ibm.com/objectstorage/create"> IBM Cloud Object Storage </a>
* Haber hecho login sobre IBM Cloud desde la CLI con el siguiente comando: "ibmcloud login"
* Tener acceso al clúster de Openshift mediante los comandos kubectl, de no ser así, ir al clúster de Openshift sobre IBM Cloud, hacer click en el menú “Actions”, elegir la opción “Connect via CLI”, desplegar el token que se muestra en la nueva ventana y ejecutar, a través de una terminal, el comando que se muestra al haber desplegado el token

## Consideraciones 📑
* Estos comandos se pueden ejecutar desde la terminal de su computadora personal o desde la terminal de IBM Cloud
* Para copiar los comandos de esta guía debe de omitir las "" que se encuentran al inicio y al final de este
* Todos los parámetros de los comandos que se van a usar en esta guía y que esten dentro de <> deben de ser modificados acorde a lo que se especifica

## 1. Instalación del plugin de Cloud Object Storage sobre el clúster utilizando Helm
   
1. Ingresar a su terminal o a la terminal de IBM Cloud

2. Ingrese el siguiente comando para iniciar la instalación: "brew install helm"

3. Ingrese el siguiente comando para agregar el repositorio ibmc: "helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm"

4. Ingrese el siguiente comando para actualizar los repositorios: "helm repo update"

5. Ingrese el siguiente comando para desempaquetar el repositorio descargado: "helm fetch --untar ibm-helm/ibm-object-storage-plugin"

6. Ingrese el siguiente comando para la instalación del plugin de forma local: "helm plugin install ./ibm-object-storage-plugin/helm-ibmc"
   <br /> **Posibles errores**
   * ```Para sistemas operativos MAC```: Ingrese el siguiente comando para dar permisos de escritura y lectura: "chmod 755 ~/Library/helm/plugins/helm-ibmc/ibmc.sh"
   * ```Para sistemas operativos Windows```: Ingrese el siguiente comando para dar permisos de escritura y lectura: "chmod 755 ~/.helm/plugins/helm-ibmc/ibmc.sh"

7. Ingrese el siguiente comando para verificar la instalación del plugin: "helm ibmc –help"

8. Ingrese el siguiente comando para la instalación del plugin en el cluster: "helm ibmc install ibm-object-storage-plugin ibm-helm/ibm-object-storage-plugin --set license=true --set bucketAccessPolicy=true"
   **Tener en cuenta**
   * ```En caso que el cluster no se encuentre dentro de una VPC```: Modificar la porción del comando "bucketAccessPolicy=true" por "bucketAccessPolicy=false" para permitir la conexión del bucktet con el cluster
   * ```En caso que el cluster no se encuentre dentro de una VPC```: Mantener la porción del comando "bucketAccessPolicy=true"

9. Ingrese el siguiente comando para verificar los Pods creados para el plugin: "kubectl get pod --all-namespaces -o wide | grep object"
   **Tener en cuenta**
   * Debemos tener un Pod ejecutando y una cantidad igual a la cantidad de worker nodes de nuestro clúster de ibmcloud-object-storage-driver ejecutandose

10. Ingrese el siguiente comando para verificar los Storage Class creados para el plugin: "kubectl get storageclass | grep s3" 
<br />


## 2. Almacenamiento de la información del Cloud Object Storage en el Cluster

1. Anotar el nombre del Cloud Object Storage
   **Pasos para ubicar y generar una APIKEY**
   * En el portal de IBM Cloud expandir la barra lateral izquierda
   * Dentro de esta barra ingresar a la sección "Resource list"
   * En el listado de los recursos ingresar al apartado "Storage"
   * Copiar y almacenar el nombre del CLoud Object Storage

2. Anotar el APIKEY de tu cuenta de IBM Cloud
   **Pasos para ubicar y generar una APIKEY**
   * Verificar que la cuenta de IBM Cloud posea permisos de Manager para la creación de las credenciales de acceso
   * En el portal de IBM Cloud dirigirse al botón "Manage" dentro de este botón ingresar a la sección "Access (IAM)"
   * Dentro de la barra lateral izquierda ingresar a la sección "API keys"
   * Seleccionar el botón "Create +"
   * Rellenar los datos que te piden para la creación del APIKEY
   * Copiar y almacenar el APIKEY generado

3. Anotar el GUID del Cloud Object Storage
   **Pasos para ubicar y generar una APIKEY**
   * En el terminal donde se viene trabajando ingresar el siguiente comando para visualizar el GUID: "ibmcloud resource service-instance <service_name> | grep GUID"
   * Copiar y almacenar el GUID mostrado

4. Ingrese el siguiente comando para crear un Secret y almacenarlo en el cluster: "kubectl create secret generic <nombre_secret> --type=ibm/ibmc-s3fs --from-literal=api-key=<api_key> --from-literal=service-instance-id=<service_instance_guid>"

5. Copiar y almacenar el <nombre_secret> que se ha creado
<br />
 
## 3. Asoicación de un Bucket con el cluster

1. Crear el bucket desde la consola de IBM Cloud
   **Pasos para ubicar y crear un Bucket**
   * En el portal de IBM Cloud expandir la barra lateral izquierda
   * Dentro de esta barra ingresar a la sección "Resource list"
   * En el listado de los recursos ingresar al apartado "Storage"
   * Ingresar a la instancia de CLoud Object Storage que posee
   * Dentro de la barra lateral izquierda ingresar a la sección "Bucktes"
   * Seleccionar el botón "Create bucket +"
   * Dentro de esta vista seleccionar el icono "->" dentro del cuadrado que posee el título de "Quickly get started"
   * Seguir la guía de configuración del Bucket
   * Copiar y almacenar el nombre del Bucket creado

2. Crear el archivo de configuración "pvc.yaml" para configurar los parámetros del Bucket dentro del cluster
   **Pasos crear el archivo de configuración**
   * Descargar el archivo de configuración "pvc.yaml" que se encuentra en el repositorio
   * Modificar el archivo de configuración en base a las variables que posea, puede guiarse del archivo de configuración "pvcTemplate.yaml" que se encuentra en el repositorio
   * En caso necesite más información acerca del archivo de configuración ingrese a la siguiente <a href="https://cloud.ibm.com/docs/openshift?topic=openshift-storage_cos_apps&mhsrc=ibmsearch_a&mhq=Persistent+Volume+Claim"> documentación </a>
   * Guardar los cambios realizados en el archivo de configuración "pvc.yaml"
   * Ingresar al terminal donde se viene trabajando e ingrese el siguiente comando para ejecutar el yaml en el cluster: "kubectl apply -f pvc.yaml"

  <p align="center">
   <img src=https://github.com/emeloibmco/Gateway-Appliance-Juniper-vSRX-version-20.4/blob/main/Imagenes/Segmentos.gif>
   </p>
<br />

## Referencias :mag:
* <a href="https://github.com/emeloibmco/VPC-Conexion-VPN"> VPC Conexión VPN</a>. 
* <a href="https://github.com/emeloibmco/PowerVS-Conectividad"> PowerVS-Conectividad</a>. 
* <a href="https://www.juniper.net/documentation/us/en/software/junos/nat/topics/topic-map/nat-security-source-and-source-pool.html"> Network Address Translation User Guide</a>. 


## Autores :black_nib:
Italo Silva IBM Cloud Tech Sales Perú.
