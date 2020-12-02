.. _flow_secure_app:

----------------
Flow: Secure App
----------------

*The estimated time to complete this lab is 30 minutes.*

Overview
++++++++

**Flow** เป็นเพิ่มความปลอดภัยเครือข่ายในรูปแบบแอปพลิเคชันเป็นศูนย์กลาง หรือที่เรียกว่า **"application-centric"** ซึ่งจะทำงานร่วมกับ **Nutanix AHV** และ **Prism Central**. **Flow** นั้นมีความสามารถมากมายไม่ว่าจะเป็นการแสดงภาพกราฟฟิกการเข้าถึงของเครือข่ายที่สมบูรณ์, ระบบอัตโนมัติ รวมไปถึงการรักษาความปลอดภัยในระดับ VM ที่ทำงานบน AHV.

**Microsegmentation** เป็นส่วนประกอบของ Flow ที่ใช้การจัดการตามนโยบายอย่างง่ายเพื่อรักษาความปลอดภัยเครือข่าย VM ด้วยการกำหนด **"categories"** ใน **Prism Central** เราสามารถสร้าง **"Distributed Firewall"** ที่มีประสิทธิภาพซึ่งทำให้ผู้ดูแลระบบมีเครื่องมือการจัดการนโยบายที่เน้นไปแอปพลิเคชันเป็นศูนย์กลางสำหรับการรักษาความปลอดภัยของ VM
และยังสามารถปรับใช้กับแอปพลิเคชันได้โดยอัตโนมัติหลังจากที่เราสร้างนโยบายความปลอดภัยนั้นขึ้นมาแล้ว

ใน Lab นี้เราจะต้องสร้างนโยบายความปลอดภัยเพื่อ จำกัด การสื่อสารระหว่าง VM ของแอปพลิเคชัน

Lab Setup
+++++++++

ใน Labs **"Flow"** ทั้งหมดนี้ เราจะใช้ เว็บแอฟพลิเคชั่น ที่มีชื่อว่า **Task Manger** ซึ่งได้ถูกเตรียมไว้ให้แล้ว ประกอบไปด้วย

.. list-table::
  :widths: 25 25 25 25
  :header-rows: 1

  * - VM
    - Service
    - Username
    - Password
  * - *Initial*-WinClient
    - Windows Client
    - Administrator
    - nutanix/4u
  * - *Initial*-HAProxy
    - HA Load Balance
    - root
    - SSH Private Key
  * - *Initial*-WebServer-#
    - Web Server 2 VMs
    - root
    - SSH Private Key
  * - *Initial*-MYSQL
    - DB Server
    - root
    - SSH Private Key

.. note::

  *Initials* หรือ *XYZ* คือ ชื่อ User ของผู้เรียน เช่น *User01*\ **-WinClient**

เมื่อทำการตรวจสอบว่า VM ต่างๆ ภายใน **Task Manager** แอฟพลิเคชั่น มีครบแล้ว, เราก็สามารถเริ่มทำ Labs ได้เลย.

Securing An Application
+++++++++++++++++++++++

Flow มาพร้อมกับ System: categories ต่างๆ แบบ out of the box เช่น AppType, AppTier และ Environment ที่ซึ่งสามารถนำไปจัดกลุ่ม VM ต่างๆเข้ากับ security policies ได้อย่างรวดเร็ว นอกจากนั้น เรายังสามารถเพิ่ม   categories สำหรับการ custom กลุ่มเองได้อีกด้วย.

กำหนดค่า Category Values
........................

Prism Central จะใช้ categories เป็นเสมือนกับ metadata ในการ tag VM ต่างๆ เพื่อกำหนดว่า Policies เหล่านั้นจะถูกนำไปใช้งานอย่างไร.

#. ใน **Prism Central**, คลิกที่ :fa:`bars` บริเวณมุมซ้ายบน จากนั้นเลือกไปที่เมนู **> Virtual Infrastructure > Categories**.

#. ติ้กเครื่องหมายถูกที่หน้า **AppType** จากนั้นคลิก  **Actions** ที่อยู่ด้านบน แล้วเลือก **Update**.

   .. figure:: images/12.png

#. คลิกที่เครื่องหมาย :fa:`plus-circle` ตรง value บรรทัดสุดท้าย เพื่อทำการเพิ่ม "Category value".

#. กำหนดชื่อของ value เป็น  *Initials*-**TaskMan**.

   .. figure:: images/13.png

#. คลิก **Save**.

#. ติ้กเครื่องหมายถูกที่หน้า  **AppTier** จากนั้นคลิก  **Actions** ที่อยู่ด้านบน แล้วเลือก **Update**.

#. คลิกที่เครื่องหมาย :fa:`plus-circle` ที่อยู่ด้านหลังของ value ล่างสุด เพื่อเพิ่ม Category value.

#. กำหนดชื่อ value เป็น *Initials*-**TMWeb**. (category นี้จะถูก apply ไปยัง application web tier)

#. คลิกที่เครื่องหมาย :fa:`plus-circle` และกำหนดชื่อ value เป็น *Initials*-**TMDB**. (category นี้จะถูก apply ไปยัง MySQL database)

#. คลิกที่เครื่องหมาย :fa:`plus-circle` และกำหนดชื่อ value เป็น *Initials*-**TMLB**. (category นี้จะถูก apply ไปยัง HAProxy load balancer)

   .. figure:: images/14.png

#. คลิก **Save**.

สร้าง Security Policy
..........................

Nutanix Flow ประกอบไปด้วย policy-driven security framework ที่มุ้งเน้นไปที่การทำ workload-centric แทนที่ network-centric ดังนั้น Flow จึงสามารถวิเคราะห์ traffic ของ VM ต่างๆ ไม่ว่า network configuration จะเปลี่ยนแปลง หรือ อยู่ที่ส่วนใดของ data center ก็ตาม ทั้งนี้ workload-centric ยังมีแนวคิดแบบ network-agnostic จึงทำให้ virtualization team สามารถทำการสร้าง security policy ด้วยตนเอง โดยไม่จำเป็นต้องพึ่งพา network security team อีกด้วย

นอกจาก Security policies จะถูกนำไปใช้กับตัว VM ต่างๆแล้ว ก็ยังถูกนำไปใช้กับ categories (logical group ของ VM) อีกด้วย ดังนั้นแล้ว ไม่ว่าจะมี VM กี่ตัวที่ถูก start ขึ้นมาใน category นั้นๆ traffic ที่เกี่ยวข้องกับ VM ใน category ก็จะถูก secure ไว้เช่นกัน โดยที่ admin  ไม่จำเป็นต้องเข้ามาจัดการ

#. ใน **Prism Central**, กดที่ :fa:`bars` เลือก **> Policies > Security Policies**.

#. คลิก **Create Security Policy > Secure Applications (App Policy) > Create**.

#. กำหนดค่าต่างๆ ดังนี้:

   - **Name** - *Initials*-AppTaskMan
   - **Purpose** - Restrict unnecessary access to Task Manager
   - **Secure this app** - AppType: *Initials*-TaskMan
   - **ไม่ต้อง** เลือก **Filter the app type by category**.

   .. figure:: images/18.png

#. คลิก **Next**.

#. คลิก **OK, Got it!** หากมี tutorial ของหน้าต่างของ  **Create App Security Policy** แสดงขึ้นมา.

#. เพื่อให้สามารถกำหนดค่านโยบายความปลอดภัยได้ละเอียดยิ่งขึ้น, คลิก **Set rules on App Tiers** ซึ่งเราสามารถกำหนดในแต่ละส่วนของ Application นี้ได้.

   .. figure:: images/19.png

#. คลิก **+ Add Tier**.

#. เลือก **AppTier:**\ *Initials*-**TMLB** from the drop down.

#. ทำซ้ำในขั้นตอนที่ 7-8 เพื่อเพิ่ม **AppTier:**\ *Initials*-**TMWeb** และ **AppTier:**\ *Initials*-**TMDB**.

   .. figure:: images/20.png

   ในส่วนถัดไป ท่านจะต้องทำการกำหนด **Inbound** rules ที่สามารถช่วยควบคุม source ที่ allow ให้ติดต่อสื่อสารกับ application ได้ โดยที่ท่านสามารถทำการ allow inbound traffic ทั้งหมด หรือเลือกกำหนด whitelist source ที่ต้องการได้เช่นกัน (โดย default แล้ว security policy จะถูกตั้งค่าไว้ให้ deny incoming traffic ทั้งหมด) 

   ใน scenario นี้ เราจะทำการ allow inbound TCP traffic ทั้งหมดบน port 80 จาก client ทั้งหมดบน production network


#. กด + Add Source บริเวณส่วนของ Inbound ซ้ายมือ

#. พิมพ์ **Environment:Production** และคลิก **Add**.

   .. note::

     Sources สามารถระบุเป็น IP หรือ subnet ได้เช่นกัน แต่หากระบุเป็น Categories จะมีความยืนหยุ่นมากกว่า เพราะข้อมูลจะ follow ตาม VM โดยไม่คำนึงถึงการเปลี่ยนแปลงของ network location

#. 12.  สำหรับการสร้าง inbound rule คลิกเครื่องหมาย **+** ที่ปรากฏอยู่บริเวณด้านซ้ายของ  **AppTier:**\ *Initials*-**TMLB**.

   .. figure:: images/21.png

#. กำหนดค่าต่างๆ ดังนี้:

   - **Protocol** - TCP
   - **Ports** - 80

   .. figure:: images/22.png

   .. note::

     สามารถทำการเพิ่มได้หลายๆ protocols และ ports บน rule เดียวกัน.

#. คลิก **Save**.

   .. note::

     Calm สามารถ access ไปที่ VM ต่างๆเพื่อทำ workflows ตลอดจนการทำ scaling out, scaling in หรือ upgrade ได้อีกด้วย โดยที่ Calm จะติดต่อสื่อสารกับ VM เหล่านี้ผ่าน SSH ด้วย TCP port 22

#. คลิก **+ Add Source** บริเวณส่วนของ Inbound ทางซ้ายมือ.

#. กำหนดค่าต่างๆ ดังนี้:

   - **Add source by:** - เลือก **Subnet/IP**
   - o  กำหนดเป็น *Prism Central IP*\ ของผู้เรียน แล้วตามด้วย /32

   .. note::

    "/32" หมายความว่า subnet range นี้จะสามารถใช้ได้ IP ที่ระบุเพียง IP เดียวเท่านั้น.

   .. figure:: images/23.png

#. คลิก **Add**.

#. คลิกเครื่องหมาย **+** ที่ปรากฏอยู่บริเวณด้านซ้ายของ  **AppTier:**\ *Initials*-**TMLB**, จากนั้นกำหนด **TCP** port **22** และคลิก **Save**.

#. ทำซ้ำในขั้นตอนที่ 18 สำหรับ **AppTier:**\ *Initials*-**TMWeb** และ **AppTier:**\ *Initials*-**TMDB** เพื่ออนุญาตให้ Calm สามารถติดต่อสื่อสารกับ web tier VM และ database VM ได้

   .. figure:: images/24.png

   โดยปกติแล้ว security policy จะอนุญาตให้ application สามารถส่ง outbound traffic ทั้งหมดไปที่ destination ใดก็ได้ แต่สำหรับ application ของใน Lab นี้ เราต้องการเพียง outbound สำหรับ database VM เพื่อให้สามารถติดต่อสื่อสารกับ DNS Server ได้

#. เลือก **Whitelist Only** จาก drop down และคลิก **+ Add Destination** บริเวณ **Outbound** ด้านขวามือ

#. กำหนดค่าต่างๆ ดังนี้:

   - **Add source by:** - เลือก **Subnet/IP**
   - กำหนดค่าเป็น *Domain Controller IP*\ ของผู้เรียน แล้วตามด้วย /32

   .. figure:: images/25.png

#. คลิก **Add**.

#. เลือกคลิกที่เครื่องหมาย **+** บริเวณขวามือของ **AppTier:**\ *Initials*-**TMDB**, และกำหนดค่า **UDP** port **53** และคลิก **Save** เพื่ออนุญาตให้สื่อสารกับ DNS ได้.

   .. figure:: images/26.png

   แต่ละ tier ของ application จะติดต่อสื่อสารกับ tier อื่นๆ และ policy ต้องทำการ allow traffic นั้นๆ ในขณะที่บาง tier เช่น load balancer และ web ไม่จำเป็นต้องติดต่อสื่อสารภายใน tier เดียวกัน .

#. เพื่อกำหนดการสื่อสารกันภายในตัว application เอง, คลิก **Set Rules within App**.

   .. figure:: images/27.png

#. คลิก **AppTier:**\ *Initials*-**TMLB** และเลือก **No** เพื่อป้องกันการเชื่อมต่อกันของ VM ต่างๆ ภายใน tier (มีแค่ load balancer VM เดียวภายใน tier นี้)

#. ตรวจสอบว่า **AppTier:**\ *Initials*-**TMLB** ยังถูกเลือกค้างไว้อยู่ จากนั้นคลิกที่เครื่องหมาย :fa:`plus-circle` ของ **AppTier:**\ *Initials*-**TMWeb** เพื่อสร้าง rule แบบ tier to tier.

#. กำหนดค่าต่างๆ ดังนี้ เพื่อกำหนดการเชื่อมต่อของ TCP port 80 ระหว่าง load balancer และ web tier:

   - **Protocol** - TCP
   - **Ports** - 80

   .. figure:: images/28.png

#. คลิก **Save**.

#. คลิก **AppTier:**\ *Initials*-**TMWeb** และเลือก **No** เพื่อป้องกันการเชื่อมต่อกันของ VM ต่างๆ ภายใน tier (web server VM ต่างๆ ไม่จำเป็นต้องเชื่อมต่อกันและกัน)

#. ตรวจสอบว่า **AppTier:**\ *Initials*-**TMWeb** ยังถูกเลือกค้างไว้อยู่ จากนั้นคลิกที่เครื่องหมาย :fa:`plus-circle` ทางด้านขวาของ **AppTier:**\ *Initials*-**TMDB** เพื่อสร้าง rule แบบ tier to tier 

#. กำหนดค่าต่างๆ ดังนี้ เพื่อ allow database connection ระหว่าง web server และ MySQL database:

   - **Protocol** - TCP
   - **Ports** - 3306

   .. figure:: images/29.png

#. คลิก **Save**.

#. คลิก **Next** เพื่อตรวจสอบ security policy ที่สร้างไว้

#. คลิก **Save and Monitor** เพื่อ save policy นี้

Assigning Category Values
.........................

ขณะนี้ ท่านจะสามารถ apply categories ที่ถูกสร้างไว้ก่อนหน้า ไปยัง VM ต่างๆ ที่ถูกเตรียมไว้ จาก Task Manage Application ได้ ทั้งนี้ Flow categories ก็ยังสามารถถูกกำหนดให้เป็นส่วนหนึ่งของ Calm blueprint ได้เช่นกัน แต่จุดประสงค์ของ lab นี้ เพื่อให้เข้าใจการกำหนด category ไปใช้กับ VM ที่มีอยู่แล้วในระบบ

#. ใน **Prism Central**, เลือก :fa:`bars` **> Virtual Infrastructure > VMs**.

#. คลิก **Filters** และกรอกชื่อ VM (*Initials*-) เพื่อค้นหาและแสดง VMs ของผู้เรียน

   .. figure:: images/15.png

#. กดที่ checkboxes เพื่อเลือก VM ทั้ง 4 ตัวที่เกี่ยวข้องกับ application (HAProxy, MySQL, WebServer-0, WebServer-1) จากนั้น เลือก **Actions > Manage Categories**.

   .. figure:: images/16.png

   .. note::

     เราสามารถใช้ฟังก์ชั่น Label เพื่อช่วยอำนวยความสะดวกในการค้นหา VM ได้รวดเร็วขึ้นอีกด้วย.

     .. figure:: images/16b.png

#. พิมพ์ **AppType:**\ *Initials*-**TaskMan** ในช่อง search bar และคลิก **Save** เพื่อกำหนด categoty ไปยัง VM ทั้ง 4 ตัว.

#. เลือกเฉพาะ *Initials*\ **-HAProxy** VM เท่านั้น, จากนั้นไปที่ **Actions > Manage Categories**, จากนั้นระบุ **AppTier:**\ *Initials*-**TMLB** category ให้กับ VM นี้ แล้วคลิก **Save**.

   .. figure:: images/17.png

#. ทำซ้ำในขั้นตอนที่ 5 เพื่อระบุ **AppTier:**\ *Initials*-**TMWeb** ให้กับ web tier VMs ทั้ง 2 ตัว.

#. ทำซ้ำในขั้นตอนที่ 5 เพื่อระบุ **AppTier:**\ *Initials*-**TMDB** ให้กับ MySQL VM.

#. และสุดท้าย, ทำซ้ำในขั้นตอนที่ 5 เพื่อระบุ **Environment:Dev** ให้กับ Windows client VM.

Monitoring and Applying a Security Policy
+++++++++++++++++++++++++++++++++++++++++

โปรดตรวจสอบอีกครั้งเพื่อให้มั่นใจว่า Task Manager application ทำงานได้อย่างถูกต้องตามที่ต้องการ ก่อนจะทำการ apply Flow policy.

ทดสอบการทำงานของ Application
............................

#. จากหน้า **Prism Central > Virtual Infrastructure > VMs**, จด IP address VM  *Initials*\ **-HAPROXY** และ *Initials*\ **-MYSQL** VMs.

#. เปิดหน้า console ของ *Initials*\ **-WinClient** VM ของผู้เรียน.

   VM นี้ถูกเตรียมขึ้นมาเป็นส่วนหนึ่งของ Task Manager application.

#. จากหน้า console ของ *Initials*\ **-WinClient** เปิด browser แล้วไปที่ \http://*HAPROXY-VM-IP*/.

#. ตรวจสอบว่า application สามารถเปิดขึ้นมาและทำงานได้อย่างถูกต้อง (สามารถ add และ delete task ได้).

   .. figure:: images/30.png

#. เปิด **Command Prompt** ขึ้นมาแล้ว ``ping -t <MYSQL-VM-IP>`` ทิ้งไว้ เพื่อตรวจสอบการเชื่อมต่อระหว่าง windows client กับ database

#. เปิด **Command Prompt** ขึ้นมาอีกหนึ่งหน้าจอ แล้ว ``ping -t <HAPROXY-VM-IP>`` ทิ้งไว้ เพื่อตรวจสอบการเชื่อมต่อระหว่าง windows client กับ database

   .. figure:: images/31.png

Using Flow Visualization
........................

#. กลับไปที่ **Prism Central** แล้วเลือก :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies >**\ *Initials*-**AppTaskMan**.

#. ตรวจสอบว่า **Environment: Dev** ปรากฏขึ้นมาที่บริเวณ inbound source โดยที่ source และเส้นจะแสดงเป็นสีเหลือง เพื่อบ่งชี้ว่า traffic นั้นๆได้ถูกตรวจพบ จากการที่เราใช้ windows client ping ไปยัง VMs ต่างๆ ใน Task Manager applicaiton ที่เราได้ทำการกำหนด Security Policy ไว้แล้ว.

   .. figure:: images/32.png

#. เลื่อนเมาส์ไปวางไว้บนเส้นที่เชื่อมต่อระหว่าง **Environment: Dev** กับ **AppTier:**\ *Initials*-**TMLB** เพื่อแสดง protocol และข้อมูลของการเชื่อมต่อ.

#. คลิกเข้าไปที่เส้นสีเหลือง เพื่อดูกราฟ connections attempts ในช่วง 24 ช่วงโมงที่ผ่านมา.

   .. figure:: images/33.png

   *หากตรวจพบว่ามี outbound traffic อื่นๆเกิดขึ้น เลื่อนเมาส์ไปวางบน connections เหล่านั้น เพื่อดูว่ามี port อะไรที่กำลังใช้งานอยู่ได้

#. คลิก **Update** เพื่อแก้ไข policy.

   .. figure:: images/34.png

#. คลิก **Next** และรอให้ traffic flow ที่ถูกตรวจพบแสดงขึ้นมา.

#. เลื่อนเมาส์ไปวางไว้บน **Environment: Dev** source ที่เชื่อมต่อไปยัง **AppTier:**\ *Initials*-**TMLB** และคลิกที่เครื่องหมาย :fa:`check` ที่ปรากฏขึ้นมา.

   .. figure:: images/35.png

#. คลิก **OK** เพื่อเสร็จสิ้นการเพิ่ม rule.

   ตอนนี้ Environment: Dev ที่ source ควรจะเปลี่ยนเป็นสีฟ้า เพื่อเป็นการบ่งชี้ว่ามันได้กลายเป็นส่วนหนึ่งของ policy เรียบร้อยแล้ว จากนั้นลากเมาส์ไปไว้บนเส้นเพื่อตรวจสอบว่ามีทั้ง ICMP (ping traffic) และ TCP port 80 แสดงขึ้นมา .

#. คลิก **Next > Save and Monitor** เพื่อ update policy.

Applying Flow Policies
......................

In order to enforce the policy you have defined, the policy must be applied.

#. เลือก *Initials*-**AppTaskMan** และคลิก **Actions > Apply**.

   .. figure:: images/36.png

#. พิมพ์ **APPLY** เพื่อยืนยันจากนั้นคลิก **OK** เพื่อเริ่มทำการห้ามไม่ให้ traffic ที่เราไม่ได้อนุญาตใน Security Polivy นี้สื่อสารกันได้.

#. กลับไปที่หน้า console ของ *Initials*\ **-WinClient**.

   ตรวจสอบว่าเกิดอะไรขึ้นกับ ping traffic จาก Windows client ไปยัง database server เช่น traffic ถูก block หรือไม่?

#. ตรวจสอบว่า Windows client VM ยังสามารถเข้าถึง application Task Manager โดยใช้ web browser  และ load balancer IP ได้หรือไม่ รวมไปถึง 

   เรายังสามารถเพิ่ม new task ที่ต้องการเชื่อมต่อระหว่าง web server และ database ได้หรือไม่?

Takeaways
+++++++++

- Microsegmentation ช่วยเพิ่มการป้องกันจากภัยคุกคามที่เป็นอันตราย ที่อาจจะเกิดจากเครื่องหนึ่งที่อยู่ภายใน data center แล้วแพร่กระจายไปยังเครื่องอื่นๆ
- เราสามารถสร้าง Categories ที่มีการกำหนด policy ไว้แล้ว หรือไม่ก็ตาม แล้วนำไปใช้กับ Calm blueprints ได้ เพื่อตอบสนองในเรื่องของ Application Security Automation.
- Flow ใช้ Prism Central ในการจัดการ ดังนั้น เราสามารถใช้ Flow Securiey Policy ร่วมกันภายใต้ Prism Central ได้ แม้ว่า VMs จะอยู่กันคนละ Nutanix Cluster
- Flow สามารถใช้กำหนด Policy ให้กับ VM ที่ทำงานอยู่บน AHV เท่านั้น
- Policy ที่ถูกสร้างขึ้น และปรับให้อยู่ในโหมด **Save and Monitor** ซึ่งหมายความว่าการเชื่อมต่อสื่อสารต่างๆ จะไม่ถูกปิดกั้นจนกว่าจะมีการ **APPLY** policy นั้นๆ เสียก่อน ประโยชน์ของโหมดนี้คือการให้ระบบได้เรียนรู้การเชื่อมต่อสื่อการทั้ง Inbound และ Outbound กับ Application ที่เราต้องการสามารถ Security Policy นั้น และเพื่อให้แน่ใจว่าไม่มีการปิดกั้นการเชื่อมต่อสื่อสารโดยไม่ได้ตั้งใจ