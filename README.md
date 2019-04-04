# kubernetes
Es un sistema de gestión y orquestación de conenedores. Ejecuta y gestiona aplicaciones
aplicaciones containerizadas sobre un cluster.

# Minikube

Minikube es una herramienta que facilita la ejecución local de Kubernetes. Minikube ejecuta un clúster Kubernetes de un solo nodo dentro de una máquina virtual 

Proporciona una solución sencilla para usuarios que desean probar Kubernetes o desarrollar soluciones sobre él.

Los despliegues (deployments) son una colección de recursos y referencias. Describe:
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

## 6. Kubectl
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

### PODs en ejecución

Comprobar cuantos pods tengo en marcha:
```
kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
hello-minikube-5857d96c67-nt4x7      1/1     Running   1          6d19h
tomcat-deployment-5c4b9b9c99-zp7t5   1/1     Running   0          13m
```
En este caso vemos que tenemos 2 pods.

Vamos a ver más información de un pod concreto si le pasamos el nombre:
```
kubectl describe pods tomcat-deployment-5c4b9b9c99-zp7t5
Name:               tomcat-deployment-5c4b9b9c99-zp7t5
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               minikube/10.0.2.15
Start Time:         Wed, 03 Apr 2019 12:13:19 +0200
Labels:             app=tomcat
                    pod-template-hash=5c4b9b9c99
Annotations:        <none>
Status:             Running
IP:                 172.17.0.5
Controlled By:      ReplicaSet/tomcat-deployment-5c4b9b9c99
Containers:
  tomcat:
    Container ID:   docker://d2a6f90d7b42598dd8971862bcbfb3f56d43acf7f55f1a5f4e2b4ab737050ce6
    Image:          tomcat:9.0
    Image ID:       docker-pullable://tomcat@sha256:3f4b2548996ffd6d7984f76557fc4db75f92e155340191f7a7325b1f751d10ac
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 03 Apr 2019 12:14:39 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-6zhlc (ro)
```
### Kubectl exec

Podemos ejecutar un comando dentro de un pod pasándoselo como parámetro.
En este caso abrimos un shell dentro del pod, que nos puede servir para debuguear.

```
kubectl exec -it tomcat-deployment-5c4b9b9c99-zp7t5 bash
root@tomcat-deployment-5c4b9b9c99-zp7t5:/usr/local/tomcat# ls
BUILDING.txt     LICENSE  README.md      RUNNING.txt  conf     lib   native-jni-lib  webapps
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin          include  logs  temp            work
root@tomcat-deployment-5c4b9b9c99-zp7t5:/usr/local/tomcat# whoami
root
root@tomcat-deployment-5c4b9b9c99-zp7t5:/usr/local/tomcat# uname -r
4.15.0
```

### Kubectl run

Podemos utilizarlo para desplegar PODs directamente sin tener que crear un archivo de despliegue.

```
kubectl run hazelcast --image=hazelcast --port=5701
deployment.apps/hazelcast created
```
## 7. Arquitectura de kubernetes

### Cluster
Un cluster es un conjunto de nodos. cada uno de estos nodos puede ser:
* Una máquina real física
* Una máquina virtual
### Nodos worker
Los nodos worker son máquinas que ejecutan aplicaciones dentro de contenedores. 
Ejecutan, monitorizan y proveen de servicios a las aplicaciones a través de diferentes componentes:
* Docker (u otro sistema) ejecuta los contenedores
* Los kubeletes se comunican con la API del servidor y gestionan los contenedores en su propio nodo
* Un proxy de red balancea el tráfico entre los diferente componentes
### Pods
Se trata de un grupo de uno o más contenedores que comparten almacenamiento y red, y una manera común de utilizarse. Los contenedores dentro de un pod se colocan, programan y ejecutan conjuntamente en un mismo contexto.

Los contenedores dentro de un pod comparten IP y puertos, y se pueden comunicar a través de localhost. Asímismo, los contenedores dentro de un mismo pod suelen compartir volúmenes.

## 8. Escalado

Nos puede interesar tener más de una instancia de un POD, pero por necesidades de demanda o por disponibilidad, nos
puede interesar tener réplicas de un POD.

Vamos a ver cómo utilizar esto en aplicaciones stateless. La forma más habitual es especificar el parámetro
replica en nuestro despliegue.

Podemos hacer varias cosas, entre ellas podemos cambiar el despliegue para que nos cree 4 réplicas del POD.

```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  selector:
    matchLabels:
      app: tomcat
  replicas: 4
 ...
 ...
```
Otra opción es no modificar el despliegue (dejarlo a replicas=1 y ejecutar el comando scale para decirle 
cuantas réplicas quiero

```
kubectl scale --replicas=4 deployment/tomcat-deployment
deployment.extensions/tomcat-deployment scaled
```
Veremos que ahora los despliegues nos mostrarán el número de réplicas:
```
kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
hazelcast           0/1     1            0           29m
hello-minikube      1/1     1            1           6d20h
tomcat-deployment   4/4     4            4           53m
```
Ahora tenemos un problema de red, puesto que cuando únicamente teníamos un POD, mapeábamos un puerto suyo
con uno accesible externamente. Para que los 4 puedan escuchar en el mismo puerto y se repartan las peticiones
de servicio entre todos, necesitaremos un balanceador de carga.

Utilizaremos un load balancer para exponer un único puerto externo y balancear la carga a los diferentes pods.

```
kubectl expose deployment tomcat-deployment --type=LoadBalancer --port=8080 --target-port 8080 --name=tomcat-load-balancer
service/tomcat-load-balancer exposed
```
Vamos a ver cómo ha quedado la cosa, mirando la descripción del balanceador que hemos creado:
```
kubectl describe service tomcat-load-balancer
```
La respuesta nos indica que hemos mapeado el puerto 8080 al 8080 de cada uno de los 4 PODs que tenemos en ejecución. 
Se encargará de redirigir las peticiones a uno de ellos, repartiendo la carga.

```
Name:                     tomcat-load-balancer
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=tomcat
Type:                     LoadBalancer
IP:                       10.96.43.114
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30854/TCP
Endpoints:                172.17.0.5:8080,172.17.0.7:8080,172.17.0.8:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
# 9. Despliegues

Cosas que puedo hacer con despliegues:
* Crear uno nuevo
* Actualizar un despliegue existente
* Aplicar actualizaciones a pods que se estén ejecutando en el cluster
* Volver a una versión anterior (rollback)
* Pausar y arrancar un despliegue

Gracias a kubectl puedo trabajar e interactuar con los diferentes despliegues y:
* Listar despliegues
* Ver el estado de los rollouts
* Establecer la imagen para un despliegue
* Ver el histórico de rollouts

# 10. Etiquetas

Puedo etiquetar diferentes nodos (máquinas), de tal forma que en los despliegues puedo especificar
en qué tipo de nodos quiero que se despliegue un pod en concreto. Para ello:
* Etiqueto el nodo o nodos que quiero agrupar
* Modifico el YAML del despliegue agregando una opción NodeSelector, especificando la etiqueta sobre la 
que quiero hacer el despliegue.
* De este modo, el pod sólo se va a ejecutar sobre los nodos que estén etiquetados de este modo.
