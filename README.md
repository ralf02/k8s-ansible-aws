# Demo Kubernetes + Ansible + AWS

Propuesta de automatización de Instalación de Kubernetes y Docker en Instancias Debian AWS con Ansible.

## Clusters con Kubeadm (k8s)

Este documento describe el proceso completo para instalar un cluster de Kubernetes utilizando kubeadm en un entorno con 3 Instancias AWS.

## Requisitos del Sistema

### Características de las Máquinas Virtuales
- **3 Instancias EC2** corriendo Debian con las siguientes especificaciones:
  - **RAM**: Mínimo 1GB
  - **Almacenamiento**: 20GB de espacio en disco duro
  - **Procesador**: Al menos 1 procesador configurado
- **Acceso SSH** mediante archivo `.pem` (keypair de AWS).
- **1 Instancia EC2 como "Ansible Control Node"** Ansible en la máquina de control.

## Preparación Inicial de Servidores

### Tener listo el AnsibleControlNode
Disponer de las EC2 y una que funcione con ansible a la que denomino **Ansible Control Node** con Debian, esto con el fin de tener lo necesario para las maquinas que harán parte del cluster k8s.

## Creación del Cluster

### Inicializar el Cluster Master

- Ejecutar kubeadm init con red compatible con Calico:
   ```bash
   kubeadm init --pod-network-cidr=19.16.0.0/16
   ```
- Configurar kubectl para usuario (no-root)
   Después de la inicialización, habilita el acceso al clúster mediante kubectl para tu usuario regular:

   ```bash
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
   Esto permite ejecutar comandos kubectl sin necesidad de ser root.

### Instalar Plugin de Red Calico

- Desplegar el operador de Calico (Tigera Operator):
   Ejecutar el siguiente comando para instalar el operador responsable de gestionar los componentes de Calico:
   ```bash
   kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.1/manifests/tigera-operator.yaml
   ```

- Aplicar la configuración personalizada de Calico:
   Esta configuración define los recursos necesarios para implementar Calico como proveedor de red:
   ```bash
   kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.1/manifests/custom-resources.yaml
   ```

- Verificar el estado de los pods de Calico
   Observar en tiempo real que los pods de la red Calico se estén desplegando correctamente
   ```bash
   watch kubectl get pods -n calico-system
   ```
   Validar que todos los pods estén en estado Running o Completed antes de continuar con el resto de la configuración del clúster.

### Configuración Final del Nodo Master

- Permitir la programación de pods en el nodo master (untaint):
   Por defecto, Kubernetes impide agendar pods en nodos con el rol de control plane. Para permitirlo (útil en entornos de desarrollo o clústeres de un solo nodo), ejecuta:
   ```bash
   kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-
   ```

- Verificar el estado del clúster y los nodos:
   Comprobar que el nodo esté en estado `Ready` y que no haya taints activos:
   ```bash
   kubectl get nodes
   ```
   Esto te permitirá validar que el clúster está operativo y listo para ejecutar cargas de trabajo.

## Añadir Nodos Worker al Cluster

### Preparación del Nodo Worker

### Unir un nodo `Worker` al Clúster
- Ejecutar el comando `kubeadm join` en el nodo designado como worker
   Usar el comando que se generó al finalizar kubeadm init en el nodo master:
   ```bash
   kubeadm join 19.16.1.11:6443 --token [TOKEN] --discovery-token-ca-cert-hash sha256:[HASH]
   ```
   > El token y hash deben ser copiados directamente del nodo master o regenerados si ya expiraron:

       Generar nuevo token: kubeadm token create
       Obtener hash: openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | sha256sum

- Verificar que el nodo se ha unido correctamente al clúster
   Desde el nodo master, ejecutar:
   ```bash
   kubectl get nodes
   ```

## Verificación Final del cluster

Una vez completados todos los pasos:
- El nodo master está inicializado y operativo.
- Los nodos worker se han unido correctamente al clúster.
- El plugin de red Calico está desplegado y funcional.
- Todos los nodos aparecen en estado `Ready`

## Validación y Reinicio del Clúster

- Verificar Componentes del Clúster
   ```bash
      kubectl get pods -A
      kubectl get cs
   ```
- Comando para Reset en caso de error
   En caso de que algo falle, puedes resetear un nodo (master o worker) con:
   ```bash
   sudo kubeadm reset -f
   sudo systemctl restart kubelet
   delete ~/.kube/config  # Si aplica
   ```
Luego, se puede volver a ejecutar los pasos de unión o inicialización según el rol del nodo.

---

¡Clúster K8s listo para usar!
