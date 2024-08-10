Meta-ROS Documentation
====================================

.. toctree::
   :hidden:

   docs/getting-started.md
   docs/perception/index.md
   docs/decision/index.md
   docs/decomposition/index.md
   docs/execution/index.md

Meta-ROS is a complete rewrite of the original `Meta-Embedded <https://github.com/Meta-Team/Meta-Embedded>`_ project based on ROS 2 (Humble).
It is designed to be run on embedded Linux devices with RT kernels.

Why rewrite ``Meta-Embedded``?
-------------------------------------------
``Meta-Embedded`` is based on STM32 and ChibiOS, which is a great combination for small embedded devices.
However, as vision and navigation systems are added. The latency between the microcontroller and the main computer
becomes a big issue. We believe it is hard for us to design a time synchronization mechanism between the two systems, so we 
decided to move the whole system into a single computer.

Why ROS 2?
-------------------------------------------
ROS 2 provides a great sender-receiver model and a lot of useful tools like ``rviz`` and ``tf2`` that makes robotics
development much easier. We prefer ROS 2 over ROS mainly because our vision system (which is earlier than this project) is 
written in ROS 2.
