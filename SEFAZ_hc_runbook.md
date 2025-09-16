# SEFAZ Health Check Runbook


## CP4D Checks

### Cluster, Nodes, Operators and Operands.

```bash
cpd-cli health runcommand \
--commands=cluster,nodes,operands,operators \
--control_plane_ns="${PROJECT_CPD_INST_OPERANDS}" \
--operator_ns="${PROJECT_CPD_INST_OPERATORS}" \
--log-level=debug \
--verbose \
--save
```

### Storage Validation.

1. Create parameter file for validation health check.

```bash
cat <<EOF > storage_val.yml
# OCP Parameters
ocp_url: ${OCP_URL}
ocp_username: ${OCP_USERNAME}
ocp_password: ${OCP_PASSWORD}
ocp_token: ${OCP_TOKEN}

storageClass_ReadWriteOnce: ${STG_CLASS_BLOCK}
storageClass_ReadWriteMany: ${STG_CLASS_FILE}
arch: ${IMAGE_ARCH}

imageurl: icr.io/cpopen/cpd/k8s-storage-test:${VERSION}

run_storage_readiness: true

storage_validation_namespace: ibm-storage-validation
prefix: "readiness"
storageSize: "1Gi"

options: ""
backoffLimit: 5
EOF
```

2. Run command.

```bash
cpd-cli health storage-validation \
--param ./storage_val.yml \
--verbose \
--save
```

### Storage Performance.

1. Create parameter file for validation health check.

```bash
cat <<EOF > storage_perf.yml
# OCP Parameters
ocp_url: ${OCP_URL}
ocp_username: ${OCP_USERNAME}
ocp_password: ${OCP_PASSWORD}
ocp_token: ${OCP_TOKEN}
ocp_apikey: <required if neither user/password or token not available>

storageClass_ReadWriteOnce: ${STG_CLASS_BLOCK}
storageClass_ReadWriteMany: ${STG_CLASS_FILE}
arch: ${IMAGE_ARCH}

imageurl: icr.io/cpopen/cpd/k8s-storage-perf:${VERSION}

run_storage_perf: true

storage_perf_namespace: ibm-storage-performance
logfolder: '.logs'

cluster_infrastructure: <optional>
cluster_name: <optional>
storage_type: <optional>

dedicated_compute_node:
   label_key: "<optional>"
   label_value: "<optional>"

rwx_storagesize: 10Gi
rwo_storagesize: 10Gi

file_extra_flags: dsync

sysbench_random_read: false
rread_threads: 8
rread_fileTotalSize: 128m
rread_fileNum: 128
rread_fileBlockSize: 4k

sysbench_random_write: true
rwrite_threads: 8
rwrite_fileTotalSize: 4096m
rwrite_fileNum: 4
rwrite_fileBlockSize: 4k

sysbench_sequential_read: false
sread_threads: 2
sread_fileTotalSize: 4096m
sread_fileNum: 4
sread_fileBlockSize: 1g

sysbench_sequential_write: true
swrite_threads: 2
swrite_fileTotalSize: 4096m
swrite_fileNum: 4
swrite_fileBlockSize: 1g
EOF
```

2. Run command.

```bash
cpd-cli health storage-performance \
--param ./storage_perf.yml \
--verbose \
--save
```

### Network Connectivity.

```bash
cpd-cli health network-connectivity \
--control_plane_ns="${PROJECT_CPD_INST_OPERANDS}" \
--verbose \
--save
```

### Network Performance.

```bash
cpd-cli health network-performance \
--log-level=debug \
--minbandwidth=350 \
--verbose \
--save
```