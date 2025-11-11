# Mirroring images

# Install IBM Software Hub CLI (cpd-cli)

1. Download Version 14.2.2:

https://www.ibm.com/links?url=https%3A%2F%2Fgithub.com%2FIBM%2Fcpd-cli%2Freleases

For a Linux workstation, run this command:

```bash
wget https://github.com/IBM/cpd-cli/releases/download/v14.2.2/cpd-cli-linux-EE-14.2.2.tgz
```

2. Extract the content of the package.

3. As a best practice, make the `cpd-cli` executable from any directory:

* 3.1 Add the following line to your `~/.basrhc` file
```bash
export PATH=<fully-qualified-path-to-the-cpd-cli>:$PATH
```

4. Validate the installation.

```bash
cpd-cli version
```

## Setting up installation enviroment variables

1. You might use the existing `cpd-vars.sh` file and modifying for this installation, so copy and save with a different name. Otherwise, you can copy the following and modify accordingly.

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

2. Update VERSION value in the `cpd-vars.sh`, if you copy the file form the existing installation to `5.2.2`.

3. For COMPONENTS value in the `cpd-vars.sh`, please remove the existing componets add the following components.

```bash
db2oltp,datagate
```

<br>**NOTE:**<br>
Don't remove these components.

```bash
ibm-licensing,scheduler,cpfs,cpd_platform
```

4. Source the `cpd-vars.sh`
```bash
source cpd-vars.sh
```

<br>**NOTE:**<br>
Use the new `cpd-vars.sh` and the new name accordingly.

5. Validation of the variables. 

* 5.1 Verify VERSION.
```bash
echo $VERSION
```
Output expected
```bash
[root@localhost ~]# echo $VERSION
5.2.2
```

* 5.2 Verify COMPONENTS.

```bash
echo $COMPONENTS
```
Output expected
```bash
[root@localhost ~]# echo $COMPONENTS
ibm-licensing,scheduler,cpfs,cpd_platform,db2oltp,datagate
```

## Downloading CASE packages and obtaining the olm-utils-v3 image

1. Download the CASE packages for the components that you plan to install.
```bash
cpd-cli manage case-download \
--components=${COMPONENTS} \
--release=${VERSION}
```

2. Restart the container.
```bash
cpd-cli manage restart-container
```

Wait for the cpd-cli to return the following messages:
```bash
[SUCCESS] ... Successfully pulled the container image icr.io/cpopen/cpd/olm-utils-v3:latest
[SUCCESS] ... Successfully started the container olm-utils-play
[SUCCESS] ... Container olm-utils-play has been re-created
```

3. Verify olm-utils-v3 container was created properly.

```bash
[root@alocalhost ~]# podman ps
CONTAINER ID  IMAGE                                  COMMAND     CREATED         STATUS         PORTS       NAMES
65675f908df0  icr.io/cpopen/cpd/olm-utils-v3:latest              17 seconds ago  Up 17 seconds              olm-utils-play-v3
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

