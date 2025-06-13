# Installation IBM Match 360 for version 5.1.1

## IBM Documentation
* [IBM Match 360](https://www.ibm.com/docs/en/software-hub/5.1.x?topic=services-match-360)

## 1. Health Check

1. General OCP

```bash
oc get nodes,mcp,co > ocp_status.txt
oc version > oc_version.txt
oc get pods -A > pod_list.txt
oc describe nodes > nodes_desc.txt
```

2. Review Custom Resources status.

```bash
cpd-cli manage get-cr-status --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} > cr_status.txt
```

3. Validate Cluster, Nodes, Operands and Operators

```bash
cpd-cli health runcommand \
--commands=cluster,nodes,operands,operators \
--control_plane_ns=${PROJECT_CPD_INST_OPERANDS} \
--log-level=debug \
--operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--save \
--verbose
```

4. Storage status.

```bash
# Get more details of the ocs cluster

oc describe cephcluster ocs-storagecluster-cephcluster -n openshift-storage > odf-status.txt

# Verify that the provisioner was created and that the corresponding daemon sets were created by:

oc get all -n openshift-local-storage > local-storage.txt

# Verify the created storage class:

oc get sc
```

## 2. Install Instuctions

1. Verify the `cpd_vars.sh` and load it.

```bash
source ./cpd_vars.sh
```

2. Login ROCP using `cpd-cli` tool.

```bash
${CPDM_OC_LOGIN}
```

3. Apply OLM objects for IBM Match 360.

```bash
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=match360
```

4. Create the Custom Resource for IBM Match 360.

```bash
cpd-cli manage apply-cr \
--components=match360 \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--license_acceptance=true
```

5. Review Custom Resources status.

```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=match360
```

## 3. Install Hotfix

1. Apply patch:

```bash
oc patch mdm mdm-cr --type=merge -p '{"spec":{"image_digests":{"mdm_configuration":"sha256:fdcea573b9fbc1a8107d303d7df2061a566990461ef880b33c5af2f3c33e6c8d","mdm_data":"sha256:64ef6e1ecb5780c3628c7f4b6bbb80beb1af72f9b14b10bf9a34da079256078c","mdm_matching":"sha256:5e43cebb400bdeecc0914ad92e7a49c3f7dbe8847535feaa1506f264ba62dac"}}}' 
 
oc patch mdm mdm-cr --type=merge -p '{"spec":{"mdm_data":{"spark":{"image": {"tag":"sha256:f7fc1b38363e2b7b0df178fac566441e938de17951ca87412bf335d9aec00eed" }}}}}'
```

2. Monitor status IBM Match 360 operator reconcilation.

```bash
oc get mdm mdm-cr -n ${PROJECT_CPD_INST_OPERANDS}
```

3.  Update mdm operator configmap.

- 3.1 Capture `mdm-instance-id` value

```bash
oc get mdm mdm-cr -n ${PROJECT_CPD_INST_OPERANDS}
```

- 3.2 Patch configmap.

```bash
oc label cm mdm-operator-<mdm-instance-id> icpdsupport/addOnId=mdm -n ${PROJECT_CPD_INST_OPERANDS} 