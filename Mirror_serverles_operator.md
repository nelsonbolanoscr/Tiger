# 🪞 Mirroring the Serverless Operator to a Private Registry

This guide explains how to mirror the **Serverless Operator** to a private container registry for use in disconnected or air-gapped OpenShift environments.

---

## 🔧 Prerequisites

- A system with internet access to pull images.
- A private container registry (e.g., Quay, Harbor).
- OpenShift CLI (`oc`) and the `oc-mirror` plugin installed.
- A valid Red Hat pull secret from [cloud.redhat.com](https://cloud.redhat.com).

---

## 📄 Step 1: Create an ImageSet Configuration File

Create a file named `imageset-config.yaml` with the following content:

```yaml
apiVersion: mirror.openshift.io/v1alpha2
kind: ImageSetConfiguration
mirror:
  platform:
    channels:
      - name: stable-4.14
        type: ocp
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.14
      packages:
        - name: serverless-operator
          channels:
            - name: stable
  additionalImages: []
  helm: {}
storageConfig:
  local:
    path: ./mirror-data
```

## 🚀 Step 2: Run `oc-mirror` to Mirror the Content

Use the following command to mirror the content to your private registry:

```bash
oc mirror --config imageset-config.yaml docker://<your-private-registry>
```

 ## 🛠️ Step 3: Configure OpenShift to Use the Mirrored Registry

After mirroring is complete, follow these steps to configure your OpenShift cluster:

1. **Apply the ImageContentSourcePolicy (ICSP):**

   The `oc mirror` command generates an `ImageContentSourcePolicy` YAML file. Apply it to your cluster:

   ```bash
   oc apply -f ./mirror-data/oc-mirror-workspace/results-*/imageContentSourcePolicy.yaml
   ```
