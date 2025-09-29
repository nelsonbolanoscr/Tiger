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

## Cleanup

After running the health check, it is a good practice to clean up the resources created during the health check.

1. Save the script as `cleanup.sh`.

```bash
#!/usr/bin/env sh

HELP="
This script lists storage test resources in the supplied namespace. If the '--delete' flag is provided, it removes those resources.

USAGE:
  cleanup.sh --namespace <namespace> <--delete>

FLAGS:
  -c | --command:           (Required) The cpd-cli storage command resources to list and/or clean up, storage-performance|storage-validation.
  --delete:                 Delete storage test resources created in --namespace.
  -n | --namespace:         (Required) The namespace specified in param.yml, storage_validation_namespace|storage_perf_namespace.
  -h | --help:              Show help for cleanup.sh
"
NAMESPACE=
COMMAND=
DELETE_RESOURCES="0"

CM_SEARCH=
JOB_SEARCH=
PVC_SEARCH=
LOCAL_CONTAINER=

log() {
  prefix=""
  case "${1}" in
  info)
    prefix="[INFO ]"
    ;;
  debug)
    prefix="[DEBUG]"
    ;;
  error)
    prefix="[ERROR]"
    ;;
  *)
    prefix=""
    ;;
  esac
  echo "${prefix} ${2}"
}

print_usage() {
  log noop "${HELP}"
}

set_vars() {
  if [ "${COMMAND}" == "storage-performance" ]; then
    PVC_SEARCH="pvc-sysbench-rwo\|pvc-sysbench-rwx"
    JOB_SEARCH="sysbench-random-\|sysbench-sequential-"
    LOCAL_CONTAINER="k8s-storage-perf"
  fi

  if [ "${COMMAND}" == "storage-validation" ]; then
    CM_SEARCH="consumer\|producer"
    PVC_SEARCH="readiness-readwritemany\|readiness-readwriteonce"
    JOB_SEARCH="readiness-consumer\|readiness-create\|readiness-edit\|readiness-file\|readiness-mount\|readiness-producer\|readiness-read"
    LOCAL_CONTAINER="k8s-storage-val"
  fi
}

list() {
  echo "\nListing ${COMMAND} cluster resources in namespace: ${NAMESPACE}"
  
  if [ -n "${CM_SEARCH}" ]; then
    echo "\nCONFIGMAPS:"
    oc get cm -n ${NAMESPACE} | sed -n "1p;/${CM_SEARCH}/p"
  fi
  
  echo "\nPVCS:"
  oc get pvc -n ${NAMESPACE} | sed -n "1p;/${PVC_SEARCH}/p"
  echo "\nJOBS:"
  oc get job -n ${NAMESPACE} | sed -n "1p;/${JOB_SEARCH}/p"

  echo "\nLocal ${COMMAND} playbook container:"
  podman ps -a --filter="name=${LOCAL_CONTAINER}"
  echo ""
}

delete_resources() {
    echo "\nDeleting ${COMMAND} cluster resources from namespace: ${NAMESPACE}"
    oc get job -n ${NAMESPACE} -o name | grep "${JOB_SEARCH}" | xargs -I % -n 1 oc delete % -n ${NAMESPACE}

    if [ -n "${CM_SEARCH}" ]; then
      oc get cm -n ${NAMESPACE} -o name | grep "${CM_SEARCH}" | xargs -I % -n 1 oc delete % -n ${NAMESPACE}
    fi
    
    sleep 10
    oc get pvc -n ${NAMESPACE} -o name | grep "${PVC_SEARCH}" | xargs -I % -n 1 oc delete % -n ${NAMESPACE}

    echo "\nRemoving local ${COMMAND} container:"
    podman rm -f ${LOCAL_CONTAINER}
    echo ""
}

run() {
  # confirm required flags have been provided  
  if [ -z "${NAMESPACE}" ]; then
    log error "You must specify the --namespace option. For example: --namespace <project-name>"
    print_usage
    exit 1
  fi
  if [ "${COMMAND}" != "storage-performance" ] && [ "${COMMAND}" != "storage-validation" ]; then
    log error "You must specify the --command option. For example: --command <storage-performance|storage-validation>"
    print_usage
    exit 1
  fi

  # set command-specific variables
  set_vars

  # show resources
  list

  # delete storage test resources and re-run list to confirm deletion
  if [ "${DELETE_RESOURCES}" -eq "1" ]; then
    delete_resources
  fi

  exit 0
}

# Parse CLI args
while [ "${1-}" != "" ]; do
    case $1 in
    --namespace | -n)
        shift
        NAMESPACE="${1}"
        ;;
    --command | -c)
        shift
        COMMAND="${1}"
        ;;    
    --delete)
        shift
        DELETE_RESOURCES="1"
        ;;
    --help | -h)
        print_usage 0
        exit 0
        ;;
    *)
        echo "Invalid Option ${1}" >&2
        exit 1
        ;;
    esac
    shift
done

run
```

2. Change permissions.

```bash
chmod +x ./cleanup.sh
```

3. List resources.<br> 
If you set the variable of `STOR_VAL` and `STOR_PERF`, please use the following command, otherwise replace with the namespace accordingly.

```bash
./cleanup.sh --namespace ${STOR_VAL} --command storage-validation

OR

./cleanup.sh --namespace ${STOR_PERF} --command storage-performance
```

4. Delete resources.<br> 
If you set the variable of `STOR_VAL` and `STOR_PERF`, please use the following command, otherwise replace with the namespace accordingly.

```bash 
./cleanup.sh --namespace ${STOR_VAL} --command storage-validation --delete

OR

./cleanup.sh --namespace ${STOR_PERF} --command storage-performance --delete
```