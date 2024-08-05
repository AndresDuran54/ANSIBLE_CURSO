# Variables

En Ansible, las variables son fundamentales para hacer tus playbooks y roles más dinámicos y reutilizables. Permiten almacenar valores que pueden cambiar según el entorno o el contexto donde se ejecuten, como nombres de hosts, direcciones IP, credenciales, y más.

Aquí te explico cómo trabajar con variables en Ansible, incluyendo ejemplos y mejores prácticas:

### Tipos de Variables en Ansible

1. **Variables Definidas en Inventario**
   - Puedes definir variables específicas para cada host o grupo en tus archivos de inventario.
   
   **Ejemplo:**
   
   ```ini
   [webservers]
   server1 ansible_host=192.168.1.10 http_port=80
   server2 ansible_host=192.168.1.11 http_port=8080
   ```

2. **Variables en Playbooks**
   - Se pueden definir variables directamente en un playbook bajo la clave `vars`.
   
   **Ejemplo:**
   
   ```yaml
   - name: Configurar servidores web
     hosts: webservers
     vars:
       http_port: 80
       max_clients: 200
     tasks:
       - name: Asegurar que Apache esté instalado
         apt:
           name: apache2
           state: present
       - name: Configurar Apache
         template:
           src: templates/apache.conf.j2
           dest: /etc/apache2/sites-available/000-default.conf
   ```

3. **Variables en Archivos de Variables**
   - Puedes almacenar variables en archivos separados y cargarlos en tus playbooks.
   
   **Ejemplo:**
   
   Archivo `vars/main.yml`:
   
   ```yaml
   http_port: 80
   max_clients: 200
   ```
   
   Playbook:
   
   ```yaml
   - name: Configurar servidores web
     hosts: webservers
     vars_files:
       - vars/main.yml
     tasks:
       - name: Asegurar que Apache esté instalado
         apt:
           name: apache2
           state: present
   ```

4. **Variables de Roles**
   - Los roles pueden tener sus propias variables definidas en directorios específicos, como `defaults` y `vars`.
   - `defaults` contiene variables por defecto que pueden ser sobrescritas.
   - `vars` contiene variables que no pueden ser sobrescritas fácilmente.

5. **Variables de Línea de Comando**
   - Puedes pasar variables en la línea de comandos al ejecutar Ansible usando la opción `-e`.

   **Ejemplo:**

   ```bash
   ansible-playbook site.yml -e "http_port=8080 max_clients=500"
   ```

6. **Variables de Hechos (Facts)**
   - Ansible recopila automáticamente datos sobre los hosts gestionados (hechos) que puedes usar como variables.
   
   **Ejemplo:**

   ```yaml
   - name: Mostrar el sistema operativo del host
     debug:
       msg: "El sistema operativo es {{ ansible_distribution }}"
   ```

### Ámbitos de Variables

Ansible aplica un orden de precedencia a las variables, desde el menos prioritario al más prioritario:

1. Variables de línea de comandos (`-e`).
2. Variables definidas en la tarea (`vars` en la tarea).
3. Variables de inventario del host/grupo.
4. Variables definidas en el archivo `host_vars` o `group_vars`.
5. Variables de hechos (recopiladas automáticamente por Ansible).
6. Variables definidas en roles, en el archivo `vars/main.yml`.
7. Variables definidas en roles, en el archivo `defaults/main.yml`.

### Uso de Variables en Templates

Las variables son especialmente útiles cuando se utilizan plantillas Jinja2 para generar archivos de configuración.

**Ejemplo de Template (apache.conf.j2):**

```jinja
<VirtualHost *:{{ http_port }}>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Mejores Prácticas

- **Nombrado Claro:** Usa nombres de variables claros y descriptivos. Evita nombres ambiguos.
- **Agrupación Lógica:** Agrupa variables en archivos separados según su contexto o función.
- **Valores Por Defecto:** Usa `defaults/main.yml` en roles para establecer valores por defecto.
- **Documentación:** Comenta tus variables para explicar su propósito y posibles valores.

### Ejemplo Completo de Playbook con Variables

Aquí tienes un ejemplo de un playbook que utiliza varias formas de definir y usar variables:

```yaml
---
- name: Desplegar aplicación web
  hosts: webservers
  vars:
    app_name: "mi_aplicacion"
    version: "1.0.0"
  vars_files:
    - "vars/{{ inventory_hostname }}.yml"
  tasks:
    - name: Asegurar que el servidor web esté instalado
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - apache2
        - libapache2-mod-wsgi

    - name: Desplegar aplicación
      git:
        repo: "https://github.com/mi_organizacion/{{ app_name }}"
        version: "{{ version }}"
        dest: "/var/www/{{ app_name }}"

    - name: Configurar Apache
      template:
        src: templates/apache.conf.j2
        dest: /etc/apache2/sites-available/{{ app_name }}.conf
      notify:
        - Reiniciar Apache

  handlers:
    - name: Reiniciar Apache
      service:
        name: apache2
        state: restarted
```

### Conclusión

Las variables en Ansible son esenciales para construir playbooks flexibles y escalables. Te permiten manejar configuraciones complejas de una manera organizada y eficiente. Usando variables correctamente, puedes asegurarte de que tus despliegues sean consistentes y adaptables a diferentes entornos.

Si tienes más preguntas o necesitas ayuda específica sobre un caso de uso en tu proyecto, ¡no dudes en preguntar!