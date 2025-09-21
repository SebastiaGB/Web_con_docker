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

## 1. Repositorios de imágenes (ECR)

1. Se crean **3 repositorios en ECR**, uno por cada aplicación o servicio.
2. Se construyen imágenes Docker localmente (`docker build`).
3. Se suben al ECR con `docker push` utilizando credenciales de AWS.

## 2. Cluster ECS

* Se crea un **cluster ECS privado** dentro de la **VPC**.
* Dos opciones de ejecución:

  * **EC2 launch type**: se crean instancias EC2 que forman parte del cluster.
  * **Fargate**: ejecución serverless sin necesidad de instancias EC2.

> En este ejemplo: se utilizarán instancias **EC2** en subredes privadas para ejecutar los contenedores.

---

## 3. Definición de tareas (Task Definitions)

Cada aplicación se define como una **Task Definition** que incluye:

* Imagen de contenedor desde ECR.
* Asignación de CPU y memoria.
* Variables de entorno y configuración de puertos.
* Uso del **driver awslogs** para enviar logs a CloudWatch.

## 4. Servicios ECS

* Se crean **servicios ECS** basados en las Task Definitions.
* Cada servicio garantiza que siempre haya un número deseado de tareas en ejecución.
* Se asocian a un **Target Group** del **Application Load Balancer (ALB)**.
* Configuración del ALB:

  * Listener en **HTTP (80)** o **HTTPS (443)**.
  * Enruta tráfico al target group → tareas ECS.

---

## 5. Networking y seguridad

* **VPC**: con 2 subnets públicas (ALB, NAT Gateway) y 2 privadas (ECS/EC2).
* **Security groups**:

  * ALB: permite tráfico entrante HTTP/HTTPS desde Internet.
  * ECS/EC2: permite tráfico solo desde el ALB.
* **IAM roles**:

  * **Task execution role**: para descargar imágenes de ECR y enviar logs.
  * **Task role**: para que el contenedor acceda a otros servicios AWS (si es necesario).

---

## 6. Autoescalado de servicios ECS

* Se habilita **Service Auto Scaling** para cada servicio ECS.
* Se configuran políticas de escalado basadas en métricas de **CloudWatch**.

  * Ejemplo: escalar entre **2 y 10 tareas** según el uso de CPU > 70%.


## 7. Autoescalado de infraestructura (EC2 Auto Scaling Group)

Si se usa **EC2 launch type**:

* Las instancias del cluster ECS están en un **Auto Scaling Group (ASG)**.
* Se configura para que escale entre un mínimo y un máximo de instancias.
* Métricas típicas:

  * CPU media de las instancias.
  * Capacidad reservada de ECS.
* El escalado de EC2 se coordina con el de ECS para mantener capacidad suficiente.

---

## 8. Monitoreo y registros

* Los logs de los contenedores se envían a **CloudWatch Logs**.
* Se crean alarmas en **CloudWatch Alarms** para:

  * Uso de CPU/Memory alto en las tareas.
  * Estado de salud de las instancias EC2.
  * Fallos en los targets del ALB.

---

## Arquitectura general

```text
Usuarios -> ALB (subred pública) -> Target Groups -> Servicios ECS -> Tareas (contenedores)
                                                   |
                                                   |-> Imágenes desde ECR
                                                   |-> Logs hacia CloudWatch
                                                   |-> Escalado automático (tareas / EC2)
```

---
