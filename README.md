# device-by-id-ocp
This is a super simple and all fashion playbook for discovery local drives and prepare the local-storage-block.yaml that you can use for create the CR with the following command:
    oc create -f local-storage-block.yaml

For each node that will be included in:
    oc get node -l cluster.ocs.openshift.io/openshift-storage=

It will create a pv for discovered device-byid and those local pvs will be consumed by OpenShift Container Storage
