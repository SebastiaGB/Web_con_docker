# Blockstellart Bootcamp
Este proyecto fue generado para el Bootcamp de AWS de **Joan Amengual**
![ECS](./Images/Arquitectura%20ECS.jpg)

# Despliegue de VPC en AWS

1. Creación de VPC
![VPC](./Images/VPC.jpg)

Para hacer el [despliegue](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/02-setup/02-createaccount/01-stackdeploy) de la VPC puede utilizarse el archivo **vpc.yaml**. En este archivo se omite la parte de Cloud9.

# Amazon ECR

#### Creación de repositorios
Pueden seguirse los pasos tal cual estan en la documentación [oficial](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/03-ecr/create)


#### Construir imagenes de Docker
Las carpetas; Cats, Dogs y Web contienen sus propios archivos de Docker utilizando los siguientes comandos:
```
docker build -t cats .
docker build -t dogs .
docker build -t web .
```
#### Etiquetar y enviar imágenes a ECR
Para este proceso puede utilizarse la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/03-ecr/push) original


# Amazon ECS

#### Creación de Cluster
![ECS](./Images/ECS%20Cluster.jpg)
La creación del cluster puede ser aplicada con el proceso de la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/01-cluster/create)

#### Grupos de seguridad de clústeres
Configurar los grupos de seguridad también puede ser aplicado tal como se menciona en la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/01-cluster/asg)

#### Definición de tareas
![ECSTareas](./Images/ECS%20Tareas.jpg)

Para la creación de tareas puede ser siguiendo las instrucciones:
1. Definición de [tarea web](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/02-taskdef/web)
2. Definición de [tarea cats](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/02-taskdef/cats)
3. Definición de [tarea dogs](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/02-taskdef/dogs)
4. Modificar el [rol IAM](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/02-taskdef/iam) de la tarea ECS


# Servicios ECS
![ECSServicios](./Images/ECS%20Servicios.jpg)

#### Creación de ALB y Servicios
1. [Creación de ALB](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/03-service/alb) la documentación es la correcta.
2. [Creación de Servicio Web](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/03-service/web)
3. [Creación de Servicio Cats](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/03-service/cats)
4. [Creación de Servicio Dogs](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/03-service/dogs)

#### Validación de servicio
Para verificar la funcionalidad de los servicios la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/04-ecs/check) muestra el proceso para confirmar el estatus de los servicios.


# Monitoreo
![Monitoreo](./Images/Monitoreo.jpg)

#### Container Insights
El proceso de monitorio funciona acorde a la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/05-monitoring/01-insights). Algunos cambios en la interfaz, como posiciones o ubicaciones de algunos opciones

#### Log Routing
Las consultas y visualización de la información funciona tal como se muestra en la [sección](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/05-monitoring/02-firelens)

# Auto Scaling del Servicio

#### Configurar el escalado automático del servicio
Seguir el proceso de la configuración de escalado funciona acorde a la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/07-autoscale/01-service/config)


#### Prueba de carga de servicio
El proceso de auto scaling fue realizado desde de Cloud9 utilizando **siege**, sin embargo al no configurar lo desde el inicio hay otras alternativas:
1. Crear una nueva instancia EC2 con una AMI de Amazon Linux 2 e instalar **siege** con los siguientes comandos:
```
sudo yum update -y
sudo amazon-linux-extras install epel -y
sudo yum install siege -y
siege --version
```

y después ejecutar:
```
siege -c 200 -i http://<<url ALB>>/
```
y el proceso sería muy similar.
**Nota:** La conexión puede ser desde instance connect o desde la linea de comandos de un equipo personal.

2. Utilizar [Artillery](https://www.npmjs.com/package/artillery)

```
npm install -g artillery
```
Se debe crear un archivo .yaml con una configuración similar a:
``` js
config:
  target: 'http://your-alb-dns-name/'
  phases:
    - duration: 60 # Duración en segundos
      arrivalRate: 200 # Usuarios por segundo
scenarios:
  - flow:
      - get:
          url: "/"

```

**Nota:** Tenga encuenta que puede depender de la velocidad de internet y puede tener intermitencia o puede fallar el proceso mostrando un mensaje como: *errors.ETIMEDOUT*.


#### Monitoreo
El monitoreo funciona como se menciona en la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/07-autoscale/01-service/monitoring)


# Auto Scaling del Cluster

#### Capacity Provider
La [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/07-autoscale/02-cluster/capacityprovider) funciona y muestra la información mencionada.

#### Configurar grupo de autoescalado
Puede ser utilizada la [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/07-autoscale/02-cluster/asg) proporcionada.

#### Prueba de carga del clúster
La prueba de carga del clúster puede ser realizada como se realizó para los servicios.

#### Monitoreo
La [documentación](https://catalog.us-east-1.prod.workshops.aws/workshops/8c9036a7-7564-434c-b558-3588754e21f5/en-US/03-console/07-autoscale/02-cluster/monitoring) proporciona el contenido necesario para realizar la actividad.