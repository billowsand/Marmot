================
OpenStack快照分析
================

快照分析
========

#) nova.api.ec2.cloud.CloudController **create_snapshot**
#) nova.volumn.cinder.API **create_snapshot**
#) nova.compute.api.API **snapshot**

  #) nova.compute.api.API **_create_image**
#) nova.compute.rpcapi.ComputeAPI **snapshot_instance**
#) nova.compute.manager.ComputeManager **snapshot_instance**
#) nova.compute.manager.ComputeManager **_snapshot_instance**
#) nova.virt.driver **snapshot**
#) nova.virt.libvirt.driver.LibvirtDriver **snapshot**
#) nova.virt.libvirt.imagebackend **snapshot**


Live Snapshot过程分析
====================
#) 获取

Cold Snapshot
=============
#) nova.virt.lobvirt.imagebackend.Qcow2 **snapshot_extract**
#) nova.virt.libvrit.utils **extract_snapshot**
