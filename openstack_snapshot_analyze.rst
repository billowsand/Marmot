================
OpenStack快照分析
================

快照分析
========

#) nova.api.ec2.cloud.CloudController **create_snapshot**
#) nova.volumn.cinder.API **create_snapshot**
#) nova.compute.api.API **snapshot**

  1. nova.compute.api.API **_create_image**

2. nova.compute.rpcapi.ComputeAPI **snapshot_instance**
3. nova.compute.manager.ComputeManager **snapshot_instance**
3. nova.compute.manager.ComputeManager **_snapshot_instance**
4. nova.virt.driver **snapshot**
4. nova.virt.libvirt.driver.LibvirtDriver **snapshot**
4. nova.virt.libvirt.imagebackend **snapshot**


Live Snapshot过程分析
====================
1. 获取

Cold Snapshot
=============
5. nova.virt.lobvirt.imagebackend.Qcow2 **snapshot_extract**
6. nova.virt.libvrit.utils **extract_snapshot**
