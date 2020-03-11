# Role Ansible Master

## Caractéristiques

* Installation et configuration d'un serveur Mariadb en standalone ou en cluster
* Architecture : X86_64
* Distribution : CENTOS7

## Configuration

cf : <http://docs.ansible.com/ansible/playbooks_variables.html>

## Variables

### Variables de playbook

* `mariadb_root_password`:
  * mot de passe root pour se connecter Ã  Mariadb
  * Variable obligatoire
  
* `mariadb_databases`  (cf [documentation module ansible mysql_db](https://docs.ansible.com/ansible/latest/modules/mysql_db_module.html)):
  * Liste des bases de données Ã  créer
  * Variable non obligatoire
  * Stucture de la liste:

``` yaml
mariadb_databases:
  - name: db1
    encoding: <optionel, valeur par défaut `latin1`>
    collation: <optionel, valeur par défaut `latin1_swedish_ci`>
    state: <optionel, valeur par défaut `present`>
```

* `mariadb_users`:
  * Liste des utilisateurs Ã crée
  * Variable non obligatoire
  * Stucture de la liste (cf [documentation module ansible mysql_user](https://docs.ansible.com/ansible/latest/modules/mysql_user_module.html)):

``` yaml
mariadb_users:
  - name: test1
    password: test1
    priv: "test1:ALL,GRANT"
    host: *optionel, valeur par défaut `localhost`*
    state: *optionel, valeur par défaut `present`*
```

* `mariadb_root_password`:
  * mot de passe root pour se connecter Ã  Mariadb
  * Variable obligatoire

* `mariadb_server_package`:
  * Nom du package pour le serveur mariadb
  * Variable obligatoire

``` yaml
mariadb_server_package: "mariadb-server"
```

### Variables fixe du rÃ´le (non-surchargeables)

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

### Variables par defaut du rÃ´le (surchargeables)

#### Variables communes

* `is_not_openstack`:
  * Défini si l'on est sur openstack ou non
  * Positionne le firewalld en fonction

``` yaml
is_not_openstack: false
```

* `mode`:
  * Défini le mode d'installation de mariadb:
    * `Auto` (par défaut): utilisation du mode automatique, le mode d'installation va se baser sur le nombre d'item dans `mariadb_inventory_group`
    * `Standalone`: Installation en mode standalone sur chaque serveur
    * `Cluster`: Installation en mode cluster sur l'ensemble des serveurs. La variable `mariadb_inventory_group`est obligatoire pour la configuration du mode cluster dans les fichiers de config (server.cnf)

``` yaml
mode: Auto
```

* `mariadb_packages`:
  * Liste des paquets yum Ã  installer
  * Sera concaténé Ã  la liste suivante:

``` yaml
  - MySQL-python
  - rsync
  - xinetd
  - policycoreutils-python
```

* `mariadb_datadir`:
  * Répertoire de data de Mariadb
  * Valeur par défaut:

``` yaml
mariadb_datadir: /var/lib/mysql
```

* `mariadb_logdir`:
  * Répertoire de log de Mariadb
  * Valeur par défaut:

``` yaml
mariadb_logdir: /var/log/mysql
```

* `mariadb_inventory_group`:
  * Nom du groupe de serveurs dans l'inventaire
  * Cette variable sert Ã  déterminer le mode d'installation:
    * `standalone` si le groupe ne contient qu'un serveur
    * `cluster` si il en contient plus de un
  * Valeur par défaut:

``` yaml
mariadb_inventory_group: mariadb_nodes
```

* `mariadb_service_name`:
  * Nom du service
  * Valeur par défaut:

``` yaml
mariadb_service_name: mariadb
```

* `mariadb_bind_address`:
  * Adresse sur laquelle le serveur mariadb doit écouter
  * Valeur par défaut:

``` yaml
mariadb_bind_address: 0.0.0.0
```

* `mariadb_socket`:
  * Emplacement du fichier socket de mariaDB
  * Valeur par défaut:

``` yaml
mariadb_socket: "{{ mariadb_datadir }}/mysql.sock"
```

* `mariadb_ports`:
  * Port d'écoute pour mariadb. Défini également le port Ã  ouvrir sur firewalld.
  * Valeur par défaut:

``` yaml
mariadb_ports:
  - port: 3306
    proto: tcp
```

* `host_RAM`:
  * RAM disponible sur le serveur. Récupérée automatiquement par anible mais peut Ãªtre positionnée Ã  une valeur inférieure si le serveur n'est pas dédié.
  * Valeur par défaut:

``` yaml
host_RAM: "{{ ansible_memory_mb.real.total }}"
```

* `mariadb_type`:
  * Moteur utilisé par la base de données.
  * Valeurs possibles :
    * InnoDB
    * MyISAM
  * Valeur par défaut:

``` yaml
mariadb_type: InnoDB
```

* `mariadb_InnoDB`:
  * Variable utilisé si le moteur de base de données est InnoDB.
  * Permet de positionner les variables dans my.cnf:
    * key_buffer_size: "the size of the buffer used for index blocks". Recommandé pour une base InnoDB : 10M
    * innodb_buffer_pool_size:"contrÃ´le la taille du cache mémoire pour les données et les index"
    Recommandé pour une base InnoDB : 70% de la RAM du serveur (basé sur la variable `host_RAM` )
  * Valeur par défaut:

``` yaml
mariadb_InnoDB:
  SGA: "{{ host_RAM *0.7|round|int }}M"
  key_buff: 10M
```

* `mariadb_MyISAM`:
  * Variable utilisé si le moteur de base de données est MyISAM.
  * Permet de positionner les variables dans my.cnf:
    * key_buffer_size: "the size of the buffer used for index blocks". Recommandé pour une base MyISAM : 20% de la RAM du serveur (bassé sur la variable `host_RAM` )
    * innodb_buffer_pool_size:"contrÃ´le la taille du cache mémoire pour les données et les index"
    Recommandé pour une base MyISAM : 0M
  * Valeur par défaut:

``` yaml
mariadb_MyISAM:
  key_buff: "{{ host_RAM *0.2|round|int }}M"
  SGA: 0M
```

#### Variables en mode cluster Galera

* `mariadb_galera_cluster_name`:
  * Nom du cluster Galera
  * Valeur par défaut:

``` yaml
mariadb_galera_cluster_name: cluster_name
```

* `mariadb_ports_cluster`:
  * Liste des ports Ã  ouvrir sur firewalld
  * Cette variable ajoute également la valeur `mariadb_ports` en cas de cluster aux valeur par défaut. En cas de surcharge, penser Ã  ajout le port d'écoute de MariaDB (tcp/3306 par défaut).
  * Valeur par défaut:

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
  
## Divers

Par défaut, le repo utilisé pour télécharger MariaDB et ses pré-requis n'est pas présent sur Centos7. Il faut donc l'ajouter avant d'exécuter ce role.

Donner un exemple de fichier de conf
