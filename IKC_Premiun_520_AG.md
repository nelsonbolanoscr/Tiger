# Installation IKC Premium for version 5.2.0

## IBM Documentation
* [IKC Premiun](https://www.ibm.com/docs/en/software-hub/5.2.x?topic=services-knowledge-catalog)

## 1. Mirror images to Private Registry

1. Log in to the IBM Entitled Registry registry.

```bash
cpd-cli manage login-entitled-registry \
${IBM_ENTITLEMENT_KEY}
```

2. Log in to the private container registry.

```bash
cpd-cli manage login-private-registry \
${PRIVATE_REGISTRY_LOCATION} \
${PRIVATE_REGISTRY_PUSH_USER} \
${PRIVATE_REGISTRY_PUSH_PASSWORD}
```

If your private registry is not secured omit the following arguments:

    ${PRIVATE_REGISTRY_PUSH_USER}
    ${PRIVATE_REGISTRY_PUSH_PASSWORD}

**Ask Hong Wei ^^**

3. Confirm that you have access to the images that you want to mirror from the IBM Entitled Registry.

* 3.1 Inspect the IBM Entitled Registry.

```bash
export COMPONENTS=wkc
```
```bash
cpd-cli manage list-images \
--components=${COMPONENTS} \
--release=${VERSION} \
--inspect_source_registry=true
```

4. Confirm that the images were mirrored to the private container registry.

```bash
cpd-cli manage list-images \
--components=${COMPONENTS} \
--release=${VERSION} \
--target_registry=${PRIVATE_REGISTRY_LOCATION} \
--case_download=false
```

## 2. Before you Begin

1. Load `cpd_vars.sh` file.

```bash
source ./cpd_vars.sh
```

2. Kernel Parameters.

3. GPU operations.

4. Specify your IBM Knowledge Catalog Edition.

    IBM Knowledge Catalog Premium
    ```bash
    export IKC_TYPE=ikc_premium
    ```

5. Specifying installation options.


## 3. Installing the Service

1. Log in ROCP using `cpd-cli` tool.

```bash
${CPDM_OC_LOGIN}
```

2. Apply OLM objects for IBM Knowledge Catalog.

```bash
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=${IKC_TYPE}
```

3. Create Custom Resource for IBM Knowledge Catalog.

```bash

```

4. Veryfy installation

```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=${IKC_TYPE}
```