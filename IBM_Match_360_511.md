# Installation IBM Match 360 for version 5.1.1

## IBM Documentation
* [IBM Match 360](https://www.ibm.com/docs/en/software-hub/5.1.x?topic=services-match-360)

## 1. Install Instuctions

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
--param-file=/tmp/work/install-options.yml \
--license_acceptance=true
```

5. Review Custom Resources status.

```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=match360
```