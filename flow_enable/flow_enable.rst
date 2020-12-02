.. _flow_enable:

-------------
Flow: Enable
-------------

*The estimated time to complete this lab is 10 minutes.*

Overview
++++++++

In this exercise you will enable Nutanix Flow.

Enabling Flow
++++++++++++++++++++++++++

**Flow** เป็น service สามารถ enable ได้จากบน Prism Central โดยไม่จำเป็นต้องมี appliance หรือ console อื่นๆเพิ่มเติม ก่อนเริ่มใช้งาน โปรดตรวจสอบให้แน่ใจก่อนว่า Flow ได้ถูก enable ขึ้นมาเรียบร้อยแล้ว

.. note::

  ถ้า **Flow** ขึ้นเป็นเครื่องหมายถูกสีเขียว หมายความ Flow ได้ถูก enable ขึ้นมาบน Prism Central นั้นๆแล้ว ท่านสามารถดำเนินการต่อที่ส่วนของ :ref:`flow_secure_app` ได้เลย.

#. 1.	บนหน้า **Prism Central** คลิกที่เครื่องหมาย **?** บริเวณด้านบน และกดเลือกที่ **Flow**.

   .. figure:: images/10.png

   สำหรับการ enable Flow แต่ละ Prism Central VM จะต้องการ memory เพิ่มขึ้น 1 GB โดยส่วนนี้ระบบจะทำการปรับ memory ขึ้นแบบอัตโนมัติ.

   หาก **Flow** ขึ้นเป็นเครื่องหมายถูกสีเขียว หมายความ Flow ได้ถูก enable ขึ้นมาบน Prism Central นั้นๆแล้ว ท่านสามารถดำเนินการต่อที่ส่วนของ :ref:`flow_secure_app` ได้เลย.

#. เลือก **Enable Microsegmentation** และคลิก **Enable**.

   .. figure:: images/11.png


Takeaways
+++++++++

- Microsegmentation, part of Flow, is a decentralized security enforcement framework centrally managed from Prism Central.
- Once Flow is enabled in the cluster, VMs can be easily protected through Security Policies created in the Prism Central UI. Security Policies use simple text based categories to define groups of VMs. These function as labels that can easily be applied to VMs without any additional network setup.
