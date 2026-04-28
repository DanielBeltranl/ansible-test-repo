# 🧪 Ansible Test Playbooks

Repositorio de **playbooks de prueba** para validar roles de Ansible en entornos de laboratorio aislados (sin salida a internet). Cada playbook está diseñado para verificar el correcto funcionamiento de un rol específico en condiciones controladas.

---

## 📁 Estructura del repositorio

```
.
├── tm-test-playbook.yml       # Playbook de prueba básica (rol: prueba)
├── deploy-nexus-proxy.yml     # Playbook de despliegue vía Nexus proxy (rol: setup-nexus-proxy)
└── roles/
    ├── prueba/                # Rol de auditoría y configuración base
    └── setup-nexus-proxy/     # Rol de instalación de paquetes desde Nexus Hosted
```

---

## 📋 Playbooks disponibles

### 1. `tm-test-playbook.yml` — Rol: `prueba`

Playbook de prueba básica que ejecuta el rol `prueba`. Este rol realiza las siguientes acciones en el nodo objetivo:

- **Imprime un mensaje de depuración** en el motor de Ansible para confirmar que el provisionamiento offline ha iniciado.
- **Crea un archivo de auditoría** en `/tmp/auditoria_devops.txt` con permisos `0644` y propietario `root`.
- **Inyecta un MOTD (Message of the Day)** en `/etc/motd` identificando el servidor como parte del entorno de laboratorio gestionado por Ansible.

#### Comando de ejecución

```bash
ansible-playbook tm-test-playbook.yml -i inventario/hosts.ini
```

Con usuario y contraseña (si no usas llaves SSH):

```bash
ansible-playbook tm-test-playbook.yml -i inventario/hosts.ini -u usuario --ask-pass --ask-become-pass
```

Con modo verbose para depuración:

```bash
ansible-playbook tm-test-playbook.yml -i inventario/hosts.ini -v
```

---

### 2. `deploy-nexus-proxy.yml` — Rol: `setup-nexus-proxy`

Playbook de despliegue que ejecuta el rol `setup-nexus-proxy`. Está orientado a entornos **sin acceso a internet**, donde los paquetes se obtienen desde un repositorio **Nexus Hosted** interno. Este rol realiza:

- **Descarga el binario** `net-tools` desde el repositorio raw de Nexus (`nexus-lab:8081`) usando credenciales parametrizadas.
- **Instala el paquete `.deb`** directamente con `dpkg -i`, evitando que Ansible intente resolver repositorios externos o validaciones de APT.

> ⚠️ **Nota:** Este playbook requiere las variables `nexus_user` y `nexus_pass`. Se recomienda gestionarlas con **Ansible Vault**.

#### Comando de ejecución

```bash
ansible-playbook deploy-nexus-proxy.yml -i inventario/hosts.ini
```

Pasando variables directamente (solo para pruebas, no recomendado en producción):

```bash
ansible-playbook deploy-nexus-proxy.yml -i inventario/hosts.ini \
  -e "nexus_user=admin nexus_pass=secreto123"
```

Usando un archivo de variables cifrado con Vault:

```bash
ansible-playbook deploy-nexus-proxy.yml -i inventario/hosts.ini \
  -e @vars/nexus_creds.yml --ask-vault-pass
```

---

## ⚙️ Requisitos

| Requisito         | Detalle                              |
|-------------------|--------------------------------------|
| Ansible           | >= 2.10                              |
| Acceso SSH        | Al nodo objetivo                     |
| Privilegios       | `become: true` (sudo)                |
| Nexus Hosted      | Disponible en `nexus-lab:8081` (solo para `deploy-nexus-proxy.yml`) |

---

## 🔐 Variables requeridas

Para el playbook `deploy-nexus-proxy.yml`:

| Variable      | Descripción                         |
|---------------|-------------------------------------|
| `nexus_user`  | Usuario de autenticación en Nexus   |
| `nexus_pass`  | Contraseña de autenticación en Nexus|

Se recomienda almacenarlas en un archivo cifrado con Ansible Vault:

```bash
ansible-vault create vars/nexus_creds.yml
```

---

## 📝 Notas

- Todos los playbooks están diseñados para ejecutarse en **entornos de laboratorio aislados**, sin salida a internet.
- El uso de `command: dpkg -i` en lugar del módulo `apt` es intencional: evita que Ansible intente validar repositorios externos o resolver dependencias en red.
- Los archivos generados por el rol `prueba` (`/tmp/auditoria_devops.txt`, `/etc/motd`) son de carácter temporal y de prueba.

---

*Autor: **Daniel Beltran Lastra***