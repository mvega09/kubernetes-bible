# Guía Completa de Comandos Kubernetes y Minikube

> **Referencia**: https://pabpereza.dev/docs/cursos/kubernetes

## Tabla de Contenidos
- [Gestión de Minikube](#gestión-de-minikube)
- [Gestión de Clusters](#gestión-de-clusters)
- [Gestión de Pods](#gestión-de-pods)
- [Deployments](#deployments)
- [Services](#services)
- [ConfigMaps y Secrets](#configmaps-y-secrets)
- [Namespaces](#namespaces)
- [Aplicar Manifiestos YAML](#aplicar-manifiestos-yaml)
- [Troubleshooting y Debugging](#troubleshooting-y-debugging)
- [Ejecución de Comandos en Pods](#ejecución-de-comandos-en-pods)
- [StatefulSets](#statefulsets)
- [DaemonSets](#daemonsets)
- [ReplicaSets](#replicasets)
- [Ingress](#ingress)
- [Persistent Volumes y Claims](#persistent-volumes-y-claims)
- [Jobs y CronJobs](#jobs-y-cronjobs)
- [RBAC (Control de Acceso)](#rbac-control-de-acceso)
- [Recursos y Límites](#recursos-y-límites)
- [Network Policies](#network-policies)
- [Horizontal Pod Autoscaler (HPA)](#horizontal-pod-autoscaler-hpa)
- [Comandos de Información y Utilidades](#comandos-de-información-y-utilidades)
- [Comandos Dry-Run y Templates](#comandos-dry-run-y-templates)
- [Helm (Gestor de Paquetes)](#helm-gestor-de-paquetes)
- [Kubectl Plugins y Kustomize](#kubectl-plugins-y-kustomize)
- [K9s - Interfaz de Terminal](#k9s---interfaz-de-terminal)
- [Cleanup y Mantenimiento](#cleanup-y-mantenimiento)
- [Tips y Shortcuts](#tips-y-shortcuts)

---

## Gestión de Minikube

Comandos para iniciar, gestionar y configurar tu cluster local de Kubernetes con Minikube.

```bash
minikube start                          # Inicia el cluster local de Kubernetes
minikube dashboard                      # Abre el dashboard web de Kubernetes para visualización gráfica
minikube addons list                    # Lista todos los addons disponibles en Minikube
minikube addons list -p multi-cluster   # Lista todos los addons disponibles en el cluster llamado multi-cluster
minikube addons enable metrics-server   # Habilita el servidor de métricas para monitoreo de recursos
minikube node add                       # Agregar nodo al cluster
minikube node delete                    # Eliminar nodo al cluster
minikube node stop <name_nodo>          # Detener un nodo especifico
minikube start --network-plugin=cni --cni=calico --container-runtime=containerd #Iniciar un cluster local con cni de calico

minikube start -p multi-cluster --nodes 3 --driver=docker       #Creaun cluster con 3 nodos en minikube
kubectl label node multi-cluster-m02 kubernetes.io/role=worker  #Modificar el rol de none a worker en el nodo de minikube
kubectl label nodes minikube-m02 node-type=superfast            #Modificar el tipo de nodo para afinidad/antiafinidad
kubectl label nodes --overwrite minikube node-type=slow         #Modificar el tipo de nodo para afinidad/antiafinidad
kubectl label nodes --overwrite minikube-m02 node-type=fast     #MModificar el tipo de nodo para afinidad/antiafinidad
kubectl label nodes --overwrite minikube-m03 node-type=superfast  #Modificar el tipo de nodo para afinidad/antiafinidad         
minikube start -p multi-cluster                                 #Iniciar cluster
minikube stop -p multi-cluster                                  #Detener cluster
minikube delete -p multi-cluster                               #Eliminar cluster

#Iniciar un cluster con cni calico
minikube start -p multi-cluster \
--driver=docker \
--nodes=3 \
--cni=calico \
--cpus=2 \
--memory=2048 \
--kubernetes-version=v1.33.0 \
--container-runtime=containerd
```

---

## Gestión de Clusters

Comandos para administrar y obtener información sobre tu cluster de Kubernetes.

```bash
kubectl cluster-info                    # Muestra información del cluster (API server, DNS, etc.)
kubectl version                         # Muestra la versión de kubectl y del cluster
kubectl config view                     # Visualiza la configuración actual de kubectl
kubectl config current-context          # Verificar el nombre del contexto que estás usando actualmente
kubectl config get-contexts             # Lista todos los contextos disponibles
kubectl config use-context <contexto>   # Cambia al contexto especificado
kubectl get nodes                       # Lista todos los nodos del cluster
kubectl describe node <nombre>          # Muestra detalles completos de un nodo específico
kubectl cordon <nodo>                   # Marca un nodo como no programable (no schedule)
kubectl uncordon <nodo>                 # Marca un nodo como programable nuevamente
kubectl drain <nodo>                    # Drena todos los pods de un nodo para mantenimiento
kubectl get componentstatuses           # Muestra el estado de los componentes del plano de control
```

---

## Gestión de Pods

Comandos para crear, listar, monitorear y gestionar pods en Kubernetes.

```bash
# Listado y visualización
kubectl get pods                        # Lista todos los pods en el namespace actual
kubectl get pods -A                     # Lista todos los pods en todos los namespaces
kubectl get pods -n kube-system         # Lista pods en el namespace kube-system
kubectl get pods -o wide                # Muestra información extendida (IP, nodo, etc.)
kubectl get pods --watch                # Observa cambios en los pods en tiempo real
kubectl get pods -l app=nginx           # Filtra pods por label específico
kubectl get pods --namespace default --output=custom-columns="NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName" #Ver los Pods y nombres de Nodos.

# Creación de pods
kubectl run nginx --image=nginx         # Crea un pod simple con la imagen de nginx

# Información detallada
kubectl describe pod <nombre>           # Muestra detalles completos del pod (eventos, estado, recursos)
kubectl describe pod coredns-66bc5c9577-mlmsq -n kube-system  # Describe un pod en namespace específico
kubectl describe pod -n mysql mysql-stateful-0|grep Image # Verificar que imagen de docker esta ejecutando el pod en este caso mysql-stateful-0 en el ns mysql

# Logs
kubectl logs <pod>                      # Muestra los logs del pod
kubectl logs <pod> -f                   # Sigue los logs en tiempo real (follow)
kubectl logs <pod> -c <contenedor>      # Logs de un contenedor específico en el pod
kubectl logs -f <pod> -c app            # Muestra en tiempo real los logs del contenedor 'app'
kubectl logs <pod> --previous           # Logs del contenedor anterior (útil si crasheó)
kubectl logs <pod> --tail=100           # Muestra las últimas 100 líneas de logs
kubectl logs <pod> --since=1h           # Logs de la última hora
kubectl logs -f -l app=nginx            # Sigue logs de todos los pods con el label app=nginx

# Acceso y ejecución
kubectl exec -it <pod> -- /bin/bash     # Accede de forma interactiva al pod con bash
kubectl exec -it <pod> -- /bin/sh       # Accede al pod con shell sh
kubectl exec -it <pod> -c <contenedor> -- /bin/bash  # Accede a contenedor específico
kubectl attach <pod> -it                # Se adjunta a un pod en ejecución

# Port forwarding
kubectl port-forward pod/nginx 8080:80  # Redirige puerto local 8080 al puerto 80 del pod
kubectl port-forward pod/nginx :80      # Asigna puerto aleatorio (30000-32000) al puerto 80 del pod

# Recursos y eliminación
kubectl top pod                         # Muestra el uso de CPU y memoria de los pods
kubectl delete pod <nombre>             # Elimina el pod especificado
kubectl delete pod <nombre> --force --grace-period=0  # Fuerza la eliminación inmediata

# Copiar archivos
kubectl cp <pod>:/path/file ./file      # Copia archivo desde el pod al local
kubectl cp ./file <pod>:/path/file      # Copia archivo local hacia el pod

#Ejemplo tomado del capitulo 21 de k8s bible
kubectl cp troubles/test.txt blog-675df44d5-gkrt2:/app/test.txt -n trouble-ns # copiar un archivo desde tu máquina local a un contenedor:
kubectl cp blog-675df44d5-gkrt2:/app/app.py /tmp/app.py -n trouble-ns #copiar un archivo desde un contenedor a tu máquina local

# Recuperar manifesto
kubectl get pods/nginx-pod -o yaml > nginx-pod.yaml  # Exporta el manifiesto YAML del pod
```

---

## Deployments

Comandos para gestionar deployments, que controlan el despliegue y escalado de aplicaciones.

```bash
# Listado y creación
kubectl get deployments                 # Lista todos los deployments
kubectl get deploy                      # Alias corto de deployments
kubectl create deployment <nombre> --image=<imagen>  # Crea un deployment con la imagen especificada

# Información
kubectl describe deployment <nombre>    # Muestra detalles del deployment
kubectl describe deployment <nombre> -o wide     # Confirmar que estás en la versión correcta


# Escalado
kubectl scale deployment <nombre> --replicas=3       # Escala el deployment a 3 réplicas
kubectl scale deployment nginx-deployment --replicas=5  # Ejemplo: escala nginx a 5 réplicas
kubectl autoscale deployment <nombre> --min=2 --max=10 --cpu-percent=80  # Autoescalado basado en CPU

# Actualización de imagen
kubectl set image deployment/<nombre> <contenedor>=<nueva-imagen>  # Actualiza la imagen del contenedor
kubectl set image deployment nginx-deployment nginx=nginx:1.29-alpine  # Ejemplo: actualiza a nginx alpine

# Gestión de rollouts
kubectl rollout status deployment/<nombre>                  # Muestra el estado del rollout actual
kubectl rollout history deployment/<nombre>                 # Historial de todos los rollouts
kubectl rollout history deployment/<nombre> --revision=1    # 
kubectl rollout undo deployment/<nombre>                    # Hace rollback al deployment anterior
kubectl rollout undo deployment nginx-deployment            # Ejemplo: vuelve a la versión anterior de nginx
kubectl rollout undo deployment/<nombre> --to-revision=2    # Rollback a una revisión específica
kubectl rollout pause deployment/<nombre>                   # Pausa el rollout en progreso
kubectl rollout resume deployment/<nombre>                  # Reanuda un rollout pausado

# Edición y eliminación
kubectl edit deployment <nombre>        # Abre el deployment en editor para modificación
kubectl delete deployment <nombre>      # Elimina el deployment y sus pods
```

---

## Services

Comandos para exponer y gestionar servicios que permiten la comunicación entre pods.

```bash
# Listado
kubectl get services                    # Lista todos los servicios
kubectl get svc                         # Alias corto de services

# Información
kubectl describe service <nombre>       # Muestra detalles del servicio
kubectl get endpoints                   # Muestra los endpoints de los servicios

# Creación y exposición
kubectl expose pod nginx --port 80 --target-port 80 --type NodePort  # Expone un pod como servicio
kubectl expose deployment <nombre> --port=80 --type=LoadBalancer     # Expone deployment como LoadBalancer
kubectl expose deployment <nombre> --port=80 --target-port=8080 --type=NodePort  # Expone con mapeo de puertos

# Port forwarding
kubectl port-forward svc/nginx 8080:80  # Redirige puerto local al servicio

# Eliminación
kubectl delete service <nombre>         # Elimina el servicio especificado
```

---

## ConfigMaps y Secrets

Comandos para gestionar configuraciones y datos sensibles en Kubernetes.

### ConfigMaps
Almacenan configuración en formato key-value o archivos.

```bash
kubectl get configmaps                  # Lista todos los ConfigMaps
kubectl get cm                          # Alias corto
kubectl get cm -n demo                  # Alias corto mas el namespace
kubectl create configmap <nombre> --from-file=<archivo>      # Crea ConfigMap desde archivo
kubectl create configmap <nombre> --from-literal=key=value   # Crea ConfigMap con valores literales
kubectl describe configmap <nombre>     # Muestra detalles del ConfigMap
kubectl get configmap <nombre> -o yaml  # Muestra el ConfigMap en formato YAML
kubectl delete configmap <nombre>       # Elimina el ConfigMap
```

### Secrets
Almacenan información sensible codificada en base64.

```bash
kubectl get secrets                     # Lista todos los Secrets
kubectl create secret generic <nombre> --from-literal=key=value  # Crea Secret con valor literal
kubectl create secret generic postgre-pass --from-literal=password=123456789  # Ejemplo: secret de password
kubectl create secret generic <nombre> --from-file=<archivo>     # Crea Secret desde archivo

# Docker registry secrets
kubectl create secret docker-registry <nombre> --docker-server=<servidor> --docker-username=<user> --docker-password=<pass>
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=mvega0911 --docker-password=dckr_pat_xxx --docker-email=mvega0911
# Este comando crea credenciales para extraer imágenes de registros privados

# Visualización
kubectl get secrets postgre-pass -o yaml  # Muestra el Secret en formato YAML (password en base64)
kubectl describe secret <nombre>        # Describe el Secret (sin mostrar valores)
kubectl get secret <nombre> -o yaml     # Obtiene el Secret completo en YAML

# Decodificar secrets
echo "MTIzNDU2Nzg5" | base64 -d         # Decodifica el valor base64 del secret

# Verificar credenciales docker
kubectl describe secret regcred         # Describe el secret de docker registry
cat ~/.docker/config.json               # Muestra la configuración de autenticación docker

# Eliminación
kubectl delete secret <nombre>          # Elimina el Secret
```

---

## Namespaces

Comandos para gestionar namespaces, que permiten aislar recursos en el cluster.

```bash
kubectl get namespaces                  # Lista todos los namespaces del cluster
kubectl get ns                          # Alias corto de namespaces
kubectl create namespace <nombre>       # Crea un nuevo namespace
kubectl delete namespace <nombre>       # Elimina un namespace y todos sus recursos
kubectl get pods -n <namespace>         # Lista pods en un namespace específico
kubectl get all -n <namespace>          # Lista todos los recursos en un namespace
kubectl describe namespace <nombre>     # Muestra detalles del namespace

# Cambiar namespace por defecto
kubectl config set-context --current --namespace=<nombre>  # Establece namespace por defecto
kubectl config get-contexts             # Verifica el namespace que estás usando actualmente
kubectl config view --minify --output 'jsonpath={..namespace}' #Verificar si algún namespace está configurado en el contexto
```

---

## Aplicar Manifiestos YAML

Comandos para aplicar, crear y gestionar recursos usando archivos de configuración YAML.

```bash
kubectl apply -f <archivo.yaml>         # Aplica o actualiza recursos desde archivo YAML
kubectl apply -f nginx.yaml             # Ejemplo: aplica configuración de nginx
kubectl apply -f <directorio>/          # Aplica todos los archivos YAML en el directorio
kubectl apply -f <url>                  # Aplica configuración desde una URL
kubectl delete -f <archivo.yaml>        # Elimina los recursos definidos en el archivo
kubectl diff -f <archivo.yaml>          # Muestra diferencias antes de aplicar
kubectl replace -f <archivo.yaml>       # Reemplaza un recurso existente
kubectl create -f <archivo.yaml>        # Crea recursos (error si ya existe)
kubectl get -f <archivo.yaml>           # Obtiene información de los recursos del archivo
```

---

## Troubleshooting y Debugging

Comandos esenciales para diagnosticar y resolver problemas en el cluster.

```bash
# Eventos
kubectl get events                      # Lista eventos recientes del cluster
kubectl get events -n <namespace>       # Eventos en un namespace específico
kubectl get events --sort-by=.metadata.creationTimestamp  # Ordena eventos por tiempo
kubectl get events --watch              # Observa eventos en tiempo real

# Información detallada
kubectl describe <recurso> <nombre>     # Describe cualquier tipo de recurso

# Logs avanzados
kubectl logs <pod> --previous           # Logs del contenedor anterior (si crasheó)
kubectl logs <pod> --tail=100           # Últimas 100 líneas de logs
kubectl logs <pod> --since=1h           # Logs de la última hora
#Ejemplos con pods multicontenedores
kubectl logs --since=2h pods/multi-container-pod -c nginx-container #Recuperar los logs escritos en las últimas 2 horas
kubectl logs --tail=30 pods/multi-container-pod --container nginx-container #Usar la opción --tail para recuperar las líneas más recientes de la salida de un log

# Port forwarding
kubectl port-forward <pod> 8080:80      # Redirección local:pod
kubectl port-forward service/<svc> 8080:80  # Port-forward a un servicio

# Debugging con contenedor temporal
kubectl debug <pod> -it --image=busybox # Crea contenedor de debug en el pod

# Copiar archivos
kubectl cp <pod>:/path/file ./file      # Copia archivo desde pod
kubectl cp ./file <pod>:/path/file      # Copia archivo hacia pod

# Búsqueda de pods fallidos
kubectl get pods --field-selector status.phase=Failed  # Lista pods en estado fallido
```

---

## Ejecución de Comandos en Pods

Comandos para ejecutar acciones dentro de los pods sin necesidad de archivos externos.

```bash
# Acceso interactivo
kubectl exec -it <pod-name> -- sh       # Abre shell sh interactivo en el pod
kubectl exec -it <pod-name> -- bash     # Abre shell bash interactivo
kubectl exec -it <pod-name> -- /bin/sh  # Especifica ruta completa del shell
kubectl exec -it pods/<pod-name> -- sh  # Notación completa con tipo de recurso

# Acceso a pods con múltiples contenedores
kubectl exec -it <pod-name> -c <container-name> -- sh    # Accede a contenedor específico
kubectl exec -it <pod-name> -c <container-name> -- bash  # Bash en contenedor específico
kubectl exec -it multi-container-pod -c nginx-container -- /bin/bash #Ejemplo de uso de acceso a pods multiples contenedores
kubectl exec -it multi-container-pod -c nginx-container -- ls  #Listar los archivos que contiene el contenedor sin acceder a el

# En namespace específico
kubectl exec -it <pod-name> -n <namespace> -- sh         # Ejecuta en pod de otro namespace

# Comandos únicos (sin entrar al pod)
kubectl exec <pod-name> -- ls /app      # Lista archivos en directorio del pod
kubectl exec <pod-name> -- cat /etc/hosts  # Muestra contenido de archivo
kubectl exec <pod-name> -- env          # Muestra variables de entorno

# Comandos complejos con pipes
kubectl exec <pod-name> -- sh -c "ps aux | grep nginx"   # Busca procesos nginx
kubectl exec <pod-name> -- sh -c "df -h"                 # Muestra uso de disco
kubectl exec <pod-name> -- sh -c "ls -la /var/log"       # Lista detallada de logs

# Verificar conectividad de red
kubectl exec <pod-name> -- ping google.com               # Prueba conectividad externa
kubectl exec <pod-name> -- curl http://service-name      # Prueba servicio interno
kubectl exec <pod-name> -- wget -O- http://localhost:8080  # Descarga respuesta HTTP
kubectl exec <pod-name> -- nslookup kubernetes.default   # Resuelve DNS interno

kubectl exec -it k8sutils -- nslookup nginx-service-example.default.svc.cluster.local # Resuelve DNS del servicio nginx-service-example (El DNS se forma service_name.namespace.svc.cluster.local)
kubectl exec -it k8sutils -- curl nginx-service-example.default.svc.cluster.local # Prueba del servicio interno 
kubectl exec -it k8sutils -- wget -O- http://nginx-service-example:80 # Descarga respuesta HTTP del servicio
kubectl exec -it k8sutils -- wget -O index.html  http://nginx-service-example:80 #Guarda archivos del pod en este archivo index.html

kubectl cp <namespace>/<pod>:/ruta/en/pod /ruta/local
kubectl cp default/k8sutils:/index.html ./index.html   # Este comando copia archivos entre tu máquina local y un Pod de Kubernetes


# Monitoreo de procesos
kubectl exec <pod-name> -- ps aux                        # Lista todos los procesos
kubectl exec <pod-name> -- top -n 1                      # Muestra uso de recursos una vez

# Diagnóstico de red
kubectl exec <pod-name> -- netstat -tulpn                # Puertos abiertos y conexiones
kubectl exec <pod-name> -- ip addr                       # Configuración de red del pod
kubectl exec <pod-name> -- route -n                      # Tabla de rutas
kubectl exec <pod-name> -- cat /etc/resolv.conf          # Configuración DNS

# Verificar archivos y configuración
kubectl exec <pod-name> -- cat /app/config.json          # Lee archivo de configuración
kubectl exec <pod-name> -- tail -f /var/log/app.log      # Sigue logs en tiempo real
kubectl exec <pod-name> -- which python                  # Verifica existencia de comando

# Ejemplos de troubleshooting con MySQL
kubectl exec -it mysql-6fdc66985f-tmmpj -- /bin/sh      # Accede al pod de MySQL
# Dentro del pod ejecutar: mysql -u root -p

# Cliente temporal de MySQL
kubectl run mysql-client --image=mysql/mysql-server -it --rm --restart=Never -- /bin/bash
# Conectar desde el cliente: mysql -h <nombre_svc> -u appuser -p
```

---

## StatefulSets

Comandos para gestionar StatefulSets, usados para aplicaciones que requieren identidad persistente.

```bash
kubectl get statefulsets                # Lista todos los StatefulSets
kubectl get sts                         # Alias corto
kubectl describe statefulset <nombre>   # Muestra detalles del StatefulSet
kubectl scale statefulset <nombre> --replicas=3  # Escala el StatefulSet
kubectl delete statefulset <nombre>     # Elimina el StatefulSet
kubectl rollout status statefulset/<nombre>      # Estado del rollout
```

---

## DaemonSets

Comandos para gestionar DaemonSets, que ejecutan un pod en cada nodo del cluster.

```bash
kubectl get daemonsets                  # Lista todos los DaemonSets
kubectl get ds                          # Alias corto
kubectl describe daemonset <nombre>     # Muestra detalles del DaemonSet
kubectl describe ds                     # Alias corto
kubectl delete daemonset <nombre>       # Elimina el DaemonSet
kubectl rollout status daemonset/<nombre>        # Estado del rollout
```

---

## ReplicaSets

Comandos para gestionar ReplicaSets, que mantienen un número específico de réplicas de pods.

```bash
kubectl get replicasets                 # Lista todos los ReplicaSets
kubectl get rs                          # Alias corto
kubectl describe replicaset <nombre>    # Muestra detalles del ReplicaSet
kubectl scale replicaset <nombre> --replicas=3   # Escala el ReplicaSet
kubectl delete replicaset <nombre>      # Elimina el ReplicaSet
kubectl delete rs/nginx-replicaset-livenessprobe-example --cascade=orphan # Eliminar el objeto ReplicaSet y dejar los Pods sin afectar. (Después de este comando, si inspeccionas qué Pods están en el clúster, aún verás todos los Pods que eran propiedad del objeto ReplicaSet nginx-replicaset-livenessprobe-example. Estos Pods ahora pueden, por ejemplo, ser adquiridos por otro objeto ReplicaSet que tenga un selector de etiquetas coincidente.)
```

---

## Ingress

Comandos para gestionar Ingress, que controla el acceso HTTP/HTTPS externo a los servicios.

```bash
kubectl get ingress                     # Lista todos los Ingress
kubectl get ing                         # Alias corto
kubectl describe ingress <nombre>       # Muestra detalles del Ingress
kubectl delete ingress <nombre>         # Elimina el Ingress
kubectl get ingressclass                # Lista las clases de Ingress disponibles
```

---

## Persistent Volumes y Claims

Comandos para gestionar almacenamiento persistente en Kubernetes.

### Persistent Volumes (PV)
```bash
kubectl get persistentvolumes           # Lista todos los PersistentVolumes
kubectl get pv                          # Alias corto
kubectl describe pv <nombre>            # Muestra detalles del PV
kubectl delete pv <nombre>              # Elimina el PersistentVolume
```

### Persistent Volume Claims (PVC)
```bash
kubectl get persistentvolumeclaims      # Lista todos los PersistentVolumeClaims
kubectl get pvc                         # Alias corto
kubectl describe pvc <nombre>           # Muestra detalles del PVC
kubectl delete pvc <nombre>             # Elimina el PersistentVolumeClaim
```

### Storage Classes
```bash
kubectl get storageclasses              # Lista todas las clases de almacenamiento
kubectl get sc                          # Alias corto
kubectl describe sc <nombre>            # Muestra detalles de la StorageClass
```

---

## Jobs y CronJobs

Comandos para gestionar tareas únicas y programadas en Kubernetes.

### Jobs
Ejecutan tareas que deben completarse una vez.

```bash
kubectl get jobs                        # Lista todos los Jobs
kubectl create job <nombre> --image=<imagen>  # Crea un nuevo Job
kubectl create job <nombre> --from=cronjob/<cronjob-nombre>  # Crea Job desde CronJob
kubectl describe job <nombre>           # Muestra detalles del Job
kubectl delete job <nombre>             # Elimina el Job
kubectl logs job/<nombre>               # Ver logs del Job
kubectl delete jobs hello-world-job --cascade=false #eliminar los Jobs pero no los Pods que creó
```

### CronJobs
Ejecutan tareas en horarios programados.

```bash
kubectl get cronjobs                    # Lista todos los CronJobs
kubectl get cj                          # Alias corto
kubectl create cronjob <nombre> --image=<imagen> --schedule="*/5 * * * *"  # Crea CronJob
kubectl describe cronjob <nombre>       # Muestra detalles del CronJob
kubectl delete cronjob <nombre>         # Elimina el CronJob
kubectl suspend cronjob <nombre>        # Suspende el CronJob (K8s 1.21+)
```

---

## RBAC (Control de Acceso)

Comandos para gestionar permisos y control de acceso basado en roles.

### Roles y RoleBindings (Namespace scope)
```bash
kubectl get roles                       # Lista roles en el namespace actual
kubectl get rolebindings                # Lista rolebindings
kubectl describe role <nombre>          # Detalles del role
kubectl describe rolebinding <nombre>   # Detalles del rolebinding
```

### ClusterRoles y ClusterRoleBindings (Cluster scope)
```bash
kubectl get clusterroles                # Lista clusterroles
kubectl get clusterrolebindings         # Lista clusterrolebindings
kubectl describe clusterrole <nombre>   # Detalles del clusterrole
kubectl describe clusterrolebinding <nombre>  # Detalles del clusterrolebinding
```

### ServiceAccounts
```bash
kubectl get serviceaccounts             # Lista service accounts
kubectl get sa                          # Alias corto
kubectl describe sa <nombre>            # Detalles de la service account
```

### Verificación de permisos
```bash
kubectl auth can-i <verbo> <recurso>    # Verifica si puedes realizar una acción
kubectl auth can-i create deployments   # Ejemplo: ¿puedo crear deployments?
kubectl auth can-i get pods --as <usuario>  # Verifica permisos de otro usuario
kubectl auth can-i '*' '*'              # Verifica si tienes permisos de admin
```

---

## Recursos y Límites

Comandos para monitorear el uso de recursos (CPU, memoria) en el cluster.

```bash
kubectl top nodes                       # Muestra uso de recursos en todos los nodos
kubectl top pods                        # Muestra uso de recursos de los pods
kubectl top pods -n <namespace>         # En namespace específico
kubectl top pods -n kube-system         # Ejemplo: monitorea pods del sistema
kubectl top pods --containers           # Muestra uso por contenedor individual
kubectl describe limitrange             # Ver límites de recursos configurados
kubectl get resourcequota               # Lista cuotas de recursos
kubectl get resourcequota -n custom-ns  # Lista cuotas de recursos de un namespace
kubectl get quota -n custom-ns          # Lista la quota dentro de una ns en este caso custom-ns
kubectl describe resourcequota <nombre> # Detalles de la cuota de recursos
```

---

## Network Policies

Comandos para gestionar políticas de red que controlan el tráfico entre pods.

```bash
kubectl get networkpolicies             # Lista todas las Network Policies
kubectl get netpol                      # Alias corto
kubectl describe networkpolicy <nombre> # Muestra detalles de la política
kubectl delete networkpolicy <nombre>   # Elimina la Network Policy
```

---

## Horizontal Pod Autoscaler (HPA)

Comandos para gestionar el autoescalado horizontal de pods basado en métricas.

```bash
kubectl get hpa                         # Lista todos los HPA configurados
kubectl describe hpa <nombre>           # Muestra detalles del HPA
kubectl autoscale deployment <nombre> --min=2 --max=10 --cpu-percent=80  # Crea HPA
kubectl delete hpa <nombre>             # Elimina el HPA
```

---

## Comandos de Información y Utilidades

Comandos útiles para obtener información general del cluster y recursos.

```bash
# Listado general
kubectl get all                         # Lista los recursos principales
kubectl get all -A                      # En todos los namespaces
kubectl get all -n <namespace>          # En namespace específico

# API y recursos
kubectl api-resources                   # Lista todos los tipos de recursos disponibles
kubectl api-versions                    # Lista todas las versiones de API
kubectl explain <recurso>               # Documentación de un recurso
kubectl explain pod.spec.containers     # Documentación de campos específicos

# Versión
kubectl version --short                 # Versión resumida de kubectl y cluster

# Formatos de salida
kubectl get <recurso> -o wide           # Salida extendida con más columnas
kubectl get <recurso> -o yaml           # Salida en formato YAML
kubectl get <recurso> -o json           # Salida en formato JSON
kubectl get <recurso> -o jsonpath='{.items[*].metadata.name}'  # Extrae campos con JSONPath

# Labels y anotaciones
kubectl label <recurso> <nombre> key=value      # Agrega o actualiza un label
kubectl annotate <recurso> <nombre> key=value   # Agrega o actualiza una anotación
```

---

## Comandos Dry-Run y Templates

Comandos para generar manifiestos YAML sin aplicarlos, útil para aprender y crear plantillas.

```bash
# Generar YAML sin aplicar (dry-run)
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl create service clusterip my-svc --tcp=80:80 --dry-run=client -o yaml
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl expose pod nginx --port=80 --dry-run=client -o yaml

# Generar y guardar en archivo
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

---

## Helm (Gestor de Paquetes)

Helm es el gestor de paquetes de Kubernetes que facilita el despliegue de aplicaciones complejas.

### Gestión de Repositorios
```bash
helm repo add <nombre> <url>            # Agrega un repositorio de charts
helm repo update                        # Actualiza la lista de charts de todos los repos
helm repo list                          # Lista todos los repositorios configurados
helm repo remove <nombre>               # Elimina un repositorio
helm search repo <keyword>              # Busca charts en los repositorios
```

### Instalación y Gestión de Releases
```bash
helm list                               # Lista releases instalados en el namespace actual
helm list -A                            # Lista releases en todos los namespaces
helm install <nombre> <chart>           # Instala un chart
helm install <nombre> <chart> -f values.yaml  # Instala con archivo de valores personalizado
helm install <nombre> <chart> --set key=value # Instala con valores inline
helm upgrade <nombre> <chart>           # Actualiza un release existente
helm upgrade --install <nombre> <chart> # Instala o actualiza (idempotente)
helm rollback <nombre> <revision>       # Hace rollback a una revisión anterior
helm uninstall <nombre>                 # Desinstala un release y limpia recursos
helm status <nombre>                    # Muestra el estado actual del release
```

### Información y Debugging
```bash
helm show chart <chart>                 # Muestra información del chart
helm show values <chart>                # Muestra los valores por defecto del chart
helm get values <nombre>                # Obtiene los valores de un release instalado
helm get manifest <nombre>              # Muestra los manifiestos del release
helm history <nombre>                   # Ver historial de revisiones del release
```

### Desarrollo de Charts
```bash
helm create <nombre>                    # Crea una estructura de chart nueva
helm lint <chart>                       # Valida un chart para errores de sintaxis
helm template <nombre> <chart>          # Renderiza templates localmente sin instalar
helm package <chart>                    # Empaqueta el chart en un archivo .tgz
```

---

## Kubectl Plugins y Kustomize

### Kustomize
Herramienta nativa de Kubernetes para personalizar manifiestos YAML.

```bash
kubectl apply -k <directorio>           # Aplica recursos con kustomize
kubectl kustomize <directorio>          # Muestra el output sin aplicar
kubectl delete -k <directorio>          # Elimina recursos kustomizados
```

### Plugins Comunes
Estos requieren instalación adicional.

```bash
kubectl tree <recurso> <nombre>         # Visualiza árbol de recursos relacionados (plugin)
kubectl neat                            # Limpia output YAML removiendo campos automáticos (plugin)
kubectl ctx                             # Cambia contexto rápidamente (kubectx)
kubectl ns                              # Cambia namespace rápidamente (kubens)
```

---

## K9s - Interfaz de Terminal

K9s es una interfaz de terminal interactiva para gestionar clusters de Kubernetes.

### Instalación en WSL Ubuntu 22.04
```bash
# Requiere tener Go instalado
go install github.com/derailed/k9s@latest
```

### Comandos de K9s
```bash
k9s version                             # Muestra la versión instalada
k9s info                                # Información sobre logs, configs, etc.
k9s help                                # Lista todas las opciones CLI disponibles
k9s -n mycoolns                         # Inicia K9s en un namespace específico
k9s --context coolCtx                   # Inicia en un contexto de KubeConfig existente
k9s --readonly                          # Inicia en modo solo lectura (sin modificaciones)
```

---

## Comandos Avanzados de Selección

Comandos para filtrar recursos usando labels y field selectors.

### Por Labels
```bash
kubectl get pods -l app=nginx           # Pods con label app=nginx
kubectl get pods -l 'env in (prod,staging)'  # Múltiples valores posibles
kubectl get pods -l app=nginx,env=prod  # Múltiples labels (AND)
kubectl get pods -l 'app!=nginx'        # Negación (todos excepto nginx)
kubectl get pods --show-labels          # Muestra las etiquetas de los pods
kubectl get pods -A --show-labels       # Ver etiquetas de todos los namespaces
```

### Por Field Selector
```bash
kubectl get pods --field-selector status.phase=Running    # Solo pods en ejecución
kubectl get pods --field-selector metadata.namespace=default  # En namespace default
```

### Combinados
```bash
kubectl get pods -l app=nginx --field-selector status.phase=Running  # Combina ambos filtros
kubectl get pod/multi-container-pod -o jsonpath="{.spec.containers[*].name}" #Este comando obtiene todos los contenedores que existen en un pod a traves de los filtros de jsonpath, seria bueno crearle un alias para su uso.
```

---

## Cleanup y Mantenimiento

Comandos para limpiar y mantener el cluster en buen estado.

```bash
# Eliminación forzada
kubectl delete pod <nombre> --grace-period=0 --force  # Fuerza eliminación inmediata
kubectl delete pods --all -n <namespace>    # Elimina todos los pods del namespace
kubectl delete all --all -n <namespace>     # Elimina todos los recursos del namespace

# Limpieza de pods fallidos
kubectl get pods --field-selector status.phase=Failed  # Lista pods en estado Failed
kubectl delete pods --field-selector status.phase=Failed  # Elimina pods fallidos
```

---

## Tips y Shortcuts

Configuraciones útiles para trabajar más eficientemente con kubectl.

### Aliases Útiles
Agrega estos a tu `~/.bashrc` o `~/.zshrc`:

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kd='kubectl describe'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias klf='kubectl logs -f'
alias kex='kubectl exec -it'
alias ka='kubectl apply -f'
alias kdel='kubectl delete'
```

### Autocompletado

#### Bash
```bash
source <(kubectl completion bash)       # Habilita autocompletado para kubectl
complete -F __start_kubectl k           # Habilita autocompletado para alias 'k'
```

#### Zsh
```bash
source <(kubectl completion zsh)        # Habilita autocompletado para kubectl en zsh
```

---

## Recursos Adicionales

### Documentación Oficial
- **Documentación oficial de Kubernetes**: https://kubernetes.io/docs/
- **Cheat Sheet oficial**: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- **Helm Documentation**: https://helm.sh/docs/
- **Kustomize**: https://kustomize.io/

### Curso de Referencia
- **Curso de Kubernetes - Pablo Pérez**: https://pabpereza.dev/docs/cursos/kubernetes

---

## Notas Importantes

**Reemplaza los siguientes placeholders con valores reales de tu entorno:**
- `<nombre>` - Nombre del recurso
- `<imagen>` - Nombre de la imagen de contenedor
- `<contexto>` - Nombre del contexto de kubectl
- `<namespace>` - Nombre del namespace
- `<pod>` - Nombre del pod
- `<contenedor>` - Nombre del contenedor
- `<archivo.yaml>` - Ruta al archivo YAML
- `<directorio>` - Ruta al directorio
- `<url>` - URL del recurso
- `<usuario>` - Nombre de usuario
- `<nodo>` - Nombre del nodo

---

## Ejemplos Prácticos Comunes

### Ejemplo 1: Despliegue completo de Nginx
```bash
# Crear deployment
kubectl create deployment nginx --image=nginx

# Exponer como servicio
kubectl expose deployment nginx --port=80 --type=NodePort

# Escalar a 3 réplicas
kubectl scale deployment nginx --replicas=3

# Ver estado
kubectl get pods -l app=nginx --watch

# Ver logs
kubectl logs -f -l app=nginx

# Hacer port-forward
kubectl port-forward svc/nginx 8080:80
```

### Ejemplo 2: Actualizar imagen con rollback
```bash
# Actualizar imagen
kubectl set image deployment/nginx-deployment nginx=nginx:1.29-alpine

# Observar el rollout
kubectl get pods --watch

# Si hay problemas, hacer rollback
kubectl rollout undo deployment/nginx-deployment
```

### Ejemplo 3: Debugging de un pod
```bash
# Verificar estado
kubectl describe pod <pod-name>

# Ver logs
kubectl logs <pod-name> -f

# Acceder al pod
kubectl exec -it <pod-name> -- /bin/bash

# Dentro del pod, verificar conectividad
curl http://service-name
ping google.com
ps aux
env
```

### Ejemplo 4: Trabajar con Secrets
```bash
# Crear secret
kubectl create secret generic db-password --from-literal=password=mysecretpass

# Ver secret (codificado)
kubectl get secret db-password -o yaml

# Decodificar el valor
kubectl get secret db-password -o jsonpath='{.data.password}' | base64 -d
```

---

**Última actualización**: Diciembre 2025  
**Versión de Kubernetes**: Compatible con 1.21+