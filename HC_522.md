# Health Check Runbook

## Mirror images

### Mirror the network performance image
```bash
cpd-cli manage copy-image \
--from=icr.io/cpopen/cpd/ibm-network-performance:${VERSION} \
--to=${PRIVATE_REGISTRY_LOCATION}/cpopen/cpd/ibm-network-performance:${VERSION}.${IMAGE_ARCH}
```

### Mirror the storage performance image.

```bash
cpd-cli manage copy-image \
--from=icr.io/cpopen/cpd/k8s-storage-perf:${VERSION}  \
--to=${PRIVATE_REGISTRY_LOCATION}/cpopen/cpd/k8s-storage-perf:${VERSION}.${IMAGE_ARCH}
```

## Run Network and Storage healthcheck

### Network Performance Check

```bash
cpd-cli health network-performance \
--image-prefix=${PRIVATE_REGISTRY_LOCATION}/cpopen/cpd \
--image-tag=${VERSION}.${IMAGE_ARCH} \
--verbose \
--save
```

### Storage Performance Check

1. Log into the private image registry 
Depends on the container runtime in your environment, choose from below two options.

* Option1 - Podman login

```bash
podman login ${PRIVATE_REGISTRY_LOCATION} \
-u ${PRIVATE_REGISTRY_PULL_USER} \
-p ${PRIVATE_REGISTRY_PULL_PASSWORD}
```

* Option2 - Docker login

```bash
docker login ${PRIVATE_REGISTRY_LOCATION} \
-u ${PRIVATE_REGISTRY_PULL_USER} \
-p ${PRIVATE_REGISTRY_PULL_PASSWORD}
```

2. Create parameter file for validation health check.

* 2.1 Export variable for namespace.

```bash
export STOR_PERF=ibm-storage-performance
```

* 2.2 Create file.

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

imageurl: ${PRIVATE_REGISTRY_LOCATION}/cpopen/cpd/k8s-storage-perf:${VERSION}.${IMAGE_ARCH}

run_storage_perf: true

storage_perf_namespace: ${STOR_PERF}
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

**NOTE:**
<br>
Remember that the `image-tag` option, it needs to match what it is in the `imageurl` in the `param.yml` file.

3. Run command

```bash
cpd-cli health storage-performance \
--image-prefix=${PRIVATE_REGISTRY_LOCATION}/cpopen/cpd \
--image-tag=${VERSION}.${IMAGE_ARCH} \
--param=storage_perf.yml \
--verbose \
--save
```


## Collect additionanl information

1. Figure out the path of the Health folder
```bash
export HEALTH_PATH=`cpd-cli health runcommand --commands=cluster --log-level=debug --save | grep "Healthcheck resources report path" | awk -F': ' '{print $2}'`
```

2. Collect the limit range
```bash
oc get limitrange -o yaml -n ${PROJECT_CPD_INST_OPERATORS} > $HEALTH_PATH/limitrange.yaml
oc get limitrange -o yaml -n ${PROJECT_CPD_INST_OPERANDS} >> $HEALTH_PATH/limitrange.yaml
```

3. Collect the resource quota
```bash
oc get resourcequota -o yaml -n ${PROJECT_CPD_INST_OPERATORS} > $HEALTH_PATH/resourcequota.yaml
oc get resourcequota -o yaml -n ${PROJECT_CPD_INST_OPERANDS} >> $HEALTH_PATH/resourcequota.yaml
```
4. Collect the network policy
```bash
oc get networkpolicy -o yaml -n ${PROJECT_CPD_INST_OPERATORS} > $HEALTH_PATH/networkpolicy.yaml
oc get networkpolicy -o yaml -n ${PROJECT_CPD_INST_OPERANDS} >> $HEALTH_PATH/networkpolicy.yaml
```