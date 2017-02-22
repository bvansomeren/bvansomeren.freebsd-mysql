Role Name
=========

Installs MySQL (5.7 by default) on FreeBSD.

Current functionality:  
* Install MySQL 5.7 or 5.6 from PKG (or other PKG version by overriding _freebsd\_mysql\_packages_  
* Place InnoDB data and logs in other locations (for example ZFS datasets)  
* Setup my.cnf  
* Changes the random MySQL root password to the value of _freebsd\_mysql\_rootpw_  
* Sets auto login for selected users by writing a .my.cnf in their home.  
* Installs Python-mysql using pip (to avoid dependency problems)  
* Creates databases from _freebsd\_mysql\_databases_  
* Creates users from _freebsd\_mysql\_users_  


Requirements
------------

Tested on FreeBSD 11 with MySQL 5.7  
Tested inside jails

Role Variables
--------------

```
freebsd_mysql_service: "mysql-server"
```

The default name of the service to enable

```
freebsd_mysql_version: "57"
freebsd_mysql_vendor: mysql
freebsd_mysql_packages:
- "{{ freebsd_mysql_vendor }}{{ freebsd_mysql_version }}-server"
- "{{ freebsd_mysql_vendor }}{{ freebsd_mysql_version }}-client"
- "{{ freebsd_mysql_extra_utilities }}"
```

Controls the packages that will be installed. Change _freebsd\_mysql\_vendor_ to percona for the Percona version of MySQL.

```
freebsd_mysql_extra_utilities:
- mysqltuner
```

Feel free to add your utilities (from PKG)  

```
_freebsd_mysql_adminusers:
- /root
- "{{ freebsd_mysql_adminusers }}"
```
These users will have automatic logins through ~/.my.cnf enabled (as root!)

```
freebsd_mysql_adminusers: []
```

Add the home folders of the users you want to have automatic root login to MySQL

```
freebsd_mysql_changerootpw: no
```
This is ignored if the .mysql_secret file is set on initial install


```
freebsd_mysql_innodb_psize: 2G
freebsd_mysql_key_buffer: 256M
freebsd_mysql_innodb_logsize: 128M
freebsd_mysql_innodb_read_io_threads: "{{ ansible_processor_count }}"
freebsd_mysql_innodb_write_io_threads: "{{ ansible_processor_count }}"
```

These are performance settings, set them to your own requirements.  
TODO: Add more.

```
freebsd_mysql_innodb_data_dir: /var/db/mysql_innodb/data
```
The InnoDB data folder where table data and indices are stored. For performance select a blocksize equal to that of InnoDB (16Kb)

```
freebsd_mysql_innodb_log_dir: /var/db/mysql_innodb/log
```

The InnoDB log dir. 128Kb works best here for blocksize

```
freebsd_mysql_restart_on_change: yes
```

Automatically restart MySQL when making changes. Setting this to no prevents the Ansible handlers from restarting MySQL after runs.  

```
freebsd_mysql_databases: []
```

A list of databases to create minimum is a name (defaults to encoding: utf8 and collation: utf8_general_ci)  
Example:  

```
freebsd_mysql_databases:
- name: db1
- name: db2
- name: latin_db
  encoding: latin1
  collation: latin1_general_ci
```

```
freebsd_mysql_users: []
```
A list of database users and their privileges. Mirrors the mysql_user module in Ansible.  

Example:  

```
freebsd_mysql_users:
    - name: example_user
      host: "%"
      password: DatSecurityDoh
      priv: "db1.*:ALL/latin_db.*:ALL"
```



Dependencies
------------

This role installs the Python-mysql module for Ansible on the machines it touches. You can disable this by setting:  
 
freebsd_mysql_install_mysqldb: no

Example Playbook
----------------

Example 1 (default latest MySQL)

    - hosts: mysql
  	   user: support
  	   become: yes
  	   become_user: root
  	   vars:
        freebsd_mysql_rootpw: SomeLamePassword
        freebsd_mysql_adminusers:
    	  - "/home/support"
    	  freebsd_mysql_databases:
         - name: db1
         - name: db2
         - name: latin_db
           encoding: latin1
           collation: latin1_general_ci
    	  freebsd_mysql_users:
    	  - name: example_user
    	    host: "%"
    	    password: 0000
    	    priv: "db1.*:ALL/latin_db.*:ALL"
      roles:
      - bvansomeren.freebsd-mysql

Example 2 (change the vendor for the latest Percona MySQL)

    - hosts: percona
  	   user: coffeesprout
  	   become: yes
  	   become_user: root
  	   vars:
    	  freebsd_mysql_rootpw: SomeLamePassword
    	  freebsd_mysql_vendor: percona
    	  freebsd_mysql_adminusers:
    	  - "/home/coffeesprout"
  	   roles:
  	   - bvansomeren.freebsd-mysql
      

License
-------

BSD

Author Information
------------------

