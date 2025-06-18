# Ansible EC2 Basic
Este proyecto permite utilizar Ansible desde una máquina controladora para automatizar la instalación base en instancias Debian (hosts gestionados) en AWS EC2, orientada a entornos de desarrollo y pruebas personales. Incluye la configuración de paquetes esenciales como Docker y GitLab Runner.

---

## Requisitos previos

- Instancias EC2 corriendo Debian.
- Acceso SSH mediante archivo `.pem` (keypair de AWS).
- Ansible instalado en la máquina desiganada como master.

---

### Instalacion Ansible

En una máquina con Debian:

```bash
sudo apt update && sudo apt install ansible -y

```

### Estructura
```
	.
	├── inventory.ini # Define los nodos gestionados sobre los que Ansible va a ejecutar tareas.
	├── setup-k8s.yml # Playbook las tareas que ansible usa para automatizar la instalación de paquetes en instacias EC2.
	└── README.md # Documentacion
```

### Ejecutación del playbook

```bash
ansible-playbook -i inventory.ini basic-setup.yml

```

---

¡Ansible listo para automatizar!

![Evidencia Ansible](../img/evidence-ansible-basic.png)
