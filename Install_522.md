

## 1. Set up client workstation

### Prepare the client workstation
1. Prepare a RHEL 9 machine with internet.

**NOTE:**
<br> The following instruction works to update the current version of `cpd-cli`.

* 1.1 Create a directory for cpd-cli utility.

```bash
export CPD522_WORKSPACE=/ibm/cpd/522
mkdir -p ${CPD522_WORKSPACE}
cd ${CPD522_WORKSPACE}
``` 

* 1.2 Download the cpd-cli for 5.2.2.

```bash
wget https://github.com/IBM/cpd-cli/releases/download/v14.2.2/cpd-cli-linux-EE-14.2.2.tgz
```

2. Install the tool.

* 2.1 Untar content.
```bash
tar xvf cpd-cli-s390x-EE-14.2.2.tgz
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
<br>https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/security_and_compliance/cert-manager-operator-for-red-hat-openshift#cert-manager-operator-install


4. Obtaining IBM Entitlement API key.

* 4.1 Log in to [Container software library on My IBM](https://myibm.ibm.com/products-services/containerlibrary) with the IBMid and password that are associated with the entitled software.

* 4.2 On the **Entitlement** keys tab, select **Copy** to copy the entitlement key to the clipboard.

* 4.3 Save the API key in a text file.

5. Determining components to install.

```bash
db2oltp,datagate
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

export VERSION=5.2.2


# ------------------------------------------------------------------------------
# Components
# ------------------------------------------------------------------------------

export COMPONENTS=ibm-licensing,scheduler,cpfs,cpd_platform
# export COMPONENTS_TO_SKIP=<component-ID-1>,<component-ID-2>
# export IMAGE_GROUPS=<image-group-1>,<image-group-2>
```

* 6.2 Update the file with the corresponding information and save it.

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
cpd-cli manage add-cred-to-global-pull-secret \
--registry=${PRIVATE_REGISTRY_LOCATION} \
--registry_pull_user=${PRIVATE_REGISTRY_PULL_USER} \
--registry_pull_password=${PRIVATE_REGISTRY_PULL_PASSWORD}
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

You cannot change the timeout settings in Azure Red Hat OpenShift (ARO) environments. The default timeout value is 4 minutes (240 seconds).

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

**Managed OpenShift**
<br>You cannot change the node settings. You must allow Db2U to run with elevated privileges.
<br>During the installation of the services, an instance administrator [specifies the privileges that Db2U runs with](https://www.ibm.com/docs/en/software-hub/5.2.x?topic=services-specifying-privileges-that-db2u-runs). In Managed OpenShift environments, the administrator must set Db2U to run with elevated privileges.

* 6.4 Specifying the privileges that Db2U runs with

Eleveted Privileges.

```
oc apply -f - <<EOF
apiVersion: v1
data:
  DB2U_RUN_WITH_LIMITED_PRIVS: "false"
kind: ConfigMap
metadata:
  name: db2u-product-cm
  namespace: ${PROJECT_CPD_INST_OPERATORS}
EOF
```

## 3. Prepare to install

1. Manually creating projects (namespaces) for an instance.

```bash
oc new-project ${PROJECT_CPD_INST_OPERATORS}
```

```bash
oc new-project ${PROJECT_CPD_INST_OPERANDS}
```

2. Applying the required permissions by running the authorize-instance-topology command

```bash
cpd-cli manage authorize-instance-topology \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

3. Annotating IBM Software Hub projects (namespaces) to enable embedded Db2 databases to use the restricted-v2 SCC

https://www.ibm.com/docs/en/software-hub/5.2.x?topic=hub-annotating-projects-embedded-db2-databases

## 4. Installing an instance of IBM Software Hub

1. Review the license terms.

```bash
cpd-cli manage get-license \
--release=${VERSION} \
--license-type=EE
```

```bash
Standard Edition
cpd-cli manage get-license \
--release=${VERSION} \
--component=db2oltp \
--license-type=DB2SE

Or
Advanced Edition
cpd-cli manage get-license \
--release=${VERSION} \
--component=db2oltp \
--license-type=DB2AE
```
```bash
IBM Data Gate for watsonx
cpd-cli manage get-license \
--release=${VERSION} \
--component=datagate \
--license-type=DGWXD
```

2. Install the required components for an instance.

```bash
cpd-cli manage setup-instance \
--release=${VERSION} \
--license_acceptance=true \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE}
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
2. Installing the IBM Software Hub configuration admission controller webhook

* 2.1 Install the configuration admission controller webhook.

```bash
cpd-cli manage install-cpd-config-ac \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

* 2.2 Enable the configuration admission controller webhook.

```bash
cpd-cli manage enable-cpd-config-ac \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

3. Applying Entitlement.

```bash
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=cpd-enterprise
```

## 6. Install Db2 Service

1. Installing the service

* 1.1 Run the following command to create the required OLM objects for Db2 in the operators project for the instance.

```bash
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=db2oltp
```

* 1.2 Create the custom resource for Db2.

```bash
cpd-cli manage apply-cr \
--components=db2oltp \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--license_acceptance=true
```

* 1.3 Confirm that the custom resource status is Completed.

```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=db2oltp
```

## 7. Install DataGate Service

1. Prerequisite

* 1.1 You must install, provision, and configure a Db2 or Db2 Warehouse instance on Cloud Pak for Data for each Data Gate instance.

2. Installing the service.

* 2.1 Run the following command to create the required OLM objects for Data Gate in the operators project for the instance.

```bash
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=datagate
```

* 2.2 Create the custom resource for Data Gate.

```bash
cpd-cli manage apply-cr \
--components=datagate \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--license_acceptance=true
```
* 2.3 Confirm that the custom resource status is Completed.

```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=datagate
```