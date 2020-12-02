.. _flow_quarantine_vm:

-------------------
Flow: Quarantine VM
-------------------

*The estimated time to complete this lab is 10 minutes.*

Overview
++++++++

**Quatantine**  คือการจำกัดการเข้าถึงของ VM ด้วย Policy ทำให้ผู้ดูแลระบบสามารถเลือกได้ว่าจะปิดกั้นสื่อสารทั้งหมดของ VM นั้นๆ หรือยังสามารถสื่อสารได้กับช่องทางที่กำหนดไว้เท่านั้น โดยที่การปิดกั้นจากการสื่อสารทั้งหมดนั้นเราจะเรียกว่า **Strict quarantine** ในขณะที่ **forensic quarantine** นั้นจะสามารถอนุญาตให้สื่อสารหรือรับส่งข้อมูลจากการกำหนด inbound และ outbound ไว้ล่วงหน้า

Quarantining a VM
+++++++++++++++++

ในส่วนนี้ ท่านจะสามารถย้าย VM ไปไว้ใน quarantine เพื่อสังเกตุพฤติกรรมที่ผิดปกติของ VM ได้ อีกทั้งยังสามารถตรวจสอบตัวเลือก configure ต่างภายใน quarantine policy ได้อีกด้วย

.. note::

  *Initials* หรือ *XYZ* คือ ชื่อ User ของผู้เรียน เช่น *User01*\ **-WinClient**

#. ไปยัง *Initials*\ **-WinClient-0** console.

#. เปิด **Command Prompt** ขึ้นมา และพิมพ์ ``ping -t <HAPROXY-VM-IP>`` เพื่อตรวจสอบการเชื่อมต่อระหว่าง windows client กับ load balancer.

   .. note::

     หากไม่สามารถ ping ได้ ผู้เรียนอาจจำเป็นที่จะต้อง update Inbound Rule สำหรับ **Environment:Dev** ไปยัง **AppTier:**\ *Initials*-**TMLB** ของผู้เรียนตามรูปด้านล่าง และทำการ apply **AppTaskMan**\-*Initials* policy จึงจะสามารถกลับมา ping ได้ตามปกติ .

     .. figure:: images/41.png

#. ใน **Prism Central > Virtual Infrastructure > VMs**, เลือก *Initials*\ **-HAPROXY** VM ของผู้เรียน.

#. คลิก **Actions > Quarantine VMs**.

   .. figure:: images/42.png

#. เลือก **Forensic** จากนั้นคลิก **Quarantine**.

   ลองตรวจสอบว่าเกิดอะไรขึ้นกับการ ping ระหว่าง windows client กับ load balancer? รวมไปถึง ท่านยังสามารถเข้าถึง Task Manager web page จาก windows client VM ได้หรือไม่?

#. ใน **Prism Central**, เลือก :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies > Quarantine** เพื่อแสดง VMs ที่ถูก Quarantined ทั้งหมด.

#. คลิก **Update** เพื่อทำการแก้ไข Quarantine policy.

   To illustrate the capabilities of this special Flow policy, you will add your client VM as a "forensic tool". In production, VMs allowed inbound access to quarantined VMs could be used to run security and forensic suites such as Kali Linux or SANS SIFT.
   เพื่อให้มองเห็นภาพถึงความสามารถของ policy พิเศษของ Flow ตัวนี้ เราจะทำการเพิ่ม windows client VM ของเราเป็น “forensic tool”. 

   ซึ่งในการใช้งานกับ Production จริงนั้นเราจะอนุญาตให้ VMs ที่จะสามารถเข้าถึง VMs ที่ถูก quarantine อยู่นั้น จะต้องเป็น VM เพื่อไว้สำหรับใช้ run security/forensic suites เช่น Kali Linux หรือ SANS SIFT เท่านั้น.

#. คลิก **+ Add Source** บริเวณ **Inbound**.

#. กำหนดค่าต่างๆ ดังนี้:

   - **Add source by:** - เลือก **Subnet/IP**
   - ระบุ *WinClient VM IP*\ ของผู้เรียน แล้วตามด้วย /32

   แล้ว quarantine mode แบบ Forensic กับ Strict ต่างกันอย่างไร?

   .. note::

     VMs ที่ถูกเลือกให้อยู่ใน **Strict** Quarantine policy นั้นจะไม่สามารถเชื่อมต่อสื่อสารกับผู้อื่นได้. สำหรับ **Strict** policy นั้นควรถูกใช้กับ VM ที่มีท่าทีว่าจะเป็นอันตรายต่อระบบเป็นวงกว้าง.

#. คลิกที่เครื่องหมาย the :fa:`plus-circle` ที่ด้านซ้ายของ **Quarantine: Forensic** เพื่อสร้าง Inbound Rule.

#. คลิก **Save** เพื่อ allow protocol และ port ใดๆก็ตามระหว่าง client VM และ **Quarantine: Forensic** category.

   .. figure:: images/43.png

#. คลิก **Next > Apply Now** เพื่อ save และ apply policy ที่ได้ทำการ update.

   ลองตรวจสอบว่าเกิดอะไรขึ้นกับ ping ที่ไปยัง load balancer หลังจาก source ถูกเพิ่มเข้ามา รวมไปถึง Task Manager web application สามารถเข้าได้หรือไม่? 

#. ท่านจะสามารถลบ load balancer VM ออกจาก **Quarantine: Forensic** category ได้โดยการเลือก VM นั้น บนหน้า Prism Central และคลิก **Actions > Unquarantine VMs**.

Takeaways
+++++++++

- ใน Labs นี้เราได้รู้จักกับ Flow: Quarantine Policy ที่ใช้ในการจำกัดการเข้าถึงของ VM มี่ที่อยู่สองรูปแบบคือ Forensic และ Strict 
- Quarentine Policy นั้นจะถูกลำดับความสำคัญที่สูงกว่า Application Policy. เมื่อ VM ถูกกำหนดให้อยู่ใน Quarantined ก็จะไม่สามารถเชื่อมต่อสื่อสารกับคนอื่นได้ถึงแม้ App Policy จะอนุญาต
- Foransic คือ mode ที่เราสามารถจำกัดการเชื่อมต่อที่จำเป็นเท่านั้นไปยัง VM ในขณะที่ VM นั้นๆ ถูก quarantined อยู่