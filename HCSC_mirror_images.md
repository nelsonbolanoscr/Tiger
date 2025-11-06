# Mirroring images

## Obtaining the olm-utils-v3 image

1. Same workstation inside the cluster network.

```bash
cpd-cli manage restart-container
```
Wait for the cpd-cli to return the following messages:
```bash
[SUCCESS] ... Successfully pulled the container image icr.io/cpopen/cpd/olm-utils-v3:latest
[SUCCESS] ... Successfully started the container olm-utils-play
[SUCCESS] ... Container olm-utils-play has been re-created
```

2. Different workstation inside the cluster network.

From a workstation that can connect to the internet:

    Ensure that Docker or Podman is running on the workstation.
    Run the following command to save the olm-utils-v3 image to the client workstation:

    x86-64 clusters

        cpd-cli manage save-image \
        --from=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.amd64

    This command saves the image as a compressed TAR file named icr.io_cpd_olm-utils-v3_${VERSION}.amd64.tar.gz in the cpd-cli-workspace/olm-utils-workspace/work/offline directory.
ppc64le clusters

    cpd-cli manage save-image \
    --from=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.ppc64le

    This command saves the image as a compressed TAR file named icr.io_cpopen_cpd_olm-utils-v3_${VERSION}.ppc64le.tar.gz in the cpd-cli-workspace/olm-utils-workspace/work/offline directory.
s390x clusters

    cpd-cli manage save-image \
    --from=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.s390x

        This command saves the image as a compressed TAR file named icr.io_cpopen_cpd_olm-utils-v3_${VERSION}.s390x.tar.gz in the cpd-cli-workspace/olm-utils-workspace/work/offline directory.

Transfer the compressed file to a client workstation that can connect to the cluster.

Ensure that you place the TAR file in the work/offline directory:

x86-64 clusters
    icr.io_cpd_olm-utils-v3_${VERSION}.amd64.tar.gz
ppc64le clusters
    icr.io_cpopen_cpd_olm-utils-v3_${VERSION}.ppc64le.tar.gz
s390x clusters
    icr.io_cpopen_cpd_olm-utils-v3_${VERSION}.s390x.tar.gz

From the workstation that can connect to the cluster:

    Ensure that Docker or Podman is running on the workstation.
    Run the following command to load the olm-utils-v3 image on the client workstation:

    x86-64 clusters

        cpd-cli manage load-image \
        --source-image=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.amd64

    The command returns the following message when the image is loaded:

    Loaded image: icr.io/cpopen/cpd/olm-utils:latest

ppc64le clusters

    cpd-cli manage load-image \
    --source-image=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.ppc64le

    The command returns the following message when the image is loaded:

    Loaded image: icr.io/cpopen/cpd/olm-utils:latest.ppc64le

s390x clusters

    cpd-cli manage load-image \
    --source-image=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.s390x

    The command returns the following message when the image is loaded:

    Loaded image: icr.io/cpopen/cpd/olm-utils:latest.s390x

Set the OLM_UTILS_IMAGE environment variable to ensure that the cpd-cli uses the version of the image on the client workstation:

x86-64 clusters

    export OLM_UTILS_IMAGE=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.amd64

ppc64le clusters

    export OLM_UTILS_IMAGE=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.ppc64le

s390x clusters

    export OLM_UTILS_IMAGE=icr.io/cpopen/cpd/olm-utils-v3:${VERSION}.s390x

## Downloading CASE packages

1. Download the CASE packages for the components that you plan to install. IBM Cloud Pak Open Container Initiative.
```bash
cpd-cli manage case-download \
--components=${COMPONENTS} \
--release=${VERSION} \
--from_oci=true
```
2. If you plan to install IBM Software Hub Control Center, download the CASE package for the ibm_swhcc component. IBM Cloud Pak Open Container Initiative.
```bash
cpd-cli manage case-download \
--components=ibm_swhcc \
--release=${VERSION} \
--from_oci=true
```

3. Set the following permissions on the work directory.
```bash
chown -R 1001 ./work
chmod -R 775 ./work
```

4. Restart the container.
```bash
cpd-cli manage restart-container
```

## Mirroring images directly to the private container registry

1. Log in to the IBM Entitled Registry registry.
```bash
cpd-cli manage login-entitled-registry \
${IBM_ENTITLEMENT_KEY}
```
2. Log in to the private container registry.

The following command assumes that you are using private container registry that is secured with credentials:

```bash
cpd-cli manage login-private-registry \
${PRIVATE_REGISTRY_LOCATION} \
${PRIVATE_REGISTRY_PUSH_USER} \
${PRIVATE_REGISTRY_PUSH_PASSWORD}
```

**Note:** <br>
If your private registry is not secured omit the following arguments:

* ${PRIVATE_REGISTRY_PUSH_USER}
* ${PRIVATE_REGISTRY_PUSH_PASSWORD}

3. Confirm that you have access to the images that you want to mirror from the IBM Entitled Registry:

    * 3.1 Inspect the IBM Entitled Registry.
    
        * 3.1.1 You already have the CASE packages on the client workstation:
        ```bash
        cpd-cli manage list-images \
        --components=${COMPONENTS} \
        --release=${VERSION} \
        --inspect_source_registry=true
        ```
        * 3.1.2 Check for errors.
        ```bash
        grep "level=fatal" list_images.csv
        ```

4. Mirror the images to the private container registry.
```bash
cpd-cli manage mirror-images \
--components=${COMPONENTS} \
--release=${VERSION} \
--target_registry=${PRIVATE_REGISTRY_LOCATION} \
--arch=${IMAGE_ARCH} \
--case_download=false
```

5. Confirm that the images were mirrored to the private container registry.
    * 5.1 Inspect the contents of the private container registry.
        ```bash
        cpd-cli manage list-images \
        --components=${COMPONENTS} \
        --release=${VERSION} \
        --target_registry=${PRIVATE_REGISTRY_LOCATION} \
        --case_download=false
        ```
    * 5.2 Check the output for errors.
        ```bash
        grep "level=fatal" list_images.csv
        ```

