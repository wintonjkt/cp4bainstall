# cp4bainstall

###############################################################################
#
# Licensed Materials - Property of IBM
#
# (C) Copyright IBM Corp. 2021. All Rights Reserved.
#
# US Government Users Restricted Rights - Use, duplication or
# disclosure restricted by GSA ADP Schedule Contract with IBM Corp.
#
###############################################################################
apiVersion: icp4a.ibm.com/v1
kind: ICP4ACluster
metadata:
  name: icp4adeploy
  namespace: cp4ba
  labels:
    app.kubernetes.io/instance: ibm-dba
    app.kubernetes.io/managed-by: ibm-dba
    app.kubernetes.io/name: ibm-dba
    release: 21.0.3
  annotations:
    ansible.sdk.operatorframework.io/verbosity: "3"
spec:
  appVersion: 21.0.3
  ibm_license: "accept"

  ## MUST exist, used to accept ibm license, valid value only can be "accept"
  #ibm_license: ""

  ##########################################################################
  ## This section contains the shared configuration for all CP4A components #
  ##########################################################################
  shared_configuration:

    ## Added per IBM support
    show_sensitive_log: true
    ## Business Automation Workflow (BAW) license and possible values are: user, non-production, and production.
    ## This value could be different from the other licenses in the CR.
    sc_deployment_baw_license: "non-production"

    ## FileNet Content Manager (FNCM) license and possible values are: user, non-production, and production.
    ## This value could be different from the other licenses in the CR.
    sc_deployment_fncm_license: "non-production"

    ## The deployment context, which has a default value of "CP4A".  Unless you are instructed to change this value or
    ## know the reason to change this value, please leave the default value.
    sc_deployment_context: "CP4A"

    ## Please keep it false and do not change it
    sc_deploy_zen_with_iaf: false

    ## Use this parameter to specify the license for the CP4A deployment and
    ## the possible values are: non-production and production and if not set, the license will
    ## be defaulted to production.  This value could be different from the other licenses in the CR.
    sc_deployment_license: "non-production"

    ## All CP4A components must use/share the image_pull_secrets to pull images
    image_pull_secrets:
    - admin.registrykey

    ## All CP4A components must use/share the same docker image repository.  For example, if IBM Entitled Registry is used, then
    ## it should be "cp.icr.io".  Otherwise, it will be a local docker registry.
    sc_image_repository: cp.icr.io

    ## Specify the RunAsUser for the security context of the pod.  This is usually a numeric value that corresponds to a user ID.
    ## For non-OCP (e.g., CNCF platforms such as AWS, GKE, etc), this parameter is optional. It is not supported on OCP and ROKS.
    sc_run_as_user:

    images:
      keytool_job_container:
        repository: cp.icr.io/cp/cp4a/baw/dba-keytool-jobcontainer
        tag: 21.0.3-IF002
      dbcompatibility_init_container:
        repository: cp.icr.io/cp/cp4a/baw/dba-dbcompatibility-initcontainer
        tag: 21.0.3-IF002
      keytool_init_container:
        repository: cp.icr.io/cp/cp4a/baw/dba-keytool-initcontainer
        tag: 21.0.3-IF002
      umsregistration_initjob:
        repository: cp.icr.io/cp/cp4a/baw/dba-umsregistration-initjob
        tag: 21.0.3-IF002

      ## All CP4A components should use this pull_policy as the default, but it can override by each component
      pull_policy: IfNotPresent

    ## All CP4A components must use/share the root_ca_secret in order for integration
    root_ca_secret: icp4a-root-ca

    ## Shared secret containing a wildcard certificate (and concatenated signers) to be used by all routes, unless overwritten for a specific component route.
    ## If this is not defined, all external routes will be signed with root_ca_secret.
    external_tls_certificate_secret:

    ## CP4A patterns or capabilities to be deployed.  This CR represents the "workflow" pattern, which includes the following
    ## mandatory components: ban(Business Automation Navigator), ums (User Management Service), rr (Resource registry), app_engine( Application Engine) and optional components: bai
    sc_deployment_patterns: workflow

    ## The optional components to be installed if listed here.  This is normally populated by the User script based on input from the user.
    ## The optional components are: bai,ae_data_persistence
    sc_optional_components: ae_data_persistence

    ## The deployment type as selected by the user.  Possible values are: Starter and Production.
    sc_deployment_type: Production

    ## The platform to be deployed specified by the user.  Possible values are: OCP and other.  This is normally populated by the User script
    ## based on input from the user.
    sc_deployment_platform: "other"

    ## Optional: You can specify a profile size for CloudPak - valid values are small,medium,large - default is small.
    sc_deployment_profile_size: "small"

    ## For ROKS, this is used to enable the creation of ingresses. The default value is "false", which routes will be created.
    sc_ingress_enable: true

    ## For ROKS Ingress, provide TLS secret name for Ingress controller. If you are not using ROKS, comment out this line.
    #sc_ingress_tls_secret_name: <Required>

    ## This is the deployment hostname suffix, this is optional and the default hostname suffix will be used as {meta.namespace}.router-canonicalhostname
    ##sc_deployment_hostname_suffix: "{{ meta.namespace }}.router-default.itzroks-270002337a-j8dmah-4b4a324f027aea19c5cbc0c3275c4656-0000.us-south.containers.appdomain.cloud"
    sc_deployment_hostname_suffix: "cp4ba.34.101.122.212.nip.io"
    ## If the root certificate authority (CA) key of the external service is not signed by the operator root CA key, provide the TLS certificate of
    ## the external service to the component's truststore.
    trusted_certificate_list: []

    ## Shared encryption key secret name that is used for Workflow or Workstream Services and Process Federation Server integration.
    ## This secret is also used by Workflow and BAStudio to store AES encryption key.
    encryption_key_secret: ibm-iaws-shared-key-secret

    ## Enable/disable ECM (FNCM) / BAN initialization (e.g., creation of P8 domain, creation/configuration of object stores,
    ## creation/configuration of CSS servers, and initialization of Navigator (ICN)).  If the "initialize_configuration" section
    ## is defined with the required parameters in the CR (below) and sc_content_initialization is set to "true" (or the parameter doesn't exist), then the initialization will occur.
    ## However, if sc_content_initialization is set to "false", then the initialization will not occur (even with the "initialize_configuration" section defined)
    sc_content_initialization: true


    ## Enable/disable the ECM (FNCM) / BAN verification (e.g., creation of test folder, creation of test document,
    ## execution of CBR search, and creation of Navigator demo repository and desktop).  If the "verify_configuration"
    ## section is defined in the CR, then that configuration will take precedence overriding this parameter.
    sc_content_verification: false

    ## On OCP 3.x and 4.x, the User script will populate these three (3) parameters based on your input for "production" deployment.
    ## If you manually deploying without using the User script, then you would provide the different storage classes for the slow, medium
    ## and fast storage parameters below.  If you only have 1 storage class defined, then you can use that 1 storage class for all 3 parameters.
    storage_configuration:
      sc_slow_file_storage_classname: "standard"
      sc_medium_file_storage_classname: "standard"
      sc_fast_file_storage_classname: "standard"

  ## The beginning section of LDAP configuration for CP4A
  ldap_configuration:
    ## The possible values are: "IBM Security Directory Server" or "Microsoft Active Directory"
    lc_selected_ldap_type: "IBM Security Directory Server"

    ## The name of the LDAP server to connect
    lc_ldap_server: "10.184.0.6"

    ## The port of the LDAP server to connect.  Some possible values are: 389, 636, etc.
    lc_ldap_port: "389"

    ## The LDAP bind secret for LDAP authentication.  The secret is expected to have ldapUsername and ldapPassword keys.  Refer to Knowledge Center for more info.
    lc_bind_secret: ldap-bind-secret

    ## The LDAP base DN.  For example, "dc=example,dc=com", "dc=abc,dc=com", etc
    lc_ldap_base_dn: "dc=asia-southeast2-a,dc=c,dc=poc-ibmbpm,dc=internal"

    ## Enable SSL/TLS for LDAP communication. Refer to Knowledge Center for more info.
    lc_ldap_ssl_enabled: false

    ## The name of the secret that contains the LDAP SSL/TLS certificate.
    lc_ldap_ssl_secret_name: ""

    ## The LDAP user name attribute. Semicolon-separated list that must include the first RDN user distinguished names. One possible value is "*:uid" for TDS and "user:sAMAccountName" for AD. Refer to Knowledge Center for more info.
    lc_ldap_user_name_attribute: "*:uid"

    ## The LDAP user display name attribute. One possible value is "cn" for TDS and "sAMAccountName" for AD. Refer to Knowledge Center for more info.
    lc_ldap_user_display_name_attr: "cn"

    ## The LDAP group base DN.  For example, "dc=example,dc=com", "dc=abc,dc=com", etc
    lc_ldap_group_base_dn: "dc=asia-southeast2-a,dc=c,dc=poc-ibmbpm,dc=internal"

    ## The LDAP group name attribute.  One possible value is "*:cn" for TDS and "*:cn" for AD. Refer to Knowledge Center for more info.
    lc_ldap_group_name_attribute: "*:cn"

    ## The LDAP group display name attribute.  One possible value for both TDS and AD is "cn". Refer to Knowledge Center for more info.
    lc_ldap_group_display_name_attr: "cn"

    ## The LDAP group membership search filter string.  One possible value is "(|(&(objectclass=groupofnames)(member={0}))(&(objectclass=groupofuniquenames)(uniquemember={0})))" for TDS
    ## and "(&(cn=%v)(objectcategory=group))" for AD.
    lc_ldap_group_membership_search_filter: "(&(cn=%v)(objectcategory=group))"

    ## The LDAP group membership ID map.  One possible value is "groupofnames:member" for TDS and "memberOf:member" for AD.
    lc_ldap_group_member_id_map: "memberOf:member"

    ## The LDAP recursive search. Set true or false to enable or disable recursive searches.
    lc_ldap_recursive_search: false

    ## The maximum search results. Specify a higher value if you expect more search results.
    lc_ldap_max_search_results: 4500

    ## The User script will uncomment the section needed based on user's input from User script.  If you are deploying without the User script,
    ## uncomment the necessary section (depending if you are using Active Directory (ad) or Tivoli Directory Service (tds)) accordingly.
    ad:
     lc_ad_gc_host: "10.184.0.6"
     lc_ad_gc_port: "389"
    # lc_user_filter: "(&(sAMAccountName=%v)(objectcategory=user))"
    # lc_group_filter: "(&(cn=%v)(objectcategory=group))"
    # tds:
       lc_user_filter: "(&(cn=%v)(objectclass=person))"
       lc_group_filter: "(&(cn=%v)(|(objectclass=groupofnames)(objectclass=groupofuniquenames)(objectclass=groupofurls)))"


    ## This section allows to enhance the ldap configuration for the UMS SCIM capability. If lc_user_filter or lc_group_filter cannot handle a custom LDAP filter for user or group searches this section should be enabled.
    ## optional: enables the liberty ldapEntityType configuration and disables the usage of lc_user_filter, lc_group_filter, lc_ldap_group_member_id_map, lc_ldap_user_name_attribute and lc_ldap_group_name_attribute in the UMS capabilities.
    ## for detailed information about the ldapEntityType, loginProperty and groupProperties  parameters please see the liberty documentation: https://www.ibm.com/docs/en/was-liberty/nd?topic=configuration-ldapregistry
    ## default is false
    lc_use_ldap_entity_type:
    ## optional: only used if lc_use_ldap_entity_type is true
    ## default is uid
    lc_ldap_login_property:
    ## optional: only used if lc_use_ldap_entity_type is true
    ## the defaults depends on the lc_selected_ldap_type
    lc_ldap_entity_type_user:
      object_class:
      search_base:
      searchfilter:
    ## optional: only used if lc_use_ldap_entity_type is true
    ## the defaults depends on the lc_selected_ldap_type
    lc_ldap_entity_type_group:
      object_class:
      search_base:
      searchfilter:
    ## optional: only used if lc_use_ldap_entity_type is true
    ## the defaults depends on the lc_selected_ldap_type
    lc_ldap_group_properties:
      # member_attribute:
        # The name of the member. Required if member_attribute is set
        # name:
        # The name of the object class. Required member_attribute is set
        # object_class:
        ## the scope options are: all, direct, nested
        # scope:
      #membership_attribute:
        # The name of the membership. Required if membership_attribute is set
        # name:
        ## the scope options are: all, direct, nested
        # scope:


  ## The beginning section of database configuration for CP4A
  datasource_configuration:
    ## The dc_ssl_enabled parameter is used to support database connection over SSL for DB2/Oracle/PostgreSQL.
    dc_ssl_enabled: false
    ## The database_precheck parameter is used to enable or disable CPE/Navigator database connection check.
    ## If set to "true", then CPE/Navigator database connection check will be enabled.
    ## if set to "false", then CPE/Navigator database connection check will not be enabled.
   # database_precheck: true
    ## The database configuration for the GCD datasource for CPE
    dc_gcd_datasource:
      ## Provide the database type from your infrastructure.  The possible values are "db2" or "db2HADR" or "oracle" or "postgresql".
      dc_database_type: "postgresql"
      ## The GCD non-XA datasource name.  The default value is "FNGCDDS".
      dc_common_gcd_datasource_name: "FNGCDDS"
      ## The GCD XA datasource name. The default value is "FNGCDDSXA".
      dc_common_gcd_xa_datasource_name: "FNGCDDSXA"
      ## Provide the database server name or IP address of the database server.
      database_servername: "10.77.225.3"
      ## Provide the name of the database for the GCD for CPE.  For example: "GCDDB"
      database_name: "GCD_DB_USER"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521"
      database_port: "5432"
      ## The name of the secret that contains the DB2/Oracle/PostgreSQL SSL certificate.
      database_ssl_secret_name: ""
      ## If the database type is Oracle, provide the Oracle DB connection string.  For example, "jdbc:oracle:thin:@//<oracle_server>:1521/orcl"
      ## dc_oracle_gcd_jdbc_url: "jdbc:oracle:thin:@//cdldvddp007or-scan.es.ad.adp.com:1521/ibp021d_svc1"

      ## If the database type is Db2 HADR, then complete the rest of the parameters below.
      ## Provide the database server name or IP address of the standby database server.
      #dc_hadr_standby_servername: "<Required>"
      ## Provide the standby database server port.  For Db2, the default is "50000".
      #dc_hadr_standby_port: "<Required>"
      ## Provide the validation timeout.  If not preference, keep the default value.
      #dc_hadr_validation_timeout: 15
      ## Provide the retry internal.  If not preference, keep the default value.
      #dc_hadr_retry_interval_for_client_reroute: 15
      ## Provide the max # of retries.  If not preference, keep the default value.
      #dc_hadr_max_retries_for_client_reroute: 3
      ## Connection manager for a data source.
      #connection_manager:
        ## Minimum number of physical connections to maintain in the pool.
        #min_pool_size: 0
        ## Maximum number of physical connections for a pool.
        #max_pool_size: 50
        ## Amount of time a connection can be unused or idle until it can be discarded during pool maintenance, if doing so does not reduce the pool below the minimum size.
        #max_idle_time: 1m
        ## Amount of time between runs of the pool maintenance thread.
        #reap_time: 2m
        ## Specifies which connections to destroy when a stale connection is detected in a pool.
        #purge_policy: EntirePool


    ## The database_precheck parameter is used to enable or disable CPE/Navigator database connection check.
    ## If set to "true", then CPE/Navigator database connection check will be enabled.
    ## if set to "false", then CPE/Navigator database connection check will not be enabled.
   # database_precheck: true
    ## The database configuration for ICN (Navigator) - aka BAN (Business Automation Navigator)
    dc_icn_datasource:
      ## Provide the database type from your infrastructure.  The possible values are "db2" or "db2HADR" or "oracle" or "postgresql".  This should be the same as the
      ## GCD and object store configuration above.
      dc_database_type: "postgresql"
      ## Provide the ICN datasource name.  The default value is "ECMClientDS".
      dc_common_icn_datasource_name: "ECMClientDS"
      database_servername: "10.77.225.3"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521"
      database_port: "5432"
      ## Provide the name of the database for ICN (Navigator).  For example: "ICNDB"
      database_name: "icndb"
      ## The name of the secret that contains the DB2/Oracle/PostgreSQL SSL certificate.
      database_ssl_secret_name: ""
      ## If the database type is Oracle, provide the Oracle DB connection string.  For example, "jdbc:oracle:thin:@//<oracle_server>:1521/orcl"
      ## dc_oracle_icn_jdbc_url: "jdbc:oracle:thin:@//cdldvddp007or-scan.es.ad.adp.com:1521/ibp021d_svc1"
      ######################################################################################
      ## If the database type is "Db2HADR", then complete the rest of the parameters below.
      ## Otherwise, remove or comment out the rest of the parameters below.
      ######################################################################################
      #dc_hadr_standby_servername: "<Required>"
      ## Provide the standby database server port.  For Db2, the default is "50000".
      #dc_hadr_standby_port: "<Required>"
      ## Provide the validation timeout.  If not preference, keep the default value.
      #dc_hadr_validation_timeout: 15
      ## Provide the retry internal.  If not preference, keep the default value.
      #dc_hadr_retry_interval_for_client_reroute: 15
      ## Provide the max # of retries.  If not preference, keep the default value.
      #dc_hadr_max_retries_for_client_reroute: 3
      ## Connection manager for a data source.
      #connection_manager:
        ## Minimum number of physical connections to maintain in the pool.
        #min_pool_size: 0
        ## Maximum number of physical connections for a pool.
        #max_pool_size: 50
        ## Amount of time a connection can be unused or idle until it can be discarded during pool maintenance, if doing so does not reduce the pool below the minimum size.
        #max_idle_time: 1m
        ## Amount of time between runs of the pool maintenance thread.
        #reap_time: 2m
        ## Specifies which connections to destroy when a stale connection is detected in a pool.
        #purge_policy: EntirePool

    ## The database configuration for UMS (User Management Service)
    dc_ums_datasource:
      ## Provide the datasource configuration for oauth
      ## Possible dc_ums_oauth_type values are "derby" for test only and "db2", "oracle", "sqlserver" and "postgresql" for production.
      ## This configuration should be the same as the other datasource configuration in the dc_ums_datasource section.
      ## db2 with HADR is automatically activated if dc_ums_oauth_alternate_hosts and dc_ums_oauth_alternate_ports are set.
      ## For Oracle RAC, specify the host name of the SCAN listener as the value of the dc_ums_oauth_host parameter
      dc_ums_oauth_type: "postgresql"
      ## Provide the database server name or IP address of the database server.
      dc_ums_oauth_host: "10.77.225.3"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521". For MSSQL, the default is "1433"- For PostgreSQL, the default is "5432".
      dc_ums_oauth_port: "5432"
      ## Provide the name of the database for UMS.  For example: "UMSDB"
      #dc_ums_oauth_name: "<required>"
      dc_ums_oauth_schema: "umsdb"
      ## dc_ums_oauth_oracle_service_name: "ibp021d_svc1"
      dc_ums_oauth_ssl: false
      dc_ums_oauth_ssl_secret_name: ""
      ## For "oracle", "sqlserver" and "postgresql" provide the names of the driver files
      dc_ums_oauth_driverfiles: "postgresql-42.4.0.jar"
      ## For db2 HADR only
      #dc_ums_oauth_alternate_hosts:
      #dc_ums_oauth_alternate_ports:

      ## Provide the datasource configuration for the teamserver
      ## Possible dc_ums_teamserver_type values are "derby" for test only and "db2", "oracle", "sqlserver" and "postgresql" for production.
      ## This configuration should be the same as the other datasource configuration in the dc_ums_datasource section.
      ## db2 with HADR is automatically activated if dc_ums_teamserver_alternate_hosts and dc_ums_teamserver_alternate_ports are set.
      ## For Oracle RAC, specify the host name of the SCAN listener as the value of the dc_ums_teamserver_host parameter
      dc_ums_teamserver_type: "postgresql"
      dc_ums_teamserver_host: "10.77.225.3"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521". For MS SQL, the default is "1433"- For PostgreSQL, the default is "5432".
      dc_ums_teamserver_port: "5432"
      ## Provide the name of the database for UMS teamserver.  For example: "UMSDB"
      #dc_ums_teamserver_name: "<Required>"
      dc_ums_teamserver_schema: "umsdb"
      ##dc_ums_teamserver_oracle_service_name: "ibp021d_svc1"
      dc_ums_teamserver_ssl: false
      dc_ums_teamserver_ssl_secret_name: ""
      ## For "oracle", "sqlserver" and "postgresql" provide the names of the driver files
      dc_ums_teamserver_driverfiles: "postgresql-42.4.0.jar"
      ## For db2 HADR only
      #dc_ums_teamserver_alternate_hosts:
      #dc_ums_teamserver_alternate_ports:
    ## The database configuration for the document object store (DOCS) datasource for CPE
    dc_os_datasources:
    ## The database configuration for the target object store (BAW DOCS) datasource for CPE. Provide the database type from your infrastructure.  The possible values are "db2" or "db2HADR" or "oracle" or "postgresql".  This should be the same as the GCD
    - dc_database_type: "postgresql"
      ## Provide the object store label for the object store.  The default value is "os" or not defined.
      ## This label must match the OS secret you define in ibm-fncm-secret.
      ## For example, if you define dc_os_label: "abc", then your OS secret must be defined as:
      ## --from-literal=abcDBUsername="<your os db username>" --from-literal=abcDBPassword="<your os db password>"
      ## If you don't define dc_os_label, then your secret will be defined as:
      ## --from-literal=osDBUsername="<your os db username>" --from-literal=osDBPassword="<your os db password>".
      ## If you have multiple object stores, then you need to define multiple datasource sections starting
      ## at "dc_database_type" element.
      ## If all the object store databases share the same username and password, then dc_os_label value should be the same
      ## in all the datasource sections.
      dc_os_label: "bawdocs"
      ## The DOCS non-XA datasource name.  The default value is "BAWINS1DOCS".
      dc_common_os_datasource_name: "BAWINS1DOCS"
      ## The DOCS XA datasource name.  The default value is "BAWINS1DOCSXA".
      dc_common_os_xa_datasource_name: "BAWINS1DOCSXA"
      ## Provide the database server name or IP address of the database server.  This should be the same as the
      ## GCD configuration above.
      database_servername: "10.77.225.3"
      ## Provide the name of the database for the object store 1 for CPE.  For example: "OS1DB"
      database_name: "docsdb"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521"
      database_port: "5432"
      ## The name of the secret that contains the DB2/Oracle/PostgreSQL SSL certificate.
      database_ssl_secret_name: ""
      ## If the database type is Oracle, provide the Oracle DB connection string.  For example, "jdbc:oracle:thin:@//<oracle_server>:1521/orcl"
      ## ## dc_oracle_os_jdbc_url: "jdbc:oracle:thin:@//cdldvddp007or-scan.es.ad.adp.com:1521/ibp021d_svc1"
      ######################################################################################
      ## If the database type is "Db2HADR", then complete the rest of the parameters below.
      ## Otherwise, remove or comment out the rest of the parameters below.
      ######################################################################################
      #dc_hadr_standby_servername: "<Required>"
      ## Provide the standby database server port.  For Db2, the default is "50000".
      #dc_hadr_standby_port: "<Required>"
      ## Provide the validation timeout.  If not preference, keep the default value.
      #dc_hadr_validation_timeout: 15
      ## Provide the retry internal.  If not preference, keep the default value.
      #dc_hadr_retry_interval_for_client_reroute: 15
      ## Provide the max # of retries.  If not preference, keep the default value.
      #dc_hadr_max_retries_for_client_reroute: 3
      ## Connection manager for a data source.
      #connection_manager:
        ## Minimum number of physical connections to maintain in the pool.
        #min_pool_size: 0
        ## Maximum number of physical connections for a pool.
        #max_pool_size: 50
        ## Amount of time a connection can be unused or idle until it can be discarded during pool maintenance, if doing so does not reduce the pool below the minimum size.
        #max_idle_time: 1m
        ## Amount of time between runs of the pool maintenance thread.
        #reap_time: 2m
        ## Specifies which connections to destroy when a stale connection is detected in a pool.
        #purge_policy: EntirePool

    ## The database configuration for the target object store (TOS) datasource for CPE
    - dc_database_type: "postgresql"
      ## Provide the object store label for the object store.  The default value is "os" or not defined.
      ## This label must match the OS secret you define in ibm-fncm-secret.
      ## For example, if you define dc_os_label: "abc", then your OS secret must be defined as:
      ## --from-literal=abcDBUsername="<your os db username>" --from-literal=abcDBPassword="<your os db password>"
      ## If you don't define dc_os_label, then your secret will be defined as:
      ## --from-literal=osDBUsername="<your os db username>" --from-literal=osDBPassword="<your os db password>".
      ## If you have multiple object stores, then you need to define multiple datasource sections starting
      ## at "dc_database_type" element.
      ## If all the object store databases share the same username and password, then dc_os_label value should be the same
      ## in all the datasource sections.
      dc_os_label: "bawtos"
      ## The TOS non-XA datasource name.  The default value is "BAWINS1TOS".
      dc_common_os_datasource_name: "BAWINS1TOS"
      ## The TOS XA datasource name.  The default value is "BAWINS1TOSXA".
      dc_common_os_xa_datasource_name: "BAWINS1TOSXA"
      ## Provide the database server name or IP address of the database server.  This should be the same as the
      ## GCD configuration above.
      database_servername: "10.77.225.3"
      ## Provide the name of the database for the object store 1 for CPE.  For example: "OS1DB"
      database_name: "tosdb"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521"
      database_port: "5432"
      ## The name of the secret that contains the DB2/Oracle/PostgreSQL SSL certificate.
      database_ssl_secret_name: ""
      ## If the database type is Oracle, provide the Oracle DB connection string.  For example, "jdbc:oracle:thin:@//<oracle_server>:1521/orcl"
      ## ## dc_oracle_os_jdbc_url: "jdbc:oracle:thin:@//cdldvddp007or-scan.es.ad.adp.com:1521/ibp021d_svc1"
      ######################################################################################
      ## If the database type is "Db2HADR", then complete the rest of the parameters below.
      ## Otherwise, remove or comment out the rest of the parameters below.
      ######################################################################################
      #dc_hadr_standby_servername: "<Required>"
      ## Provide the standby database server port.  For Db2, the default is "50000".
      #c_hadr_standby_port: "<Required>"
      ## Provide the validation timeout.  If not preference, keep the default value.
      #dc_hadr_validation_timeout: 15
      ## Provide the retry internal.  If not preference, keep the default value.
      #dc_hadr_retry_interval_for_client_reroute: 15
      ## Provide the max # of retries.  If not preference, keep the default value.
      #dc_hadr_max_retries_for_client_reroute: 3
      ## Connection manager for a data source.
      #connection_manager:
        ## Minimum number of physical connections to maintain in the pool.
        #min_pool_size: 0
        ## Maximum number of physical connections for a pool.
        #max_pool_size: 50
        ## Amount of time a connection can be unused or idle until it can be discarded during pool maintenance, if doing so does not reduce the pool below the minimum size.
        #max_idle_time: 1m
        ## Amount of time between runs of the pool maintenance thread.
        #reap_time: 2m
        ## Specifies which connections to destroy when a stale connection is detected in a pool.
        #purge_policy: EntirePool

    ## The database configuration for the design object store (DOS) datasource for CPE
    - dc_database_type: "postgresql"
      ## Provide the object store label for the object store.  The default value is "os" or not defined.
      ## This label must match the OS secret you define in ibm-fncm-secret.
      ## For example, if you define dc_os_label: "abc", then your OS secret must be defined as:
      ## --from-literal=abcDBUsername="<your os db username>" --from-literal=abcDBPassword="<your os db password>"
      ## If you don't define dc_os_label, then your secret will be defined as:
      ## --from-literal=osDBUsername="<your os db username>" --from-literal=osDBPassword="<your os db password>".
      ## If you have multiple object stores, then you need to define multiple datasource sections starting
      ## at "dc_database_type" element.
      ## If all the object store databases share the same username and password, then dc_os_label value should be the same
      ## in all the datasource sections.
      dc_os_label: "bawdos"
      ## The DOS non-XA datasource name.  The default value is "BAWINS1DOS".
      dc_common_os_datasource_name: "BAWINS1DOS"
      ## The DOS XA datasource name.  The default value is "BAWINS1DOSXA".
      dc_common_os_xa_datasource_name: "BAWINS1DOSXA"
      ## Provide the database server name or IP address of the database server.  This should be the same as the
      ## GCD configuration above.
      database_servername: "10.77.225.3"
      ## Provide the name of the database for the object store 1 for CPE.  For example: "OS1DB"
      database_name: "dosdb"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521"
      database_port: "5432"
      ## The name of the secret that contains the DB2/Oracle/PostgreSQL SSL certificate.
      database_ssl_secret_name: ""
      ## If the database type is Oracle, provide the Oracle DB connection string.  For example, "jdbc:oracle:thin:@//<oracle_server>:1521/orcl"
      #### dc_oracle_os_jdbc_url: "jdbc:oracle:thin:@//cdldvddp007or-scan.es.ad.adp.com:1521/ibp021d_svc1"
      ######################################################################################
      ## If the database type is "Db2HADR", then complete the rest of the parameters below.
      ## Otherwise, remove or comment out the rest of the parameters below.
      ######################################################################################
      #dc_hadr_standby_servername: "<Required>"
      ## Provide the standby database server port.  For Db2, the default is "50000".
      #dc_hadr_standby_port: "<Required>"
      ## Provide the validation timeout.  If not preference, keep the default value.
      #dc_hadr_validation_timeout: 15
      ## Provide the retry internal.  If not preference, keep the default value.
      #dc_hadr_retry_interval_for_client_reroute: 15
      ## Provide the max # of retries.  If not preference, keep the default value.
      #dc_hadr_max_retries_for_client_reroute: 3
      ## Connection manager for a data source.
      #connection_manager:
        ## Minimum number of physical connections to maintain in the pool.
        #min_pool_size: 0
        ## Maximum number of physical connections for a pool.
        #max_pool_size: 50
        ## Amount of time a connection can be unused or idle until it can be discarded during pool maintenance, if doing so does not reduce the pool below the minimum size.
        #max_idle_time: 1m
        ## Amount of time between runs of the pool maintenance thread.
        #reap_time: 2m
        ## Specifies which connections to destroy when a stale connection is detected in a pool.
        #purge_policy: EntirePool

    ## object store for AEOS
    - dc_database_type: "postgresql"
      ## Provide the object store label for the object store.  The default value is "os" or not defined.
      ## This label must match the OS secret you define in ibm-fncm-secret.
      ## For example, if you define dc_os_label: "abc", then your OS secret must be defined as:
      ## --from-literal=abcDBUsername="<your os db username>" --from-literal=abcDBPassword="<your os db password>"
      ## If you don't define dc_os_label, then your secret will be defined as:
      ## --from-literal=osDBUsername="<your os db username>" --from-literal=osDBPassword="<your os db password>".
      ## If you have multiple object stores, then you need to define multiple datasource sections starting
      ## at "dc_database_type" element.
      ## If all the object store databases share the same username and password, then dc_os_label value should be the same
      ## in all the datasource sections.
      dc_os_label: "aeos"
      ## The AEOS non-XA datasource name.  The default value is "AEOS".
      dc_common_os_datasource_name: "AEOS"
      ## The AEOS XA datasource name.  The default value is "AEOSXA".
      dc_common_os_xa_datasource_name: "AEOSXA"
      ## Provide the database server name or IP address of the database server.  This should be the same as the
      ## GCD configuration above.
      database_servername: "10.77.225.3"
      ## Provide the name of the database for the object store AEOS for CPE.  For example: "AEOSDB"
      database_name: "aeosdb"
      ## Provide the database server port.  For Db2, the default is "50000".  For Oracle, the default is "1521"
      database_port: "5432"
      ## The name of the secret that contains the DB2/Oracle/PostgreSQL SSL certificate.
      database_ssl_secret_name: ""
      ## If the database type is Oracle, provide the Oracle DB connection string.  For example, "jdbc:oracle:thin:@//<oracle_server>:1521/orcl"
      ## dc_oracle_os_jdbc_url: "jdbc:oracle:thin:@//cdldvddp007or-scan.es.ad.adp.com:1521/ibp021d_svc1"
      ######################################################################################
      ## If the database type is "Db2HADR", then complete the rest of the parameters below.
      ## Otherwise, remove or comment out the rest of the parameters below.
      ######################################################################################
      #dc_hadr_standby_servername: "<Required>"
      ## Provide the standby database server port.  For Db2, the default is "50000".
      #dc_hadr_standby_port: "<Required>"
      ## Provide the validation timeout.  If not preference, keep the default value.
      #dc_hadr_validation_timeout: 15
      ## Provide the retry internal.  If not preference, keep the default value.
      #dc_hadr_retry_interval_for_client_reroute: 15
      ## Provide the max # of retries.  If not preference, keep the default value.
      #dc_hadr_max_retries_for_client_reroute: 3
      ## Connection manager for a data source.
      #connection_manager:
        ## Minimum number of physical connections to maintain in the pool.
        #min_pool_size: 0
        ## Maximum number of physical connections for a pool.
        #max_pool_size: 50
        ## Amount of time a connection can be unused or idle until it can be discarded during pool maintenance, if doing so does not reduce the pool below the minimum size.
        #max_idle_time: 1m
        ## Amount of time between runs of the pool maintenance thread.
        #reap_time: 2m
        ## Specifies which connections to destroy when a stale connection is detected in a pool.
        #purge_policy: EntirePool

  ########################################################################
  ########   IBM Business Automation Navigator configuration      ########
  ########################################################################
  navigator_configuration:

    ## Navigator secret that contains user credentials for LDAP and database
    ban_secret_name: ibm-ban-secret

    ## The architecture of the cluster.  This is the default for Linux and should not be changed.
    arch:
      amd64: "3 - Most preferred"

    ## The number of replicas or pods to be deployed.  The default is 2 replica and for high availability in a production env,
    ## it is recommended to have 2 or more.
    replica_count: 2

    ## This is the image repository and tag that correspond to image registry, which is where the image will be pulled.
    image:

      ## The default repository is the IBM Entitled Registry
      repository: cp.icr.io/cp/cp4a/ban/navigator-sso
      tag: ga-3011-icn-la001

      ## This will override the image pull policy in the shared_configuration.
      pull_policy: IfNotPresent

    ## Logging for workloads.  This is the default setting.
    log:
      format: json

    ## This is the initial default resource requests.  If more resources are needed,
    ## make the changes here to meet your requirement.
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 1
        memory: 1536Mi

    ## By default "Autoscaling" is enabled with the following settings with a minimum of 1 replca and a maximum of 3 replicas.  Change
    ## this settings to meet your requirement.
    auto_scaling:
      enabled: false
      max_replicas: 3
      min_replicas: 2
      ## This is the default cpu percentage before autoscaling occurs.
      target_cpu_utilization_percentage: 80

    ## Below are the default ICN Production settings.  Make the necessary changes as you see fit.
    icn_production_setting:
      timezone: Etc/UTC
      jvm_initial_heap_percentage: 40
      jvm_max_heap_percentage: 66
      jvm_customize_options:
      icn_jndids_name: ECMClientDS
      icn_schema: ICN_DB_USER
      icn_table_space: ICN_DB_DATA1
      allow_remote_plugins_via_http: false
      ## uncomment copy_files_to_war parameter to copy customized files into Navigator web application.
      ## The <custom-dir>/navigator_war_filesources.xml must be located in config volume mapping, which is /opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides
      # copy_files_to_war: <custom-dir>/navigator_war_filesources.xml

      ## The WalkMe URL references a WalkMe snippet.  This snippet is a piece of JavaScript code that allows WalkMe to be displayed in the application.
      ## Each WalkMe Editor account has a unique snippet code that can be accessed inside the Editor.
      #  walkme_url: https://cdn.walkme.com/users/4e7c687193414395aa0411837a9eee4b/test/walkme_4e7c687193414395aa0411837a9eee4b_https.js

    ## Default settings for monitoring
    monitor_enabled: false
    ## Default settings for logging
    logging_enabled: false

    ## Persistent Volume Claims for Navigator.  The Operator will create the PVC using the names below by default.
    datavolume:
      existing_pvc_for_icn_cfgstore: "icn-cfgstore"
      existing_pvc_for_icn_logstore: "icn-logstore"
      existing_pvc_for_icn_pluginstore: "icn-pluginstore"
      existing_pvc_for_icnvw_cachestore: "icn-vw-cachestore"
      existing_pvc_for_icnvw_logstore: "icn-vw-logstore"
      existing_pvc_for_icn_aspera: "icn-asperastore"

    ## Default values for both rediness and liveness probes.  Modify these values to meet your requirements.
    probe:
      readiness:

        initial_delay_seconds: 120
        period_seconds: 5
        timeout_seconds: 10
        failure_threshold: 6
      liveness:
        initial_delay_seconds: 600
        period_seconds: 5
        timeout_seconds: 5
        failure_threshold: 6

    ## Only use this parameter if you want to override the image_pull_secrets setting in the shared_configuration above.
    image_pull_secrets:
      name: "admin.registrykey"

  ########################################################################
  ########   IBM User and Group Management Service configuration  ########
  ########################################################################
  ums_configuration:
    existing_claim_name: "operator-shared-pvc"
    dedicated_pods: true
    service_type: Ingress
    routes_ingress_annotations:
    # your external UMS host name, only required if there is no sc_deployment_hostname_suffix given
    #hostname:
    port: 443
    images:
      ums:
        repository: cp.icr.io/cp/cp4a/ums/ums
        tag: 21.0.3-IF002
    admin_secret_name: ibm-dba-ums-secret
    ## optional: create routes for backwards compatibility
    backwards_compatibility_routes:
    ## optional for secure communication with UMS
    external_tls_secret_name:
    ## optional for secure communication with UMS
    external_tls_ca_secret_name:
    ## optional for secure communication with UMS
    external_tls_teams_secret_name:
    ## optional for secure communication with UMS
    external_tls_scim_secret_name:
    ## optional for secure communication with UMS
    external_tls_sso_secret_name:
    ##Set to true if you are using Oracle, PostgreSQL databases, or custom db2 JDBC drivers.
    use_custom_jdbc_drivers: true
    custom_jdbc_pvc: "operator-shared-pvc"
    jdbc_driver_files: "postgresql-42.4.0.jar"
    use_custom_binaries: false
    custom_secret_name:

    oauth:
      ## optional: full DN of an LDAP group that is authorized to manage OIDC clients, in addition to primary admin from admin secret
      client_manager_group:
      ## optional: full DN of an LDAP group that is authorized to manage app_tokens, in addition to primary admin from admin secret
      token_manager_group:
      ## optional: lifetime of OAuth access_tokens. default is 7200s
      access_token_lifetime:
      ## optional: lifetime of app-tokens. default is 366d
      app_token_lifetime:
      ## optional: lifetime of app-passwords. default is 366d
      app_password_lifetime:
      ## optional: maximum number of app-tokens or app-passwords per client. default is 100
      app_token_or_password_limit:
      ## optional: encoding / encryption when storing client secrets in OAuth database. Default is xor for compatibility. Recommended value is PBKDF2WithHmacSHA512
      client_secret_encoding:

    #### If dedicated_pods is set to false, the UMS capabilities sso, scim and teamserver
    #### run in the same pods and share this configuration
    replica_count: 2
    resources:
      limits:
        cpu: 500m
        memory: 512Mi
      requests:
        cpu: 200m
        memory: 256Mi
    autoscaling:
      enabled: true
      min_replicas: 2
      max_replicas: 5
      target_average_utilization: 98
    custom_xml:
    logs:
      traceSpecification: "*=info"

    #### If dedicated_pods is set to true, the UMS capabilities sso, scim and teamserver
    #### run in dedicated pods and are configured separately

    # Configuration for sso pods
    sso:
      replica_count: 2
      resources:
        limits:
          cpu: 500m
          memory: 512Mi
        requests:
          cpu: 200m
          memory: 256Mi
      autoscaling:
        enabled: true
        minReplicas: 2
        maxReplicas: 5
        targetAverageUtilization: 98
      custom_xml:
      logs:
        traceSpecification: "*=info"

    # configuration for scim pods
    scim:
      replica_count: 2
      resources:
        limits:
          cpu: 500m
          memory: 512Mi
        requests:
          cpu: 200m
          memory: 256Mi
      autoscaling:
        enabled: true
        minReplicas: 2
        maxReplicas: 5
        targetAverageUtilization: 98
      custom_xml:
      logs:
        traceSpecification: "*=info"

    # configuration for teamserver pods
    teamserver:
      replica_count: 2
      resources:
        limits:
          cpu: 500m
          memory: 512Mi
        requests:
          cpu: 200m
          memory: 256Mi
      autoscaling:
        enabled: true
        minReplicas: 2
        maxReplicas: 5
        targetAverageUtilization: 98
      custom_xml:
      logs:
        traceSpecification: "*=info"
      # optional, default false: enable debug output for Teamserver-to-BTS migration
      migration_debug: false
      # optional, default false: drop tables in BTS database before Teamserver-to-BTS migration
      migration_droptables: false
      # optional, default false: execute Teamserver-to-BTS migration regardless of Teamserver uninstall
      migration_test: false


  ##################################################################
  ########   Resource Registry configuration                ########
  ##################################################################
  resource_registry_configuration:
    # If you inputed hostname and port here. They will be used always
    # If you are using pattern mode (the shared_configuration.sc_deployment_patterns contains value)
    # Then you don't need to fill the hostname and port. It will use shared_configuration.sc_deployment_hostname_suffix to generate one
    # But if you haven't input suffix. And no hostname port assigned. A error will be reported in operator log during deploy
    # For non pattern mode you must assign a valid hostname and port here
    #hostname: "{{ 'rr-' + shared_configuration.sc_deployment_hostname_suffix }}"
    #port: 443
    images:
      pull_policy: IfNotPresent
      resource_registry:
        repository: cp.icr.io/cp/cp4a/aae/dba-etcd
        tag: 21.0.3-IF002
    admin_secret_name: resource-registry-admin-secret
    replica_size: 1
    probe:
      liveness:
        initial_delay_seconds: 60
        period_seconds: 10
        timeout_seconds: 5
        success_threshold: 1
        failure_threshold: 3
      readiness:
        initial_delay_seconds: 10
        period_seconds: 10
        timeout_seconds: 5
        success_threshold: 1
        failure_threshold: 3
    resource:
      limits:
        cpu: "500m"
        memory: "512Mi"
      requests:
        cpu: "100m"
        memory: "256Mi"
    auto_backup:
      enable: true
      minimal_time_interval: 300
      pvc_name: "{{ meta.name }}-dba-rr-pvc"
      log_pvc_name: 'cp4a-shared-log-pvc'
      dynamic_provision:
        enable: true
        size: 3Gi
        size_for_logstore:
        storage_class: "{{ shared_configuration.storage_configuration.sc_fast_file_storage_classname }}"

  application_engine_configuration:
  ## The application_engine_configuration is a list, you can deploy multiple instances of AppEngine, you can assign different configurations for each instance.
  ## For each instance, application_engine_configuration.name, database, admin_secret_name and hostname must be assigned to different values.
  ## Each application_engine_configuration.name can consist of lowercase alphanumeric characters or '-', and must start and end with an alphanumeric character. Keep the instance name as short as possible.
  ## You should use different database, admin_secret_name, hostname for playback server and the application engine servers
  - name: workspace
    images:
      pull_policy: IfNotPresent
      solution_server:
        repository: cp.icr.io/cp/cp4a/aae/solution-server
        tag: 21.0.3-IF002
      db_job:
        repository: cp.icr.io/cp/cp4a/aae/solution-server-helmjob-db
        tag: 21.0.3-IF002
    # If you inputed hostname and port here. They will be used always
    # If you are using pattern mode (the shared_configuration.sc_deployment_patterns contains value)
    # Then you don't need to fill the hostname and port. It will use shared_configuration.sc_deployment_hostname_suffix to generate one
    # But if you haven't input suffix. And no hostname port assigned. A error will be reported in operator log during deploy
    # For non pattern mode you must assign a valid hostname and port here
    hostname: "{{ 'ae-workspace-' + shared_configuration.sc_deployment_hostname_suffix }}"
    port: 443
    # Inside the admin secret. There are two must fields
    admin_secret_name: "workspace-aae-app-engine-admin-secret"
    #-----------------------------------------------------------------------
    # The app engine admin Secret template will be
    #-----------------------------------------------------------------------
    # apiVersion: v1
    # stringData:
    #   AE_DATABASE_PWD: "<Your database password>"
    #   AE_DATABASE_USER: "<Your database username>"
    #   REDIS_PASSWORD: "<Your Redis server password>"
    # kind: Secret
    # metadata:
    #   name: icp4adeploy-workspace-aae-app-engine-admin-secret
    # type: Opaque
    #-----------------------------------------------------------------------
    # Designate an existing LDAP user for the Application Engine admin user.
    # This user ID should be in the IBM Business Automation Navigator administrator role, as specified as appLoginUsername in the Navigator secret.
    # This user should also belong to the User Management Service (UMS) Teams admin group or the UMS Teams Administrators team.
    # If not, follow the instructions in "Completing post-deployment tasks for Business Automation Studio and Application Engine" in the IBM Documentation to add it to the Navigator Administrator role and UMS team server admin group.
    admin_user: "ibpmServices_flt"
    external_tls_secret:
    external_connection_timeout: 90s
    replica_size: 1
    # data_persistence is for Business Automation Application Data Persistence(ae_data_persistence).
    # If you are using pattern mode, the shared_configuration.sc_deployment_patterns contains value and sc_optional_components contains ae_data_persistence, then you do not need input any value to data_persistence.enable, it is enabled by default.
    # If you are using non-pattern mode, you can set data_persistence.enable to true to enable it.
    # Notes: ae_data_persistence is not supported in starter pattern mode and when AE is as playback server
    data_persistence:
        enable:
        ## If ae_data_persistence is enabled. Then you must input one CPE object store name. If you keep the default object store configuration. Then the default name filled should be AEOS.
        object_store_name: "AEOS"
    ## When the database type is Db2 you can set this to false, must be set to true when the database type is Oracle, PostgreSQL.
    use_custom_jdbc_drivers: true
    service_type: Ingress
    autoscaling:
      enabled: false
      max_replicas: 5
      min_replicas: 2
      target_cpu_utilization_percentage: 80
    server_identifier: ""
    database:
      # AE Database host name or IP when the database type is Db2, PostgreSQL, SQLSERVER.
      host: "cdldvddp007or-scan.es.ad.adp.com"
      # AE Database name when the database type is Db2, PostgreSQL, SQLSERVER.
      #Provide the database name for runtime application engine use
      #Please pay attention that if you selected authoring environment also.
      #The database used by playback server and this one should be different
      #name: "AEOSDB"
      # AE database port number when the database type is Db2, PostgreSQL, SQLSERVER.
      #port: "1521"
      ## If you setup Db2 HADR or PostgreSQL, SQLSERVER Connection Fail-over and want to use it, you need to configure alternative_host and alternative_port, or else, leave is as blank.
      ## If more than one server name is specified, delimit the server names with commas (,). The number of values that is specified for alternative_host must match the number of values that is specified for alternative_port.
      #alternative_host:
      #alternative_port:
      ## Only Db2, Oracle, PostgreSQL, SQLSERVER are supported.
      type: postgresql
      ## Required only when the database type is Oracle, both ssl and non-ssl. The format must be purely Oracle descriptor like (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<your database host/IP>)(PORT=<your database port>))(CONNECT_DATA=(SERVICE_NAME=<your Oracle service name>))).
      ##oracle_url_without_wallet_directory: (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=cdldvddp007or-scan.es.ad.adp.com)(PORT=1521))(CONNECT_DATA=(SERVICE_NAME=ibp021d_svc1)))
      enable_ssl: false
      ## Required only when type is Oracle and enable_ssl is true. The format must be purely oracle descriptor. SSO wallet directory must be specified and fixed to (MY_WALLET_DIRECTORY=/shared/resources/oracle/wallet).
      #oracle_url_with_wallet_directory:
      ## Required only when enable_ssl is true, when the database type is Db2, Oracle, SQLSERVER or PostgreSQL
      db_cert_secret_name: ""
      ## Required only when type is oracle and enable_ssl is true.
      #oracle_sso_wallet_secret_name:
      ## Optional. If it is empty, the DBASB is default when the database type is Db2 or PostgreSQL; the AE_DATABASE_USER set in the admin_secret_name is default when the database type is Oracle and SQLSERVER.
      current_schema: AE_DB_USER
      initial_pool_size: 1
      max_pool_size: 100
      max_lru_cache_size: 1000
      max_lru_cache_age: 600000
      dbcompatibility_max_retries: 30
      dbcompatibility_retry_interval: 10
      ## The persistent volume claim for custom JDBC Drivers if using the custom JDBC drivers is enabled(use_custom_jdbc_drivers is true).
      custom_jdbc_pvc: "operator-shared-pvc"
    log_level:
      node: info
      browser: 2
    content_security_policy:
      enable: false
      whitelist:
      frame_ancestor:
    env:
      max_size_lru_cache_rr: 1000
      server_env_type: development
      purge_stale_apps_interval: 86400000
      # Number of preview-only automation application must be more to trigger purge,
      apps_threshold: 100
      # Age of preview-only automation application since publish to be stale in milliseconds
      stale_threshold: 172800000
      # Number of preview-only automation services must be more to trigger purge,
      service_threshold: 100
      # Age of preview-only automation service since publish to be stale in milliseconds
      service_stale_threshold: 172800000
      # Service socket connection timeout in milliseconds
      connection_timeout: 120000
      uv_thread_pool_size: 40
      # Set custom enviroment varaible on app engine pods
      custom_environment_variables:
      # # example entry for setting timezone on pod
      # - key: TZ
      #   value: Europe/Warsaw
    max_age:
      auth_cookie: "900000"
      csrf_cookie: "3600000"
      static_asset: "2592000"
      hsts_header: "2592000"
    probe:
      liveness:
        failure_threshold: 5
        initial_delay_seconds: 60
        period_seconds: 10
        success_threshold: 1
        timeout_seconds: 180
      readiness:
        failure_threshold: 5
        initial_delay_seconds: 10
        period_seconds: 10
        success_threshold: 1
        timeout_seconds: 180
    #-----------------------------------------------------------------------
    # If you want better HA experience.
    # - Set the session.use_external_store to true
    # - Fill in your redis server information
    #-----------------------------------------------------------------------
    redis:
      # Your external redis host/ip
      host:
      # Your external redis port
      port: '6379'
      ttl: 1800
      # If your redis enabled TLS connection set this to true
      # You should add redis server CA certificate in tls_trust_list or trusted_certificate_list
      tls_enabled: false
      # If you are using Redis V6 and above with username fill in this field.
      # Otherwise leave this field as empty
      username:
    resource_ae:
      limits:
        cpu: 500m
        memory: 1Gi
      requests:
        cpu: 300m
        memory: 256Mi
    resource_init:
      limits:
        cpu: 500m
        memory: 256Mi
      requests:
        cpu: 100m
        memory: 128Mi
    session:
      check_period: "3600000"
      duration: "1800000"
      max: "10000"
      resave: "false"
      rolling: "true"
      save_uninitialized: "false"
      #-----------------------------------------------------------------------
      # If you want better HA experience.
      # - Set the session.use_external_store to true
      # - Fill in your redis server information
      #-----------------------------------------------------------------------
      use_external_store: "false"
    tls:
      tls_trust_list: []
    # If you want to make the replicate size more than 1 for this cluster. Then you must enable the shared storage
    share_storage:
      enabled: true
      # If you create the PV manually. Then please provide the PVC name bind here
      pvc_name:
      auto_provision:
        enabled: true
        # Required if you enabled the auto provision
        storage_class:
        size: 20Gi
    log_storage:
      enabled: true
      pvc_name: 'cp4a-shared-log-pvc'
      log_file_size: '20M'
      log_rotate_size: 5
      auto_provision:
        enabled: true
        # By default it will reuse the operator shared log pvc. If you assgined other name
        # And enabled the auto provision. We will provision that with fast storage class by default
        # If you want to adjust that please fill in this value.
        storage_class: ""
        size: '5Gi'



  ########################################################################
  ########   IBM Business Automation Workflow configuration     ########
  ########################################################################
  baw_configuration:
  ## The baw_configuration is a list. You can deploy multiple instances of Workflow Server and assign different configurations for each instance.
  ## For each instance, baw_configuration.name and hostname must be assigned different values.
  ## For each instance's database configuration, you can choose to use either different database instances, or one shared database instance. If you use a shared database instance, in Db2 or PostgreSQL, you must assign different database names (baw_configuration[x].database.database_name); in Oracle, you must assign different database users (the dbUser in the baw_configuration[x].database.secret_name).
  ## Each baw_configuration.name can consist of lowercase alphanumeric characters or '-', and must start and end with an alphanumeric character. Keep the instance name as short as possible.
  ## For baw_configuration.tls.tls_secret_name, if you choose to use a customized Workflow Server TLS certificate, ensure that each BAW instance has a different value.
  - name: bawins1
    ## Whether the Business Automation Workflow instance hosts federated Process Portal and integrates with Intelligent Task Prioritization.
    host_federated_portal: true
    ## Workflow Server service type.
    service_type: "Ingress"
    ## Workflow Server hostname.
    hostname: "{{ 'bawins1-' + shared_configuration.sc_deployment_hostname_suffix }}"
    ## Workflow Server port.
    port: 443
    ## Workflow Server node port.
    nodeport: 30026
    ## Workflow Server environment type. Possible values are Development, Test, Staging, and Production.
    env_type: "Production"
    ## Workflow Server capability.
    capabilities: "workflow"
    ## Workflow Server replica count.
    replicas: 1
    ## Designate an existing LDAP user for the Workflow Server admin user.
    admin_user: "ibpmServices_flt"
    ## The name of Workflow Server admin secret. This secret name is optional, if the secret name is null, default secret named {{ meta.name }}-<instance-name>-baw-admin-secret will be generated.
    admin_secret_name: "{{ meta.name }}-instance1-baw-admin-secret"
    ## Whether to use the built-in monitoring capability.
    monitor_enabled: false

    # Optional: You can specify a profile size for BAW Server if different from BAW (see shared_configuration.sc_deployment_profile_size).  In other words,
    # you can override "shared_configuration.sc_deployment_profile_size" with "deployment_profile_size" defined here.
    # The valid values are small, medium, large - default is small.  The resources in this file are reflecting a "small" profile.
    #deployment_profile_size: "small"

    ## Required if you implemented your own portal. For example, https://portal.mycompany.com.
    customized_portal_endpoint: ""
    federated_portal:
      ## Content security policy additional origins for federating on premise BAW systems. E.g ["https://on-prem-baw1","https://on-prem-baw2"]
      content_security_policy_additional_origins: []
    ## External connection timeout.
    external_connection_timeout: ""

    ## The secret that contains the Transport Layer Security (TLS) key and certificate for external https visits. You can enter the secret name here.
    ## If you do not want to use a customized external TLS certificate, leave it empty.
    external_tls_secret:
    ## Certificate authority (CA) used to sign the external TLS secret. It is stored in the secret with the TLS key and certificate. You can enter the secret name here.
    ## If you don't want to use a customized CA to sign the external TLS certificate, leave it empty.
    external_tls_ca_secret:

    tls:
      ## Workflow Server TLS secret that contains tls.key and tls.crt.
      ## If you want to use a customized Workflow Server TLS certificate, ensure it is signed by the CA in shared_configuration.root_ca_secret.
      ## If you do not want to use a customized Workflow Server TLS certificate, leave it empty.
      #tls_secret_name: ibm-baw-tls
      ## Workflow Server TLS trust list.
      ## You might specify a list of secrets, every secret stores a trusted CA
      ## Use command `kubectl create secret generic baw_custom_trust_ca_secret1 --from-file=tls.crt=./ca1.crt` to generate the secret.
      tls_trust_list:
      ## Secret to store your custom trusted keystore (optional). The type for the keystore must be JKS or PKCS12. All certificates from the keystore are imported into the trust keystore of the Workflow Server.
      ## You might run the following sample command to create the secret:
      ## `kubectl create secret generic baw_custom_trusted_keystore_secret --from-file=truststorefile=./trust.p12 --from-literal=type=PKCS12  --from-literal=password=WebAS`
      tls_trust_store:

    image:
      ## Workflow Server (Process Server) image repository URL.
      repository: cp.icr.io/cp/cp4a/baw/workflow-server
      ## Image tag for Workflow Server container.
      tag: 21.0.3-IF002
      ## Pull policy for Workflow Server container. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
      pullPolicy: IfNotPresent
    pfs_bpd_database_init_job:
      ## Database initialization image repository URL for Process Federation Server.
      repository: cp.icr.io/cp/cp4a/baw/pfs-bpd-database-init-prod
      ## Database initialization image repository tag for Process Federation Server.
      tag: 21.0.3-IF002
      ## Pull policy for Process Federation Server database initialization image. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
      pullPolicy: IfNotPresent
    upgrade_job:
      ## Workflow Server database handling image repository URL.
      repository: cp.icr.io/cp/cp4a/baw/workflow-server-dbhandling
      ## Workflow Server database handling image repository tag.
      tag: 21.0.3-IF002
      ## Pull policy for database handling. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
      pullPolicy: IfNotPresent
    bas_auto_import_job:
      ## Workflow Server Business Automation Studio toolkit init image repository URL.
      repository: cp.icr.io/cp/cp4a/baw/toolkit-installer
      ## Workflow Server Business Automation Studio toolkit init image repository tag.
      tag: 21.0.3-IF002
      ## Pull policy for Workflow Server Business Automation Studio toolkit init image. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
      pullPolicy: IfNotPresent
    ibm_workplace_job:
      ## Workflow Server IBM Workplace deployment job image repository URL
      repository: cp.icr.io/cp/cp4a/baw/iaws-ibm-workplace
      ## Workflow Server IBM Workplace deployment job image repository tag.
      tag: 21.0.3-IF002
      ## Pull policy for Workflow Server IBM Workplace deployment job image. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
      pull_policy: IfNotPresent

    ## The database configuration for Workflow Server
    database:
      ## Whether to enable Secure Sockets Layer (SSL) support for the Workflow Server database connection.
      enable_ssl: false
      ## Secret name for storing the database TLS certificate when an SSL connection is enabled, e.g  {{ meta.name }}-db-ssl-secret.
      ## Required only when enable_ssl is true. If enable_ssl is false, comment out this line.
      db_cert_secret_name: ""
      ## Workflow Server database type. Possible values are: db2, oracle, postgresql
      type: "postgresql"
      ## Workflow Server database server name. It must be an accessible address, such as IP, hostname, or Kubernetes service name.
      ## This parameter is required.
      server_name: "10.77.225.3"
      ## Workflow Server database name. This parameter is required.
      database_name: "bawdb"
      ## Workflow Server database port. This parameter is required. For DB2, the default value is "50000"
      port: "5432"
      ## Workflow Server database secret name. This parameter is required.
      ## apiVersion: v1
      ## kind: Secret
      ## metadata:
      ##   name: ibm-baw-wfs-server-db-secret
      ## type: Opaque
      ## data:
      ##   dbUser: <DB_USER>
      ##   password: <DB_USER_PASSWORD>
      secret_name: "ibm-baw-wfs-server-db-secret"
      ## Oracle and PostgreSQL database connection string.
      ## If the database type is Oracle, provide the Oracle database connection string. For example, jdbc:oracle:thin:@//<oracle_server>:1521/orcl.
      ## If the database type is PostgreSQL, this parameter is optional, you can choose inputs server_name, database_name, and port with or without this parameter here. If you do not need this parameter when PostgreSQL, remove or comment this parameter.
      ## In any other cases, remove or comment this parameter.
      #jdbc_url: "jdbc:oracle:thin:@//cdldvddp007or-scan.es.ad.adp.com:1521/ibp021d_svc1"
      ## Whether to use custom JDBC drivers. Set to true if you are using Oracle, PostgreSQL, or a special Db2 driver.
      use_custom_jdbc_drivers: true
      ## If use_custom_jdbc_drivers is set to true, input the name of the persistent volume claim (PVC) that binds to the persistent volume (PV) where the custom JDBC driver files are stored.
      ## If use_custom_jdbc_drivers is set to false, remove or comments this parameter.
      custom_jdbc_pvc: "operator-shared-pvc"
      ## The set of JDBC driver files.
      ## For DB2, it is normally "db2jcc4.jar db2jcc_license_cisuz.jar db2jcc_license_cu.jar".
      ## For Oracle, it is normally "ojdbc8.jar".
      ## For PostgreSQL, it is normally "postgresql-42.2.14.jar".
      jdbc_driver_files: "postgresql-42.4.0.jar"
      ## Workflow Server database connect pool maximum number of physical connections.
      cm_max_pool_size: 200
      dbcheck:
        ## The maximum wait time (seconds) to check the database intialization status. The server fails to start after wait_time.
        wait_time: 900
        ## The interval time (seconds) to check the database initialization status before the database is ready and bootstrapped with system data.
        interval_time: 15
      #hadr:
        ## Database standby host for high availability disaster recovery (HADR)
        ## To enable database HADR, configure both standby host and port.
        #standbydb_host:
        ## Database standby port for HADR. To enable database HADR, configure both standby host and port.
        #standbydb_port:
        ## Retry interval for HADR
        #retryinterval:
        ## Maximum retries for HADR
        #maxretries:

    ## Workflow center configuration
    workflow_center:
      ## The URL of workflow Center
      url: ""
      ## The secret name of workflow center that contains username and password.
      ## apiVersion: v1
      ## kind: Secret
      ## metadata:
      ##  name: ibm-baw-wc-secret
      ## type: Opaque
      ## stringData:
      ##  username: deadmin
      ##  password: deadmin
      secret_name: ""
      ## Heartbeat interval (seconds) to connect to Workflow Center.
      heartbeat_interval: 30
      ## URL that is used by Workflow Center to link to the web Process Designer. For example, https://hostname:port/WebPD.
      webpd_url: ""

    ## The configurations for content integration for attachment in process
    content_integration:
      init_job_image:
        ## Image name for content integration container.
        repository: cp.icr.io/cp/cp4a/baw/iaws-ps-content-integration
        ## Image tag for content integration container.
        tag: 21.0.3-IF002
        ## Pull policy for content integration container. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
        pull_policy: IfNotPresent
      ## Domain name for content integration. The value must be the same as initialize_configuration.ic_domain_creation.domain_name.
      domain_name: "P8DOMAIN"
      ## Object Store name for content integration.
      ## The value must be an existing object store in CPE.
      ## If use initialize_configuration for the object store initilization, the value must be one of initialize_configuration.ic_obj_store_creation.object_stores.
      object_store_name: "BAWINS1DOCS"
      ## Admin secret for connecting to Content Platform Engine (CPE). This parameter is optional. If not set, it will autodetect CPE's admin secret in the same namespace.
      cpe_admin_secret: ""

    ## The configuration for case
    case:
      init_job_image:
        ## Image name for CASE init job container.
        repository: cp.icr.io/cp/cp4a/baw/workflow-server-case-initialization
        ## Image tag for CASE init job container.
        tag: 21.0.3-IF002
        ## Pull policy for CASE init job container. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
        pull_policy: IfNotPresent

      ## Domain name for CASE. The value must be the same as initialize_configuration.ic_domain_creation.domain_name.
      domain_name: "P8DOMAIN"
      ## Design Object Store name of CASE.
      ## The value must be the same as the oc_cpe_obj_store_symb_name value of one of the object stores defined in initialize_configuration.ic_obj_store_creation.object_stores.
      object_store_name_dos: "BAWINS1DOS"
      ## Target Object Store name of CASE
      ## The value must be the same as the oc_cpe_obj_store_symb_name value of one of the object stores defined in initialize_configuration.ic_obj_store_creation.object_stores.
      object_store_name_tos: "BAWINS1TOS"
      ## Connection point name for Target Object Store.
      ## See initialize_configuration.ic_obj_store_creation.object_stores[x].oc_cpe_obj_store_workflow_pe_conn_point_name.
      ## If oc_cpe_obj_store_workflow_pe_conn_point_name is not specified explicitly, the default value is pe_conn_<TOS_OS_DB_NAME>.
      connection_point_name_tos: "cpe_conn_tos"
      ## Specify the name of the non-XA datasource of Target Object Store (from dc_common_os_datasource_name in the dc_os_datasources section)
      ## It will not take effect if TOS object store exists in initialize_configuration.ic_obj_store_creation.object_stores.
      datasource_name_tos: "BAWINS1TOS"

      ## Name of the target environment or project area to register with the case components and associate with an IBM Content Navigator desktop.
      target_environment_name: "target_env"

      ## Persistent volume claim (PVC) name for case network shared directory.
      ## This parameter must be set to the same value as the Business Automation Navigator pvc_for_icn_pluginstore parameter.
      ## If navigator_configuration.datavolume.existing_pvc_for_icn_pluginstore is not specified explicitly, the default value is icn-pluginstore.
      network_shared_directory_pvc: "{{ navigator_configuration.datavolume.existing_pvc_for_icn_pluginstore | default('icn-pluginstore', true) }}"
      ## Custom package names for installing custom packages, where the value format is similar to package1.zip, package2.zip.
      custom_package_names: ""
      ## Custom extension names for installing custom packages, where the value format is similar to extension1.zip, extension2.zip.
      custom_extension_names: ""

    ## Application engine configuration, because application engine is an array,
    ## when there is only one Application engine deployed along with this CR, below four parameters are not required.
    ## when there is more then one application engine deployed, below four parameters are required.
    appengine:
      ## App Engine hostname
      hostname: ""
      ## App Engine port
      port: "443"
      ## App Engine admin secret name
      admin_secret_name: ""
      ## App Engine context root.
      ## The application default context root will be /{{ meta.name}}-{{ application engine instance name }}-aae
      ## Please adjust this value according to the App Engine instance you selected. Don't miss the slash in front of value
      context_root: ""

    ## The configuration for Java Messaging Service(JMS)
    jms:
      image:
        ## Image name for Java Messaging Service container.
        repository: cp.icr.io/cp/cp4a/baw/jms
        ## Image tag for Java Messaging Service container.
        tag: 21.0.3-IF002
        ## Pull policy for Java Messaging Service container. Default value is IfNotPresent. Possible values are IfNotPresent, Always.
        pull_policy: IfNotPresent
      tls:
        ## TLS secret name for Java Message Service (JMS)
        tls_secret_name: ibm-jms-tls-secret
      resources:
        limits:
          ## Memory limit for JMS configuration
          memory: "1Gi"
          ## CPU limit for JMS configuration
          cpu: "1000m"
        requests:
          ## Requested amount of memory for JMS configuration
          memory: "512Mi"
          ## Requested amount of CPU for JMS configuration
          cpu: "100m"
      storage:
        ## Whether to enable persistent storage for JMS
        persistent: true
        ## Size for JMS persistent storage
        size: "1Gi"
        ## Whether to enable dynamic provisioning for JMS persistent storage
        use_dynamic_provisioning: true
        ## Access modes for JMS persistent storage
        access_modes:
        - ReadWriteOnce
        ## Storage class name for JMS persistent storage
        storage_class: "{{ shared_configuration.storage_configuration.sc_fast_file_storage_classname }}"
      ## Default values for liveness probes. Modify these values to meet your requirements.
      liveness_probe:
        ## Number of seconds after the Workflow Server container starts before the liveness probe is initiated
        initial_delay_seconds: 180
        ## Number of seconds to wait before the next probe.
        period_seconds: 20
        ## Number of seconds after which the probe times out.
        timeout_seconds: 10
        ## When a probe fails, number of times that Kubernetes will try before giving up and restarting the container.
        failure_threshold: 3
        ## Minimum consecutive successes for the probe to be considered successful after it failed.
        success_threshold: 1
      ## Default values for rediness probes. Modify these values to meet your requirements.
      readiness_probe:
        ## Number of seconds after the Workflow Server container starts before the liveness probe is initiated
        initial_delay_seconds: 30
        ## Number of seconds to wait before the next probe.
        period_seconds: 5
        ## Number of seconds after which the probe times out.
        timeout_seconds: 5
        ## When a probe fails, number of times that Kubernetes will try before giving up and restarting the container.
        failure_threshold: 6
        ## Minimum consecutive successes for the probe to be considered successful after it failed.
        success_threshold: 1

    ## Resource configuration for init job
    resources_init:
      limits:
        ## CPU limit for init job containers.
        cpu: "500m"
        ## Memory limit for init job containers.
        memory: 256Mi
      requests:
        ## Requested amount of CPU for init job containers.
        cpu: "200m"
        ## Requested amount of memory for init job containers.
        memory: 128Mi

    ## Resource configuration for heavy init job such as database init job
    resources_init_heavy_job:
      limits:
        ## CPU limit for Workflow Server database init job container.
        cpu: 1
        ## Memory limit for Workflow Server database init job container.
        memory: 2048Mi
      requests:
        ## Requested amount of CPU for Workflow Server database init job container.
        cpu: "500m"
        ## Requested amount of memory for Workflow Server database init job container.
        memory: 512Mi

    ## Resource configuration
    resources:
      limits:
        ## CPU limit for Workflow Server.
        cpu: 2
        ## Memory limit for Workflow Server.
        memory: 3060Mi
      requests:
        ## Requested amount of CPU for Workflow Server.
        cpu: "500m"
        ## Requested amount of memory for Workflow Server.
        memory: 2048Mi

    ## liveness and readiness probes configuration
    probe:
      ws:
        liveness_probe:
          ## Number of seconds after the Workflow Server container starts before the liveness probe is initiated
          initial_delay_seconds: 300
          ## Number of seconds to wait before the next probe.
          period_seconds: 10
          ## Number of seconds after which the probe times out.
          timeout_seconds: 10
          ## When a probe fails, number of times that Kubernetes will try before giving up and restarting the container.
          failure_threshold: 3
          ## Minimum consecutive successes for the probe to be considered successful after it failed.
          success_threshold: 1
        readinessProbe:
          ## Number of seconds after the Workflow Server container starts before the readiness probe is initiated
          initial_delay_seconds: 240
          ## Number of seconds to wait before the next probe.
          period_seconds: 5
          ## Number of seconds after which the probe times out.
          timeout_seconds: 5
          ## When a probe fails, number of times that Kubernetes will try before giving up and restarting the container.
          failure_threshold: 6
          ## Minimum consecutive successes for the probe to be considered successful after it failed.
          success_threshold: 1

    ## log trace configuration
    logs:
      ## Format for printing logs on the console.
      console_format: "json"
      ## Log level for printing logs on the console.
      console_log_level: "INFO"
      ## Source of the logs for printing on the console.
      console_source: "message,trace,accessLog,ffdc,audit"
      ## Required format for the messages.log file. Valid values are SIMPLE and JSON.
      message_format: "SIMPLE"
      ## Format of the trace log. Options values are: BASIC, ADVANCED, and ENHANCED.
      trace_format: "ENHANCED"
      ## Specification for printing trace logs.
      trace_specification: "*=info"
      # Maximum number of log files that are kept before the oldest file is removed.
      max_files: 10
      # The maximum size (in MB) that a log file can reach before it is rolled.
      max_filesize: 50

    ## storage configuration
    storage:
      ## Set to true to use dynamic storage provisioning. If set to false, you must set existing_pvc_for_logstore and existing_pvc_for_dumpstore.
      use_dynamic_provisioning: true
      ## Persistent volume claim (PVC) for logs.
      existing_pvc_for_logstore: ""
      ## Minimum size of the persistent volume (PV) that is mounted as the log store.
      size_for_logstore: "1Gi"
      ## Persistent volume claim (PVC) for dump files.
      existing_pvc_for_dumpstore: ""
      ## Minimum size of the persistent volume (PV) that is mounted as the dump store.
      size_for_dumpstore: "5Gi"
      ## Persistent volume claim (PVC) for generic files.
      existing_pvc_for_filestore: ""
      ## Minimum size of the persistent volume (PV) that is mounted as the generic file store.
      size_for_filestore: "1Gi"

    ## autoscaling
    autoscaling:
      ## Whether to enable automatically scaling the number of pods.
      enabled: false
      ## Upper limit for the number of pods that can be set by the autoscaler. If it is not specified or negative, the server uses the default value.
      max_replicas: 3
      ## Lower limit for the number of pods that can be set by the autoscaler. If it is not specified or negative, the server uses the default value.
      min_replicas: 2
      ## Target average CPU utilization (represented as a percent of requested CPU) over all the pods. If it is not specified or negative, the default is used.
      target_cpu_utilization_percentage: 80

    ## customize environment settings
    environment_config:
      ## Whether to show the Intelligent Task Prioritization service toggle button in the web user interface to allow the user to enable or disable this service. This parameter is valid only for the first Workflow instance.
      show_task_prioritization_service_toggle: false
      ## Whether to display the Intelligent Task Prioritization service toggle button. If this parameter is set to true, the previous parameter is ignored. This parameter is valid only for the first Workflow instance.
      always_run_task_prioritization_service: false
      csrf:
        ## Acceptable values in the Origin header field. The value of this property must be a comma-separated list of prefixes in the format protocol://host:port, e.g "https://example.com, http://example2.com:8080".
        origin_whitelist:
        ## Acceptable values in the Referer header field. The value of this property must be a comma-separated list of fully qualified host names, e.g "example1.com, example2.com".
        referer_whitelist:
        ## Comma-separated list of user agents. For the REST API requests with the path pattern “/rest/bpm/wle/v1/*” that is sent by the agents in the list, the server will not validate the “XSRF-TOKEN” cookie. The value of this property must be a comma-separated list, e.g “agentkeyworkd1, agentkeyworkd2”.
        user_agent_keyword_white_list_for_old_restapi_csrf_check: "java,wink client,httpclient,curl,jersey,httpurlconnection"
        ## Whether to validate the cookie "XSRF-TOKEN" against incoming REST API requests (POST/PUT/DELETE) with the path pattern "/rest/bpm/wle/v1/*". The value must be "true" or "false".
        check_xsrf_for_old_restapi: "true"
      ## Whether to disable the compliance with Federal Information Processing Standards (FIPS) requirements
      disable_fips: false

    ## federation config
    federation_config:
      workflow_server:
          ## Number of primary shards of the Elasticsearch index used to store Workflow Server data.
          index_number_of_shards: 3
          ## Number of shard replicas of the Elasticsearch index used to store Workflow Server data.
          index_number_of_replicas: 1
      case_manager:
            ## Case Manager object store name.
          - object_store_name: BAWINS1TOS
            ## Number of primary shards of the Elasticsearch index used to store Case Manager object store data.
            index_number_of_shards: 3
            ## Number of shard replicas of the Elasticsearch index used to store Case Manager object store data.
            index_number_of_replicas: 1

    ## JVM options separated with spaces, for example: -Dtest1=test -Dtest2=test2.
    jvm_customize_options:

    ##  Workflow Server custom plain XML snippet
    ##  liberty_custom_xml: |+
    ##    <server>
    ##      <!-- custom propeties here -->
    ##    </server>
    ## The custom_xml_secret_name is also used for Workflow Server customization. Ensure the configuration value exists either in liberty_custom_xml or custom_xml_secret_name.
    ## Do not set the configuration value in both places.
    liberty_custom_xml:

    ## Workflow Server custom XML secret name that contains custom configuration in Liberty server.xml,
    ## put the custom.xml in secret with key "sensitiveCustomConfig"
    ## kubectl create secret generic wfs-custom-xml-secret --from-file=sensitiveCustomConfig=./custom.xml
    ## The liberty_custom_xml property is also used for Workflow Server customization. Ensure the configuration value exists either in liberty_custom_xml or custom_xml_secret_name.
    ## Do not set the configuration value in both places.
    custom_xml_secret_name:

    ## Workflow Server Lombardi custom XML secret name that contains custom configuration in 100Custom.xml
    ## put the 100Custom.xml in secret with key "sensitiveCustomConfig"
    #  kubectl create secret generic wfs-lombardi-custom-xml-secret --from-file=sensitiveCustomConfig=./100Custom.xml
    lombardi_custom_xml_secret_name:

  ########################################################################
  ########   IBM Process Federation Server configuration          ########
  ########################################################################
  pfs_configuration:
    ## If you input hostname and port here, they will always be used.
    ## If you are using pattern mode (the shared_configuration.sc_deployment_patterns contains value),
    ## you don't need to fill in the hostname and port: the shared_configuration.sc_deployment_hostname_suffix will be used to generate them.
    ## But if you didn't input a suffix, and no hostname port is assigned, an error will be reported in the operator log during deployment.
    ## For non pattern mode you must assign a valid hostname and port here.
    ## Process Federation Server hostname
    #hostname: ""
    ## Process Federation Server port
    #port: 443
    ## How the HTTPS endpoint service should be published. Possible values are ClusterIP, NodePort, Route
    service_type: Route

    # Optional: You can specify a profile size for BAW Server if different from BAW (see shared_configuration.sc_deployment_profile_size).  In other words,
    # you can override "shared_configuration.sc_deployment_profile_size" with "deployment_profile_size" defined here.
    # The valid values are small, medium, large - default is small.  The resources in this file are reflecting a "small" profile.
    #deployment_profile_size: "small"

    ## The elasticsearch settings which will be used by PFS
    elasticsearch:
      ## Required only when using external elasticsearch. Endpoint of external elasticearch, such as: https://<external_es_host>:<external_es_port>.
      endpoint: ""
      ## Required only when using external elasticsearch. The external elasticsearch administrative secret that contains the keys: username and password.
      admin_secret_name: ""
      ## The number of seconds for elasticsearch connection timeout setting.
      connect_timeout: 10s
      ## The number of seconds for elasticsearch read timeout setting.
      read_timeout: 30s
      ## elasticsearch thread count setting
      thread_count: 0
      ## The maximum number of connections allowed across all routes when PFS connects to the elasticsearch cluster to call its REST API.
      ## Specify a positive integer. If the provided value is less than or equal to 0, then the default Elasticsearch High Level REST Client value will be used.
      max_connection_total: -1
      ## The maximum number of connections allowed for a route when PFS connects to the elasticsearch cluster to call its REST API.
      ## Specify a positive integer. If the provided value is less than or equal to 0, then the default Elasticsearch High Level REST Client value will be used.
      max_connection_per_route: -1

    ## This is the image repository and tag that correspond to image registry, which is where the image will be pulled.
    image:
      ## Process Federation Server image
      ## The default repository is the IBM Entitled Registry.
      repository: cp.icr.io/cp/cp4a/baw/pfs-prod
      ## Process Federation Server image tag
      tag: "21.0.3-IF002"
      ## Process Federation Server image pull policy. This will override the image pull policy in the shared_configuration.
      pull_policy: IfNotPresent

    ## The number of initial Process Federation Server pods.
    replicas: 1
    ## Service account name for Process Federation Server pod. If not set, the default service account named "{{ meta.name }}-pfs-service-account" will be created.
    service_account:
    ## Whether Kubernetes can (soft) or must not (hard) deploy Process Federation Server pods onto the same node. Possible values are "soft" and "hard".
    anti_affinity: hard

    ## Whether to enable notification server. Possible values are: true and false
    enable_notification_server: true
    ## Whether to enable default security roles. Possible values are: true and false
    enable_default_security_roles: true
    ## Designate a list of users for the Process Federation Server administrator by entering the distinguished name for the LDAP user, e.g "uid=cp4baAdminUser,ou=cp4ba,dc=company,dc=com".
    ## This parameter is only used when you set enable_default_security_roles to true.
    admin_user_id:

    ## Designate a list of groups for the Process Federation Server administrator by entering the distinguished name for the LDAP group, e.g "CN=cp4baAdminGroup,ou=cp4ba,dc=company,dc=com".
    ## This parameter is only used when you set enable_default_security_roles to true.
    admin_group_id:

    ## Name of the secret containing the Process Federation Server administration passwords, such as ltpaPassword, oidcClientPassword, sslKeyPassword
    admin_secret_name: ibm-pfs-admin-secret
    ## Name of the secret containing the files that will be mounted in the /config/configDropins/overrides folder
    config_dropins_overrides_secret: ""
    ## Name of the secret containing the files that will be mounted in the /config/resources/security folder
    resources_security_secret: ""
    ## Name of the custom libraries containing the files that will be mounted in the /config/resources/libs folder
    custom_libs_pvc: ""

    ## The secret that contains the Transport Layer Security (TLS) key and certificate for external https visits. You can enter the secret name here.
    ## If you do not want to use a customized external TLS certificate, leave it empty.
    external_tls_secret:
    ## Certificate authority (CA) used to sign the external TLS secret. It is stored in the secret with the TLS key and certificate. You can enter the secret name here.
    ## If you don't want to use a customized CA to sign the external TLS certificate, leave it empty.
    external_tls_ca_secret:

    ## Whether to use the built-in monitoring capability.
    monitor_enabled: false

    tls:
      ## Existing TLS secret containing keys tls.key and tls.crt. The certificate should be signed by the CA in shared_configuration.root_ca_secret.
      ## If you do not want to use a customized TLS certificate, leave it empty.
      tls_secret_name:
      ## Existing TLS trust secret list
      ## You might specify a list of secrets like the external Elastichsearch applied , every secret stores a trusted CA
      ## kubectl create secret generic pfs-custom-trust-ca-secret1 --from-file=tls.crt=./ca1.crt
      tls_trust_list:
      ## The parameter is optional, if you want PFS server to trust your custom trusted CAs, you can add them to a keystore and then create a secret to store the trusted keystore.
      ## The keystore type must be JKS or PKCS12.
      ## kubectl create secret generic pfs-custom-trusted-keystore-secret --from-file=truststorefile=./trust.p12 --from-literal=type=PKCS12  --from-literal=password=WebAS
      tls_trust_store:

    ## The initial resources (CPU, memory) requests and limits.
    ## If more resources are needed, make the changes here to meet your requirement.
    resources:
      requests:
        ## The minimum amount of CPU required to start a PFS pod.
        cpu: 200m
        ## The minimum memory required (including JVM heap and file system cache) to start a PFS pod.
        memory: 512Mi
      limits:
        ## The maximum amount of CPU to allocate to each PFS pod.
        cpu: 1
        ## The maximum memory (including JVM heap and file system cache) to allocate to each PFS pod.
        memory: 1024Mi

    ## Default values for liveness probes. Modify these values to meet your requirements.
    liveness_probe:
      ## Number of seconds after Process Federation Server container starts before the liveness probe is initiated
      initial_delay_seconds: 300

    ## Default values for readiness probes. Modify these values to meet your requirements.
    readiness_probe:
      ## Number of seconds after Process Federation Server container starts before the readiness probe is initiated
      initial_delay_seconds: 240

    ## Configurations for saved searches
    saved_searches:
      ## Name of the Elasticsearch index used to store saved searches.
      index_name: ibmpfssavedsearches
      ## Number of shards of the Elasticsearch index used to store saved searches.
      index_number_of_shards: 3
      ## Number of replicas (pods) of the Elasticsearch index used to store saved searches.
      index_number_of_replicas: 1
      ## Batch size used when retrieving saved searches.
      index_batch_size: 100
      ## Amount of time before considering an update lock as expired. Valid values are numbers with a trailing 'm' or 's' for minutes or seconds.
      update_lock_expiration: 5m
      ## Amount of time before considering a unique constraint as expired. Valid values are numbers with a trailing 'm' or 's' for minutes or seconds.
      unique_constraint_expiration: 5m

    security:
      sso:
        ## The ssoDomainNames property of the <webAppSecurity> tag.
        domain_name:
        ## The ssoCookieName property of the <webAppSecurity> tag.
        cookie_name: "ltpatoken2"
        ltpa:
          ## The keysFileName property of the <ltpa> tag.
          filename: "ltpa.keys"
          ## The expiration property of the <ltpa> tag.
          expiration: "120m"
          ## The monitorInterval property of the <ltpa> tag.
          monitor_interval: "60s"
      ## The sslProtocol property of the <ssl> tag used as default SSL config.
      ssl_protocol: SSL

    executor:
      ## Value of the maxThreads property of the <executor> tag.
      max_threads: "80"
      ## Value of the coreThreads property of the <executor> tag.
      core_threads: "40"

    rest:
      ## Value of the userGroupCheckInterval property of the <ibmPfs_restConfig> tag.
      user_group_check_interval: "300s"
      ## Value of the systemStatusCheckInterval property of the <ibmPfs_restConfig> tag.
      system_status_check_interval: "60s"
      ## Value of the bdFieldsCheckInterval property of the <ibmPfs_restConfig> tag.
      bd_fields_check_interval: "300s"

    custom_env_variables:
      ## Names of the custom environment variables defined in the secret referenced in pfs.customEnvVariables.secret
      names:
      # - name: MY_CUSTOM_ENVIRONMENT_VARIABLE
      ## Secret holding custom environment variables
      secret:

    ## Log trace configuration.
    logs:
      ## Format for printing logs on the console.
      console_format: "json"
      ## Log level for printing logs on the console.
      console_log_level: "INFO"
      ## Source of the logs for printing on the console.
      console_source: "message,trace,accessLog,ffdc,audit"
      ## The required format for the messages.log file. Valid values are SIMPLE or JSON format.
      message_format: "SIMPLE"
      ## Format for printing trace logs on the console.
      trace_format: "ENHANCED"
      ## Specification for printing trace logs.
      trace_specification: "*=info"
      storage:
        ## Use dynamic provisioning for PFS logs data storage.
        use_dynamic_provisioning: true
        ## The minimum size of the persistent volume used mounted as PFS Liberty server /logs folder.
        size: 1Gi
        ## Storage class of the persistent volume used mounted as PFS Liberty server /logs folder.
        storage_class: "{{ shared_configuration.storage_configuration.sc_medium_file_storage_classname }}"
        ## The persistent volume claim for log data storage if dynamic provisioning is not used (use_dynamic_provisioning is false).
        existing_pvc_name: ""

    ## Dump configuration.
    dump:
      ## Whether to persist dump files. Possible values are: true and false.
      persistent: false
      storage:
        ## Whether to use dynamic provisioning for PFS dump storage. Possible values are: true and false.
        use_dynamic_provisioning: true
        ## Minimum size of the persistent volume (PV) that is mounted as the dump store.
        size: 5Gi
        ## Storage class if using dynamic provisioning for PFS dump storage
        storage_class: "{{ shared_configuration.storage_configuration.sc_slow_file_storage_classname }}"
        ## The persistent volume claim for dump files if not using dynamic provisioning.
        existing_pvc_name:

    ## When PFS is deployed in an environment that includes the Resource Registry,
    ## the following additional parameters can be used to configure the integration between PFS and the Resource Registry.
    dba_resource_registry:
      ## Time to live of the lease that creates the PFS entry in the DBA Resource Registry, in seconds.
      lease_ttl: 120
      ## The interval at which to check that PFS is running, in seconds.
      pfs_check_interval: 10
      ## The number of seconds after which PFS will be considered as not running if no connection can be perfomed.
      pfs_connect_timeout: 10
      ## The number of seconds after which PFS will be considered as not running if has not yet responded.
      pfs_response_timeout: 30
      ## The key under which PFS should be registered in the DBA Service Registry when running.
      pfs_registration_key: /dba/appresources/IBM_PFS/PFS_SYSTEM
      ## The initial resources (CPU, memory) requests and limits.
      ## If more resources are needed, make the changes here to meet your requirement.
      resources:
        limits:
          ## The maximum memory to allocate to each PFS registration pod.
          memory: '512Mi'
          ## The maximum amount of CPU to allocate to each PFS registration pod.
          cpu: '100m'
        requests:
          ## The minimum memory required to start a PFS registration pod.
          memory: '512Mi'
          ## The minimum amount of CPU required to start a PFS registration pod.
          cpu: '50m'
      liveness_probe:
        ## When a probe fails, number of times that Kubernetes will try before giving up and restarting the container.
        failure_threshold: 3
        ## Number of seconds after the PFS registration container starts before the liveness probe is initiated.
        initial_delay_seconds: 5
        ## Number of seconds to wait before the next probe.
        period_seconds: 5
        ## Minimum consecutive successes for the probe to be considered successful after it failed.
        success_threshold: 1
        ## Number of seconds after which the probe times out.
        timeout_seconds: 5
      readiness_probe:
        ## When a probe fails, number of times that Kubernetes will try before giving up and restarting the container.
        failure_threshold: 3
        ## Number of seconds after the PFS registration container starts before the readiness probe is initiated.
        initial_delay_seconds: 1
        ## Number of seconds to wait before the next probe.
        period_seconds: 10
        ## Minimum consecutive successes for the probe to be considered successful after it failed.
        success_threshold: 1
        ## Number of seconds after which the probe times out.
        timeout_seconds: 5


  ########################################################################
  ########   Embedded Elasticsearch configuration                 ########
  ########################################################################
  elasticsearch_configuration:
    es_image:
      ## Elasticsearch image
      repository: cp.icr.io/cp/cp4a/baw/pfs-elasticsearch-prod
      ## Elasticsearch image tag
      tag: "21.0.3-IF002"
      ## Elasticsearch image pull policy
      pull_policy: IfNotPresent
    es_init_image:
      ## The image used by the privileged init container to configure Elasticsearch system settings.
      ## This value is only relevant if elasticsearch_configuration.privileged is set to true
      repository: cp.icr.io/cp/cp4a/baw/pfs-init-prod
      ## The image tag for Elasticsearch init container
      tag: "21.0.3-IF002"
      ## The pull policy for Elasticsearch init container
      pull_policy: IfNotPresent
    es_nginx_image:
      ## The name of the Nginx docker image to be used by Elasticsearch pods
      repository: cp.icr.io/cp/cp4a/baw/pfs-nginx-prod
      ## The image tag of the Nginx docker image to be used by Elasticsearch pods
      tag: "21.0.3-IF002"
      ## The pull policy for the Nginx docker image to be used by Elasticsearch pods
      pull_policy: IfNotPresent

    ## Number of initial Elasticsearch pods
    replicas: 1
    ## How the HTTPS endpoint service should be published. The possible values are ClusterIP and NodePort
    service_type: ClusterIP
    ## The port to which the Elasticsearch server HTTPS endpoint will be exposed externally.
    ## This parameter is relevant only if elasticsearch_configuration.service_type is set to NodePort
    external_port:
    ## The elasticsearch admin secret that contains the username, password and .htpasswd.
    ## If not provided, the defualt admin secret named "{{ meta.name }}-elasticsearch-admin-secret" is used.
    admin_secret_name:
    ## Whether Kubernetes "may" (soft) or "must not" (hard) deploy Elasticsearch pods onto the same node
    ## The possible values are "soft" and "hard"
    anti_affinity: hard
    ## The Elasticsearch pods require the hosting worker nodes to be configured to:
    ## - disable memory swapping by setting the sysctl value vm.swappiness to 1.
    ## - increase the limit on the number of open files descriptors for the user running Elasticsearch by setting sysctl value vm.max_map_count to 65,536 or higher.
    ## When set to true, a privileged init container will execute the appropriate sysctl commands to update the worker node configuration to match Elasticsearch requirements.
    ## When set to false, you must ask the cluster administrator to change the memory swapping and descriptor properties on each worker node.
    privileged: false
    ## If elasticsearch_configuration.privileged is set to true, you must create a service account that has the privileged SecurityContextConstraint to allow running privileged containers. See "Preparing SecurityContextConstraints (SCC) requirements" for instructions.
    ## If elasticsearch_configuration.service_account not set, default service account "{{ meta.name }}-elasticsearch-service-account" will be used.
    ## If elasticsearch_configuration.privileged is set to false, see "Preparing SecurityContextConstraints (SCC) requirements" for instructions.
    service_account: "ibm-pfs-es-service-account"
    ## Initial delay for liveness and readiness probes of Elasticsearch pods
    probe_initial_delay: 90
    ## The JVM heap size to allocate to each Elasticsearch pod
    heap_size: "1024m"
    ## Whether to use the built-in monitoring capability.
    monitor_enabled: false

    ## The initial resources (CPU, memory) requests and limits.
    ## If more resources are needed, make the changes here to meet your requirement.
    resources:
      requests:
        ## The minimum amount of CPU required to start a Elasticsearch pod.
        cpu: "100m"
        ## The minimum memory required (including JVM heap and file system cache) to start a Elasticsearch pod.
        memory: "2Gi"
      limits:
        ## The maximum amount of CPU to allocate to each Elasticsearch pod.
        cpu: "1000m"
        ## The maximum memory (including JVM heap and file system cache) to allocate to each Elasticsearch pod.
        memory: "2Gi"

    storage:
      ## If persistent the elasticsearch data. Set to false for non-production or trial-only deployment.
      persistent: true
      ## Set to true to use dynamic storage provisioner
      use_dynamic_provisioning: true
      ## The minimum size of the persistent volume
      size: 10Gi
      ## Storage class name for Elasticsearch persistent storage
      storage_class: "ibmc-block-gold"

    snapshot_storage:
      ## If persistent the elasticsearch snapshot storage. Set to true for production deployment.
      enabled: false
      ## Set to true to use dynamic storage provisioner
      use_dynamic_provisioning: true
      ## The minimum size of the persistent volume
      size: 30Gi
      ## Storage class name for Elasticsearch persistent snapshot storage
      storage_class_name: "ibmc-block-gold"
      ## By default, a new persistent volume claim is be created. Specify an existing claim here if one is available.
      ## existing_claim_name: ""



  ########################################################################
  ########      IBM FileNet Content Manager configuration         ########
  ########################################################################
  ecm_configuration:
    ## FNCM secret that contains GCD DB user name and password, Object Store DB user name and password,
    ## LDAP user and password, CPE username and password, keystore password, and LTPA passs, etc.
    fncm_secret_name: ibm-fncm-secret

    ## By default all the components create ingress and routes with required annotations. In case any custom annotation is needed for the environment provide below.
    #    route_ingress_annotations:
    #      - haproxy.router.openshift.io/balance: roundrobin
    route_ingress_annotations:

    # Optional: You can specify a profile size for FNCM if different from CloudPak (see shared_configuration.sc_deployment_profile_size).  In other words,
    # you can override "shared_configuration.sc_deployment_profile_size" with "deployment_profile_size" defined here.
    # The valid values are small, medium, large - default is small.  The resources in this file are reflecting a "small" profile.
    #deployment_profile_size: "small"

    # Optional: Use an existing certificate for automatic creation of OpenShift routes.
    # Set this key if you have an external TLS certificate. Leave this empty if you don't have an external TLS certificate, operator will generate one for you.
    fncm_ext_tls_secret_name:
    ## Optional. The Certificate Authority (CA) used to sign the external TLS secret for automatic creation of OpenShift routes.
    ## Set this key if you have a CA to sign the external TLS certificate, leave this parameter empty if you don't have the CA of your external TLS certificate.
    fncm_auth_ca_secret_name:
    ####################################
    ## Start of configuration for CPE ##
    ####################################
    cpe:
      ## The architecture of the cluster.  This is the default for Linux on x86 and should not be changed.
      arch:
        amd64: "3 - Most preferred"

      ## The number of replicas or pods to be deployed.  The default is 1 replica and for high availability in a production env,
      ## it is recommended to have 2 or more.
      replica_count: 2

      ## This is the image repository and tag that correspond to image registry, which is where the image will be pulled.
      image:
        ## The default repository is the IBM Entitled Registry.
        repository: cp.icr.io/cp/cp4a/fncm/cpe
        tag: ga-558-p8cpe-la003

        ## This will override the image pull policy in the shared_configuration.
        pull_policy: IfNotPresent

      ## Logging for workloads.  This is the default setting.
      log:
       format: json

      ## The initial resources (CPU, memory) requests and limits.  If more resources are needed,
      ## make the changes here to meet your requirement.
      resources:
        requests:
          cpu: 500m
          memory: 512Mi
        limits:
          cpu: 1
          memory: 3072Mi

      ## By default "Autoscaling" is disabled with the following settings with a minimum of 2 replca and a maximum of 3 replicas.  Change
      ## this settings to meet your requirement.
      auto_scaling:
        enabled: false
        max_replicas: 3
        min_replicas: 2
        ## This is the default cpu percentage before autoscaling occurs.
        target_cpu_utilization_percentage: 80

      ## Below are the default CPE Production settings.  Make the necessary changes as you see fit.  Refer to Knowledge Center documentation for details.
      cpe_production_setting:
        time_zone: Etc/UTC

        ## The initial use of available memory.
        jvm_initial_heap_percentage: 18
        ## The maximum percentage of available memory to use.
        jvm_max_heap_percentage: 33

        ## Use this "jvm_customize_options" parameter to specify JVM arguments using comma separation. For example, if you want to set the following JVM arguments:
        ##  -Dmy.test.jvm.arg1=123
        ##  -Dmy.test.jvm.arg2=abc
        ##  -XX:+SomeJVMSettings
        ##  -XshowSettings:vm"
        ## Then set the following: jvm_customize_options="-Dmy.test.jvm.arg1=123,-Dmy.test.jvm.arg2=abc,-XX:+SomeJVMSettings,-XshowSettings:vm"
        jvm_customize_options:

        ## Default JNDI name for GCD for non-XA data source
        gcd_jndi_name: FNGCDDS
        ## Default JNDI name for GCD for XA data source
        gcd_jndixa_name: FNGCDDSXA
        license_model: FNCM.PVUNonProd

        # The license must be set to "accept" in order for the component to install.  This is the default value.
        license: accept

        # Enable/disable FIPS (default value is "false")
        disable_fips: false

      ## Enable/disable monitoring where metrics can be sent to Graphite or scraped by Prometheus
      monitor_enabled: false
      ## Enable/disable logging where logs can be sent to Elasticsearch.
      logging_enabled: false

      ## By default, the plugin for Graphite is enable to emit container metrics.
      collectd_enable_plugin_write_graphite: false

      ## Persistent Volume Claims for CPE.  If the storage_configuration in the shared_configuration is configured,
      ## the Operator will create the PVC using the names below.
      datavolume:
        existing_pvc_for_cpe_cfgstore: "cpe-cfgstore"
        existing_pvc_for_cpe_logstore: "cpe-logstore"
        existing_pvc_for_cpe_filestore: "cpe-filestore"
        existing_pvc_for_cpe_icmrulestore: "cpe-icmrulesstore"
        existing_pvc_for_cpe_textextstore: "cpe-textextstore"
        existing_pvc_for_cpe_bootstrapstore: "cpe-bootstrapstore"
        existing_pvc_for_cpe_fnlogstore: "cpe-fnlogstore"

      ## Default values for both rediness and liveness probes.  Modify these values to meet your requirements.
      probe:
        readiness:
          initial_delay_seconds: 180
          period_seconds: 30
          timeout_seconds: 10
          failure_threshold: 6
        liveness:
          initial_delay_seconds: 600
          period_seconds: 30
          timeout_seconds: 5
          failure_threshold: 6

      ## Only use this parameter if you want to override the image_pull_secrets setting in the shared_configuration above.
      image_pull_secrets:
        name: "admin.registrykey"

    #####################################
    ## Start of configuration for CMIS ##
    #####################################
    cmis:
      ## The architecture of the cluster.  This is the default for Linux on x86 and should not be changed.
      arch:
        amd64: "3 - Most preferred"

      ## The number of replicas or pods to be deployed.  The default is 1 replica and for high availability in a production env,
      ## it is recommended to have 2 or more.
      replica_count: 2

      ## This is the image repository and tag that correspond to image registry, which is where the image will be pulled.
      image:
        ## The default repository is the IBM Entitled Registry.
        repository: cp.icr.io/cp/cp4a/fncm/cmis
        tag: ga-306-cmis-la102

        ## This will override the image pull policy in the shared_configuration.
        pull_policy: IfNotPresent

      ## Logging for workloads.  This is the default setting.
      log:
        format: json

      ## The initial resources (CPU, memory) requests and limits.  If more resources are needed,
      ## make the changes here to meet your requirement.
      resources:
        requests:
          cpu: 500m
          memory: 256Mi
        limits:
          cpu: 1
          memory: 1536Mi

      ## By default "Autoscaling" is disabled with the following settings with a minimum of 2 replca and a maximum of 3 replicas.  Change
      ## this settings to meet your requirement.
      auto_scaling:
        enabled: false
        max_replicas: 3
        min_replicas: 2
        ## This is the default cpu percentage before autoscaling occurs.
        target_cpu_utilization_percentage: 80

      ## Below are the default CMIS Production settings.  Make the necessary changes as you see fit.  Refer to Knowledge Center documentation for details.
      cmis_production_setting:
        ## By default, this parameter is set by the Operator using the CPE service endpoint (e.g., "http://{{ meta.name }}-cpe-svc:9080/wsi/FNCEWS40MTOM")
        cpe_url:

        time_zone: Etc/UTC

        ## The initial use of available memory.
        jvm_initial_heap_percentage: 40
        ## The maximum percentage of available memory to use.
        jvm_max_heap_percentage: 66

        ## Use this "jvm_customize_options" parameter to specify JVM arguments using comma separation. For example, if you want to set the following JVM arguments:
        ##  -Dmy.test.jvm.arg1=123
        ##  -Dmy.test.jvm.arg2=abc
        ##  -XX:+SomeJVMSettings
        ##  -XshowSettings:vm"
        ## Then set the following: jvm_customize_options="-Dmy.test.jvm.arg1=123,-Dmy.test.jvm.arg2=abc,-XX:+SomeJVMSettings,-XshowSettings:vm"
        jvm_customize_options:

        ## Enable/disable FIPS (default value is "false")
        disable_fips: false

        ## Enable/disable Websphere Security
        ws_security_enabled: false

        # Enable/disable the content-stream of the Private Working Copy should be copied from the Document that was checked out.
        checkout_copycontent: true
        # The default value for the optional maxItems input argument on paging-related services.
        default_maxitems: 25

        # Enable/disable whether ChoiceLists will be cached once for all users.
        cvl_cache: true
        secure_metadata_cache: false

        # Enable/disalbe hidden P8 domain properties should appear in CMIS type definitions and folder or document instance data.
        filter_hidden_properties: true

        # Timeout in seconds for the queries that specify timeout.
        querytime_limit: 180

        # If true, then a faster response time for REST next line. If false, the next link for REST will re-issue query.
        resumable_queries_forrest: true

        # Specifies whether to escape characters that are not valid for XML unicode as specified by the XML 1.0 standard.
        escape_unsafe_string_characters: false

        # Limits the maximum allowable Web Service SOAP message request size.
        max_soap_size: 180

        # Enable/disable the printing of the full stack trace in the response.
        print_pull_stacktrace: false

        # Configures the sequence in which CMIS tries to identify objects (folder or document first).
        folder_first_search: false

        # To ignore the reading or writing contents in root folder, set this parameter to true.
        ignore_root_documents: false

        # Enable/disable the support type mutability.
        supporting_type_mutability: false

        # The license must be set to "accept" in order for the component to install.  This is the default value.
        license: accept

      ## Enable/disable monitoring where metrics can be sent to Graphite or scraped by Prometheus
      monitor_enabled: false
      ## Enable/disable logging where logs can be sent to Elasticsearch.
      logging_enabled: false

      ## By default, the plugin for Graphite is enable to emit container metrics.
      collectd_enable_plugin_write_graphite: false

      ## Persistent Volume Claims for CMIS.  If the storage_configuration in the shared_configuration is configured,
      ## the Operator will create the PVC using the names below.
      datavolume:
        existing_pvc_for_cmis_cfgstore: "cmis-cfgstore"
        existing_pvc_for_cmis_logstore: "cmis-logstore"

      ## Default values for both rediness and liveness probes.  Modify these values to meet your requirements.
      probe:
        readiness:
          initial_delay_seconds: 90
          period_seconds: 10
          timeout_seconds: 10
          failure_threshold: 6
        liveness:
          initial_delay_seconds: 180
          period_seconds: 10
          timeout_seconds: 5
          failure_threshold: 6

      ## Only use this parameter if you want to override the image_pull_secrets setting in the shared_configuration above.
      image_pull_secrets:
        name: "admin.registrykey"

    ########################################
    ## Start of configuration for GraphQL ##
    ########################################
    graphql:
      ## The architecture of the cluster.  This is the default for Linux on x86 and should not be changed.
      arch:
        amd64: "3 - Most preferred"

      ## The number of replicas or pods to be deployed.  The default is 1 replica and for high availability in a production env,
      ## it is recommended to have 2 or more.
      replica_count: 2

      ## This is the image repository and tag that correspond to image registry, which is where the image will be pulled.
      image:
        ## The default repository is the IBM Entitled Registry.
        repository: cp.icr.io/cp/cp4a/fncm/graphql
        tag: ga-558-p8cgql-la003

        ## This will override the image pull policy in the shared_configuration.
        pull_policy: IfNotPresent

      ## Logging for workloads.  This is the default setting.
      log:
        format: json

      ## The initial resources (CPU, memory) requests and limits.  If more resources are needed,
      ## make the changes here to meet your requirement.
      resources:
        requests:
          cpu: 500m
          memory: 512Mi
        limits:
          cpu: 1
          memory: 1536Mi

      ## By default "Autoscaling" is disabled with the following settings with a minimum of 2 replca and a maximum of 3 replicas.  Change
      ## this settings to meet your requirement.
      auto_scaling:
        enabled: false
        min_replicas: 2
        max_replicas: 3
        ## This is the default cpu percentage before autoscaling occurs.
        target_cpu_utilization_percentage: 80

      ## Below are the default CMIS Production settings.  Make the necessary changes as you see fit.  Refer to Knowledge Center documentation for details.
      graphql_production_setting:
        time_zone: Etc/UTC

        ## The initial use of available memory.
        jvm_initial_heap_percentage: 40
        ## The maximum percentage of available memory to use.
        jvm_max_heap_percentage: 66

        ## Use this "jvm_customize_options" parameter to specify JVM arguments using comma separation. For example, if you want to set the following JVM arguments:
        ##  -Dmy.test.jvm.arg1=123
        ##  -Dmy.test.jvm.arg2=abc
        ##  -XX:+SomeJVMSettings
        ##  -XshowSettings:vm"
        ## Then set the following: jvm_customize_options="-Dmy.test.jvm.arg1=123,-Dmy.test.jvm.arg2=abc,-XX:+SomeJVMSettings,-XshowSettings:vm"
        jvm_customize_options:

        license_model: FNCM.PVUNonProd

        # The license must be set to "accept" in order for the component to install.  This is the default value.
        license: accept

        enable_graph_iql: false

      ## Enable/disable monitoring where metrics can be sent to Graphite or scraped by Prometheus
      monitor_enabled: false
      ## Enable/disable logging where logs can be sent to Elasticsearch.
      logging_enabled: false

      ## By default, the plugin for Graphite is enable to emit container metrics.
      collectd_enable_plugin_write_graphite: false

      ## Persistent Volume Claims for GraphQL.  If the storage_configuration in the shared_configuration is configured,
      ## the Operator will create the PVC using the names below.
      datavolume:
        existing_pvc_for_graphql_cfgstore: "graphql-cfgstore"
        existing_pvc_for_graphql_logstore: "graphql-logstore"

      ## Default values for both rediness and liveness probes.  Modify these values to meet your requirements.
      probe:
        readiness:
          initial_delay_seconds: 120
          period_seconds: 10
          timeout_seconds: 10
          failure_threshold: 6
        liveness:
          initial_delay_seconds: 600
          period_seconds: 10
          timeout_seconds: 5
          failure_threshold: 6
      ## Only use this parameter if you want to override the image_pull_secrets setting in the shared_configuration above.
      image_pull_secrets:
        name: "admin.registrykey"


  ########################################################################
  ########  IBM FileNet Content Manager initialize configuration  ########
  ########################################################################
  ## The deployment of FNCM will be initialized with the default values assigned to the parameters below.
  ## The initialization process includes the creation of the P8 domain, the creation of the directory services,
  ## the assignments of users/groups to the P8 domain and object store(s), the creation of the object store(s),
  ## the creation/addition of add-ons for each object store, the enablement of workflow for each object store, the
  ## creation of Content Search Services servers, index areas, and the enabling of Content-based Retrieval (CBR) for each object store.
  ## In addition, the creation of Navigator desktop will also occur.
  ## If any of the values below does not fit your infrastructure, then change the value to correpond to your configuration
  ## (e.g., "CEAdmin" is the default user for ic_ldap_admin_user_name parameter and if you do not have "CEAdmin" user in your directory
  ## server and have a different user, then replace "CEAdmin" with your own user).  Otherwise, the rest of the values should remain as default.
  initialize_configuration:
    ic_domain_creation:
      ## Provide a name for the domain
      domain_name: "P8DOMAIN"
      ## The encryption strength
      encryption_key: "128"
    ic_ldap_creation:
      ## Administrator user
      ic_ldap_admin_user_name:
      - "ibpmServices_flt" # user name for P8 domain admin, for example, "CEAdmin".  This parameter accepts a list of values.
      ## Administrator group
      ic_ldap_admins_groups_name:
      - "iBPM_ES_Service_Accounts" # group name for P8 domain admin, for example, "P8Administrators".  This parameter accepts a list of values.
      ## Name of the LDAP directory
      ic_ldap_name: "ldap_name"
    ic_obj_store_creation:
      object_stores:
      ## Configuration for the document object store
      ## Display name for the document object store to create
      - oc_cpe_obj_store_display_name: "BAWINS1DOCS"
        ## Symbolic name for the document object store to create
        oc_cpe_obj_store_symb_name: "BAWINS1DOCS"
        oc_cpe_obj_store_conn:
          ## Object store connection name
          name: "DOCS_connection" #database connection name
          ## The name of the site
          site_name: "InitialSite"
          ## Specify the name of the non-XA datasource (from dc_common_os_datasource_name in the dc_os_datasources section above)
          dc_os_datasource_name: "BAWINS1DOCS"
          ## The XA datasource
          dc_os_xa_datasource_name: "BAWINS1DOCSXA"
        ## Admin user group
        oc_cpe_obj_store_admin_user_groups:
        - "iBPM_ES_Service_Accounts" # user name and group name for object store admin, for example, "CEAdmin" or "P8Administrators".  This parameter accepts a list of values.
        ## An array of users with access to the object store
        oc_cpe_obj_store_basic_user_groups:
        ## Specify whether to enable add-ons
        oc_cpe_obj_store_addons: true
        ## Add-ons to enable for Content Platform Engine
        oc_cpe_obj_store_addons_list:
        - "{CE460ADD-0000-0000-0000-000000000004}"
        - "{CE460ADD-0000-0000-0000-000000000001}"
        - "{CE460ADD-0000-0000-0000-000000000003}"
        - "{CE460ADD-0000-0000-0000-000000000005}"
        - "{CE511ADD-0000-0000-0000-000000000006}"
        - "{CE460ADD-0000-0000-0000-000000000008}"
        - "{CE460ADD-0000-0000-0000-000000000007}"
        - "{CE460ADD-0000-0000-0000-000000000009}"
        - "{CE460ADD-0000-0000-0000-00000000000A}"
        - "{CE460ADD-0000-0000-0000-00000000000B}"
        - "{CE460ADD-0000-0000-0000-00000000000D}"
        - "{CE511ADD-0000-0000-0000-00000000000F}"
        ## Provide a name for the Advance Storage Area
        oc_cpe_obj_store_asa_name: "demo_storage"
        ## Provide a name for the file system storage device
        oc_cpe_obj_store_asa_file_systems_storage_device_name: "demo_file_system_storage"
        ## The root directory path for the object store storage area
        oc_cpe_obj_store_asa_root_dir_path: "/opt/ibm/asa/os01_storagearea"
        ## Specify whether to enable workflow for the object store
        oc_cpe_obj_store_enable_workflow: false
        # Enable/disable object store database compression.  Default is false.
        oc_cpe_obj_store_enable_compression: false
      ## Configuration for the design object store
      ## Display name for the design object store to create
      - oc_cpe_obj_store_display_name: "BAWINS1DOS"
        ## Symbolic name for the document object store to create
        oc_cpe_obj_store_symb_name: "BAWINS1DOS"
        oc_cpe_obj_store_conn:
          ## Object store connection name
          name: "DOS_connection"
          ## The name of the site
          site_name: "InitialSite"
          ## Specify the name of the non-XA datasource (from dc_common_os_datasource_name in the dc_os_datasources section above)
          dc_os_datasource_name: "BAWINS1DOS"
          ## The XA datasource
          dc_os_xa_datasource_name: "BAWINS1DOSXA"
        ## Admin user group
        oc_cpe_obj_store_admin_user_groups:
        - "iBPM_ES_Service_Accounts" # user name and group name for object store admin, for example, "CEAdmin" or "P8Administrators".  This parameter accepts a list of values.
        ## An array of users with access to the object store
        oc_cpe_obj_store_basic_user_groups:
        ## Specify whether to enable add-ons
        oc_cpe_obj_store_addons: true
        ## Add-ons to enable for Content Platform Engine
        oc_cpe_obj_store_addons_list:
        - "{CE460ADD-0000-0000-0000-000000000004}"
        - "{CE460ADD-0000-0000-0000-000000000001}"
        - "{CE460ADD-0000-0000-0000-000000000003}"
        - "{CE460ADD-0000-0000-0000-000000000005}"
        - "{CE511ADD-0000-0000-0000-000000000006}"
        - "{CE460ADD-0000-0000-0000-000000000008}"
        - "{CE460ADD-0000-0000-0000-000000000007}"
        - "{CE460ADD-0000-0000-0000-000000000009}"
        - "{CE460ADD-0000-0000-0000-00000000000A}"
        - "{CE460ADD-0000-0000-0000-00000000000B}"
        - "{CE460ADD-0000-0000-0000-00000000000D}"
        - "{CE511ADD-0000-0000-0000-00000000000F}"
        ## Provide a name for the Advance Storage Area
        oc_cpe_obj_store_asa_name: "demo_storage"
        ## Provide a name for the file system storage device
        oc_cpe_obj_store_asa_file_systems_storage_device_name: "demo_file_system_storage"
        ## The root directory path for the object store storage area
        oc_cpe_obj_store_asa_root_dir_path: "/opt/ibm/asa/os02_storagearea"
        ## Specify whether to enable workflow for the object store
        oc_cpe_obj_store_enable_workflow: false
        # Enable/disable object store database compression.  Default is false.
        oc_cpe_obj_store_enable_compression: false
      ## Configuration for the target object store
      ## Display name for the target object store to create
      - oc_cpe_obj_store_display_name: "BAWINS1TOS"
        ## Symbolic name for the document object store to create
        oc_cpe_obj_store_symb_name: "BAWINS1TOS"
        oc_cpe_obj_store_conn:
          ## Object store connection name
          name: "TOS_connection"
          ## The name of the site
          site_name: "InitialSite"
          ## Specify the name of the non-XA datasource (from dc_common_os_datasource_name in the dc_os_datasources section above)
          dc_os_datasource_name: "BAWINS1TOS"
          ## The XA datasource
          dc_os_xa_datasource_name: "BAWINS1TOSXA"
        ## Admin user group
        oc_cpe_obj_store_admin_user_groups:
        - "iBPM_ES_Service_Accounts" # user name and group name for object store admin, for example, "CEAdmin" or "P8Administrators".  This parameter accepts a list of values.
        ## An array of users with access to the object store
        oc_cpe_obj_store_basic_user_groups:
        ## Specify whether to enable add-ons
        oc_cpe_obj_store_addons: true
        ## Add-ons to enable for Content Platform Engine
        oc_cpe_obj_store_addons_list:
        - "{CE460ADD-0000-0000-0000-000000000004}"
        - "{CE460ADD-0000-0000-0000-000000000001}"
        - "{CE460ADD-0000-0000-0000-000000000003}"
        - "{CE460ADD-0000-0000-0000-000000000005}"
        - "{CE511ADD-0000-0000-0000-000000000006}"
        - "{CE460ADD-0000-0000-0000-000000000008}"
        - "{CE460ADD-0000-0000-0000-000000000007}"
        - "{CE460ADD-0000-0000-0000-000000000009}"
        - "{CE460ADD-0000-0000-0000-00000000000A}"
        - "{CE460ADD-0000-0000-0000-00000000000B}"
        - "{CE460ADD-0000-0000-0000-00000000000D}"
        - "{CE511ADD-0000-0000-0000-00000000000F}"
         ## Provide a name for the Advance Storage Area
        oc_cpe_obj_store_asa_name: "demo_storage"
        ## Provide a name for the file system storage device
        oc_cpe_obj_store_asa_file_systems_storage_device_name: "demo_file_system_storage"
        ## The root directory path for the object store storage area
        oc_cpe_obj_store_asa_root_dir_path: "/opt/ibm/asa/os03_storagearea"
        ## Specify whether to enable workflow for the object store
        oc_cpe_obj_store_enable_workflow: true
        ## Specify a name for the workflow region
        oc_cpe_obj_store_workflow_region_name: "tos_region_name"
        ## Specify the number of the workflow region
        oc_cpe_obj_store_workflow_region_number: 3
        ## Specify a table space for the workflow data
        oc_cpe_obj_store_workflow_data_tbl_space: "TOS_DB_DATA1"
        ## Optionally specify a table space for the workflow index
        oc_cpe_obj_store_workflow_index_tbl_space: "TOS_DB_UPPER_INDEX1"
        ## Optionally specify a table space for the workflow blob.
        oc_cpe_obj_store_workflow_blob_tbl_space: "TOS_DB_UPPER_BLOB"
        ## Designate an LDAP group for the workflow admin group.
        oc_cpe_obj_store_workflow_admin_group: "iBPM_ES_Service_Accounts"
        ## Designate an LDAP group for the workflow config group
        oc_cpe_obj_store_workflow_config_group: "iBPM_ES_Service_Accounts"
        ## Default format for date and time
        oc_cpe_obj_store_workflow_date_time_mask: "mm/dd/yy hh:tt am"
        ## Locale for the workflow
        oc_cpe_obj_store_workflow_locale: "en"
        ## Provide a name for the connection point
        oc_cpe_obj_store_workflow_pe_conn_point_name: "cpe_conn_tos"
        # Enable/disable object store database compression.  Default is false.
        oc_cpe_obj_store_enable_compression: false
      ## Configuration for the application engine object store
      ## Display name for the application engine object store to create
      - oc_cpe_obj_store_display_name: "AEOS"
        ## Symbolic name for the application engine object store to create
        oc_cpe_obj_store_symb_name: "AEOS"
        oc_cpe_obj_store_conn:
          ## Object store connection name
          name: "AEOS_connection"
          ## The name of the site
          site_name: "InitialSite"
          ## Specify the name of the non-XA datasource (from dc_common_os_datasource_name in the dc_os_datasources section above)
          dc_os_datasource_name: "AEOS"
          ## The XA datasource
          dc_os_xa_datasource_name: "AEOSXA"
        ## Admin user group
        oc_cpe_obj_store_admin_user_groups:
        - "iBPM_ES_Service_Accounts" # user name and group name for object store admin, for example, "CEAdmin" or "P8Administrators".  This parameter accepts a list of values.
        ## An array of users with access to the object store
        oc_cpe_obj_store_basic_user_groups:
        ## Specify whether to enable add-ons
        oc_cpe_obj_store_addons: true
        ## Add-ons to enable for Content Platform Engine
        oc_cpe_obj_store_addons_list:
        - "{CE460ADD-0000-0000-0000-000000000004}"
        - "{CE460ADD-0000-0000-0000-000000000001}"
        - "{CE460ADD-0000-0000-0000-000000000003}"
        - "{CE460ADD-0000-0000-0000-000000000005}"
        - "{CE511ADD-0000-0000-0000-000000000006}"
        - "{CE460ADD-0000-0000-0000-000000000008}"
        - "{CE460ADD-0000-0000-0000-000000000007}"
        - "{CE460ADD-0000-0000-0000-000000000009}"
        - "{CE460ADD-0000-0000-0000-00000000000A}"
        - "{CE460ADD-0000-0000-0000-00000000000B}"
        - "{CE460ADD-0000-0000-0000-00000000000D}"
        - "{CE511ADD-0000-0000-0000-00000000000F}"
        ## Provide a name for the Advance Storage Area
        oc_cpe_obj_store_asa_name: "demo_storage"
        ## Provide a name for the file system storage device
        oc_cpe_obj_store_asa_file_systems_storage_device_name: "demo_file_system_storage"
        ## The root directory path for the object store storage area
        oc_cpe_obj_store_asa_root_dir_path: "/opt/ibm/asa/osae_storagearea"
        ## Specify whether to enable workflow for the object store
        oc_cpe_obj_store_enable_workflow: false
        # Enable/disable object store database compression.  Default is false.
        oc_cpe_obj_store_enable_compression: false
