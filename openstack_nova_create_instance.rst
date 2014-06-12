======================
OpenStack Nova 虚拟机启动流程分析
======================

#) nova.api.cloud. *CloudController* ::

  **run_instance**

#) nova.compute.api ::

  **create-->_create_instance**

#) nova.conductor.api. *ComputerTaskAPI* ::

  **build_instances**

#) nova.conductor.api. *LocalComputeTaskAPI* ::

  **build_instances**

#) nova.conductor.manager. *ComputeTaskManager* ::

  **build_instances**

#) nova.scheduler.rpcapi. *SchedulerManager* ::

  **run_instances**

#) nova.scheduler.driver. *Scheduler* ::
 
  **schedule_run_instance**

#) nova.compute.rpcapi. *ComputeAPI* ::

  **run_instance**

#) nova.compute.manager. *ComputerManager* ::

  **run_instance->do_run_instance->_run_instance->_build_instance->_spawn

#)
