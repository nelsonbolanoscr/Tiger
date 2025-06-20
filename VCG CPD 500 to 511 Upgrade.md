# CPD Upgrade Runbook - v.5.0.0 to 5.1.1

## Upgrade documentation

* [Upgrading from IBM Cloud Pak for Data Version 5.0.0 to Version 5.1.1](https://www.ibm.com/docs/en/software-hub/5.1.x?topic=upgrading-from-cloud-pak-data-version-50)

## Upgrade context

From

```yaml
OCP: 4.14
CDP: 5.0.0
Storage: Netapp Trident
Components: ibm-cert-manager,ibm-licensing,cpfs,cpd_platform,watson_assistant,watson_discovery
```

To

```yaml
OCP: 4.14
CPD: 5.1.1
Storage: Netapp Trident
Components: ibm-cert-manager,ibm-licensing,cpfs,cpd_platform,watson_assistant,watson_discovery
```

## Table of Content

* [1. Set up client workstation](VCG%20CPD%20511%20Upgrade.md#1-set-up-client-workstation)
  - [1.1 Prepare the client workstation](VCG%20CPD%20511%20Upgrade.md#11-prepare-the-client-workstation)
  - [1.2 Update Environment Variables for the upgrade to Version 5.1.1](VCG%20CPD%20511%20Upgrade.md#12-update-environment-variables-for-the-upgrade-to-version-511)
  - [1.3 Ensure the cpd-cli manage plug-in the latest version of the olm-utils image](VCG%20CPD%20511%20Upgrade.md#13-ensure-the-cpd-cli-manage-plug-in-the-latest-version-of-the-olm-utils-image)
  - [1.4 Health Check](VCG%20CPD%20511%20Upgrade.md#14-health-check)
* [2. Upgrade CPD to 5.1.1](VCG%20CPD%20511%20Upgrade.md#2-upgrade-cpd-to-511)
  - [2.1 Upgrading prerequisite software](VCG%20CPD%20511%20Upgrade.md#21-upgrading-prerequisite-software)
  - [2.2 Upgrading shared cluster components](VCG%20CPD%20511%20Upgrade.md#22-upgrading-shared-cluster-components)
  - [2.3 Preparing to upgrade instance of IBM Software Hub](VCG%20CPD%20511%20Upgrade.md#23-preparing-to-upgrade-instance-of-ibm-software-hub)
  - [2.4 Upgrade IBM Software Hub](VCG%20CPD%20511%20Upgrade.md#24-upgrade-ibm-software-hub)
  - [2.5 Updating cpdbr service](VCG%20CPD%20511%20Upgrade.md#25-updating-cpdbr-service)

## 1. Set up client workstation

### 1.1 Prepare the client workstation
1. Prepare a RHEL 9 machine with internet

* 1.1 Create a directory for cpd-cli utility.

```bash
export CPD511_WORKSPACE=/ibm/cpd/511
mkdir -p ${CPD511_WORKSPACE}
cd ${CPD511_WORKSPACE}
``` 

* 1.2 Download the cpd-cli for 5.1.1.

```bash
wget https://github.com/IBM/cpd-cli/releases/download/v14.1.1/cpd-cli-linux-EE-14.1.1.tgz
```

2. Install the tool.

```bash
tar xvf cpd-cli-linux-EE-14.1.1.tgz
mv cpd-cli-linux-EE-14.1.1-1650/* .
rm -rf cpd-cli-linux-EE-14.1.1-1650
```

3. Copy the cpd_vars.sh file used for previous installations to the folder `${CPD511_WORKSPACE}`.

```bash
cp <File path of the previous cpd_vars.sh> cpd_vars_511.sh
```

4. Make cpd-cli utility executable anywhere.

```bash
vi cpd_vars_511.sh
```

* 4.1 Add below two lines into the head of `cpd_vars_511.sh`.

```bash
export CPD511_WORKSPACE=/ibm/cpd/511
export PATH=${CPD511_WORKSPACE}:$PATH
```

* 4.2 Update the `CPD_CLI_MANAGE_WORKSPACE` variable.

```bash
export CPD_CLI_MANAGE_WORKSPACE=${CPD511_WORKSPACE}
```

* 4.3 Save the changes and run this command to apply `cpd_vars_511.sh`.

```bash
source cpd_vars_511.sh
```

* 4.4 Check out with this command.

```bash
cpd-cli version
```

* 4.5 Output should be like this.

```bash
cpd-cli
        Version: 14.1.1
        Build Date: 2025-02-20T18:45:49
        Build Number: 1650
        CPD Release Version: 5.1.1
```

<!-- 5. Update the OpenShift CLI

* 5.1 Check the OpenShift CLI version.

```
oc version
```

**NOTE:**
<br>If the version doesn't match the OpenShift cluster version, update it accordingly. -->

### 1.2 Update Environment Variables for the upgrade to Version 5.1.1

1. Locate the VERSION entry and update the environment variable for VERSION.

```bash
vi cpd_vars_511.sh
export VERSION=5.1.1
```

2. Locate the COMPONENTS entry and confirm the COMPONENTS entry is accurate.

```bash
COMPONENTS=ibm-cert-manager,scheduler,ibm-licensing,cpfs,cpd_platform,watson_assistant,watson_discovery
```

3. Save the changes.

4. Confirm that the script does not contain any errors.

```bash
bash ./cpd_vars_511.sh
```

5. Run this command to apply `cpd_vars_511.sh`.

```bash
source cpd_vars_511.sh
```

### 1.3 Ensure the cpd-cli manage plug-in the latest version of the olm-utils image

1. Run the following command to ensure that the cpd-cli is installed and running and that the cpd-cli manage plug-in has the latest version of the olm-utils image.

```bash
cpd-cli manage restart-container
```

2. Check and confirm the olm-utils-v3 container is up and running.

```bash
podman ps | grep olm-utils-v3
```

### 1.4 Health Check

1. Check OCP status

* 1.1 Login Bastion node and log in to OCP cluster.
```bash
${CPDM_OC_LOGIN}
```

* 1.2 Make sure All the cluster operators should be in `AVAILABLE` status, and not in `PROGRESSING` or `DEGRADED` status.

```bash
oc get co
```

* 1.3 Review nodes status, make sure all the nodes are in `READY` status.

```bash
oc get nodes
```

* 1.4 Review the machine configure pool are in healthy status.

```bash
oc get mcp
```

2. Check CPD status

* 2.1 Login Bastion node and log in to OCP cluster. Ensure `cli-cpd`command-line interface is installed properly.

```bash
${CPDM_OC_LOGIN}
```

* 2.2 Review Services are in `READY` status.

```bash
cpd-cli manage get-cr-status --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

* 2.3 Review pods are healthy.

```bash
oc get po --no-headers --all-namespaces -o wide | grep -Ev '([[:digit:]])/\1.*R' | grep -v 'Completed'
```

3. Check private container registry status if installed

* 3.1 Login Bastion node, where the private container registry is usually installed, as root. Run this command in terminal and make sure it can succeed.

```bash
podman login --username $PRIVATE_REGISTRY_PULL_USER --password $PRIVATE_REGISTRY_PULL_PASSWORD $PRIVATE_REGISTRY_LOCATION --tls-verify=false
```

* 3.2 You can run this command to verify the images in private container registry.

```bash
curl -k -u ${PRIVATE_REGISTRY_PULL_USER}:${PRIVATE_REGISTRY_PULL_PASSWORD} https://${PRIVATE_REGISTRY_LOCATION}/v2/_catalog?n=6000 | jq .
```
## 2. Upgrade CPD to 5.1.1

### 2.1 Upgrading prerequisite software

#### 2.1.1 Upgrading Red Hat OpenShift Serverless Knative Eventing

1. Upgrade IBM Events Operator.

```bash
${CPDM_OC_LOGIN}
```

```bash
cpd-cli manage authorize-instance-topology \
--release=${VERSION} \
--cpd_operator_ns=ibm-knative-events \
--cpd_instance_ns=knative-eventing
```

```bash
cpd-cli manage setup-instance-topology \
--release=${VERSION} \
--cpd_operator_ns=ibm-knative-events \
--cpd_instance_ns=knative-eventing \
--license_acceptance=true
```

<!-- ### Known issue:
https://github.ibm.com/PrivateCloud-analytics/CPD-Quality/issues/42294 -->

Review `${STG_CLASS_BLOCK}` is set in the `cpd_vars.sh` file

```bash
cpd-cli manage deploy-knative-eventing \
--release=${VERSION} \
--block_storage_class=${STG_CLASS_BLOCK} \
--upgrade=true
```

### 2.2 Upgrading shared cluster components

1. Run the cpd-cli manage login-to-ocp command to log in to the cluster.

```bash
${CPDM_OC_LOGIN}
```

2. Confirm the project which the License Service is in, run the following command.

```bash
oc get deployment -A |  grep ibm-licensing-operator
```

**NOTE:**
<br>Make sure the project returned by the command matches the environment variable `PROJECT_LICENSE_SERVICE` in your environment variables script `cpd_vars_511.sh`.

3. Confirm the scheduler services project is installed on the cluster, run the following command.

```bash
oc get scheduling -A
```

**Note:**
<br>Make sure the project returned by the command matches the environment variable `PROJECT_SCHEDULING_SERVICE` in your environment variables script `cpd_vars_511.sh`.

4. Upgrade the Certificate Manager and License Service.

```bash
cpd-cli manage apply-cluster-components \
--release=${VERSION} \
--license_acceptance=true \
--licensing_ns=${PROJECT_LICENSE_SERVICE}
```

5. Confirm License Service pods are Running or Completed.

```bash
oc get pods --namespace=${PROJECT_LICENSE_SERVICE}
```

<!-- 6. Confirm IBM Certificate Manager pods are Running or Completed.

```bash
oc get pods --namespace=${PROJECT_CERT_MANAGER}
``` -->

6. Upgrade Scheduling Service.

```bash
cpd-cli manage apply-scheduler \
--release=${VERSION} \
--license_acceptance=true \
--scheduler_ns=${PROJECT_SCHEDULING_SERVICE}
```

7. Confirm Scheduling service pods are Running or Completed.

```bash
oc get pods --namespace=${PROJECT_SCHEDULING_SERVICE}
```

### 2.3 Preparing to upgrade instance of IBM Software Hub

1. Applying Entitlements.

```bash
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=cpd-enterprise
```

```bash
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=watson-discovery
```

```bash
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=watson-assistant
```

### 2.4 Upgrade IBM Software Hub

1. Review License Term.

```bash
cpd-cli manage get-license \
--release=${VERSION} \
--license-type=EE
```

2. Upgrade operators and custom resources.

```bash
cpd-cli manage setup-instance \
--release=${VERSION} \
--license_acceptance=true \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

3. Upgrade operators for services

```bash
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--upgrade=true
```

4. Confirm all the operators pods are Running or Completed and custom resources.

* 4.1 Verify pods.
```bash
oc get pods --namespace=${PROJECT_CPD_INST_OPERATORS}
```

* 4.2 Verify Custom Resources.

```bash
cpd-cli manage get-cr-status --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

5. Upgrading CDP Services to 5.1.1.

* 5.1 Upgrading Watson Discovery.

```bash
cpd-cli manage apply-cr \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=watson_discovery \
--license_acceptance=true \
--upgrade=true
```
<!-- **NOTE:**
<br>Review install-options.yml is configured properly for the service that it is going to be upgraded. -->

* 5.2 Review installation progress.

* 5.2.1 Find the pod of the service.

```bash
oc get pods -n ${PROJECT_CPD_INST_OPERATORS} | grep -i discovery
```

The output should be something like this:

```bash
oc get pods -n ${PROJECT_CPD_INST_OPERATORS} | grep -i discovery
ibm-watson-discovery-operator-catalog-nfzg7                       1/1     Running     0               4h31m
wd-discovery-operator-f4bd9688b-qg2j6                             1/1     Running     1 (4h16m ago)   4h19m
```

* 5.2.2 Review logs of the pod:

```bash
oc logs wd-discovery-operator-f4bd9688b-qg2j6 -n ${PROJECT_CPD_INST_OPERATORS}
```

* 5.2.3 Review the progress of the update.
```bash
watch 'oc get WatsonDiscovery wd -n ${PROJECT_CPD_INST_OPERANDS} --output jsonpath="{.status.progress} {.status.componentStatus.deployed} {.status.componentStatus.verified}"'
```

* 5.3 Validate the upgrade.

```bash
cpd-cli manage get-cr-status --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

* 5.4 Removing the previous PostgreSQL cluster

*  5.4.1 Confirm that the status of the Watson Discovery service is shown as `READY`.
```bash
oc get watsonDiscovery wd --namespace=${PROJECT_CPD_INST_OPERANDS}
```

* 5.4.2 Run the following command to remove the previous PostgreSQL cluster.

```bash
oc delete cluster.postgresql wd-discovery-cn-postgres
```

* 5.5 Upgrading Watson Assistant.

```bash
cpd-cli manage apply-cr \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=watson_assistant \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--license_acceptance=true \
--upgrade=true
```

<!-- Defect:
https://github.ibm.com/PrivateCloud-analytics/CPD-Quality/issues/42297 -->

<!-- **NOTE:**
<br>Review install-options.yml is configured properly for the service that it is going to be upgraded. -->

* 5.6 Review installation progress.

* 5.6.1 Find the pod of the service.

```bash
oc get pods -n ${PROJECT_CPD_INST_OPERATORS} | grep -i assistant
```

The output should be something like this:

```bash
oc get pods -n ${PROJECT_CPD_INST_OPERATORS} | grep -i assistant
ibm-watson-assistant-operator-64f579c54-c2gtl                     1/1     Running     0               4h40m
ibm-watson-assistant-operator-catalog-xcm6x                       1/1     Running     0               4h40m
```

* 5.6.2 Review logs of the pod:

```bash
oc logs ibm-watson-assistant-operator-64f579c54-c2gtl -n ${PROJECT_CPD_INST_OPERATORS}

OR

oc exec -it <watson-assistant-pod name> -n ${PROJECT_CPD_INST_OPERATORS} sh
```
* 5.6.3 When you connect to the pod:

```bash
tail -f watsonassistant.wa.log
```

* 5.6.4 Review the progress of the update.
```bash
watch 'oc get WatsonAssistant wa -n ${PROJECT_CPD_INST_OPERANDS} --output jsonpath="{.status.progress} {.status.componentStatus.deployed} {.status.componentStatus.verified}"'
```

* 5.7 Validate the upgrade.

```bash
cpd-cli manage get-cr-status --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

6. Validate EBS Postgres Upgrade.

* 6.1 Set an instance enviromental variable

```bash
export INSTANCE=wa
```

* 6.2 Make sure wa instance goes to fully verified state.

```bash
oc get wa
```

* 6.3 Check if `$INSTANCE-postgres-16` cluster is in healthy or ready state.

```bash
oc get cluster $INSTANCE-postgres-16

oc get pods -l app=$INSTANCE-postgres-16
```

* 6.4 Check whether all the jobs are in completed state.

```bash
oc get jobs -l slot=$INSTANCE
```

* 6.5 Check whether the store cronjob is pointing to the new EDB Postgres-16 cluster.

```bash
oc get cronjob $INSTANCE-store-cronjob  -o yaml | grep postgres
```

* 6.6 If wa-postgres-16-rw.cpd.svc points to old EDB Postgres cluster instead of EDB Postgres-16, then delete the cronjob with the following command. This helps the operator to create a new cronjob with the latest config.

```bash
oc delete cronjob $INSTANCE-store-cronjob
```

* 6.6 Login to IBM Cloud Pak for Data console.

* 6.7 Test the upgrade by sending a message to one of the old instances:

    * Click Launch tool and click one of the watsonx Assistant service instances.
    * Type a message in the preview chat window to trigger the actions that are defined for the watsonx Assistant before upgrade.
    * Check whether you received the expected response without training the watsonx Assistant again.

7. Cleaning up resources.

* 7.1 Set an instance environment variable.

```bash
export INSTANCE=wa
```

* 7.2 Delete old EDB Postgres Version 12 cluster.

```bash
oc delete cluster -l app=$INSTANCE-postgres,component=postgres,service=conversation,slot=$INSTANCE
```

* 7.3 Delete old certificate created for EDB Postgres Version 12 cluster.

```bash
oc delete cert -l component=cert-postgres,service=conversation,slot=$INSTANCE
```

* 7.4 Delete old secrets created by the preceding certificate.

```bash
oc delete secret -l component=cert-postgres,service=conversation,slot=$INSTANCE
```

* 7.5 Delete the watsonx Assistant created old secrets.

```bash
oc delete secret -l app=$INSTANCE-postgres,component=postgres,service=conversation
```

8. Enable `zen-rsi-evictor-cron-job`.

```bash
oc patch CronJob zen-rsi-evictor-cron-job \
--namespace=${PROJECT_CPD_INST_OPERANDS} \
--type=merge \
--patch='{"spec":{"suspend": false}}'
```

9. Apply RSI patches.

```bash
cpd-cli manage apply-rsi-patches \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

### 2.5 Updating cpdbr service

1. Upgrade the cpdbr-tenant component for the instance. 

**NOTE:**
<br>This can be doing in a maintenance window

```bash
export OADP_OPERATOR_NS=<oadp-operator-project>
```

```bash
cpd-cli oadp install \
--component=cpdbr-tenant \
--cpdbr-hooks-image-prefix=${PRIVATE_REGISTRY_LOCATION} \
--namespace=${OADP_OPERATOR_NS} \
--tenant-operator-namespace=${PROJECT_CPD_INST_OPERATORS} \
--cpd-scheduler-namespace=${PROJECT_SCHEDULING_SERVICE} \
--skip-recipes=true \
--upgrade=true \
--log-level=debug \
--verbose
```