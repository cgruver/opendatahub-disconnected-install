# WIP - Tested on OKD 4.5 Disconnected

### Prerequisites:

1. You need cluster-admin access to a running OpenShift 4 cluster (This guide was tested on OKD 4.5)
1. An image registry that is accessible from the OpenShift cluster. You can use the OpenShift internal registry.
1. An HTTP server to host the manifest bundle at `http://your.http.srv/opendatahub/odh-manifests.tar.gz`
1. A workstation with internet access and either podman or docker
1. Dynamic storage provisioners for Block and S3 compatible Object storage
    See: https://github.com/cgruver/rook-ceph-disconnected-install for a disconnected install of Ceph that provides the needed storage capabilities

## Set up Open Data Hub

Clone this repository:

```bash
git clone https://github.com/cgruver/opendatahub-disconnected-install.git
cd opendatahub-disconnected-install
```

### Make the Open Data Hub images available to the OpenShift cluster:

The next step is to pull all of the images needed for the install, and make them available to the OpenShift cluster.

These instructions assume that you are using a personal workstation with Docker desktop.  You can also use `podman`, `skopeo`, or `buildah` to perform these steps.

The list of images is in the file `odh-images`, included with this guide.

From your internet connected workstation or bastion host:

1. Pull the images needed for the install:

    Replace the value for LOCAL_REGISTRY with the URL for your registry.

    ```bash
    export LOCAL_REGISTRY=nexus.your.domain.org:5000
    
    for i in $(cat odh-images)
    do 
        IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
        docker pull ${i}
        docker tag ${i} ${IMAGE_TAG}
    done
    ```

1. Log into your local image registry.  Since you are in a disconnected environment, you might have to change networks for this step.

    ```bash
    docker login ${LOCAL_REGISTRY}
    ```

1. Push the newly tagged images.

    ```bash
    for i in $(cat odh-images)
    do 
        IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
        docker push ${IMAGE_TAG}
    done
    ```

### Set up local registry trust

You need to establish trust between your OpenShift cluster and your image registry so that the ImageStreams will work.

If you need to install a registry to hold your mirrored images, you can follow this guide: https://github.com/cgruver/okd4-upi-lab-setup/blob/master/docs/pages/Nexus_Config.md

```bash
openssl s_client -showcerts -connect ${LOCAL_REGISTRY} </dev/null 2>/dev/null|openssl x509 -outform PEM > ca.crt
oc create configmap nexus-registry-config --from-file=${LOCAL_REGISTRY//:/..}=ca.crt -n openshift-config
oc patch image.config.openshift.io cluster --type=merge --patch '{"spec":{"additionalTrustedCA":{"name":"nexus-registry-config"}}}'
```

### Install Open Data Hub:

1. Create a working directory and stage the files:

    ```bash
    mkdir -p ~/odh-workdir
    cp -r ./odh-manifests ~/odh-workdir
    cp ./openshift-resources/ApplyOnce/*.yaml ~/odh-workdir
    cp ./openshift-resources/PerNamespace/*.yaml ~/odh-workdir
    cp kfdef.yaml ~/odh-workdir

    MANIFEST_URL=http://your.nginx.com/opendatahub/odh-manifests.tar.gz

    cd ~/odh-workdir
    for i in $(ls *.yaml)
    do
        sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" ${i}
    done

    sed -i "s|--MANIFEST_URL--|${MANIFEST_URL}|g" kfdef.yaml

    for i in $(find . -name disconnected)
    do
        for j in $(ls ${i})
        do
            sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" ${i}/${j}
        done
    done

    tar -cvf ./odh-manifests.tar ./odh-manifests
    gzip ./odh-manifests.tar
    ```

1. Now copy the `odh-manifests.tar.gz` bundle to your http server:

    ```bash
    scp ./odh-manifests.tar.gz root@your.nginx.com:/usr/local/nginx/html/opendatahub/odh-manifests.tar.gz
    ```
1. Create a namespace for your Open Data Hub instance:

    ```bash
    oc new-project my-opendatahub
    ```

1. __Not Working Yet__ If you want Grafana installed, then deploy the grafana operator: __Not Working Yet__

    ```bash
    oc apply -f grafana-crd.yaml
    oc apply -f grafana-operator-csv.yaml
    ```

1. Finally, deploy the Open Data Hub components:

    ```bash
    oc apply -f odh-role.yaml
    oc apply -f kfdef-crd.yaml
    oc apply -f opendatahub-operator-csv.yaml
    oc apply -f python.yaml
    oc apply -f postgresql.yaml
    oc apply -f object-user.yaml
    oc apply -f prometheus.yaml
    oc apply -f prometheus-servicemonitor.yaml -n my-opendatahub
    ```

### Create an instance of Open Data Hub:

This project includes a sample KfDef file that will deploy and instance of Open Data Hub in the `my-opendatahub` namespace.

```bash
oc apply -f kfdef.yaml -n my-opendatahub
```

Log into JupyterHub

```bash
oc get route jupyterhub -n my-opendatahub -o jsonpath='{.spec.host}'
```

### S3 Buckets (WIP)

```bash
AWS_ACCESS_KEY_ID=$(oc get secrets -n rook-ceph rook-ceph-object-user-s3-object-store-odh-user -o yaml | grep AccessKey | grep -v "f:AccessKey:" | awk '{print $2}' | base64 --decode)
AWS_SECRET_ACCESS_KEY=$(oc get secrets -n rook-ceph rook-ceph-object-user-s3-object-store-odh-user -o yaml | grep SecretKey | grep -v "f:SecretKey:" | awk '{print $2}' | base64 --decode)
oc get secrets -n rook-ceph rook-ceph-object-user-s3-object-store-odh-user -o yaml | grep Endpoint | awk '{print $2}' | base64 --decode
```

# Full list of images needed for a disconnected install:

## Open Data Hub:

    quay.io/opendatahub/opendatahub-operator:v0.8.0
    quay.io/odh-jupyterhub/jupyterhub-img:3.0.7-b7db22b
    quay.io/radanalyticsio/openshift-spark-py36:2.4.5-2
    quay.io/radanalyticsio/spark-operator:1.0.7
    quay.io/odh-jupyterhub/s2i-spark-scipy-notebook:3.6
    quay.io/odh-jupyterhub/s2i-spark-minimal-notebook:py36-spark2.4.5-hadoop2.7.3
    quay.io/odh-jupyterhub/nbviewer:latest
    quay.io/odh-jupyterhub/s2i-spark-container:spark2.4.5-1
    quay.io/thoth-station/s2i-lab-elyra:v0.0.2
    quay.io/thoth-station/s2i-minimal-notebook:v0.0.4
    quay.io/thoth-station/s2i-scipy-notebook:v0.0.1
    quay.io/thoth-station/s2i-tensorflow-notebook:v0.0.1
    quay.io/thoth-station/s2i-thoth-ubi8-py36:v0.15.0
    docker.io/centos/postgresql-96-centos7:latest
    docker.io/centos/postgresql-10-centos7:latest
    docker.io/centos/postgresql-12-centos7:latest
    docker.io/centos/python-27-centos7:latest
    docker.io/centos/python-36-centos7:latest
    registry.access.redhat.com/ubi7/s2i-base:latest

## Grafana:

    docker.io/grafana/grafana:6.5.1
    quay.io/openshift/origin-grafana:4.5
    quay.io/integreatly/grafana-operator:v3.5.0
    quay.io/integreatly/grafana_plugins_init:0.0.2

## Knative:

    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-operator
    registry.svc.ci.openshift.org/openshift/openshift-serverless-nightly:knative-operator
    registry.svc.ci.openshift.org/openshift/openshift-serverless-nightly:knative-openshift-ingress
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-channel-dispatcher
    registry.svc.ci.openshift.org/openshift/knative-v0.16.1:kn-cli-artifacts
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-serving-autoscaler
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-channel-dispatcher
    docker.io/maistra/proxyv2-ubi8:1.1.0
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-ping
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-apiserver-receive-adapter
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-broker-filter
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-channel-controller
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-mtbroker-filter
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-webhook
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-serving-queue
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-serving-activator
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-controller
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-serving-controller
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-mtchannel-broker
    registry.svc.ci.openshift.org/openshift/knative-v0.15.2:knative-eventing-channel-broker
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-mtbroker-ingress
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-storage-version-migration
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:kourier
    registry.svc.ci.openshift.org/openshift/knative-v0.15.2:knative-eventing-broker-filter
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-serving-webhook
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-mtping
    registry.svc.ci.openshift.org/openshift/knative-v0.15.2:knative-eventing-broker-ingress
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-broker-ingress
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-serving-storage-version-migration
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-serving-autoscaler-hpa
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-broker-cleanup
    registry.svc.ci.openshift.org/openshift/knative-v0.16.0:knative-eventing-sugar-controller

## Rook - Ceph:

    docker.io/rook/ceph:v1.4.0
    docker.io/ceph/ceph:v15.2.4
    quay.io/cephcsi/cephcsi:v3.0.0
    quay.io/k8scsi/csi-node-driver-registrar:v1.2.0
    quay.io/k8scsi/csi-resizer:v0.4.0
    quay.io/k8scsi/csi-provisioner:v1.6.0
    quay.io/k8scsi/csi-snapshotter:v2.1.1
    quay.io/k8scsi/csi-attacher:v2.1.0

## OpenShift Pipelines:

    quay.io/openshift-pipeline/tektoncd-pipeline-controller:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-kubeconfigwriter:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-creds-init:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-git-init:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-entrypoint:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-nop:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-imagedigestexporter:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-pullrequest-init:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-gcs-fetcher:v0.16.3
    quay.io/openshift-pipeline/tektoncd-pipeline-webhook:v0.16.3
    quay.io/openshift-pipeline/tektoncd-triggers-webhook:v0.8.1
    quay.io/openshift-pipeline/tektoncd-triggers-eventlistenersink:v0.8.1
    quay.io/openshift-pipeline/tektoncd-triggers-controller:v0.8.1
    quay.io/buildah/stable:v1.15.1
    quay.io/skopeo/stable:v1.1.1
    registry.access.redhat.com/ubi8/ubi-minimal:latest
    gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init:v0.14.2
    gcr.io/knative-releases/knative.dev/client/cmd/kn:v0.15.2
    # This one requires a Red Hat subscription.  It can be built from: https://github.com/openshift/source-to-image
    registry.redhat.io/ocp-tools-43-tech-preview/source-to-image-rhel8

## Workshop

    quay.io/willbenton/simple-model-s2i:cached-pipeline-s2i
    quay.io/willbenton/jh-ml-workflows-notebook:summit2020
    quay.io/willbenton/jh-ml-workflows-notebook:devconf-us-2019
