# 🎓 Kubernetes Complete Learning Guide

**Autor:** Basado en Udemy Kubernetes Course  
**Total de Lecciones:** 150  
**Enfoque:** Conceptos, Casos de Uso y Ejemplos Prácticos con kubectl

---

## 📋 Tabla de Contenidos

1. [Introducción a Kubernetes](#introducción-a-kubernetes)
2. [Conceptos Fundamentales](#conceptos-fundamentales)
3. [Tipos de Objetos en Kubernetes](#tipos-de-objetos-en-kubernetes)
4. [Instalación y Configuración](#instalación-y-configuración)
5. [Pods y Contenedores](#pods-y-contenedores)
6. [Deployments](#deployments)
7. [Services](#services)
8. [Almacenamiento](#almacenamiento)
9. [Configuración y Secretos](#configuración-y-secretos)
10. [Escalado y Autoscaling](#escalado-y-autoscaling)
11. [Ingress y Networking](#ingress-y-networking)
12. [Administración y RBAC](#administración-y-rbac)
13. [Monitoreo y Troubleshooting](#monitoreo-y-troubleshooting)

---

## Introducción a Kubernetes

### ¿Qué es Kubernetes?

Kubernetes es un **orquestador de contenedores** que automatiza:
- 🚀 **Despliegue** de aplicaciones
- 📦 **Escalado** automático
- 🔄 **Actualización** sin interrupciones
- 🛡️ **Recuperación** ante fallos
- ⚖️ **Balanceo de carga**

### Casos de Uso Reales

| Caso de Uso | Descripción | Beneficio |
|---|---|---|
| **Microservicios** | Ejecutar múltiples servicios de forma independiente | Escalado granular, actualización independiente |
| **SaaS Multi-tenant** | Múltiples clientes en una infraestructura compartida | Aislamiento con isolation, optimización de recursos |
| **CI/CD Pipeline** | Ejecutar jobs de build y test | Escalado dinámico de workers |
| **ML/AI Workloads** | Entrenar modelos en paralelo | Aprovechar GPUs, escalado dinámico |
| **API Web Scale** | APIs que manejan millones de requests | Alta disponibilidad y escalabilidad |

---

## Conceptos Fundamentales

### 1. Arquitectura de Kubernetes

```
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                   │
├─────────────────────────────────────────────────────────┤
│ Master Node (Control Plane)                             │
│  - API Server: Gestiona la API                          │
│  - etcd: Base de datos clave-valor                      │
│  - Scheduler: Asigna pods a nodos                       │
│  - Controller Manager: Mantiene estado deseado          │
├─────────────────────────────────────────────────────────┤
│ Worker Nodes                                             │
│  - Node 1: kubelet + container runtime                  │
│  - Node 2: kubelet + container runtime                  │
│  - Node 3: kubelet + container runtime                  │
└─────────────────────────────────────────────────────────┘
```

### 2. Componentes Clave

#### **Pod** (Unidad más pequeña)
Un Pod es un contenedor o grupo de contenedores que:
- Comparten red (mismo IP)
- Comparten almacenamiento
- Se despliegan juntos

```bash
# Ver pods en default namespace
kubectl get pods

# Ver pods en todos los namespaces
kubectl get pods -A

# Obtener información detallada
kubectl get pods -o wide

# Ver descripción de un pod
kubectl describe pod <pod-name>

# Ver logs de un pod
kubectl logs <pod-name>

# Ver logs de un contenedor específico en un pod multi-contenedor
kubectl logs <pod-name> -c <container-name>

# Seguir logs en tiempo real
kubectl logs -f <pod-name>
```

**Caso de Uso Práctico:**

```yaml
# Crear pod con nginx
# Comentarios:
# - apiVersion: versión de la API que gestiona este recurso (v1 para recursos básicos).
# - kind: tipo de recurso (Pod, Deployment, Service, etc.).
# - metadata: información identificadora (name, namespace, labels, annotations).
# - spec: la definición declarativa del estado deseado del recurso.
# - containers: lista de contenedores que correrán dentro del Pod. Cada contenedor tiene image, ports, resources, probes, etc.
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod    # Nombre único del pod dentro del namespace
  namespace: default # Namespace donde se crea el pod (aislamiento lógico)
  labels:
    app: nginx       # Etiqueta útil para selecciones y servicios
spec:
  containers:
  - name: nginx
    image: nginx:1.21               # Imagen de contenedor (repositorio:nombre:tag)
    imagePullPolicy: IfNotPresent   # Política para descargar la imagen: Always/IfNotPresent/Never
    ports:
    - containerPort: 80             # Puerto interno del contenedor
    resources:                       # Requests vs Limits: reservations y límites para scheduler y cgroup
      requests:
        cpu: "100m"                # CPU solicitada (millicores)
        memory: "128Mi"            # Memoria solicitada
      limits:
        cpu: "500m"                # Límite máximo de CPU
        memory: "256Mi"            # Límite máximo de memoria
    readinessProbe:                  # Indica cuándo el contenedor está listo para recibir tráfico
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:                   # Indica si el contenedor sigue vivo; si falla, kubelet reinicia el contenedor
      httpGet:
        path: /health
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
```

```bash
# Aplicar configuración
kubectl apply -f nginx-pod.yaml

# Verificar creación
kubectl get pods
kubectl describe pod nginx-pod

# Acceder al contenedor
kubectl exec -it nginx-pod -- /bin/bash

# Dentro del contenedor
curl localhost:80

# Ver recursos utilizados
kubectl top pod nginx-pod
```

#### **Deployment** (Replicación y actualización)

Los *Deployments* son la forma recomendada de gestionar aplicaciones sin estado en Kubernetes. Proporcionan:
- Declaración del número deseado de réplicas (replicas) para alta disponibilidad.
- Estrategias de actualización (rolling   update, recreate) para despliegues con mínimo downtime.
- Un historial de revisiones que permite hacer *rollbacks* en caso de fallos.

Conceptos clave:
- `selector`: define cómo el Deployment identifica los Pods que controla (labels).
- `template`: definición del Pod que será replicado; contiene `metadata` y `spec` análogos a un Pod.
- `strategy`: controla el comportamiento durante actualizaciones (maxSurge, maxUnavailable).
- `resources.requests/limits`: ayudan al scheduler a colocar Pods en nodos adecuados.
- `affinity`/`tolerations`: controlan sobre qué nodos se programan los Pods.

Antes de desplegar, piensa en:
- Definir probes (readiness/liveness) para salud y tráfico.
- Añadir `resources` para evitar overcommit y garantizar SLA.
- Usar `labels` y `annotations` para observabilidad y gestión.

```bash
# Ver deployments
kubectl get deployments

# Ver todo sobre un deployment
kubectl describe deployment <deployment-name>

# Crear deployment (forma imperative)
kubectl create deployment nginx-dep --image=nginx:1.21 --replicas=3

# Ver replicas creadas
kubectl get pods | grep nginx-dep

# Escalado manual
kubectl scale deployment nginx-dep --replicas=5

# Verificar escalado
kubectl get pods

# Actualizar imagen
kubectl set image deployment/nginx-dep nginx=nginx:1.22

# Ver historial de cambios
kubectl rollout history deployment/nginx-dep

# Ver estado del rollout
kubectl rollout status deployment/nginx-dep

# Revertir a versión anterior
kubectl rollout undo deployment/nginx-dep

# Revertir a una versión específica
kubectl rollout undo deployment/nginx-dep --to-revision=1
```

**Caso de Uso Práctico - Deployment con YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web
# Comentarios:
# - apiVersion: apps/v1: uso común para recursos de controladores (Deployment, StatefulSet).
# - kind: Deployment: gestiona réplicas y actualizaciones declarativas.
# - metadata.labels: etiquetas para selección y agrupación; útiles para servicios y selecciones.
spec:
  replicas: 3                # Número deseado de réplicas de pod
  strategy:
    type: RollingUpdate      # RollingUpdate permite actualizar sin downtime
    rollingUpdate:
      maxSurge: 1            # Pods extra que pueden crearse durante la actualización
      maxUnavailable: 0      # Máximo de pods no disponibles durante el update
  selector:
    matchLabels:
      app: web               # Selector que empareja los pods gestionados por este Deployment
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: webapp
        image: nginx:1.21      # Imagen del contenedor
        ports:
        - containerPort: 80    # Puerto expuesto por el contenedor
        resources:
          requests:
            memory: "64Mi"   # Recursos solicitados (garantizados por el scheduler)
            cpu: "100m"
          limits:
            memory: "256Mi"   # Límites máximos para el contenedor
            cpu: "500m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

```bash
# Ejecutar deployment
kubectl apply -f deployment.yaml

# Monitorear despliegue en tiempo real
kubectl rollout status deployment/web-app --watch

# Ver eventos
kubectl get events

# Inspeccionar details
kubectl describe deployment web-app
```

---

## Tipos de Objetos en Kubernetes

Todo lo que gestionas en Kubernetes es un **objeto** (o *recurso*): una entidad declarativa definida en YAML/JSON e identificada por su `apiVersion` y `kind`, que representa un estado deseado que el clúster se encarga de mantener. Los objetos se agrupan en categorías según el problema que resuelven: ejecutar cargas de trabajo, exponer red, persistir datos, gestionar configuración, controlar acceso o escalar automáticamente.

```bash
# Listar todos los tipos de objetos (kinds) disponibles en el cluster
kubectl api-resources

# Filtrar solo los que aceptan namespace
kubectl api-resources --namespaced=true

# Ver a qué apiVersion pertenece un kind específico
kubectl explain deployment
kubectl explain deployment.spec.strategy
```

### 1. Workloads (Cargas de Trabajo)

Definen cómo se ejecutan los contenedores.

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **Pod** | Unidad mínima desplegable; uno o más contenedores que comparten red y almacenamiento | Rara vez se crea directamente; casi siempre gestionado por un controlador |
| **ReplicaSet** | Garantiza que un número fijo de réplicas de un Pod estén corriendo | Casi nunca directo; lo gestiona un Deployment |
| **Deployment** | Gestiona ReplicaSets, permite rolling updates y rollbacks | Apps sin estado (stateless): APIs, web, workers |
| **StatefulSet** | Como un Deployment, pero con identidad de red y almacenamiento estable por réplica | Apps con estado (stateful): bases de datos, colas |
| **DaemonSet** | Asegura que una copia del Pod corra en todos (o algunos) los nodos | Agentes de nodo: logging, monitoreo, CNI, antivirus |
| **Job** | Ejecuta Pods hasta completar una tarea exitosamente N veces | Tareas puntuales: migraciones, batch processing |
| **CronJob** | Crea Jobs según una calendarización tipo cron | Tareas periódicas: backups, reportes, limpieza |

### 2. Networking (Red)

Controlan cómo se comunican los Pods entre sí y con el exterior.

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **Service** | Expone un conjunto de Pods bajo una IP/DNS estable (ClusterIP, NodePort, LoadBalancer) | Descubrimiento y balanceo de carga interno/externo |
| **Ingress** | Enruta tráfico HTTP/HTTPS externo hacia Services según host/path | Exponer múltiples servicios bajo un mismo punto de entrada |
| **NetworkPolicy** | Define reglas de firewall a nivel de Pod (qué tráfico entra/sale) | Aislar namespaces o restringir comunicación entre apps |
| **Endpoints / EndpointSlice** | Lista las IPs de los Pods que respaldan a un Service | Generado automáticamente; útil para debugging |

### 3. Almacenamiento

Gestionan la persistencia y el acceso a datos.

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **Volume** | Almacenamiento asociado al ciclo de vida de un Pod (`emptyDir`, `configMap`, etc.) | Compartir archivos entre contenedores de un mismo Pod, cache temporal |
| **PersistentVolume (PV)** | Recurso de almacenamiento físico/cloud provisionado en el clúster | Lo suele crear el administrador o un provisioner dinámico |
| **PersistentVolumeClaim (PVC)** | Solicitud de almacenamiento hecha por una app; se enlaza a un PV | Pedir disco persistente para bases de datos, uploads, etc. |
| **StorageClass** | Define cómo se aprovisiona dinámicamente el almacenamiento (tipo de disco, proveedor) | Automatizar la creación de PVs bajo demanda (EBS, GCE PD, etc.) |

### 4. Configuración

Separan la configuración del código de la aplicación.

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **ConfigMap** | Almacena datos de configuración no sensibles (pares clave/valor o archivos) | Variables de entorno, archivos de configuración |
| **Secret** | Igual que ConfigMap, pero pensado para datos sensibles (codificados en base64) | Contraseñas, tokens, llaves, credenciales de registry |

### 5. Seguridad y Control de Acceso (RBAC)

Definen quién puede hacer qué dentro del clúster.

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **ServiceAccount** | Identidad que usan los Pods para autenticarse ante la API de Kubernetes | Dar permisos específicos a una app (ej. acceso de lectura a Secrets) |
| **Role / ClusterRole** | Define un conjunto de permisos (verbos sobre recursos) a nivel namespace o clúster | Restringir qué puede hacer un usuario o ServiceAccount |
| **RoleBinding / ClusterRoleBinding** | Asocia un Role/ClusterRole a un usuario, grupo o ServiceAccount | Otorgar efectivamente los permisos definidos en un Role |

### 6. Escalado y Disponibilidad

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **HorizontalPodAutoscaler (HPA)** | Escala el número de réplicas según métricas (CPU, memoria, custom) | Apps con carga variable que necesitan escalar solas |
| **VerticalPodAutoscaler (VPA)** | Ajusta automáticamente `requests`/`limits` de los Pods (requiere addon aparte) | Optimizar recursos cuando no se conoce el consumo real de antemano |
| **PodDisruptionBudget (PDB)** | Limita cuántos Pods pueden estar caídos a la vez durante mantenimiento voluntario | Garantizar disponibilidad mínima durante drenados de nodos o updates |

### 7. Organización del Clúster

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **Namespace** | Divide el clúster en espacios de nombres lógicos y aislados | Separar entornos (dev/staging/prod) o equipos/proyectos |
| **ResourceQuota** | Limita el consumo total de recursos (CPU, memoria, cantidad de objetos) por namespace | Evitar que un equipo/proyecto consuma todos los recursos del clúster |
| **LimitRange** | Define límites/valores por defecto de recursos por Pod o contenedor en un namespace | Prevenir Pods sin `resources` definidos que puedan sobrecargar nodos |
| **Node** | Representa una máquina (física o virtual) worker del clúster | No se crea manualmente; lo registra el kubelet al unirse al clúster |

### 8. Extensibilidad

| Objeto | Descripción | Cuándo usarlo |
|---|---|---|
| **CustomResourceDefinition (CRD)** | Permite definir nuevos `kind` propios, extendiendo la API de Kubernetes | Modelar conceptos de dominio propio (ej. `Certificate`, `Backup`) |
| **Operator** | Combina un CRD con un controlador que automatiza la gestión de ese recurso | Automatizar tareas operativas complejas (bases de datos, backups) |

> 💡 **Resumen mental:** *Workloads* dicen qué correr, *Networking* dice cómo se comunica, *Almacenamiento* dice dónde persisten los datos, *Configuración* separa settings del código, *RBAC* dice quién puede tocar qué, y *Escalado*/*Organización* mantienen todo estable y ordenado a medida que el clúster crece.

---

## Instalación y Configuración

### 1. Opción Local: Minikube

**Caso: Desarrollador Local**

```bash
# Instalar Minikube
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Iniciar cluster
minikube start --cpus=4 --memory=8192

# Verificar estado
minikube status

# Obtener IP
minikube ip

# Ver dashboard
minikube dashboard

# Parar cluster
minikube stop

# Eliminar cluster
minikube delete
```

### 2. Opción Cloud: EKS (AWS)

**Caso: Production en AWS**

```bash
# Instalar eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz

# Crear cluster
eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name workers --node-type t3.medium --nodes 3

# Verificar cluster
kubectl get nodes

# Ver contextos
kubectl config get-contexts

# Cambiar contexto
kubectl config use-context <context-name>

# Ver configuración actual
kubectl config current-context
```

### 3. Kubeconfig

```bash
# Mergear múltiples kubeconfigs
export KUBECONFIG=~/.kube/config:~/.kube/config-prod:~/.kube/config-dev
kubectl config view

# Ver todos los clusters configurados
kubectl config get-clusters

# Ver usuarios
kubectl config get-users

# Cambiar default namespace
kubectl config set-context --current --namespace=production

# Crear nuevo contexto
kubectl config set-context prod-eks --cluster=prod-eks --user=admin@prod-eks --namespace=default
```

---

## Pods y Contenedores

Los Pods son la unidad mínima de ejecución en Kubernetes. Contienen uno o más contenedores que comparten red y almacenamiento. Al trabajar con Pods piensa en:
- Ciclo de vida efímero: los Pods pueden recrearse; no almacenar datos locales sin volúmenes.
- Probes: readiness/liveness/startup para controlar tráfico y reinicios.
- Resources: requests y limits para planificación y estabilidad.
- Labels y Selectors para agrupamiento y descubrimiento.

### Caso de Uso 1: Pod Simple

```yaml
# simple-pod.yaml
# Comentarios:
# - metadata.name: identificador del Pod.
# - labels: permiten agrupar y seleccionar el Pod desde un Service o Deployment.
# - spec.containers: cada entrada describe un contenedor (image, ports, command, resources, probes).
apiVersion: v1
kind: Pod
metadata:
  name: web-server
  labels:
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:latest          # Imagen oficial de nginx
    ports:
    - containerPort: 80          # Puerto expuesto internamente
    readinessProbe:              # Indica cuando el contenedor está listo para tráfico
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
  - name: sidecar
    image: busybox:latest
    command: ['sh', '-c', 'echo "Sidecar corriendo"']
    # Los sidecars suelen complementar la funcionalidad (logs, proxy, sincronización)
```

```bash
# Crear y verificar
kubectl apply -f simple-pod.yaml
kubectl get pods web-server
kubectl describe pod web-server

# Acceder a contenedor
kubectl exec -it web-server -c nginx -- /bin/bash

# Ver logs de sidecar
kubectl logs web-server -c sidecar
```

### Caso de Uso 2: Pod con Probes de Salud

```yaml
# pod-with-probes.yaml
# Comentarios sobre probes:
# - livenessProbe: si falla, kubelet reinicia el contenedor (definir para detectar deadlocks).
# - readinessProbe: si falla, el Pod se marca como no listo y no recibe tráfico del Service.
# - startupProbe: útil para aplicaciones con inicio lento; evita que livenessProbe reinicie prematuramente.
apiVersion: v1
kind: Pod
metadata:
  name: healthy-app
spec:
  containers:
  - name: app
    image: myapp:v1
    ports:
    - containerPort: 8000

    livenessProbe:
      httpGet:
        path: /health
        port: 8000
      initialDelaySeconds: 30
      periodSeconds: 10
      failureThreshold: 3

    readinessProbe:
      httpGet:
        path: /ready
        port: 8000
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 2

    startupProbe:
      httpGet:
        path: /startup
        port: 8000
      failureThreshold: 30
      periodSeconds: 1
```

```bash
# Desplegar
kubectl apply -f pod-with-probes.yaml

# Monitorear probes
kubectl describe pod healthy-app

# Ver eventos de los probes
kubectl get events --field-selector involvedObject.name=healthy-app

# Simular fallo de probe
kubectl port-forward healthy-app 8000:8000 &
```

---

## Deployments

### Caso de Uso 1: API REST con Scaling

**Escenario:** API de e-commerce que necesita escalar según tráfico
# network-policy.yaml
# Comentarios:
# - NetworkPolicy controla tráfico a nivel de Pod (Capa 3/4). Por defecto, muchas CNI permiten todo el tráfico; las NetworkPolicies aplican restricciones.
# - podSelector: selecciona los pods a los que se aplica la política. `{}` significa todos los pods en el namespace.
# - policyTypes: Ingress/Egress; define direcciones de tráfico afectadas.
# - from: origen permitido (podSelector, namespaceSelector, ipBlock).
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}                   # Selecciona todos los pods en el namespace
  policyTypes:
  - Ingress                         # Bloquea todo ingreso no explícitamente permitido
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend            # Permite tráfico desde Pods con label tier=frontend
    ports:
    - protocol: TCP
      port: 8080
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-db
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: backend            # Permite tráfico desde el backend hacia la BD
    ports:
    - protocol: TCP
      port: 5432

      containers:
      - name: api
        image: ecommerce-api:v2.1.0     # Imagen de la aplicación con etiqueta de versión
        imagePullPolicy: IfNotPresent   # Evita descargar imagen si ya está presente

        ports:
        - containerPort: 8080
          name: api
          protocol: TCP

        env:                            # Variables de entorno para configuración de la app
        - name: DB_HOST
          value: "postgres.default.svc.cluster.local"  # Servicio interno de BD
        - name: DB_PORT
          value: "5432"
        - name: LOG_LEVEL
          value: "info"

        resources:
          requests:
            memory: "256Mi"            # Recursos solicitados (para scheduling)
            cpu: "250m"
          limits:
            memory: "512Mi"            # Límites máximos consumibles
            cpu: "1000m"

        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          failureThreshold: 2

        volumeMounts:
        - name: logs
          mountPath: /var/log/app        # Montaje temporal para logs (emptyDir)

      volumes:
      - name: logs
        emptyDir: {}                    # Volumen efímero en cada nodo; no persiste entre reinicios
```

```bash
# Desplegar
kubectl apply -f ecommerce-api.yaml

# Verificar estado del deployment
kubectl rollout status deployment/ecommerce-api
kubectl get deployment ecommerce-api -o wide

# Ver los pods creados
kubectl get pods -l app=ecommerce

# Ver uso de recursos
kubectl top pod -l app=ecommerce

# Simular carga y ver como el deployment maneja fallos
kubectl port-forward deployment/ecommerce-api 8080:8080 &

# En otra terminal, generar carga
for i in {1..100}; do curl http://localhost:8080/api/products & done

# Verificar cómo Kubernetes resuelve fallos
kubectl delete pod -l app=ecommerce  # Eliminar pods
kubectl get pods -l app=ecommerce   # Se recrearán automáticamente

# Actualizar imagen
kubectl set image deployment/ecommerce-api api=ecommerce-api:v2.2.0

# Ver progreso
kubectl rollout status deployment/ecommerce-api --watch

# Ver historial
kubectl rollout history deployment/ecommerce-api

# Revertir si hay problemas
kubectl rollout undo deployment/ecommerce-api

# Ver cambios descritpivos
kubectl diff -f ecommerce-api.yaml
```

### Caso de Uso 2: Actualización Canary

```bash
# Actualización gradual (Canary)
kubectl set image deployment/ecommerce-api api=ecommerce-api:v2.2.0

# Verificar que solo una instancia se actualiza primero
kubectl get pods -o wide

# Esperar a que se estabilice
kubectl rollout status deployment/ecommerce-api --watch

# Monitorear métricas durante update
watch kubectl top pods -l app=ecommerce
```

---

## Services
Los Services exponen Pods dentro o fuera del cluster y proporcionan estabilidad de red mediante una IP virtual y balanceo. Conceptos importantes:
- `type`: ClusterIP, NodePort, LoadBalancer o ExternalName.
- `selector`: etiquetas que el Service usa para encontrar endpoints (Pods).
- `ports`: número del Service (`port`) y puerto del Pod (`targetPort`).
- `ClusterIP`: acceso interno; `NodePort` y `LoadBalancer` exponen externamente.

### Caso de Uso 1: ClusterIP (Acceso Interno)

```yaml
# service-internal.yaml
# Comentarios:
# - type: determina visibilidad del servicio (ClusterIP interno por defecto).
# - selector: empareja pods mediante labels para crear endpoints.
# - ports.port: puerto del servicio; targetPort: puerto del contenedor objetivo.
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-api-service
  labels:
    app: ecommerce
spec:
  type: ClusterIP                 # Servicio accesible solo dentro del cluster
  selector:
    app: ecommerce
    tier: backend
  ports:
  - name: api
    port: 80                       # Puerto en el Service
    targetPort: 8080               # Puerto en el Pod al que se enruta
    protocol: TCP
```

```bash
# Crear servicio
kubectl apply -f service-internal.yaml

# Ver servicios
kubectl get svc

# Describir servicio
kubectl describe svc ecommerce-api-service

# Verificar endpoints (pods que está balanceando)
kubectl get endpoints ecommerce-api-service

# Testear acceso desde otro pod
kubectl run debug --image=busybox --rm -it --restart=Never -- wget -O- http://ecommerce-api-service/api/products

# Ver DNS interno
kubectl run debug --image=busybox --rm -it --restart=Never -- nslookup ecommerce-api-service
```

### Caso de Uso 2: NodePort (Acceso Externo)

```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-nodeport
spec:
  type: NodePort
  selector:
    app: ecommerce
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Puerto en cada nodo (30000-32767)
```

```bash
# Crear
kubectl apply -f service-nodeport.yaml

# Obtener NodePort asignado
kubectl get svc ecommerce-nodeport

# Acceder desde fuera del cluster
curl http://<node-ip>:30080

# En minikube
minikube service ecommerce-nodeport

# Obtener la URL
minikube service ecommerce-nodeport --url
```

### Caso de Uso 3: LoadBalancer (Cloud)

```yaml
# service-lb.yaml
apiVersion: v1
kind: Service
metadata:
  name: ecommerce-lb
spec:
  type: LoadBalancer
  selector:
    app: ecommerce
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

```bash
# Crear
kubectl apply -f service-lb.yaml

# Esperar a que se asigne IP externa
kubectl get svc ecommerce-lb --watch

# Una vez asignada
curl http://<external-ip>

# Ver detalles
kubectl describe svc ecommerce-lb
```

---

## Almacenamiento

El almacenamiento en Kubernetes aborda dos necesidades principales: datos efímeros y datos persistentes. Para bases de datos y sistemas con estado se usan StatefulSets con PVCs; para caches y datos temporales se usan emptyDir o tmpfs. Conceptos clave:
- `PersistentVolume` (PV): recurso físico/proveedor (NFS, cloud disk, local) con capacidad y acceso.
- `PersistentVolumeClaim` (PVC): petición de almacenamiento por parte de un Pod; enlaza con PV.
- `volumeClaimTemplates` en StatefulSet: crea PVCs con nombres estables por réplica.
- `accessModes`: ReadWriteOnce, ReadOnlyMany, ReadWriteMany.
- `storageClassName`: clase de almacenamiento para provisión dinámica.

### Caso de Uso 1: Base de Datos Stateful

```yaml
# stateful-db.yaml
# Comentarios:
# - StatefulSet: mantiene identidad estable de pods (nombres predecibles: postgres-0, postgres-1...).
# - serviceName: servicio Headless (clusterIP: None) requerido para StatefulSets para resolver pods individualmente.
# - volumeClaimTemplates: crea PVCs por réplica con nombres estables; útil para persistencia de datos.
# - subPath: permite usar subcarpetas en un volumen compartido.
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres        # Nombre del Service headless que permite DNS estable para cada réplica
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: "ecommerce"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
          subPath: postgres
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi

---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None                # Headless Service para StatefulSets (DNS por pod)
  selector:
    app: postgres
  ports:
  - port: 5432
    name: postgres
```

```bash
# Crear secreto para contraseña
kubectl create secret generic db-secret --from-literal=password=MySecurePass123

# Desplegar StatefulSet
kubectl apply -f stateful-db.yaml

# Ver StatefulSets
kubectl get statefulset

# Ver PVCs (Persistent Volume Claims)
kubectl get pvc

# Ver volúmenes
kubectl get pv

# Conectar a la base de datos
kubectl exec -it postgres-0 -- psql -U postgres -d ecommerce

# Dentro de PostgreSQL
CREATE TABLE products (id SERIAL PRIMARY KEY, name VARCHAR(255));
INSERT INTO products (name) VALUES ('Laptop');
SELECT * FROM products;
\q

# Verificar que datos persisten incluso si el pod se recrea
kubectl delete pod postgres-0

# El pod se recrea y los datos siguen ahí
kubectl exec -it postgres-0 -- psql -U postgres -d ecommerce -c "SELECT * FROM products;"
```

### Caso de Uso 2: Volúmenes Temporales

```yaml
```yaml
# pod-with-volumes.yaml
# Comentarios:
# - emptyDir: volumen efímero ligado al ciclo de vida del Pod (se borra al eliminar el Pod).
# - mountPath: ruta dentro del contenedor donde se monta el volumen.
apiVersion: v1
kind: Pod
metadata:
  name: app-with-cache
spec:
  containers:
  - name: app
    image: myapp:v1
    volumeMounts:
    - name: cache
      mountPath: /cache       # Cache temporal en el nodo
    - name: logs
      mountPath: /var/log
  - name: log-collector
    image: busybox:latest
    command: ['sh', '-c', 'tail -f /var/log/app.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log
  volumes:
  - name: cache
    emptyDir: {}             # Volumen temporal, útil para caches intermedios
  - name: logs
    emptyDir: {}
```

```bash
# Crear
kubectl apply -f pod-with-volumes.yaml

# Verificar volúmenes
kubectl describe pod app-with-cache

# Escribir datos en cache desde la app
kubectl exec -it app-with-cache -c app -- sh -c 'echo "datos en cache" > /cache/data.txt'

# Leer desde sidecar
kubectl exec -it app-with-cache -c log-collector -- cat /cache/data.txt
# Nota: No funcionará porque están en volumenes diferentes, ejemplo educativo
```

---

## Configuración y Secretos
Las ConfigMaps y Secrets separan configuración de código, facilitando actualizaciones sin reconstruir imágenes. Buenas prácticas:
- Guardar contraseñas y credenciales en `Secrets` (codificados en base64), no en ConfigMaps.
- Usar `envFrom` o `volumeMounts` para inyectar configuración en Pods.
- Mantener configuración inmutable en producción y versionada en Git cuando sea posible.

### Caso de Uso 1: ConfigMap

```yaml
# configmap.yaml
# Comentarios:
# - ConfigMap almacena datos no sensibles en pares clave/valor o archivos completos.
# - Use `data` para valores y `binaryData` para binarios.
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.yml: |
    host: postgres.default.svc.cluster.local
    port: 5432
    pool: 10
  features.json: |
    {
      "new_checkout": true,
      "recommendation_engine": false
    }
  LOG_LEVEL: "debug"
```

```bash
# Crear
kubectl apply -f configmap.yaml

# Ver ConfigMaps
kubectl get configmap

# Ver contenido
kubectl describe configmap app-config

# Editar online
kubectl edit configmap app-config

# Crear desde archivo
kubectl create configmap db-config --from-file=database.yml

# Crear desde literales
kubectl create configmap features --from-literal=checkout=true --from-literal=recommendations=false

# Ver en YAML
kubectl get configmap app-config -o yaml

# Eliminar
kubectl delete configmap app-config
```

**Usar ConfigMap en Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-config
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: myapp:v1
        
        # Inyectar como variables de entorno
        env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database.host
        
        # Inyectar como archivos montados
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
        
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

### Caso de Uso 2: Secrets

```bash
# Crear secreto para credenciales de Docker
kubectl create secret docker-registry regcred \
  --docker-server=gcr.io \
  --docker-username=_json_key \
  --docker-password="$(cat ~/key.json)" \
  --docker-email=user@example.com

# Crear secreto genérico
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=SecurePass123

# Crear secreto desde archivos
kubectl create secret generic ssl-certs \
  --from-file=tls.crt=./cert.pem \
  --from-file=tls.key=./key.pem

# Ver secretos (no muestra valores)
kubectl get secrets

# Ver valores en base64
kubectl get secret db-credentials -o yaml

# Decodificar
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d

# Usar en Deployment
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      # Para descargar imágenes de registro privado
      imagePullSecrets:
      - name: regcred

      containers:
      - name: app
        image: gcr.io/myproject/secure-app:v1
        # Comentarios:
        # - imagePullSecrets: referencia a Secret de tipo docker-registry para acceder a registros privados.
        # - valueFrom.secretKeyRef: inyecta variables desde Secrets sin exponerlas en el manifiesto.
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password

        volumeMounts:
        - name: ssl
          mountPath: /etc/ssl
          readOnly: true

      volumes:
      - name: ssl
        secret:
          secretName: ssl-certs   # Monta certificados desde Secret como archivos en el pod
```

```bash
# Desplegar
kubectl apply -f deployment-with-secrets.yaml

# Ver variables de entorno
kubectl exec -it <pod-name> -- env | grep DB_

# Ver archivos montados
kubectl exec -it <pod-name> -- ls -la /etc/ssl/
```

---

## Escalado y Autoscaling

El escalado permite ajustar la capacidad de la aplicación según demanda. Kubernetes ofrece escalado manual y autoscaling (HPA/VPA) que pueden basarse en métricas como CPU, memoria o métricas custom.

Buenas prácticas:
- Configurar `requests` y `limits` para que el HPA tenga métricas confiables.
- Evitar escalar por métricas de aplicación inestables; usar métricas agregadas cuando sea posible.
- Probar políticas de `behavior` en HPA para evitar flapping.

### Caso de Uso 1: Escalado Manual

```bash
# Ver replicas actuales
kubectl get deployment ecommerce-api -o wide

# Escalar a 5 replicas
kubectl scale deployment ecommerce-api --replicas=5

# Verificar
kubectl get pods

# Escalar hacia abajo
kubectl scale deployment ecommerce-api --replicas=2
```

### Caso de Uso 2: Horizontal Pod Autoscaler (HPA)

```bash
# Primero, asegurarse que Metrics Server está instalado
kubectl get deployment metrics-server -n kube-system

# En minikube, habilitar
minikube addons enable metrics-server

# Crear HPA con línea de comandos
kubectl autoscale deployment ecommerce-api \
  --min=2 \
  --max=10 \
  --cpu-percent=80

# Ver HPA
kubectl get hpa

# Descripción detallada
kubectl describe hpa ecommerce-api

# Ver métricas
kubectl top pods -l app=ecommerce
```

**Con YAML:**

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-api
  minReplicas: 2                  # Réplicas mínimas que HPA puede mantener
  maxReplicas: 10                 # Réplicas máximas permitidas
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # Objetivo de utilización de CPU por pod (en %)
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80    # Objetivo de memoria (opcional)
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 30
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
# Comentarios:
# - apiVersion autoscaling/v2 permite reglas más avanzadas y métricas custom.
# - behavior.scaleUp/scaleDown controla estabilidad y velocidad de escalado para evitar flapping.
# - Siempre validar que Metrics Server u otro proveedor de métricas esté funcionando.
```

```bash
# Aplicar
kubectl apply -f hpa.yaml

# Monitorear en tiempo real
kubectl get hpa -w

# Simular carga
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh
# Dentro del pod
while sleep 0.01; do wget -q -O- http://ecommerce-api-service; done

# En otra terminal, ver el HPA escalando
kubectl get hpa ecommerce-hpa --watch

# Ver eventos
kubectl get events --field-selector involvedObject.name=ecommerce-api
```

---

## Ingress y Networking

Ingress y Network Policies controlan el tráfico de capa 7 y la seguridad de red entre pods/servicios. Ingress requiere un Ingress Controller (NGINX, Traefik) y permite TLS, virtual-host routing y reglas avanzadas.

Conceptos clave:
- `Ingress`: define reglas HTTP/HTTPS para enrutar dominios y paths hacia Services.
- `ingressClassName`: indica el controlador que atenderá este Ingress.
- `annotations`: configuraciones específicas del controlador (rate-limit, redirect, cert-manager).
- `tls.secretName`: Secret que contiene certificados TLS para los hosts definidos.

### Caso de Uso 1: Exponer múltiples servicios con Ingress

```yaml
# ingress.yaml
# Comentarios:
# - annotations: ajustes específicos del controlador (rate-limit, ssl-redirect, cert-manager).
# - ingressClassName: indica qué controlador (ingress controller) gestionará este Ingress.
# - tls.secretName: secreto con certificados TLS para los hosts listados.
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # Cert-manager configura certificados TLS
    nginx.ingress.kubernetes.io/rate-limit: "10"      # Ejemplo de anotación para nginx-ingress
    nginx.ingress.kubernetes.io/ssl-redirect: "true"  # Forzar redirect a HTTPS
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - ecommerce.example.com
    - api.ecommerce.example.com
    secretName: ecommerce-tls
  rules:
  - host: ecommerce.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.ecommerce.example.com
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: api-v1-service
            port:
              number: 8080
      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: api-v2-service
            port:
              number: 8080
```

```bash
# Instalar Ingress Controller (NGINX)
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace

# Verificar instalación
kubectl get pods -n ingress-nginx

# Aplicar Ingress
kubectl apply -f ingress.yaml

# Ver Ingress
kubectl get ingress

# Ver detalles
kubectl describe ingress ecommerce-ingress

# Obtener IP del Ingress
kubectl get ingress ecommerce-ingress -o wide

# Actualizar /etc/hosts
echo "<ingress-ip> ecommerce.example.com" >> /etc/hosts
echo "<ingress-ip> api.ecommerce.example.com" >> /etc/hosts

# Probar
curl https://ecommerce.example.com
curl https://api.ecommerce.example.com/v1/products
```

### Caso de Uso 2: Network Policies

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-db
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: backend
    ports:
    - protocol: TCP
      port: 5432
```

```bash
# Aplicar policies
kubectl apply -f network-policy.yaml

# Ver network policies
kubectl get networkpolicies

# Verificar aislamiento
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -O- http://backend-service

# Debería fallar de acuerdo a las policies
```

---

## Administración y RBAC
La administración del cluster incluye la gestión de namespaces, cuotas, políticas de acceso (RBAC) y mantenimiento de nodos. RBAC controla permisos finos a recursos y usuarios/servicios.

### Caso de Uso 1: Namespaces

```bash
# Crear namespace
kubectl create namespace production
kubectl create namespace staging

# Ver namespaces
kubectl get namespaces

# Listar recursos en namespace
kubectl get pods -n production

# Cambiar namespace por defecto
kubectl config set-context --current --namespace=production

# Crear recursos en namespace específico
kubectl apply -f deployment.yaml -n staging

# Limpiar todo un namespace
kubectl delete namespace staging
```

**Namespace con quotas:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "10"            # Límite total de CPU solicitada en el namespace
    requests.memory: 20Gi          # Límite total de memoria solicitada
    limits.cpu: "20"             # Límite total de CPU asignada
    limits.memory: 40Gi           # Límite total de memoria asignada
    pods: "30"                   # Número máximo de pods permitidos
    services: "10"               # Número máximo de services permitidos
---
apiVersion: v1
kind: LimitRange
metadata:
  name: team-a-limits
  namespace: team-a
spec:
  limits:
  - max:
      cpu: "2"
      memory: "2Gi"
    min:
      cpu: "100m"
      memory: "128Mi"
    type: Container
# Comentarios:
# - ResourceQuota controla consumos agregados por namespace para evitar abuso de recursos.
# - LimitRange establece límites por contenedor si no se especifican, ayudando a previsibilidad.
```

```bash
# Aplicar
kubectl apply -f namespace-with-quotas.yaml

# Ver quotas
kubectl describe resourcequota team-a-quota -n team-a

# Intentar exceder
kubectl create deployment big-app --image=nginx:latest --replicas=50 -n team-a --dry-run=client -o yaml | kubectl apply -f -
# Vará fallar por quotas
```

### Caso de Uso 2: RBAC (Role-Based Access Control)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
# Rol para desarrolladores
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "update", "patch"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["get", "list"]
---
# RoleBinding - Asignar rol a usuario
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
- kind: User
  name: "developer@company.com"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
# ClusterRole - Rol a nivel cluster
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-custom
rules:
- apiGroups: ["*"]
  resources: ["*"]
# Comentarios:
# - Role: permisos namespace-scoped; ClusterRole: permisos a nivel cluster.
# - RoleBinding: asocia subjects (User, Group, ServiceAccount) a un Role en un namespace.
# - roleRef: referencia al Role/ClusterRole que se asigna.
  verbs: ["*"]
---
# ClusterRoleBinding
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin-binding
subjects:
- kind: User
  name: "admin@company.com"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-custom
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Aplicar RBAC
kubectl apply -f rbac.yaml

# Ver roles
kubectl get roles -A

# Ver role bindings
kubectl get rolebindings -A

# Describir un rol
kubectl describe role developer

# Probar acceso (como usuario específico)
kubectl get pods --as=developer@company.com

# Ver permisos de usuario actual
kubectl auth can-i get pods

# Ver si un usuario puede hacer algo específico
kubectl auth can-i update deployments --as=developer@company.com
```

---

## Monitoreo y Troubleshooting

### Caso de Uso 1: Monitoreo con Prometheus y Grafana

```yaml
# monitoring.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  type: LoadBalancer
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
```

```bash
# Desplegar
kubectl apply -f monitoring.yaml

# Acceder a Prometheus
kubectl port-forward svc/prometheus 9090:9090

# Navegar a http://localhost:9090

# Consultas útiles en Prometheus
# - Pod CPU usage
container_cpu_usage_seconds_total{pod_name="ecommerce-api-xyz"}
# - Pod Memory usage
container_memory_usage_bytes{pod_name="ecommerce-api-xyz"}
# - Pod RESTART count
kube_pod_container_status_restarts_total
```

### Caso de Uso 2: Logs y Debugging

```bash
# Ver logs de un pod
kubectl logs <pod-name>

# Últimas 100 líneas
kubectl logs <pod-name> --tail=100

# Logs en tiempo real
kubectl logs -f <pod-name>

# Logs de un contenedor específico
kubectl logs <pod-name> -c <container-name>

# Logs de pod anterior (si ha reiniciado)
kubectl logs <pod-name> --previous

# Logs de múltiples pods
kubectl logs -l app=ecommerce

# Con timestamps
kubectl logs <pod-name> --timestamps=true

# Exportar logs
kubectl logs <pod-name> > pod.log

# Ver eventos del namespace
kubectl get events -n default

# Ver eventos de un pod específico
kubectl describe pod <pod-name> | grep -A 10 Events

# Ver todos los eventos
kubectl get events -A --sort-by='.lastTimestamp'
```

### Caso de Uso 3: Debugging en Vivo

```bash
# Ejecutar comandos en pod
kubectl exec -it <pod-name> -- /bin/bash

# Ejecutar comando único
kubectl exec <pod-name> -- ps aux

# Si el contenedor es mínimo (alpine, distroless)
kubectl exec <pod-name> -- sh

# Port forward para acceder a aplicación
kubectl port-forward pod/<pod-name> 8080:8080

# Port forward a un servicio
kubectl port-forward svc/my-service 8080:8080

# Copiar archivos desde pod
kubectl cp <pod-name>:/path/to/file ./local-file

# Copiar archivos al pod
kubectl cp ./local-file <pod-name>:/path/to/file

# Crear pod de debug con imagen distroless
kubectl debug node/<node-name> -it --image=ubuntu

# Adjuntarse a un contenedor ya corriendo
kubectl attach -it <pod-name>

# Ver descripción completa
kubectl describe pod <pod-name>

# Obtener YAML de un recurso
kubectl get pod <pod-name> -o yaml

# Ver cambios en tiempo real
kubectl get pods -w

# Métricas de recursos
kubectl top nodes
kubectl top pods -A
kubectl top pod <pod-name> --containers
```

### Caso de Uso 4: Troubleshooting Común

**Problema: Pod no se inicia**

```bash
# Verificar estado
kubectl describe pod <pod-name>

# Mirar sección "Events"
kubectl get events --field-selector involvedObject.name=<pod-name>

# Causas comunes:
# 1. Imagen no existe
# 2. Recurso insuficiente
# 3. Volumen no disponible
# 4. Imagen pull error

# Solucionar
kubectl delete pod <pod-name>
kubectl apply -f corrected-pod.yaml
```

**Problema: Pod en CrashLoopBackOff**

```bash
# Ver logs (incluso de pods que fallaron)
kubectl logs <pod-name> --previous

# Ver últimas líneas antes de crash
kubectl logs <pod-name> --previous --tail=50

# Ver por qué se está reiniciando
kubectl describe pod <pod-name> | grep "Last State"

# Revisar configuración
kubectl get pod <pod-name> -o yaml

# Verificar health checks
kubectl get pod <pod-name> -o yaml | grep -A 10 "Probe"
```

**Problema: Servicio no es alcanzable**

```bash
# Verificar servicio existe
kubectl get svc <service-name>

# Ver endpoints (pods a los que redirige)
kubectl get endpoints <service-name>

# Si endpoints está vacío, pods no match selectores
kubectl get pods -l <selector-key>=<selector-value>

# Testear conectividad dentro del cluster
kubectl run test --image=busybox --rm -it --restart=Never -- wget -O- http://<service-name>

# Ver si DNS resuelve
kubectl run test --image=busybox --rm -it --restart=Never -- nslookup <service-name>

# Verificar NetworkPolicies
kubectl get networkpolicies

# Ver iptables (en nodo)
sudo iptables -L -n

# Ver logs del controlador
kubectl logs -n kube-system -l component=kube-controller-manager
```

---

## Resumen de Comandos Útiles

### Comandos Esenciales

```bash
# Obtener información
kubectl get <resource> [-A] [-o wide|yaml|json]
kubectl describe <resource> <name>
kubectl explain <resource>

# Crear/Actualizar
kubectl apply -f <file.yaml>
kubectl create <resource> --from-literal key=value
kubectl set image deployment/<name> <container>=<image>
kubectl patch <resource> <name> -p '{"spec":{"replicas":3}}'

# Eliminar
kubectl delete <resource> <name>
kubectl delete -f <file.yaml>
kubectl delete <resource> --all

# Debugging
kubectl logs <pod> [-f|--previous]
kubectl exec -it <pod> -- <command>
kubectl describe <resource> <name>
kubectl get events
kubectl top <node|pod>

# Otras operaciones
kubectl scale <resource> <name> --replicas=N
kubectl rollout <status|history|undo> <resource> <name>
kubectl port-forward <resource> <local-port>:<remote-port>
kubectl proxy
```

---

## Checklist de Buenas Prácticas

- ✅ Siempre especificar `resources.requests` y `resources.limits`
- ✅ Implementar `livenessProbe` y `readinessProbe`
- ✅ Usar ConfigMaps para configuración y Secrets para datos sensibles
- ✅ Implementar RBAC y namespaces para aislamiento
- ✅ Usar labels y selectors apropiadamente
- ✅ Implementar NetworkPolicies para seguridad
- ✅ Monitorear con Prometheus y dashboards Grafana
- ✅ Usar StatefulSets para aplicaciones con estado
- ✅ Implementar PodDisruptionBudgets para alta disponibilidad
- ✅ Usar init containers para preparación
- ✅ Implementar affinity rules para distribución de pods
- ✅ Versionar y testear cambios en staging primero
- ✅ Mantener imágenes Docker pequeñas y seguras
- ✅ Usar readinessProbe antes de mandas tráfico
- ✅ Implementar health checks exhaustivos

---

**Fin del Guía Completa de Kubernetes**

*Última actualización: 2026*  
*Para preguntas o actualizaciones, consultar documentación oficial: https://kubernetes.io/docs/*
