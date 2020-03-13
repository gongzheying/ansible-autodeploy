# Step 1. Setup the hosts inventory

> Edit file *hosts*

Add the hosts to the group they belong to, like the following example

     
    [database]
    # oracle database
    # including nbs and nbs_common schemas and only the master can be specified if the cluster is configured
    db.local
        
    [bo]
    # sap bo server
    # only the master can be specified if the cluster is configured
    bo.local

    [scheduler:children]
    scheduler_master
    scheduler_slave
        
    [executor]
    # ibsps execution  server
    # all members of the execution cluster need to be added here   
    exe.local
        
    [webapp]
    # ibsps web server
    # all members of the schedule cluster need to be added here
    app.local
    
    [scheduler_master]
    # ibsps schedule server
    # all members of the schedule cluster master nodes need to be added here
    # if schedule cluster have only one node, it should to be added here.
    sch.master.local

    [scheduler_slave]
    # ibsps schedule server
    # all members of the schedule cluster slave nodes need to be added here
    # if the schedule cluster does not have slave node, it should be blank here.
    sch.slave.local


For more details, check the ansible [inventory documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html).



# Step 2. Setup the hosts connection info  

> Edit file *host_vars/\*.yml*

If a host named ‘db.local’ that the variables in YAML files at the following locations:

    host_vars/db.local.yml


Sample file:

    ---
    # connection type to the host. This can be the name of any of ansible’s connection plugins
    ansible_connection: ssh

    # the name of the host to connect to, if different from the alias you wish to give to it.
    ansible_host: db.local

    # the connection port number, if not the default (22 for ssh)
    ansible_port: 22

    # the user name to use when connecting to the host
    ansible_user: root



For more details, check the ansible [connection plugins documentation](https://docs.ansible.com/ansible/latest/plugins/connection.html)

# Step 3. Setup the ibsps environment variables  

> Refer to the template *../ibsps_environments/configuration.xml.sample* to customize the environment variables


For more details, check the ibsps document &lt;&lt;IBSPs Installation and Deployment Manual.doc&gt;&gt;

# Step 4. Install the ibsps release package

> Save ibsps packages released by ACCA to the *../ibsps_release_packages* directory  

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
    - nbs_changelog*.dmp
4. Store ibsps packages, it must include these files:
    - job-schedule-ci.ear
    - job-execution-ci-ear
    - seurat-web-ci.ear

# Step 5. Configure the playbooks environment variables  

> Edit file *group_vars/all.yml*

    ---
    # The directory is used to store the build history archive files 
    acd_build_history_path: "../history"

    # The file path of the automatic configure tool
    acd_addons_autoconfig_path: "../addons/AutomaticConfiguration.zip"

    # The file path of the template upload tool
    acd_addons_templateupload_path: "../addons/TemplateUpload.zip"

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

    # The file path of the ibsps environment configuration file 
    ibsps_environment_configure_path: "../ibsps_environments/configuration.xml.sample"

    # The oracle connect string of the nbs schema
    ibsps_db_account_nbs: nbs/nbs

    # The oracle connect string of the nbs_common schema
    ibsps_db_account_nbs_common: nbs_common/nbs_common

    # The current version of ibsps changelog
    ibsps_db_current_version: 510000

    # The upgrade version of ibsps chagnelog
    ibsps_db_next_version: 511000

    # The share folder between the jboss nodes that needs to be able to create sub directories and files
    ibsps_nas_shared_directory: /ibspdata



# Step 6. Run playbook

- Full site

> ansible-playbook *site.yml*

- Update the nbs and nbs_common schemas

> ansible-playbook *jboss_shutdown.yml*  
> ansible-playbook *database.yml*  

- Upload the bo templates  

> ansible-playbook *jboss_shutdown.yml*  
> ansible-playbook *database.yml*  

- Deploy the ibsps release packages

> ansible-playbook *jboss_shutdown.yml*  
> ansible-playbook *jboss_deploy.yml*  

- Shutdown jboss

> ansible-playbook *jboss_shutdown.yml*  

- Restart jboss

> ansible-playbook *jboss_restart.yml*  


# Step 7. Check result


All build history will be saved to the *../history* directory in zip format. These files will contain the stdout and stderr files that recording normal and error screen output respectively   

