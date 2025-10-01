# Proyecto: Despliegue de contenedores en AWS con ECS

El objetivo del proyecto es, mediante **contenedores Docker**, subir aplicaciones a AWS y crear un sistema capaz de desplegarlas de forma **segura, escalable y automatizada**.

Se utilizarán los siguientes recursos principales:

* **ECR**: repositorios de imágenes.
* **ECS**: orquestador de contenedores.
* **EC2 / Fargate**: ejecución de tareas.
* **ALB**: balanceo de carga.
* **VPC**: red privada (2 subredes públicas y 2 privadas).
* **CloudWatch**: registros y métricas.
* **Auto Scaling**: escalado automático de servicios y de la infraestructura.

---

## Descripción del proyecto

El proyecto está compuesto por tres carpetas principales: Dioses, Semidioses y WebApp.
Cada una contiene una imagen de Docker, un archivo de configuración (.config) para definir la estructura de Docker y un archivo HTML. Además, la carpeta WebApp incluye un directorio adicional con imágenes.

### Estructura del proyecto

```bash
Proyecto/
├── Dioses/
│   ├── Dockerfile
│   ├── default.conf
│   └── index.html
│
├── Semidioses/
│   ├── Dockerfile
│   ├── default.conf
│   └── index.html
│
├── WebApp/
│   ├── Dockerfile
│   ├── default.conf
│   ├── index.html
│   └── Imagenes/
│       ├── Dioses.avif
│       ├── Semidioses.avif
│       └── ...
└── vpc.yaml
└── README.md
```
Para la implementación se utiliza una pila de AWS CloudFormation (proporcionada por el instructor), lo que permite desplegar la infraestructura como código siguiendo buenas prácticas. Esta plantilla crea la VPC, conformada por dos subredes privadas y dos públicas, además de los grupos de seguridad necesarios.

### 1. Repositorios en Amazon ECR

El primer paso consiste en configurar Amazon ECR, creando tres repositorios (uno por cada servicio) donde se subirán las imágenes de Docker.
Para ello es indispensable:

Tener Docker instalado y en ejecución.

Disponer de permisos de acceso a AWS.

Generar un par de claves de acceso denominado LOCAL CLI, que permitirá autenticarse desde la consola, construir los contenedores y subir las imágenes.

### 2. Creación del cluster ECS

Se define un cluster privado de ECS que alojará los servicios mediante EC2 y Fargate. Antes de ello, se deben realizar las siguientes configuraciones:

Crear un Auto Scaling Group (ASG) con capacidad t2.medium (ya que la capa gratuita no es suficiente).

Definir un nuevo rol de instancia EC2.

Lanzar dos instancias EC2 dentro de la VPC creada por CloudFormation, ubicándolas exclusivamente en las subredes privadas.

Asociar las instancias al grupo de seguridad correspondiente.

### 3. Definición de tareas en ECS

Se configuran tres task definitions, una para cada imagen subida a ECR. Cada tarea se define con los recursos mínimos de CPU y memoria necesarios. En este paso:

Se crea un nuevo rol de ejecución de tareas, al que se asigna la política de acceso a CloudWatch para permitir la monitorización.

Se especifica como imagen la última versión subida a cada repositorio de ECR.

Se mapea el puerto 80/TCP, ya que se usará el protocolo HTTP.

La tarea Dioses se ejecuta en Fargate, mientras que Semidioses y WebApp se ejecutan en EC2.

### 4. Servicios en ECS

Se crean tres servicios en el cluster ECS para mantener y gestionar las tareas de forma continua. Cada servicio lanza 2 tareas de su definición correspondiente:

Dioses → EC2

WebApp → EC2

Semidioses → Fargate (requiere ubicarlo correctamente en las subredes privadas de la VPC).

### 5. Balanceador de carga (ALB)

Finalmente, se despliega un Application Load Balancer (ALB) expuesto a internet, configurado con un listener en el puerto 80/HTTP.

Como grupo de destino principal se define WebApp, en modo instancia EC2 y usando el protocolo HTTP.

Para Dioses y Semidioses se crean grupos de destino adicionales, cada uno con su ruta correspondiente.

En el caso de Fargate (Semidioses), es fundamental gestionar correctamente las redes para que las tareas se ubiquen en las subredes privadas de la VPC.

---
## Créditos
Proyecto desarrollado como parte del bootcamp **AWS DevOps & Cloud Computing de Blockstellar**.




