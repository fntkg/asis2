# kubernetes 2/3

## Resumen

En esta 2a parte del trabajo se ha comprendido como poner en marcha un clúster de almacenamiento `CEPH` y un gestor para este, `ROOK`.

Después se estudiaron los diferentes archivos `.yaml` que se encargaban de crear un `RBD` ("RADOS Block Devices"), es decir, un almacen de dispositivos de bloques.

Tras esto se crearon unos `PersistentVolumeClaim` y sus respectives volumenes para después lanzar una bbdd MySql y un servidor web Wordpress.

También se realizaron las pruebas indicadas en el guión respecto al `RBD`.

## Arquitectura de elementos relevantes del despliegue Kubernetes/Ceph/Aplicaciones.

Para este trabajo se han desplegado 3 nodos `worker` ya que es el número mínimo de máquinas que se requieren para poder desplegar correctamente `rook-ceph`. Por supuesto, también se tiene un nodo `master`.

Los 3 nodos necesarios es debido a que, según la documentación oficial de `Rook`, se requieren 3 `OSD` corriendo en nodos diferentes por la siguiente especificación: `replicated.size: 3`.

Se han desplegado también 2 servicios `mysql-pv-claim` y `wp-pv-claim` para hacer uso de `rook-ceph`.

## Explicación de las diferentes aplicaciones desplegadas sobre Kubernetes con la explicación de los conceptos y  los recursos Kubernetes utilizados y explicación básica de los conceptos y recursos utilizados porCeph.

**`common.yaml`**

Sse encarga de crear los recursos necesarios para, posteriormente, desplegar `ceph` y el operador `rook`. Esto lo realiza creando `CustomResourceDefinition` es decir, recursos personalizados para la situación.

También crea el espacio de nombres `rook-ceph`.

**`operator.yaml`**

Se encarga de poner en marcha `rook`.

```ruby
apiVersion: v1
kind: Namespace
metadata:
  name: rook-ceph
```

Se ha estudiado la documentación oficial y se ha visto la utilidad de los recursos `ConfigMap`: "A ConfigMap is an API object used to store non-confidential data in key-value pairs. A ConfigMap allows you to decouple environment-specific configuration from your container images , so that your applications are easily portable.".

**`cluster.yaml`**

Se encarga de desplegar ceph en los nodos

```ruby
dataDirHostPath: /var/lib/rook # Establece donde se almacenarán los ficheros de configuración de ceph

mon: # Establece el numero de monitores, en este caso uno por nodo
    count: 3
    allowMultiplePerNode: false

storage: # cluster level storage configuration and selection
  useAllNodes: true
  useAllDevices: true
```

**`storageclassRbdBlock.yaml`**

Se crea un recurso `CephBlockPool`, es uno de los `CustomResourceDefinition` definidos anteriormente. Se establecen 3 réplicas.

Se crea un recurso `StorageClass` y se establecen los secretos que contienen las credenciales del administrador y la política del recurso `Delete`. También se establece el pool en el que se va a encontrar esta unidad de almacenamiento `replicapool`.

**`mysql.yaml`**

Se ha usado `mysql-persistent-storage` como volumen para `wordpress-mysql`.

**`wordpress.yaml`**

Se ha establecido en el `PersistentVolumeClaim` lo siguiente: `storageClassName: rook-ceph-block`

## Explicación de los métodos utilizados para la validación de la operativa.

Cada elemento que se creaba en kubernetes, no se pasaba al siguiente paso hasta que todos los `pods` del recurso estuvieran en el estado `Running`.

Para comprobar el despliegue de `rook` y `ceph` se siguieron los pasos indicados en el enunciado del trabajo y se comprobaron que estuvieran los 3 monitores, el gestor y los 3 demonios en funcionamiento:

```bash
$ ceph status
  ...
  services:
    mon: 3 daemons, quorum a,b,c (age 24m)
    mgr: a(active, since 23m)
    osd: 3 osds: 3 up (since 22m), 3 in (since 22m)
  ...
```

Para comprobar el estado de `wordpress` simplemente se usó un explorador en la máquina host y se accedió a esta.

## Problemas encontrados y su solución.

Se creó un script para automatizar el arranque de todo pero los recursos no se desplegaban de manera correcta debido a la falta de tiempo entre instrucciones. Por falta de tiempo se decidió hacer dicho despliegue de manera manual.