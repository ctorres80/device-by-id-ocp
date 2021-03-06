# device-by-id-ocp
This is a super simple and all fashion (sorry for the shell module, awk, grep, sed and more) playbook for discovery local drives and prepare the local-storage-block.yaml

The playbook will dicover the /dev/disk/by-id/XXX on each OCP worker node that has the following label:

    oc get node -l cluster.ocs.openshift.io/openshift-storage=

The playbook will return local-storage-block.yaml, please review and modify or adapt to your requirements.

    [ctorres-redhat.com@clientvm 0 ~/deploy/device-by-id-ocp master ⭑|…1]$ cat local-storage-block.yaml
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
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1AC0E1AFFAB8A03A3 # ip-10-0-135-45.eu-central-1.compute.internal 	 nvme0n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6AC0E1AFFAB8A03A3 # ip-10-0-135-45.eu-central-1.compute.internal 	 nvme1n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1F1C27E2E890F5210 # ip-10-0-135-45.eu-central-1.compute.internal 	 nvme2n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6F1C27E2E890F5210 # ip-10-0-135-45.eu-central-1.compute.internal 	 nvme3n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS15C769136DCE431E1 # ip-10-0-155-116.eu-central-1.compute.internal 	 nvme0n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS65C769136DCE431E1 # ip-10-0-155-116.eu-central-1.compute.internal 	 nvme1n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1A3C0A4A49BCF144B # ip-10-0-155-116.eu-central-1.compute.internal 	 nvme2n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6A3C0A4A49BCF144B # ip-10-0-155-116.eu-central-1.compute.internal 	 nvme3n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS16F604C3B81EE5523 # ip-10-0-175-94.eu-central-1.compute.internal 	 nvme0n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS66F604C3B81EE5523 # ip-10-0-175-94.eu-central-1.compute.internal 	 nvme1n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS18C4F9CA036362729 # ip-10-0-175-94.eu-central-1.compute.internal 	 nvme2n1 	  1.7T
            - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS68C4F9CA036362729 # ip-10-0-175-94.eu-central-1.compute.internal 	 nvme3n1 	  1.7T

    
With the local-storage-block.yaml you can create the CR for consuming local drives throug Local Storage Operator LSO with the following command:

    oc create -f local-storage-block.yaml
    
It will create a pv for each discovered /dev/disk/by-id/XXX and those local pvs will be consumed by OpenShift Container Storage.

Followig an example:

    [ctorres-redhat.com@clientvm 130 ~/deploy/device-by-id-ocp master ⭑|…3]$ ansible --version
    ansible 2.9.6
      config file = /home/ctorres-redhat.com/deploy/device-by-id-ocp/ansible.cfg
      configured module search path = [u'/home/ctorres-redhat.com/deploy/device-by-id-ocp/library']
      ansible python module location = /usr/lib/python2.7/site-packages/ansible
      executable location = /usr/bin/ansible
      python version = 2.7.5 (default, May 31 2018, 09:41:32) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]

    [[ctorres-redhat.com@clientvm 0 ~/deploy/device-by-id-ocp master ⭑|✔]$ ansible-playbook devices_by_id.yml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

    PLAY [Playbook for collecting disks information] ************************************************************************************************************************************************************************************

    TASK [Collect OCS workers] **********************************************************************************************************************************************************************************************************
    Wednesday 15 April 2020  00:41:06 +0000 (0:00:00.040)       0:00:00.040 *******
    changed: [localhost]

    TASK [Collect device by-id and size] ************************************************************************************************************************************************************************************************
    Wednesday 15 April 2020  00:41:07 +0000 (0:00:00.744)       0:00:00.784 *******
    changed: [localhost]

    TASK [Collect device by-id] *********************************************************************************************************************************************************************************************************
    Wednesday 15 April 2020  00:41:07 +0000 (0:00:00.225)       0:00:01.009 *******
    changed: [localhost] => (item=ip-10-0-135-45.eu-central-1.compute.internal)
    changed: [localhost] => (item=ip-10-0-155-116.eu-central-1.compute.internal)
    changed: [localhost] => (item=ip-10-0-175-94.eu-central-1.compute.internal)

    TASK [Print to screen config file ./local-storage-block.yaml] ***********************************************************************************************************************************************************************
    Wednesday 15 April 2020  00:41:26 +0000 (0:00:18.796)       0:00:19.806 *******
    changed: [localhost]

    TASK [debug] ************************************************************************************************************************************************************************************************************************
    Wednesday 15 April 2020  00:41:26 +0000 (0:00:00.195)       0:00:20.001 *******
    ok: [localhost] =>
      msg:
      - 'apiVersion: local.storage.openshift.io/v1'
      - 'kind: LocalVolume'
      - 'metadata:'
      - '  name: local-block'
      - '  namespace: local-storage'
      - 'spec:'
      - '  nodeSelector:'
      - '    nodeSelectorTerms:'
      - '    - matchExpressions:'
      - '        - key: cluster.ocs.openshift.io/openshift-storage'
      - '          operator: In'
      - '          values:'
      - '          - ""'
      - '  storageClassDevices:'
      - '    - storageClassName: localblock'
      - '      volumeMode: Block'
      - '      devicePaths:'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1AC0E1AFFAB8A03A3 # nvme0n1    1769 GB    ip-10-0-135-45.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6AC0E1AFFAB8A03A3 # nvme1n1    1769 GB    ip-10-0-135-45.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1F1C27E2E890F5210 # nvme2n1    1769 GB    ip-10-0-135-45.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6F1C27E2E890F5210 # nvme3n1    1769 GB    ip-10-0-135-45.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS15C769136DCE431E1 # nvme0n1    1769 GB    ip-10-0-155-116.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS65C769136DCE431E1 # nvme1n1    1769 GB    ip-10-0-155-116.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS1A3C0A4A49BCF144B # nvme2n1    1769 GB    ip-10-0-155-116.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS6A3C0A4A49BCF144B # nvme3n1    1769 GB    ip-10-0-155-116.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS16F604C3B81EE5523 # nvme0n1    1769 GB    ip-10-0-175-94.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS66F604C3B81EE5523 # nvme1n1    1769 GB    ip-10-0-175-94.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS18C4F9CA036362729 # nvme2n1    1769 GB    ip-10-0-175-94.eu-central-1.compute.internal'
      - '        - /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage_AWS68C4F9CA036362729 # nvme3n1    1769 GB    ip-10-0-175-94.eu-central-1.compute.internal'

    PLAY RECAP **************************************************************************************************************************************************************************************************************************
    localhost                  : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

    Wednesday 15 April 2020  00:41:26 +0000 (0:00:00.024)       0:00:20.026 *******
    ===============================================================================
    Collect device by-id -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 18.80s
    Collect OCS workers ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 0.74s
    Collect device by-id and size ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 0.23s
    Print to screen config file ./local-storage-block.yaml ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- 0.20s
    debug ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ 0.02s
