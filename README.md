# Role Ansible Master

## Caract�ristiques

* Installation et configuration d'un serveur Mariadb en standalone ou en cluster
* Architecture : X86_64
* Distribution : CENTOS7

## Configuration

cf : <http://docs.ansible.com/ansible/playbooks_variables.html>

## Variables

### Variables de playbook

* `mariadb_root_password`:
  * mot de passe root pour se connecter à Mariadb
  * Variable obligatoire
  
* `mariadb_databases`  (cf [documentation module ansible mysql_db](https://docs.ansible.com/ansible/latest/modules/mysql_db_module.html)):
  * Liste des bases de donn�es à cr�er
  * Variable non obligatoire
  * Stucture de la liste:

``` yaml
mariadb_databases:
  - name: db1
    encoding: <optionel, valeur par d�faut `latin1`>
    collation: <optionel, valeur par d�faut `latin1_swedish_ci`>
    state: <optionel, valeur par d�faut `present`>
```

* `mariadb_users`:
  * Liste des utilisateurs àcr�e
  * Variable non obligatoire
  * Stucture de la liste (cf [documentation module ansible mysql_user](https://docs.ansible.com/ansible/latest/modules/mysql_user_module.html)):

``` yaml
mariadb_users:
  - name: test1
    password: test1
    priv: "test1:ALL,GRANT"
    host: *optionel, valeur par d�faut `localhost`*
    state: *optionel, valeur par d�faut `present`*
```

* `mariadb_root_password`:
  * mot de passe root pour se connecter à Mariadb
  * Variable obligatoire

* `mariadb_server_package`:
  * Nom du package pour le serveur mariadb
  * Variable obligatoire

``` yaml
mariadb_server_package: "mariadb-server"
```

### Variables fixe du rôle (non-surchargeables)

* `mariadb_root_login`:
  * Nom du user "root" de mariaDB:

``` yaml
mariadb_root_login: root
```

* `mariadb_pid_dir`:
  * Emplacement du fichier pid de mariaDB

``` yaml
mariadb_pid_dir: /var/run/mysqld
```

### Variables par defaut du rôle (surchargeables)

#### Variables communes

* `is_not_openstack`:
  * D�fini si l'on est sur openstack ou non
  * Positionne le firewalld en fonction

``` yaml
is_not_openstack: false
```

* `mode`:
  * D�fini le mode d'installation de mariadb:
    * `Auto` (par d�faut): utilisation du mode automatique, le mode d'installation va se baser sur le nombre d'item dans `mariadb_inventory_group`
    * `Standalone`: Installation en mode standalone sur chaque serveur
    * `Cluster`: Installation en mode cluster sur l'ensemble des serveurs. La variable `mariadb_inventory_group`est obligatoire pour la configuration du mode cluster dans les fichiers de config (server.cnf)

``` yaml
mode: Auto
```

* `mariadb_packages`:
  * Liste des paquets yum à installer
  * Sera concat�n� à la liste suivante:

``` yaml
  - MySQL-python
  - rsync
  - xinetd
  - policycoreutils-python
```

* `mariadb_datadir`:
  * R�pertoire de data de Mariadb
  * Valeur par d�faut:

``` yaml
mariadb_datadir: /var/lib/mysql
```

* `mariadb_logdir`:
  * R�pertoire de log de Mariadb
  * Valeur par d�faut:

``` yaml
mariadb_logdir: /var/log/mysql
```

* `mariadb_inventory_group`:
  * Nom du groupe de serveurs dans l'inventaire
  * Cette variable sert à d�terminer le mode d'installation:
    * `standalone` si le groupe ne contient qu'un serveur
    * `cluster` si il en contient plus de un
  * Valeur par d�faut:

``` yaml
mariadb_inventory_group: mariadb_nodes
```

* `mariadb_service_name`:
  * Nom du service
  * Valeur par d�faut:

``` yaml
mariadb_service_name: mariadb
```

* `mariadb_bind_address`:
  * Adresse sur laquelle le serveur mariadb doit �couter
  * Valeur par d�faut:

``` yaml
mariadb_bind_address: 0.0.0.0
```

* `mariadb_socket`:
  * Emplacement du fichier socket de mariaDB
  * Valeur par d�faut:

``` yaml
mariadb_socket: "{{ mariadb_datadir }}/mysql.sock"
```

* `mariadb_ports`:
  * Port d'�coute pour mariadb. D�fini �galement le port à ouvrir sur firewalld.
  * Valeur par d�faut:

``` yaml
mariadb_ports:
  - port: 3306
    proto: tcp
```

* `host_RAM`:
  * RAM disponible sur le serveur. R�cup�r�e automatiquement par anible mais peut être positionn�e à une valeur inf�rieure si le serveur n'est pas d�di�.
  * Valeur par d�faut:

``` yaml
host_RAM: "{{ ansible_memory_mb.real.total }}"
```

* `mariadb_type`:
  * Moteur utilis� par la base de donn�es.
  * Valeurs possibles :
    * InnoDB
    * MyISAM
  * Valeur par d�faut:

``` yaml
mariadb_type: InnoDB
```

* `mariadb_InnoDB`:
  * Variable utilis� si le moteur de base de donn�es est InnoDB.
  * Permet de positionner les variables dans my.cnf:
    * key_buffer_size: "the size of the buffer used for index blocks". Recommand� pour une base InnoDB : 10M
    * innodb_buffer_pool_size:"contrôle la taille du cache m�moire pour les donn�es et les index"
    Recommand� pour une base InnoDB : 70% de la RAM du serveur (bas� sur la variable `host_RAM` )
  * Valeur par d�faut:

``` yaml
mariadb_InnoDB:
  SGA: "{{ host_RAM *0.7|round|int }}M"
  key_buff: 10M
```

* `mariadb_MyISAM`:
  * Variable utilis� si le moteur de base de donn�es est MyISAM.
  * Permet de positionner les variables dans my.cnf:
    * key_buffer_size: "the size of the buffer used for index blocks". Recommand� pour une base MyISAM : 20% de la RAM du serveur (bass� sur la variable `host_RAM` )
    * innodb_buffer_pool_size:"contrôle la taille du cache m�moire pour les donn�es et les index"
    Recommand� pour une base MyISAM : 0M
  * Valeur par d�faut:

``` yaml
mariadb_MyISAM:
  key_buff: "{{ host_RAM *0.2|round|int }}M"
  SGA: 0M
```

#### Variables en mode cluster Galera

* `mariadb_galera_cluster_name`:
  * Nom du cluster Galera
  * Valeur par d�faut:

``` yaml
mariadb_galera_cluster_name: cluster_name
```

* `mariadb_ports_cluster`:
  * Liste des ports à ouvrir sur firewalld
  * Cette variable ajoute �galement la valeur `mariadb_ports` en cas de cluster aux valeur par d�faut. En cas de surcharge, penser à ajout le port d'�coute de MariaDB (tcp/3306 par d�faut).
  * Valeur par d�faut:

``` yaml
mariadb_ports_cluster:
  - port: "{{ mariadb_ports.port }}"
    proto: "{{ mariadb_ports.proto }}"
  - port: 4567
    proto: udp
  - port: 4567
    proto: tcp
  - port: 4444
    proto: tcp
```

## Exemple d'utilisation

### Playbook

```yaml
---
- hosts: mariadb_nodes
  become: false
  any_errors_fatal: true
  vars_files:
    - "{{ playbook_dir }}/vars/vars.yml"
    - "{{ playbook_dir }}/vars/secrets.yml"
  roles:
    - { role: "mariadb" }
```
  
### Requirements

 Lignes à placer dans le fichier requirements.yml du projet apr�s avoir modifi� la version.

```yaml
- src: git@sources.devtools.local:workbench/forge/ansible/mariadb.git
  scm: git
  version: v1.0.0
```

## Divers

Par d�faut, le repo utilis� pour t�l�charger MariaDB et ses pr�-requis n'est pas pr�sent sur Centos7. Il faut donc l'ajouter avant d'ex�cuter ce role.

Donner un exemple de fichier de conf
