.. _flow_isolate_environments:

--------------------------
Flow: Isolate Environments
--------------------------

*The estimated time to complete this lab is 30 minutes.*

Overview
++++++++

ใน lab นี้ ผู้เรียนจะสามารถสร้าง environment category และกำหนดให้กับ Task Manager application รวมไปถึง ผู้เรียนจะสามารถสร้างและติดตั้ง isolation security policy ที่นำไปใช้กับ category  ที่ถูกสร้างขึ้นใหม่ตามลำดับ เพื่อจำกัดการการเข้าถึงที่ไม่อนุญาตได้

Isolating Environments
++++++++++++++++++++++

**Isolation policies** ถูกนำมาใช้เมื่อเราต้องการปิดกั้นการเชื่อมต่อโดยสมบูรณ์ของกลุ่ม VMs หนึ่ง กับอีกกลุ่ม VMs หนึ่ง
ยกตัวอย่างง่ายๆ ของการใช้ isolation policies เช่น เมื่อเราต้องการ block VMs ที่ถูกจัดให้อยู่ใน category ที่มีชื่อว่า **Environment: Dev** ไม่ให้ติดต่อกับ VM ที่ถูกจัดให้อยู่ใน category ที่มีชื่อว่า **Environment: Production** 

ทั้งนี้ ไม่ควรใช้ isolation policies หากต้องการสร้างข้อยกเว้นในการเชื่อมระหว่าง VMs ทั้งสองกลุ่ม แต่ควรใช้ Application Policy เพื่อทำ whitelist แทน

ใน lab นี้ ผู้เรียนจะสามารถสร้าง environment category และกำหนดให้กับ Task Manager application รวมไปถึง ผู้เรียนจะสามารถสร้างและติดตั้ง isolation security policy ที่นำไปใช้กับ category  ที่ถูกสร้างขึ้นใหม่ตามลำดับ เพื่อจำกัดการการเข้าถึงที่ไม่อนุญาตได้


Creating and Assigning Categories
.................................

.. note::

  *Initials* หรือ *XYZ* คือ ชื่อ User ของผู้เรียน เช่น *User01*\ **-WinClient**

#. ใน **Prism Central**, เลือก :fa:`bars` **> Virtual Infrastructure > Categories**.

#. ติ้กเครื่องหมายถูกที่ **Environment** จากนั้นคลิก **Actions > Update**.

#. คลิกที่เครื่องหมาย :fa:`plus-circle` ข้าง value บรรทัดสุดท้าย เพื่อเพิ่ม category value.

#. กำหนดชื่อของ value เป็น  *Initials*-**Prod**.

   .. figure:: images/37.png

#. คลิก **Save**.

#. ใน **Prism Central**, เลือก :fa:`bars` **> Virtual Infrastructure > VMs**.

#. คลิก **Filters** ที่มุมบนขวา ติ๊กที่ **Name** และกรอกชื่อ *Initials-* เพื่อแสดง VMs ทั้งหมดของผู้เรียน.

   .. note::

     ก่อนหน้านี้เราได้ทำการสร้าง Category สำหรับ Taks Manager application ของเราเอสไว้แล้ว ผู้เรียนสามารถค้นหาจากชื่อ label นั้นๆได้ อีกทางเลือกหนึ่งคือ ท่านสามารถค้นหาจาก category **AppType:** *Initials*-**TaskMan** ในหน้า Filter ได้เช่นกัน.

     .. figure:: images/38.png

#. ติ๊กที่ checkboxes เพื่อเลือก VMs ทั้ง 4 ตัวใน application (HAProxy, MYSQL, WebServer-0, WebServer-1) จากนั้นเลือก **Actions > Manage Categories**.

#. พิมพ์ **Environment:**\ *Initials*-**Prod** ในช่อง search bar จากนั้นคลิก **Save** เพื่อกำหนด category ให้กับทั้ง 4 VMs.

   .. figure:: images/39.png

Creating an Isolation Policy
............................

#. ใน **Prism Central**, เลือก :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies**.

#. คลิก **Create Security Policy > Isolate Environments (Isolation Policy) > Create**.

#. กำหนดค่าต่างๆ ดังนี้:

   - **Name** - *Initials*-Isolate-dev-prod
   - **Purpose** - *Initials* - Isolate dev from prod
   - **Isolate This Category** - Environment:Dev
   - **From This Category** - Environment:*Initials*-Prod
   - **ไม่ต้อง** เลือก **Apply this isolation only within a subset of the datacenter**. 

   .. figure:: images/40.png

#. คลิก **Apply Now** เพื่อ save policy และเริ่มการใช้งานทันที.

#. กลับไปที่หน้า console ของ *Initials*\ **-WinClient**.

   ตรวจสอบว่า Task Manager ยังสามารถเข้าได้หรือไม่? และทำไมจึงเป็นเช่นนั้น?

   การใช้งาน policy เหล่านี้ ช่วยให้สามารถ block traffic ระหว่าง group ของ VMต่างๆ เช่น แยก production กับ development ออกจากกัน, แบ่ง zone ทำ lab ตลอดจนการทำ compliance isolation.

Deleting a Policy
.................

#. ใน **Prism Central**, เลือก :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies**.

#. เลือก *Initials*-**Isolate-dev-prod** และคลิก **Actions > Delete**.

#. พิมพ์ **DELETE** เพื่อยืนยัน จากนั้นคลิก **OK** เพื่อ disable policy.

   .. note::

     สำหรับการ disable policy นั้น เราสามารถที่จะเลือก Monitor mode แทนที่จะลบทั้ง policy ก็ได้.

#. กลับไปที่หน้า console ของ *Initials*\ **-WinClient** เพื่อตรวจสอบว่า Task Manager application สามารถกลับมาใช้งานผ่าน browser ได้ตามเดิมหรือไม่.

Takeaways
+++++++++

- ใน Labs นี้เราได้ทดลองสร้าง Categoty และ Isolation Policy ซึ่งสามารถทำได้อย่างง่ายดาย โดยที่เราไม่ได้เข้าไปเปลี่ยนแปลงหรือแก้ไข Network Configuration แต่อย่างใด
- หลังจากที่เราติดแท็กให้กับ VM ด้วย Category ที่สร้างขึ้นแล้ว, VM นั้นก็จะได้รับ Policy ที่เรากำหนดไว้โดยอัตโนมัติ
- Isolation Policy นั้นจะถูกลำดับความสำคัญที่สูงกว่า Application Policy. เมื่อ VM ถูกกำหนดให้อยู่ใน Isolation Policy ก็จะไม่สามารถเชื่อมต่อสื่อสารกับ VM ที่อยู่อีกกลุ่มภายได้ Isolation Policy เดียวกันได้ถึงแม้ App Policy จะอนุญาต