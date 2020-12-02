.. title:: Nutanix Flow (ภาษาไทย)

.. toctree::
  :maxdepth: 2
  :caption:  Flow Labs
  :name: _files_labs
  :hidden:

  flow_enable/flow_enable
  flow_secure_app/flow_secure_app
  flow_isolate_environments/flow_isolate_environments
  flow_quarantine_vm/flow_quarantine_vm
  
.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  tools_vms/windows_tools_vm
  tools_vms/linux_tools_vm
  taskman/taskman

.. _getting_started:

---------------------------------------------
ยินดีต้อนรับสู่
---------------------------------------------
---------------------------------------------
Nutanix Flow Hands-on Workshop!
---------------------------------------------

Overview
++++++++

**Flow** คือ software-defined networking ที่ integrate มาพร้อมกับ Nutanix AHV และ Prism Central ที่มีความสามารถในการทำ visualization, automation รวมไปถึง security ให้กับ VM ต่างๆที่ทำงานอยู่บน AHV ได้ อีกทั้ง Flow ยังมาพร้อมกับความสามารถในการทำ Microsegmentation ที่ช่วยลดความซับซ้อนของการบริหารจัดการ Policy ได้อีกด้วย
ด้วยความสามารถของ Flow ท่านจะสามารถสร้าง distributed firewall ที่เป็นเสมือนเครื่องมือช่วยบริหารจัดการความปลอดภัยบนtraffic ของ VM และยังสามารถช่วยทำในส่วนของการ deploy automated application ให้มีความปลอดภัยตามที่ต้องการได้โดยอาศัยการทำงานร่วมกับ Clam อีกด้วย
|
สำหรับ lab นี้ ท่านจะได้เรียนรู้การนำ Flow ไปใช้สร้าง microsegmentation policy สำหรับการทำ multi-tier web application, ทำการแยก (isolate) group ของ VM ต่างๆออกจากกัน ตลอดจนทำการควบคุม (quarantine) VM ที่เกิดปัญหาได้


What's New
++++++++++

ปัจจุบัน Nutanix AOS version 5.18.X แต่เนื่องจาก Labs ชุดนี้ถูกจัดทำขึ้นใน Nutanix AOS Version 5.11.X ดังนั้นอาจจะมีภาพและเสียงที่ไม่ตรงกับ Version ปัจจุบันที่ใช้ใน Labs จริง จึงต้องขออภัยไว้ ณ.ที่นี้ด้วย

Agenda
++++++

- Nutanix Flow Labs
    - Flow: Deploy
    - Flow: Secure an Application
    - Flow: Isolate Environments
    - Flow: Quarantine VM

- Optional Labs

Introductions
+++++++++++++

- Name
- Familiarity with Nutanix

Initial Setup
+++++++++++++

Nutanix Workshops ใช้งานอยู่บน Nutanix Hosted POC environment. เราได้เตรียมสิ่งที่จำเป็นต้องใช้ใน Labs ไว้ให้แล้วเช่น images, networks, และ VMs.

- Take note of the *Passwords* being used.
- VPN to Nutanix Labs ด้วยข้อมูลประกอบด้านล่าง
- ผู้เรียนจะถูกแบ่งออกเป็น 3 กลุ่ม จำนวนเท่าๆ กัน

.. raw:: html

  <strong><font color="red">- **ห้าม ทำการใดๆ กับ VMs, Images, Network ใดๆ ที่ไม่ใช่สิ่งที่ผู้เรียนคนนั้นๆ สร้างใน Labs นี้</font></strong>

|

.. note::
  ภายใน Labs นี้ จะมีการยกตัวอย่างด้วย *XYZ* หรือ *Initial* อยู่บ่อยครั้ง ดังนั้นให้ผู้เรียนสังเกตุ และเปลี่ยนตัวอย่างดังกล่าวให้เป็นชื่อของผู้เรียนเอง หรือเป็น User #No. ที่ได้รับมอบหมาย

|

VPN Access Instructions
+++++++++++++++++++++++

Pulse Secure VPN Client
.......................

1. หากผู้เรียนที่ใช้งาน Pulse Secure VPN อยู่แล้ว ให้ข้ามไปที่ขั้นตอนที่ 5
2. ไป download client เพื่อติดตั้งได้ที่ https://xlv-blr.xlabs.nutanix.com/ ด้วย Username & Password ด้านล่าง\
  - Direct Download\
     - :download:`Windows 32 bit <download/PulseSecure.Windows32bit.msi>`
     - :download:`Windows 64 bit <download/PulseSecure.Windows64bit.msi>`
     - :download:`Mac OS <download/PulseSecureMAC.dmg>`

3. Download and install client
4. Logout of the Web UI
5. เปิด client แล้ว ADD connection ตามข้อมูลต่อไปนี้:\
  - :Type: Policy Secure (UAC) or Connection Server(VPN)\
  - :Name: X-Labs - BLR
  - :Server URL: xlv-blr.xlabs.nutanix.com\

6. Once setup, login with the supplied credentials

Networking
..........

.. list-table::
  :widths: 25 25 25 25
  :header-rows: 1

  * - 
    - กลุ่มที่ 1
    - กลุ่มที่ 2
    - กลุ่มที่ 3
  * - UserName
    - BLR-POC224-User01, ..., BLR-POC224-User20
    - BLR-POC226-User01, ..., BLR-POC226-User20
    - BLR-POC232-User01, ..., BLR-POC232-User20
  * - Password
    - nx2Tech716!
    - nx2Tech831!
    - nx2Tech861!

Environment Details
+++++++++++++++++++

Nutanix Workshops ใช้งานอยู่บน Nutanix Hosted POC environment. เราได้เตรียมสิ่งที่จำเป็นต้องใช้ใน Labs ไว้ให้แล้วเช่น images, networks, และ VMs.

.. list-table::
  :widths: 25 25 25 25
  :header-rows: 1

  * - 
    - กลุ่มที่ 1
    - กลุ่มที่ 2
    - กลุ่มที่ 3
  * - Cluster Name
    - BLR-POC224
    - BLR-POC226
    - BLR-POC232
  * - Cluster IP
    - http://10.136.224.37
    - http://10.136.226.37
    - http://10.136.232.37
  * - PC IP
    - http://10.136.224.39
    - http://10.136.226.39
    - http://10.136.232.39
  * - PE/PC Username
    - admin
    - admin
    - admin
  * - PE/PC Password
    - nx2Tech716!
    - nx2Tech831!
    - nx2Tech861!
  * - CVM Username
    - nutanix
    - nutanix
    - nutanix
  * - CVM Password
    - nx2Tech716!
    - nx2Tech831!
    - nx2Tech861!

Each cluster has a dedicated domain controller VM, **DC**, responsible for providing AD services for the **NTNXLAB.local** domain. The domain is populated with the following Users and Groups:

.. list-table::
  :widths: 25 35 40
  :header-rows: 1

  * - Group
    - Username(s)
    - Password
  * - Administrators
    - Administrator
    - nutanix/4u
  * - SSP Admins
    - adminuser01-adminuser25
    - nutanix/4u
  * - SSP Developers
    - devuser01-devuser25
    - nutanix/4u
  * - SSP Power Users
    - poweruser01-poweruser25
    - nutanix/4u
  * - SSP Basic Users
    - basicuser01-basicuser25
    - nutanix/4u
