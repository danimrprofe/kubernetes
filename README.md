# kubernetes
Es un sistema de gestión y orquestación de conenedores. Ejecuta y gestiona aplicaciones
aplicaciones containerizadas sobre un cluster.
# Cluster
Un cluster es un conjunto de nodos. cada uno de estos nodos puede ser:
* Una máquina real física
* Una máquina virtual
# Nodos worker
Los nodos worker son máquinas que ejecutan aplicaciones dentro de contenedores. 
Ejecutan, monitorizan y proveen de servicios a las aplicaciones a través de diferentes componentes:
* Docker (u otro sistema) ejecuta los contenedores
* Los kubeletes se comunican con la API del servidor y gestionan los contenedores en su propio nodo
* Un proxy de red balancea el tráfico entre los diferente componentes
# Pods
Se trata de un grupo de uno o más contenedores que comparten almacenamiento y red, y una manera común de utilizarse. Los contenedores dentro de un pod se colocan, programan y ejecutan conjuntamente en un mismo contexto.

Los contenedores dentro de un pod comparten IP y puertos, y se pueden comunicar a través de localhost. Asímismo, los contenedores dentro de un mismo pod suelen compartir volúmenes.

# Minikube

Los deployments son una colección de recursos y referencias. Describe:
* Qué contenedores nos interesan y los describe
* Indica cómo los contenedores se relacionan
* Qué se requiere para qué funcionen correctamente
* Qué hacer si dejan de funcionar correctamente

Se suelen describir en un archivo YAML. Nos descargaremos un proyecto ya hecho con git.

Un ejemplo es el siguiente:
Solo tendremos una réplica con 1 contenedor, que será la imagen oficial de tomcat 9.0 y 
que escuchará en el puerto 8080. La imagen del contenedor la descargará de Docker Hub.

```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  selector:
    matchLabels:
      app: tomcat
  replicas: 1
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:9.0
        ports:
        - containerPort: 8080
```     

# Kubectl
Kubectl nos permite interaccionar con cualquier cluster de Kubernetes, desde un shell.

Aplicar el deployment
```
kubectl apply -f ./deployment.yaml
contestación: deployment.apps/tomcat-deployment created
```
Exponemos el despliegue para que nos asocie una IP y puerto
```
kubectl expose deployment tomcat-deployment --type=NodePort
Contestación: service/tomcat-deployment exposed
```
Para saber la IP y el puerto en el que nos ha expuesto el servicio:
```
minikube service tomcat-deployment --url
Contestación: http://192.168.99.100:30836
```
Si accedemos a esta URL veremos que está el servidor Tomcat escuchando

## Otros comandos

Comprobar cuantos pods tengo en marcha:
```
kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
hello-minikube-5857d96c67-nt4x7      1/1     Running   1          6d19h
tomcat-deployment-5c4b9b9c99-zp7t5   1/1     Running   0          13m
```
