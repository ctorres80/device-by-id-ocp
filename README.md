# device-by-id-ocp
This is a super simple and all fashion playbook for discovery local drives and prepare the local-storage-block.yaml

The playbook will dicover the /dev/disk/by-id/XXX on each OCP worker node that has the following label:

    oc get node -l cluster.ocs.openshift.io/openshift-storage=

The playbook will return local-storage-block.yaml, please review and modify or adapt to your requirements, you can create the CR for consuming local drives throug Local Storage Operator LSO with the following command:

    oc create -f local-storage-block.yaml
    
It will create a pv for each discovered /dev/disk/by-id/XXX and those local pvs will be consumed by OpenShift Container Storage.

Followig an example:

    [ctorres-redhat.com@clientvm 130 ~/deploy]$ ansible --version
    ansible 2.8.8
      config file = /home/ctorres-redhat.com/deploy/ansible.cfg
      configured module search path = [u'/home/ctorres-redhat.com/deploy/library']
      ansible python module location = /usr/lib/python2.7/site-packages/ansible
      executable location = /usr/bin/ansible
      python version = 2.7.5 (default, May 31 2018, 09:41:32) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]

    [ctorres-redhat.com@clientvm 130 ~/deploy]$ ansible-playbook devices_by_id.yml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'


    PLAY [Playbook for collecting disks information] ******************************************************************************************************************************************************

    TASK [Collect OCS workers] ****************************************************************************************************************************************************************************
    Tuesday 14 April 2020  16:29:32 +0000 (0:00:00.072)       0:00:00.072 *********
    changed: [localhost]

    TASK [Collect device by-id] ***************************************************************************************************************************************************************************
    Tuesday 14 April 2020  16:29:32 +0000 (0:00:00.683)       0:00:00.755 *********
    changed: [localhost] => (item=ip-10-0-135-45.eu-central-1.compute.internal)
    changed: [localhost] => (item=ip-10-0-155-116.eu-central-1.compute.internal)
    changed: [localhost] => (item=ip-10-0-175-94.eu-central-1.compute.internal)

    TASK [Collect device by-id and size] ******************************************************************************************************************************************************************
    Tuesday 14 April 2020  16:29:42 +0000 (0:00:09.325)       0:00:10.081 *********
    changed: [localhost]

    TASK [Collect device by-id and size] ******************************************************************************************************************************************************************
    Tuesday 14 April 2020  16:29:42 +0000 (0:00:00.131)       0:00:10.213 *********
    changed: [localhost]

    TASK [debug] ******************************************************************************************************************************************************************************************
    Tuesday 14 April 2020  16:30:20 +0000 (0:00:38.356)       0:00:48.570 *********
    ok: [localhost] =>
      msg: |-
        Following the devices available:

        apiVersion: local.storage.openshift.io/v1
        kind: LocalVolume
        metadata:
          name: local-block
          namespace: local-storage
        spec:
          nodeSelector:
            nodeSelectorTerms:
            - matchExpressions:
                - key: cluster.ocs.openshift.io/openshift-storage
                  operator: In
                  values:
                  - ""
          storageClassDevices:
            - storageClassName: localblock
              volumeMode: Block
              devicePaths:
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1AC0E1AFFAB8A03A3 # ip-10-0-135-45.eu-central-1.compute.internal      nvme0n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6AC0E1AFFAB8A03A3 # ip-10-0-135-45.eu-central-1.compute.internal      nvme1n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1F1C27E2E890F5210 # ip-10-0-135-45.eu-central-1.compute.internal      nvme2n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6F1C27E2E890F5210 # ip-10-0-135-45.eu-central-1.compute.internal      nvme3n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS15C769136DCE431E1 # ip-10-0-155-116.eu-central-1.compute.internal     nvme0n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS65C769136DCE431E1 # ip-10-0-155-116.eu-central-1.compute.internal     nvme1n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1A3C0A4A49BCF144B # ip-10-0-155-116.eu-central-1.compute.internal     nvme2n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6A3C0A4A49BCF144B # ip-10-0-155-116.eu-central-1.compute.internal     nvme3n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS16F604C3B81EE5523 # ip-10-0-175-94.eu-central-1.compute.internal      nvme0n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS66F604C3B81EE5523 # ip-10-0-175-94.eu-central-1.compute.internal      nvme1n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS18C4F9CA036362729 # ip-10-0-175-94.eu-central-1.compute.internal      nvme2n1          1.7T
                - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS68C4F9CA036362729 # ip-10-0-175-94.eu-central-1.compute.internal      nvme3n1          1.7T

    TASK [Clean tmp file] *********************************************************************************************************************************************************************************
    Tuesday 14 April 2020  16:30:20 +0000 (0:00:00.045)       0:00:48.615 *********
    changed: [localhost]

    PLAY RECAP ********************************************************************************************************************************************************************************************
    localhost                  : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

    Tuesday 14 April 2020  16:30:21 +0000 (0:00:00.314)       0:00:48.929 *********
    ===============================================================================
    Collect device by-id and size ----------------------------------------------------------------------------------------------------------------------------------------------------------------- 38.36s
    Collect device by-id --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 9.33s
    Collect OCS workers ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 0.68s
    Clean tmp file --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 0.31s
    Collect device by-id and size ------------------------------------------------------------------------------------------------------------------------------------------------------------------ 0.13s
    debug ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 0.05s
