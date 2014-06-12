======================
OpenStack Nova 虚拟机启动流程分析
======================

- nova.api.cloud. *CloudController* ::
  **run_instance**


- nova.compute.api ::
  **create-->_create_instance**


- nova.conductor.api. *ComputerTaskAPI* ::
  **build_instances**


- nova.conductor.api. *LocalComputeTaskAPI* ::
  **build_instances**


- nova.conductor.manager. *ComputeTaskManager* ::
  **build_instances**


- nova.scheduler.rpcapi. *SchedulerManager* ::
  **run_instances**


- nova.scheduler.driver. *Scheduler* ::
  **schedule_run_instance**


- nova.compute.rpcapi. *ComputeAPI* ::
  **run_instance**


- nova.compute.manager. *ComputerManager* ::
  **run_instance->do_run_instance->_run_instance->_build_instance->_spawn**


- nova.virt.driver. *ComputeDriver* ::
  **spawn**


- nova.virt.libvirt.driver. *LibvirtDriver* ::
  **spawn-> _create_image**


- nova.virt.libvirt.imagebackend.image ::
  **create->create_image**


- nova.virt.libvirt.imagebackend.qcow2 ::
  **create_image->prepare_template**


- 回调函数 nova.virt.libvirt.utils ::
  **fetch_image**


- nova.virt.images ::
  **fetch_to_raw->fetch**



