# Step 1. Setup the hosts inventory

> Edit file _hosts.yml_

Add the hosts to the group they belong to, like the following example

```yaml
all:
  children:
    database: # oracle database
      hosts: # including nbs and nbs_common schemas and only the master can be specified if the cluster is configured
        db.local: # configure ssh connection
          # connection type to the host
          ansible_connection: ssh
          # the name of the host to connect to, if different from the alias you wish to give to it.
          ansible_host: db.local
          # the user name to use when connecting to the host
          ansible_user: root
          # the password to use to authenticate to the host
          ansible_password: root
          # allows to force privilege escalation
          ansible_become: true
          # allows to set privilege escalation method
          ansible_become_method: su
          # allows to set the user you become through privilege escalation
          ansible_become_user: oracle

    bo: # sap bo server
      children: # can define one or more bo clusters here
        bo_cluster_a: # bo cluster alias
          hosts: # only one bo master node needs to be defined in this cluster
            bo.cluster_a.local: # configure ssh connection
              ansible_connection: ssh

        ### double bo cluster configuration ###
        #bo_cluster_a:
        #  hosts:
        #    bo.cluster_a.local:
        #      ansible_connection: ssh
        #bo_cluster_b:
        #  hosts:
        #    bo.cluster_b.local:
        #      ansible_connection: ssh

    webapp: # ibsps webapp server
      hosts: # can define one or more webapp node here
        app.local: # configure ssh connection
          ansible_connection: ssh

    scheduler: # ibsps schedule server
      children:
        scheduler_master: # schedule master
          hosts: # all members of the schedule master need to be added here
            sch.master.local: # configure ssh connection
              ansible_connection: ssh

        scheduler_slave: # schedule salve
          hosts: # all members of the schedule salve need to be added here
            sch.slave.local: # configure ssh connection
              ansible_connection: ssh

    executor: # ibsps execution server
      children: # can define one or more executor clusters here
        executor_cluster_a: # executor cluster alias
          hosts: # all members of this executor cluster need to be added here
            exe.cluster_a.local: # configure ssh connection
              ansible_connection: ssh
              
        ### double execution cluster configuration ###
        #executor_cluster_a:
        #  hosts:
        #    exe.cluster_a.local:
        #      ansible_connection: ssh
        #executor_cluster_b:
        #  hosts:
        #    exe.cluster_b.local:
        #      ansible_connection: ssh
```

Here are a few scenarios of configuring SSH connections, for more details, check the ansible [inventory documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) and [connection plugins documentation](https://docs.ansible.com/ansible/latest/plugins/connection.html) .

Scenario 1:	Log in with password

    ansible_connection: ssh
    ansible_host: db.local 
    ansible_user: root 
    ansible_password: root

Scenario 2:	Log in with private key

    ansible_connection: ssh
    ansible_host: db.local
    ansible_user: root 
    ansible_ssh_private_key_file: ¡°private-key-pair.pem¡±

Scenario 3:	Log in with password and need to sudo user

    ansible_connection: ssh
    ansible_host: db.local 
    ansible_user: ec2-user 
    ansible_password: ec2-user 
    ansible_become: true 
    ansible_become_method: sudo 
    ansible_become_user: oracle 
    ansible_become_flags: ¡°-i¡±

Scenario 4:	 Log in with private key and need to sudo user

    ansible_connection: ssh
    ansible_host: db.local 
    ansible_user: ec2-user
    ansible_ssh_private_key_file: ¡°private-key-pair.pem¡± 
    ansible_become: true 
    ansible_become_method: sudo 
    ansible_become_user: oracle 
    ansible_become_flags: ¡°-i¡±

Note, please execute the following command to check if the SSH connection parameters are configured correctly

    # ansible -i hosts.yml all -m ping
    
If everything is correct, you should see output for each host in your inventory, similar to this:

    db.local | SUCCESS => {"ansible_facts": {    "discovered_interpreter_python": "/usr/bin/python"},"changed": false,"ping": "pong"}


# Step 2. Setup the ibsps environment variables

This tool needs to replace the environment variables in the ibsps release packages with a configuration file on each host, so please name the configuration file according to the following rules:

    - {{inventory_hostname}}_configuration.xml    (1)
    - {{group_name}}_configuration.xml            (2)
    - configuration.xml                           (3)

1. Used for a specific host, and has the highest priority, for example _exe.cluster_a.local_\_configuration.xml
2. Used for all hosts in a specific group, and has the medium priority, for example _executor_cluster_a_\_configuration.xml
3. Used for all hosts, and has the lowest priority

Refer to the template _../ibsps_environments/configuration.xml.sample_ to customize the environment variables, for more details, check the ibsps document &lt;&lt;IBSPs Installation and Deployment Manual.doc&gt;&gt;

# Step 3. Install the ibsps release package

> Save ibsps packages released by ACCA to the _../ibsps_release_packages_ directory

Directory structure description in the ibsps package

    - IBSPS_Va.b.c                                  (1)
        - BO Tempaltes                              (2)
            - *.rpt
        - Database scripts
            - majar update to ***
                - IBSP_CHANGELOG                    (3)
                    - PCK_DBADMIN_CHANGE.pck
                    - run_changelog.pdc
                    - run_grant_synonym.pdc
                    - nbs_changelog*.dmp
        - Software Packages                         (4)
            - job-schedule-ci.ear
            - job-execution-ci-ear
            - seurat-web-ci.ear

1. The root directory is as same name as the release package, for example: IBSPS_V5.11.B
2. Store bo templates
3. Store database upgrade scripts, it must include these files:
   - PCK_DBADMIN_CHANGE.pck
   - run_changelog.pdc
   - run_grant_synonym.pdc
   - nbs_changelog\*.dmp
4. Store ibsps packages, it must include these files:
   - job-schedule-ci.ear
   - job-execution-ci-ear
   - seurat-web-ci.ear

# Step 4. Configure the playbooks environment variables

Before the first run, you need to set the properties in _playbooks/group_vars/all.yml_ correctly according to the following example 

```yaml
    # The directory is used to store the build history archive files
    acd_build_history_path: "../history"

    # The oracle home directory on the database node
    oracle_home: /opt/oracle/app/oracle/product/11.2.0/dbhome_1

    # The oracle tns name on the database node
    oracle_sid: isis2db

    # The oracle dumpdata directory on the database node
    oracle_dump_directory_path: /opt/oracle/dump

    # The oracle dumpdata directory name
    oracle_dump_directory_name: dumpdata

    # The admin account name of the bo service
    bo_administrator_name: Administrator

    # The admin account pwd of the bo service
    bo_administrator_password: ABCabc123

    # The jboss home directory on the jboss node
    jboss_home: /opt/jboss/wildfly

    # The jboss service name on the jboss node
    jboss_service_name: jboss

    # The file path of the ibsps release package
    ibsps_release_packages_path: "../ibsps_release_packages/IBSPS_V5.11.B.zip"

    # The directory containing the ibsps environment configuration files
    ibsps_environment_configure_path: "../ibsps_environments"

    # The oracle connect string of the nbs schema
    ibsps_db_account_nbs: nbs/nbs

    # The other nbs schema names which separated by commas, if nothing left an empty array
    ibsps_db_account_nbs_others: []

    # The current version of ibsps changelog
    ibsps_db_current_version: 510000

    # The upgrade version of ibsps chagnelog
    ibsps_db_next_version: 511000

    # Enable the database backup before upgrading
    ibsps_db_enable_backup: true
```

In future releases, only the following attributes need to be modified

    ibsps_release_packages_path 
    ibsps_db_current_version
    ibsps_db_current_version


# Step 5. Run playbook

- Full site

> ansible-playbook _site_bo_database_parallel.yml_

- Update the nbs and nbs_common schemas

> ansible-playbook _jboss_shutdown.yml_  
> ansible-playbook _database.yml_

- Upload the bo templates

> ansible-playbook _jboss_shutdown.yml_  
> ansible-playbook _database.yml_

- Deploy the ibsps release packages

> ansible-playbook _jboss_shutdown.yml_  
> ansible-playbook _jboss_deploy.yml_

- Shutdown jboss

> ansible-playbook _jboss_shutdown.yml_

- Restart jboss

> ansible-playbook _jboss_restart.yml_

# Step 6. Check result

All build history will be saved to the _../history_ directory in zip format. These files will contain the stdout and stderr files that recording normal and error screen output respectively
