suite_install
=============

This role install Maximo Application Suite. It internally resolve the namespace based on the `mas_instance_id` as `mas-{mas_instance_id}-core`. By default this role install MAS Operator using Manual Upgrade Strategy. Set `MAS_UPGRADE_STRATEGY` environment variable to Automatic to override it. In the `Manual` upgrade mode, IBM Common Services operators requested by MAS will inherit the upgrade strategy from MAS and their pending install plans approved.

Role Variables
--------------
- `mas_catalog_source` Defines the catalog to be used to install MAS. You can set it to ibm-operator-catalog for both release as well as for development install
- `artifactory_username` Required when using this role for development versions of MAS
- `artifactory_apikey` Required when using this role for development versions of MAS
- `mas_channel` Defines which channel of MAS to subscribe to
- `mas_domain` Opitional fact, if not provided the role will use the default cluster subdomain
- `mas_instance_id` Defines the instance id to be used for MAS installation
- `mas_icr_cp` Defines the entitled registry from the images should be pulled from. Set this to `cp.icr.io/cp` when installing release version of MAS or `docker-na-public.artifactory.swg-devops.com/wiotp-docker-local` for dev
- `mas_icr_cpopen` Defines the registry for non entitled images, such as operators. Set this to `icr.io/cpopen` when installing release version of MAS or `docker-na-public.artifactory.swg-devops.com/wiotp-docker-local/cpopen` for dev
- `mas_entitlement_username` Username for entitled registry. This username will be used to create the image pull secret. Set to `cp` when installing release or use your `w3Id` for dev.
- `mas_entitlement_key` API Key for entitled registry. This password will be used to create the image pull secret. Set to with IBM entitlement key when installing release or use your artifactory `apikey` for dev.
- `mas_config_dir` Directory containing configuration files (`*.yaml` and `*.yml`) to be applied to the MAS installation.  Intended for creating the various MAS custom resources to configure the suite post-install, but can be used to apply any kubernetes resource you need to customize any aspect of your cluster.
- `certManager.namespace` The namespace containing the cert-manager to be used by MAS
- `mas_upgrade_strategy` The Upgrade strategy for MAS Operator. Default is set to Automatic
- `mas_annotations` Optional variable having all the annotations that need to be added to the Suite CR. This variable takes a comma separated list of annotations. For example, to deploy your suite in non production mode, set this to `mas.ibm.com/operationalMode=nonproduction` 
or set `MAS_ANNOTATIONS` environment variable as `export MAS_ANNOTATIONS=mas.ibm.com/operationalMode=nonproduction`
- `mas_customize_scaling` Optional, only supported in MAS 8.10 or later, installation of earlier releases will ignore this parameter. ConfigMap name used for customizing MAS Core resources scaling settings. MAS provides the following built-in t-shirt sizes: small/medium/large. 
For example, to deploy your suite with small size, set this to `{mas_instance_id}-wl-cust-small`, for medium size set to `{mas_instance_id}-wl-cust-small` and for large size set to `{mas_instance_id}-wl-cust-small`. If using a custom scaling configuration, use any ConfigMap name previously created. Alternatively, you can use the environment variable `MAS_CUSTOMIZE_SCALING` to set the scaling ConfigMap name.
- `custom_labels` Optional environment variable, CUSTOM_LABELS, containing a list of comma separated key=value pairs for setting custom labels on instance specific resources.

Example Playbook
----------------

```yaml
- hosts: localhost
  any_errors_fatal: true
  vars:
    mas_instance_id: "inst1"
    mas_config_dir: "/home/david/masconfig"
    mas_entitlement_key: "{{ lookup('env', 'IBM_ENTITLEMENT_KEY') }}"

  roles:
    - ibm.mas_devops.suite_install
    - ibm.mas_devops.suite_config
    - ibm.mas_devops.suite_verify
```


Tekton Task
-----------
Start a run of the **mas-devops-suite-install** Task as below, you must have already prepared the namespace:

```
cat <<EOF | oc create -f -
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  generateName: mas-devops-suite-install-
spec:
  taskRef:
    kind: Task
    name: mas-devops-suite-install
  params:
  - name: mas_instance_id
    value: inst1
  - name: mas_channel
    value: 8.7.x
  - name: mas_entitlement_key
    value: xxx
  resources: {}
  serviceAccountName: pipeline
  timeout: 48h0m0s
EOF
```


License
-------

EPL-2.0
