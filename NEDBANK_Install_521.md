

## 1. Set up client workstation

### Prepare the client workstation
1. Prepare a RHEL 9 machine with internet

* 1.1 Create a directory for cpd-cli utility.

```bash
export CPD521_WORKSPACE=/ibm/cpd/521
mkdir -p ${CPD521_WORKSPACE}
cd ${CPD521_WORKSPACE}
``` 

* 1.2 Download the cpd-cli for 5.2.1.

```bash
wget https://github.com/IBM/cpd-cli/releases/download/v14.2.1/cpd-cli-s390x-EE-14.2.1.tgz
```

2. Install the tool.

* 2.1 Untar content.
```bash
tar xvf cpd-cli-s390x-EE-14.2.1.tgz *Ask*
```
 * 2.2 Make the cpd-cli executable from any directory.

 ```bash
 vi ~/.bashrc
 ```

 Add the following line at the bottom of the file.

 ```bash
 export PATH=<fully-qualified-path-to-the-cpd-cli>:$PATH
 ```

 * 2.3 After editing the `.bashrc` file, in order to load this change, we need to close the current session and login back to the terminal, or do `bash`.

 * 2.4 Test the cpd-cli tool.

 ```bash
 cpd-cli version
 ```

3. Install cert-management operator using web client procedure.

* 3.1 Prerequisites.

    * 3.1.1 cluster-admin privileges.
    * 3.1.2 Acess Openshift Container Platform web console.

* 3.2 Procedure.

    * 3.2.1 Log in to the OpenShift Container Platform web console.
    * 3.2.2 Navigate to **Operators** > **OperatorHub**.
    * 3.2.3 Enter **cert-manager Operator for Red Hat OpenShift** into the filter box.
    * 3.2.4 Select the **cert-manager Operator for Red Hat OpenShift**.
    * 3.2.5 Select the cert-manager Operator for Red Hat OpenShift version from **Version** drop-down list, and click Install.
    * 3.2.6 On the **Install Operator** page:
        * i. Update the **Update channel**, if necessary. The channel defaults to **stable-v1**, which installs the latest stable release of the cert-manager Operator for Red Hat OpenShift. 
        * ii.  Choose the **Installed Namespace** for the Operator. The default Operator namespace is `cert-manager-operator`.

            If the cert-manager-operator namespace does not exist, it is created for you.
        * iii. Select an **Update approval** strategy.
            * The **Automatic** strategy allows Operator Lifecycle Manager (OLM) to automatically update the Operator when a new version is available.
            * The **Manual** strategy requires a user with appropriate credentials to approve the Operator update. 
        * iv. Click **Install**

* 3.3 Verification.

    * 3.3.1 Navigate to **Operators** > **Installed Operators**.
    * 3.3.2 Verify that **cert-manager Operator for Red Hat OpenShift** is listed with a Status of **Succeeded** in the `cert-manager-operator` namespace.
    * 3.3.3 Verify that cert-manager pods are up and running by entering the following command.
    ```bash
    oc get pods -n cert-manager

    Example output:
    NAME                                       READY   STATUS    RESTARTS   AGE
    cert-manager-bd7fbb9fc-wvbbt               1/1     Running   0          3m39s
    cert-manager-cainjector-56cc5f9868-7g9z7   1/1     Running   0          4m5s
    cert-manager-webhook-d4f79d7f7-9dg9w       1/1     Running   0          4m9s
    ```

    You can use the cert-manager Operator for Red Hat OpenShift only after cert-manager pods are up and running.

* 3.4 Link of the documentation for refence.
https://docs.redhat.com/en/documentation/openshift_container_platform/4.15/html/security_and_compliance/cert-manager-operator-for-red-hat-openshift#cert-manager-operator-install


4. Obtaining IBM Entitlement API key.

* 4.1 Log in to [Container software library on My IBM](https://myibm.ibm.com/products-services/containerlibrary) with the IBMid and password that are associated with the entitled software.

* 4.2 On the **Entitlement** keys tab, select **Copy** to copy the entitlement key to the clipboard.

* 4.3 Save the API key in a text file.

5. Determining components to install.

```bash
db2oltp
```

6. Setting up instalation environment variables.

* 6.1 Copy the following example on your local system, example name `cpd_vars.sh`.

```bash

#===============================================================================
# IBM Software Hub installation variables
#===============================================================================

# ------------------------------------------------------------------------------
# Client workstation 
# ------------------------------------------------------------------------------
# Set the following variables if you want to override the default behavior of the IBM Software Hub CLI.
#
# To export these variables, you must uncomment each command in this section.

# export CPD_CLI_MANAGE_WORKSPACE=<enter a fully qualified directory>
# export OLM_UTILS_LAUNCH_ARGS=<enter launch arguments>


# ------------------------------------------------------------------------------
# Cluster
# ------------------------------------------------------------------------------

export OCP_URL=<enter your Red Hat OpenShift Container Platform URL>
export OPENSHIFT_TYPE=<enter your deployment type>
export IMAGE_ARCH=<enter your cluster architecture>
# export OCP_USERNAME=<enter your username>
# export OCP_PASSWORD=<enter your password>
# export OCP_TOKEN=<enter your token>
export SERVER_ARGUMENTS="--server=${OCP_URL}"
# export LOGIN_ARGUMENTS="--username=${OCP_USERNAME} --password=${OCP_PASSWORD}"
# export LOGIN_ARGUMENTS="--token=${OCP_TOKEN}"
export CPDM_OC_LOGIN="cpd-cli manage login-to-ocp ${SERVER_ARGUMENTS} ${LOGIN_ARGUMENTS}"
export OC_LOGIN="oc login ${SERVER_ARGUMENTS} ${LOGIN_ARGUMENTS}"


# ------------------------------------------------------------------------------
# Proxy server
# ------------------------------------------------------------------------------

# export PROXY_HOST=<enter your proxy server hostname>
# export PROXY_PORT=<enter your proxy server port number>
# export PROXY_USER=<enter your proxy server username>
# export PROXY_PASSWORD=<enter your proxy server password>
# export NO_PROXY_LIST=<a comma-separated list of domain names>


# ------------------------------------------------------------------------------
# Projects
# ------------------------------------------------------------------------------

export PROJECT_CERT_MANAGER=<enter your certificate manager project>
export PROJECT_LICENSE_SERVICE=<enter your License Service project>
export PROJECT_SCHEDULING_SERVICE=<enter your scheduling service project>
# export PROJECT_IBM_EVENTS=<enter your IBM Events Operator project>
# export PROJECT_PRIVILEGED_MONITORING_SERVICE=<enter your privileged monitoring service project>
export PROJECT_CPD_INST_OPERATORS=<enter your IBM Software Hub operator project>
export PROJECT_CPD_INST_OPERANDS=<enter your IBM Software Hub operand project>
# export PROJECT_CPD_INSTANCE_TETHERED=<enter your tethered project>
# export PROJECT_CPD_INSTANCE_TETHERED_LIST=<a comma-separated list of tethered projects>



# ------------------------------------------------------------------------------
# Storage
# ------------------------------------------------------------------------------

export STG_CLASS_BLOCK=<RWO-storage-class-name>
export STG_CLASS_FILE=<RWX-storage-class-name>

# ------------------------------------------------------------------------------
# IBM Entitled Registry
# ------------------------------------------------------------------------------

export IBM_ENTITLEMENT_KEY=<enter your IBM entitlement API key>


# ------------------------------------------------------------------------------
# Private container registry
# ------------------------------------------------------------------------------
# Set the following variables if you mirror images to a private container registry.
#
# To export these variables, you must uncomment each command in this section.

# export PRIVATE_REGISTRY_LOCATION=<enter the location of your private container registry>
# export PRIVATE_REGISTRY_PUSH_USER=<enter the username of a user that can push to the registry>
# export PRIVATE_REGISTRY_PUSH_PASSWORD=<enter the password of the user that can push to the registry>
# export PRIVATE_REGISTRY_PULL_USER=<enter the username of a user that can pull from the registry>
# export PRIVATE_REGISTRY_PULL_PASSWORD=<enter the password of the user that can pull from the registry>


# ------------------------------------------------------------------------------
# IBM Software Hub version
# ------------------------------------------------------------------------------

export VERSION=5.2.1


# ------------------------------------------------------------------------------
# Components
# ------------------------------------------------------------------------------

export COMPONENTS=ibm-licensing,scheduler,cpfs,cpd_platform
# export COMPONENTS_TO_SKIP=<component-ID-1>,<component-ID-2>
# export IMAGE_GROUPS=<image-group-1>,<image-group-2>
```

* 6.2 Update the file with the corresponding information and save it.

```bash
OCP_URL=https://openshift1.example.com:8443 
OPENSHIFT_TYPE=self-manage
IMAGE_ARCH=s390x
OCP_USERNAME=USERNAME
OCP_PASSWORD=PASSWORD
OCP_TOKEN=TOKEN

PROJECT_CERT_MANAGER=
PROJECT_LICENSE_SERVICE=ibm-licensing
PROJECT_SCHEDULING_SERVICE=ibm-cpd-scheduler
PROJECT_CPD_INST_OPERATORS=cpd-operators
PROJECT_CPD_INST_OPERANDS=cpd-operands

STG_CLASS_BLOCK=ibm-spectrum-scale-sc
STG_CLASS_FILE=ibm-spectrum-scale-sc

IBM_ENTITLEMENT_KEY=

COMPONENTS=ibm-licensing,scheduler,cpfs,cpd_platform,db2oltp
```
* 6.3 Confirm that the script does not contain any errors.

```bash
bash ./cpd_vars.sh
```

* 6.4 If you stored passwords in the file, prevent others from reading the file.

```bash
chmod 700 cpd_vars.sh
```

* 6.5 Source cpd_vars.sh file.

```bash
source cpd_vars.sh
```

## 2. Preparing the cluster

1. Updating the global image pull secret

```bash
${CPDM_OC_LOGIN}
```

2. Run the appropriate command to update the global image pull secret:

```bash
cpd-cli manage add-icr-cred-to-global-pull-secret \
--entitled_registry_key=${IBM_ENTITLEMENT_KEY}
```

3. Review nodes.

```bash
cpd-cli manage oc get nodes

watch 'oc get nodes'
```

**NOTE:**
<br>Wait until all the nodes are Ready before you proceed to the next step.

4. Manually creating projects (namespaces) for the shared cluster components.

```bash
oc new-project ${PROJECT_LICENSE_SERVICE}
```

```bash
oc new-project ${PROJECT_SCHEDULING_SERVICE}
```

5. Installing shared cluster components.

* 5.1 Install the License Service.
```bash
cpd-cli manage apply-cluster-components \
--release=${VERSION} \
--license_acceptance=true \
--licensing_ns=${PROJECT_LICENSE_SERVICE}
```

* 5.2 Install the scheduling service.
```bash
cpd-cli manage apply-scheduler \
--release=${VERSION} \
--license_acceptance=true \
--scheduler_ns=${PROJECT_SCHEDULING_SERVICE}
```

6. Changing required node settings.

* 6.1 Load Balancer Timeout.

The minimum recommended timeout is:

    Client timeout: 300s (5m)
    Server timeout: 300s (5m)


```bash
export TIMEOUT_SETTING=<timeout>
```

To increase the timeout client setting.
```bash
sed -i -e "/timeout client/s/ [0-9].*/ ${TIMEOUT_SETTING}/" /etc/haproxy/haproxy.cfg
```

To increase the timeout server setting.
```bash
sed -i -e "/timeout server/s/ [0-9].*/ ${TIMEOUT_SETTING}/" /etc/haproxy/haproxy.cfg
```

Run the following command to apply the changes that you made to the HAProxy configuration.
```bash
systemctl restart haproxy
```

* 6.2 Changing the process IDs limit.

```bash
oc get kubeletconfig
```

Take the appropriate action based on whether the command returns the name of a kubeletconfig.

The command returns the name of a kubeletconfig.

```bash
export KUBELET_CONFIG=<kubeletconfig-name>
```

```bash
oc patch kubeletconfig ${KUBELET_CONFIG} \
--type=merge \
--patch='{"spec":{"kubeletConfig":{"podPidsLimit":16384}}}'
```

The command returns an empty response

```bash
oc apply -f - << EOF
apiVersion: machineconfiguration.openshift.io/v1
kind: KubeletConfig
metadata:
  name: cpd-kubeletconfig
spec:
  kubeletConfig:
    podPidsLimit: 16384
  machineConfigPoolSelector:
    matchExpressions:
    - key: pools.operator.machineconfiguration.openshift.io/worker
      operator: Exists
EOF
```
* 6.3 Changing kernel parameter settings.

```bash
oc get kubeletconfig
```
```bash
oc patch kubeletconfig ${KUBELET_CONFIG} \
--type=merge \
--patch='{"spec":{"kubeletConfig":{"allowedUnsafeSysctls":["kernel.msg*", "kernel.shm*", "kernel.sem"]}}}'
```
```bash
cpd-cli manage apply-db2-kubelet
```

## 3. Prepare to install

1. Checking the health of your cluster.

```bash
cpd-cli health cluster
```

```bash
cpd-cli health nodes
```

```bash
cpd-cli health network-performance \
--verbose \
--save
```

2. Manually creating projects (namespaces) for an instance.

```bash
oc new-project ${PROJECT_CPD_INST_OPERATORS}
```

```bash
oc new-project ${PROJECT_CPD_INST_OPERANDS}
```

3. Applying the required permissions by running the authorize-instance-topology command

```bash
cpd-cli manage authorize-instance-topology \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

## 4. Installing an instance of IBM Software Hub

1. Review the license terms.

```bash
cpd-cli manage get-license \
--release=${VERSION} \
--license-type=EE
```

```bash
cpd-cli manage get-license \
--release=${VERSION} \
--component=db2oltp \
--license-type=DB2SE
```

2. Install the required components for an instance.

```bash
cpd-cli manage setup-instance \
--release=${VERSION} \
--license_acceptance=true \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--run_storage_tests=true
```

3. Confirm that the status of the operands is Completed.

```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

4. Check the health of the resources in the operators and operands project.

```bash
cpd-cli health operators \
--operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--control_plane_ns=${PROJECT_CPD_INST_OPERANDS}
```

```bash
cpd-cli health operands \
--control_plane_ns=${PROJECT_CPD_INST_OPERANDS}
```

5. Get the URL and default credentials of the web client.

```bash
cpd-cli manage get-cpd-instance-details \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--get_admin_initial_credentials=true
```

## 5. Setting up IBM Software Hub

1. Installing privileged monitors for an instance.

```bash
cpd-cli manage apply-privileged-monitoring-service \
--privileged_service_ns=${PROJECT_PRIVILEGED_MONITORING_SERVICE} \
--cluster_components_ns=${PROJECT_SCHEDULING_SERVICE} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

2. Applying Entitlement.

```bash
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=cpd-enterprise
```

## 6. Install Db2 Service.

1. Setting up dedicated nodes

* 1.1 Retrieve the name of the worker node that you want to dedicate to Db2

```bash
oc get nodes
```

* 1.2 Edit the configmap to add the toleration for the custom taint.

```bash
oc edit configmap rook-ceph-operator-config -n openshift-storage
```

* 1.3 Check the configmap to verify that the toleration has been added.

```bash
oc get configmap rook-ceph-operator-config -n openshift-storage -o yaml
```

Example:

```bash
apiVersion: v1
data:
[...]
  CSI_PLUGIN_TOLERATIONS: |
    - key: nodetype
      operator: Equal
      value: infra
      effect: NoSchedule
    - key: node.ocs.openshift.io/storage
      operator: Equal
      value: "true"
      effect: NoSchedule
[...]
kind: ConfigMap
metadata:
[...]
```

* 1.4 Restart the rook-ceph-operator if the csi-cephfsplugin-* and csi-rbdplugin-* pods fail to come up on their own on the infra nodes.

```bash
oc delete -n openshift-storage pod <name of the rook_ceph_operator pod>
```

* 1.5 Taint the node with the NoSchedule effect and safely evict all of the pods from that node.

```bash
oc adm taint node node_name icp4data=dedicated_specifier:NoSchedule --overwrite
```
```bash
oc adm drain <node_name>
```
```bash
oc adm uncordon <node_name>
```

* 1.6 Verify that the csi-cephfsplugin-* and csi-rbdplugin-* pods are running on the Tainted nodes.

```bash
oc get pods -A --field-selector spec.nodeName=<Tainted Node>
```

* 1.7 Label the node.

```bash
oc label node node_name icp4data=dedicated_specifier --overwrite
```

* 1.8 Verify that the node is labeled.

```bash
oc get node --show-labels
```

2. Configuring database storage for Db2.

Pending

3. Creating Custom SCC.

* 3.1 Manually 

* 3.1.1 Set the following environment variables

* 3.1.1.1 Set SCC_NAME to the name that you want to use for the SCC

```bash
export SCC_NAME=<scc-name>
```
* 3.1.1.2 Set SERVICE_ACCOUNT to the name of the service account that you want to bind the SCC to.

 ```bash
 export SERVICE_ACCOUNT=<sa-name>
 ```

* 3.1.1.3 Set ROLE_NAME to the name of the role that will be referenced by the role binding.

 ```bash
export ROLE_NAME=<role-name>
```

* 3.1.1.4 Set ROLEBINDING_NAME to the name of the role binding that will be used to bind the service account to the SCC.

```bash
export ROLEBINDING_NAME=<role-name>
```

* 3.1.1.5 Set PROJECT_CPD_INST_OPERANDS to the project namespace in which the Db2 service is installed. If `cpd_vars.sh` is source, just verify the variable is loaded using `echo` command.

```bash
export PROJECT_CPD_INST_OPERANDS=<namespace>
```

* 3.1.2 Create the service account.

```bash
cat <<EOF |oc apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ${SERVICE_ACCOUNT}
  namespace: ${PROJECT_CPD_INST_OPERANDS}
EOF
```

* 3.1.3 Create the role.

```bash
cat <<EOF |oc apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ${ROLE_NAME}
  namespace: ${PROJECT_CPD_INST_OPERANDS}
rules:
- apiGroups:
  - ""
  resources:
  - endpoints
  - pods
  verbs:
  - get
  - patch
  - update
- apiGroups:
  - apps
  resources:
  - StatefulSets
  - deployments
  - replicasets
  verbs:
  - get
  - list
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
  - patch
  - watch
  - list
  - update
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
  - create
  - update
- apiGroups:
  - db2u.databases.ibm.com
  resources:
  - recipes
  verbs:
  - watch
  - get
  - update
  - create
  - patch
  - list
  - delete
- apiGroups:
  - db2u.databases.ibm.com
  resources:
  - buckets
  verbs:
  - patch
- apiGroups:
  - db2u.databases.ibm.com
  resources:
  - backups
  verbs:
  - patch
  - delete
  - list
- apiGroups:
  - db2u.databases.ibm.com
  resources:
  - formations
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods/exec
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - watch
  - list
  - get
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - watch
  - list
  - get
EOF
```

* 3.1.4 Create the role binding

```bash
cat <<EOF |oc apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ${ROLEBINDING_NAME}
  namespace: ${PROJECT_CPD_INST_OPERANDS}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ${ROLE_NAME}
subjects:
- kind: ServiceAccount
  name: ${SERVICE_ACCOUNT}
  namespace: ${PROJECT_CPD_INST_OPERANDS}
EOF
```

* 3.1.5 Create the SCC

Need to ask here.

* 3.2 Specifying a custom service account, SCC, role, and role binding before deploying Db2

* 3.2.1 Set the Db2 operator replica to 0 by running the following command.

```bash
oc scale deployment ibm-db2oltp-cp4d-operator-controller-manager --replicas=0 -n ${PROJECT_CPD_INST_OPERATORS}
```

* 3.2.2 Open the db2oltp-json-cm ConfigMap object in edit mode.

```bash
oc edit cm db2oltp-json-cm -n ${PROJECT_CPD_INST_OPERANDS}
```

Type /service-account to locate the service-account section of the ConfigMap.

Under the ibm-db2oltp.json data option, change the service account name from db2u to the name of the custom service account that you manually created earlier, step `3.1.1.2 Set SERVICE_ACCOUNT to the name of the service account that you want to bind the SCC to.`

Save the change and exit the ConfigMap.

* 3.2.3 Delete the zen-database-core pod in order for the ConfigMap changes to be re-mounted to the volume.

```bash
oc delete po $(oc get po -n ${PROJECT_CPD_INST_OPERANDS} | grep zen-database-core | awk {'print $1'}) -n ${PROJECT_CPD_INST_OPERANDS}
```

* 3.2.4 Deploy the Db2 instance.

The link is broken atm.

**NOTE:**
<br> **Important:** You must deploy your instance before scaling up the operator. Scaling the operator causes your changes to be overwritten.


* 3.2.5 Set the Db2 operator replica to 1 by running the following command.

```bash
oc scale deployment ibm-db2oltp-cp4d-operator-controller-manager --replicas=1 -n ${PROJECT_CPD_INST_OPERATORS}
```

4. Installing the service

* 4.1 Run the following command to create the required OLM objects for Db2 in the operators project for the instance.

```bash
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=db2oltp
```

* 4.2 Create the custom resource for Db2.

```bash
cpd-cli manage apply-cr \
--components=db2oltp \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--license_acceptance=true
```

* 4.3 Confirm that the custom resource status is Completed.

```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=db2oltp
```

5. Adding privileges to deploy Db2 with the web console.

Pending