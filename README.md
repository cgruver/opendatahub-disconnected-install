
```bash
mkdir -p ~/odh-workdir
cp ./openshift-resources/*.yaml ~/odh-workdir
INTERNAL_REGISTRY=nexus.your.domain.com:5000
sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" ~/odh-workdir/opendatahub-operator.v0.8.0.clusterserviceversion.yaml

oc apply -f kfdef.apps.kubeflow.org.crd.yaml
oc apply -f opendatahub-operator.v0.8.0.clusterserviceversion.yaml

```
