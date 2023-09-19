# Creació de repliques

## Opció Manual

Podeu optar per desplegar les rèpliques de forma manual, creant dos nous servidors WordPress amb la mateixa configuració i assegurant la comunicació amb el servidor de base de dades. En resum, heu de fer dues vegades el **HandsOn00**.

## Opció Automàtica

Utilitzant els coneixements adquirits en el **Lab01**, podeu automatitzar la creació de rèpliques amb Ansible. 

**Requeriments previs**: Instal·lació d'ansible [Com instal·lar ansible](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html).




1. Instancieu 2 màquines virtual al servidor Stormy (**0.25 CPU,1GB RAM**). Anomeneu-los **H01-WS-WP1** i **H01-WS-WP2**.
2. Creació del vostre inventari de hosts (**inventory.ini**):

  ```yaml
  [WordPress]
  192.168.101.X ansible_ssh_user=root 
  192.168.101.Y ansible_ssh_user=root 
  ```

  on *X,Y* represente les IPs dels vostres servidors de Wordpress.

3. Creació d'un Playbook d'Ansible (**install_wordpress.yml**):

  ```yaml
  ---
  - name: Install WordPress with preinstalled database
    become: yes
    hosts: WordPress
    vars:
      wp_db_name: wordpress_db
      wp_db_user: wordpress_user
      wp_db_password: password
      wp_db_host: 192.168.101.Z 

    tasks:
      - name: Update DNF cache
        dnf:
          update_cache: yes

      - name: Install PHP 7.4
        command: "dnf module install php:7.4 -y"

      - name: Install necessary packages for WordPress
        dnf:
          name: "{{ item }}"
          state: present
        loop:
          - httpd
          - php-curl
          - php-zip
          - php-gd
          - php-soap
          - php-intl
          - php-mysqlnd
          - php-pdo
          - policycoreutils-python-utils
          - firewalld

      - name: Download and extract WordPress
        get_url:
          url: "https://wordpress.org/latest.tar.gz"
          dest: "/tmp/wordpress.tar.gz"

      - name: Extract WordPress archive
        ansible.builtin.command: "tar -zxf /tmp/wordpress.tar.gz -C /var/www/html/"

      - name: Set SELinux policy for HTTPD
        ansible.builtin.command: "semanage fcontext -a -t \ 
        httpd_sys_rw_content_t '/var/www/html/wordpress(/.*)?'"
        become: yes

      - name: Restore SELinux context for WordPress directory
        ansible.builtin.command: "restorecon -Rv /var/www/html/wordpress"
        become: yes

      - name: Set SELinux boolean httpd_can_network_connect_db to allow DB connections
        ansible.builtin.command: "setsebool -P httpd_can_network_connect_db 1"

      - name: Enable HTTP traffic in firewall
        firewalld:
          zone: public
          service: http
          state: enabled
          permanent: yes
        become: yes

      - name: Configure WordPress wp-config.php
        template:
          src: "wp-config.php.j2"
          dest: "/var/www/html/wordpress/wp-config.php"
        notify:
          - Restart Apache

    handlers:
      - name: Restart Apache
        ansible.builtin.service:
          name: httpd
          state: restarted

  ```

  **Nota**: On *Z* representa la IP del vostre servidor de bases de dades.

  Per tal de flexibilitzar la configuració, utilitzarem un template per configurar el WordPress. Anomeneu aquest fitxer **wp-config.php.j2**:

  ```php
  <?php
  define('DB_NAME', '{{ wp_db_name }}');
  define('DB_USER', '{{ wp_db_user }}');
  define('DB_PASSWORD', '{{ wp_db_password }}');
  define('DB_HOST', '{{ wp_db_host }}');
  define('DB_CHARSET', 'utf8');
  define('DB_COLLATE', '');

  $table_prefix  = 'wp_';

  define('WP_DEBUG', false);

  if ( ! defined('ABSPATH') ) {
      define('ABSPATH', __DIR__ . '/');
  }
  require_once ABSPATH . 'wp-settings.php';
  ?>

  ```

### Execució del Playbook

```sh
ansible-playbook -i inventory.ini install_wordpress.yml
```

Reviseu que s'ha executat tot correctament i que en les dues instàncies teniu un WordPress funcional.