# WIP - Not Ready For Use

```bash
git clone https://github.com/cgruver/opendatahub-disconnected-install.git
cd opendatahub-disconnected-install
```

### Make the Open Data Hub images available to the OpenShift cluster:

The next step is to pull all of the images needed for the install, and make them available to the OpenShift cluster.

These instructions assume that you are using a personal workstation with Docker desktop.  You can also use `podman`, `skopeo`, or `buildah` to perform these steps.

The list of images is in the file `odh-images`, included with this guide:

```bash
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
```

From your internet connected workstation or bastion host:

1. Pull the images needed for the install:

    ```bash
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


```bash
mkdir -p ~/odh-workdir
cp -r ./odh-manifests ~/odh-workdir
cp ./openshift-resources/*.yaml ~/odh-workdir
cp kfdef.yaml ~/odh-workdir
LOCAL_REGISTRY=nexus.your.domain.com:5000
MANIFEST_URL=http://your.nginx.com/opendatahub/odh-manifests.tar.gz

cd ~/odh-workdir
for i in $(find . -name disconnected)
do
    for j in $(ls ${i})
    do
        sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" ${i}/${j}
    done
done

tar -cvf ./odh-manifests.tar ./odh-manifests
gzip ./odh-manifests.tar
scp ./odh-manifests.tar.gz root@your.nginx.com:/usr/local/nginx/html/opendatahub/odh-manifests.tar.gz

sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" opendatahub-operator.v0.8.0.clusterserviceversion.yaml
sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" python.yaml
sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" postgresql.yaml
sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" kfdef.yaml
sed -i "s|--MANIFEST_URL--|${MANIFEST_URL}|g" kfdef.yaml

oc apply -f role.yaml
oc apply -f kfdef.apps.kubeflow.org.crd.yaml
oc apply -f opendatahub-operator.v0.8.0.clusterserviceversion.yaml
oc apply -f python.yaml
oc apply -f postgresql.yaml
oc apply -f kfdef.yaml
```

