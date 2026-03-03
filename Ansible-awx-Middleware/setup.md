## Setup provider

Prima di ogni operazione bisogna recuperare le configurazioni dei provider dal secret kubernetes provider-clients-configuration.yaml
Tramite il parametro providerClientTag passato dal middleware bisogna fare matching sul parametro definition.tag del secret
per recuperare le configurazioni del provider e chiamare il provider giusto.

## Operazioni Supportate

### 1. Purchase First Subscription
Crea la prima subscription per un nuovo utente.
- **Playbook**: `playbooks/purchase_first_subscription.yml`
- **Cosa fa**:
  1) SETUP_MASTER_PROJECT:
  - MASTER project exists by name (name = user_id)
  - Se MASTER project non esiste:
    - Crea MASTER project (name = user_id, parentId = root_project_id, enabled = true, description = "Provider project for customer {user_id}", extraParams = Map<String,String>)
  - Check MASTER Project User Role (projectId = master_project_id, userId = sync_user_id, roleId = sync_role_id)
  - Se MASTER Project User Role non assegnato:
    - Grant MASTER Project User Role (projectId = master_project_id, userId = sync_user_id, roleId = sync_role_id)
  - MASTER group exists by name and domainId (name = user_id, domainId = keycloak_users_domain_id)
  - Se MASTER group non esiste:
    - Crea MASTER group (name = user_id, domainId = keycloak_users_domain_id)
  2) SETUP_SUBSCRIPTION_PROJECT
  - SUBSCRIPTION project exists by name (name = external_id)
  - Se SUBSCRIPTION project non esiste:
    - Crea SUBSCRIPTION project (name = external_id, parentId = master_project_id, enabled = true, description = "Provider project for customer {user_id}", extraParams = Map<String,String>)
  - Check SUBSCRIPTION Project User Role (projectId = subscription_project_id, userId = sync_user_id, roleId = sync_role_id)
  - Se SUBSCRIPTION Project User Role non assegnato:
    - Grant SUBSCRIPTION Project User Role (projectId = master_project_id, userId = sync_user_id, roleId = sync_role_id)
  - Se SUBSCRIPTION group non esiste:
    - Crea SUBSCRIPTION group (name = external_id, domainId = keycloak_users_domain_id)
  - Check SUBSCRIPTION Project MASTER Group Role (projectId = subscription_project_id, group_id =master_group_id , roleId = active_group_role_id)
  - Se SUBSCRIPTION Project MASTER Group Role non assegnato:
    - Grant SUBSCRIPTION Project MASTER Group Role (projectId = subscription_project_id, group_id =master_group_id, roleId = active_group_role_id)
  - Check SUBSCRIPTION Project SUBSCRIPTION Group Role (projectId = subscription_project_id, group_id =subscription_group_id , roleId = active_group_role_id)
  - Se SUBSCRIPTION Project SUBSCRIPTION Group Role non assegnato:
    - Grant SUBSCRIPTION Project SUBSCRIPTION Group Role (projectId = subscription_project_id, group_id =subscription_group_id, roleId = active_group_role_id)
  3) SETUP_QUOTA_ON_SUBSCRIPTION_PROJECT
  - Retrieve all quota on NOVA (project_id = subscription_project_id):
  - For cores, ram if request quota != NOVA retrieved quota:
    - Update NOVA quota ( put request quota that changed and insert retrieved value for unchanged quota)
  - Retrieve quota on CINDER (project_id = subscription_project_id):
  - For gigabytes, backup_gigabytes if request quota != CINDER retrieved quota:
    - Update CINDER quota ( put request quota that changed and insert retrieved value for unchanged quota)
  - Retrieve quota on NEUTRON (project_id = subscription_project_id):
  - For floatingip if request quota != NEUTRON retrieved quota:
    - Update NEUTRON quota ( put request quota that changed and insert retrieved value for unchanged quota)

### 2. Purchase Subsequent Subscription
Crea subscription successive per un utente esistente.
- **Playbook**: `playbooks/purchase_subsequent_subscription.yml`
- **Cosa fa**:
  1) RETRIEVE_MASTER_PROJECT_AND_GROUP:
  - MASTER project exists by name (name = user_id)
  - Se MASTER project non esiste:
    - return error
  - MASTER group exists by name and domainId (name = user_id, domainId= keycloak_users_domain_id)
  - Se MASTER group non esiste:
    - return error
  2) SETUP_SUBSCRIPTION_PROJECT (uguale a Purchase First Subscription)
  - SUBSCRIPTION project exists by name (name = external_id)
  - Se SUBSCRIPTION project non esiste:
    - Crea SUBSCRIPTION project (name = external_id, parentId = master_project_id, enabled = true, description = "Provider project for customer {user_id}", extraParams = Map<String,String>)
  - Check SUBSCRIPTION Project User Role (projectId = subscription_project_id, userId = sync_user_id, roleId = sync_role_id)
  - Se SUBSCRIPTION Project User Role non assegnato:
    - Grant SUBSCRIPTION Project User Role (projectId = master_project_id, userId = sync_user_id, roleId = sync_role_id)
  - Se SUBSCRIPTION group non esiste:
    - Crea SUBSCRIPTION group (name = external_id, domainId = keycloak_users_domain_id)
  - Check SUBSCRIPTION Project MASTER Group Role (projectId = subscription_project_id, group_id =master_group_id , roleId = active_group_role_id)
  - Se SUBSCRIPTION Project MASTER Group Role non assegnato:
    - Grant SUBSCRIPTION Project MASTER Group Role (projectId = subscription_project_id, group_id =master_group_id, roleId = active_group_role_id)
  - Check SUBSCRIPTION Project SUBSCRIPTION Group Role (projectId = subscription_project_id, group_id =subscription_group_id , roleId = active_group_role_id)
  - Se SUBSCRIPTION Project SUBSCRIPTION Group Role non assegnato:
    - Grant SUBSCRIPTION Project SUBSCRIPTION Group Role (projectId = subscription_project_id, group_id =subscription_group_id, roleId = active_group_role_id)
  3) SETUP_QUOTA_ON_SUBSCRIPTION_PROJECT (uguale a Purchase First Subscription)
  - Retrieve all quota on NOVA (project_id = subscription_project_id):
  - For cores, ram if request quota != NOVA retrieved quota:
    - Update NOVA quota ( put request quota that changed and insert retrieved value for unchanged quota)
  - Retrieve quota on CINDER (project_id = subscription_project_id):
  - For gigabytes, backup_gigabytes if request quota != CINDER retrieved quota:
    - Update CINDER quota ( put request quota that changed and insert retrieved value for unchanged quota)
  - Retrieve quota on NEUTRON (project_id = subscription_project_id):
  - For floatingip if request quota != NEUTRON retrieved quota:
    - Update NEUTRON quota ( put request quota that changed and insert retrieved value for unchanged quota)

### 3. Change Subscription
Modifica le quote di una subscription esistente.
- **Playbook**: `playbooks/change_subscription.yml`
- **Cosa fa**:
  1) RETRIEVE_SUBSCRIPTION_PROJECT
  - SUBSCRIPTION project exists by name (name = external_id)
  - Se SUBSCRIPTION project non esiste:
    - return error
  2) SETUP_QUOTA_ON_SUBSCRIPTION_PROJECT (uguale a Purchase First Subscription)
  - Retrieve all quota on NOVA (project_id = subscription_project_id):
  - For cores, ram if request quota != NOVA retrieved quota:
    - Update NOVA quota ( put request quota that changed and insert retrieved value for unchanged quota)
  - Retrieve quota on CINDER (project_id = subscription_project_id):
  - For gygabytes, backup_gigabytes if request quota != CINDER retrieved quota:
    - Update CINDER quota ( put request quota that changed and insert retrieved value for unchanged quota)
  - Retrieve quota on NEUTRON (project_id = subscription_project_id):
  - For floatingip if request quota != NEUTRON retrieved quota:
    - Update NEUTRON quota ( put request quota that changed and insert retrieved value for unchanged quota)

### 4. Suspend Subscription
Sospende una subscription attiva.
- **Playbook**: `playbooks/suspend_subscription.yml`
- **Cosa fa**:
  1) RETRIEVE_SUBSCRIPTION_PROJECT_AND_GROUP :
  - SUBSCRIPTION project exists by name (name = external_id)
  - Se SUBSCRIPTION project non esiste:
    - return error
  - SUBSCRIPTION group exists by name and domainId (name = external_id, domainId= keycloak_users_domain_id)
  - Se SUBSCRIPTION group non esiste:
    - return error
  2) ASSIGN_SUSPEND_ROLE
  - List SUBSCRIPTION project group roles (projectId = subscription_project_id , groupId = subscription_group_id)
  - If SUBSCRIPTION project group roles contains suspended_group_role_id:
    - Do nothing and exit
  - Else:
    - for each SUBSCRIPTION project group role revoke Project group Role (subscription_project_id , groupId = subscription_group_id, roleId = retrieved_group_role_id )
    - Grant project group role (subscription_project_id , groupId = subscription_group_id, roleId = suspended_group_role_id )

### 5. Resume Subscription
Riattiva una subscription sospesa.
- **Playbook**: `playbooks/resume_subscription.yml`
- **Cosa fa**:
  1) RETRIEVE_SUBSCRIPTION_PROJECT_AND_GROUP:
  - SUBSCRIPTION project exists by name (name = external_id)
  - Se SUBSCRIPTION project non esiste:
    - return error
  - SUBSCRIPTION group exists by name and domainId (name = external_id, domainId= keycloak_users_domain_id)
  - Se SUBSCRIPTION group non esiste:
    - return error
  2) ASSIGN_ACTIVE_ROLE
  - List SUBSCRIPTION project group roles (projectId = subscription_project_id , groupId = subscription_group_id)
  - If SUBSCRIPTION project group roles contains active_group_role_id:
    - Do nothing and exit
  - Else:
    - for each SUBSCRIPTION project group role revoke Project group Role (subscription_project_id , groupId = subscription_group_id, roleId = retrieved_group_role_id )
    - Grant project group role (subscription_project_id , groupId = subscription_group_id, roleId = active_group_role_id )

### 6. Cancel Subscription
Cancella una subscription.
- **Playbook**: `playbooks/cancel_subscription.yml`
- **Cosa fa**:
  1) RETRIEVE_SUBSCRIPTION_PROJECT_AND_GROUP:
  - SUBSCRIPTION project exists by name (name = external_id)
  - Se SUBSCRIPTION project non esiste:
    - return error
  - SUBSCRIPTION group exists by name and domainId (name = external_id, domainId= keycloak_users_domain_id)
  - Se SUBSCRIPTION group non esiste:
    - return error
  2) ASSIGN_CANCELLED_ROLE
  - List SUBSCRIPTION project group roles (projectId = subscription_project_id , groupId = subscription_group_id)
  - If SUBSCRIPTION project group roles contains cancelled_group_role_id:
    - Do nothing and exit
  - Else:
    - for each SUBSCRIPTION project group role revoke Project group Role (subscription_project_id , groupId = subscription_group_id, roleId = retrieved_group_role_id )
    - Grant project group role (subscription_project_id , groupId = subscription_group_id, roleId = cancelled_group_role_id )