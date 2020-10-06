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

quay.io/openshift/origin-grafana:4.5
quay.io/integreatly/grafana-operator:v3.5.0

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

quay.io/openshift-pipeline/openshift-pipelines-operator-controller:v0.15.2-1

