# lerning storcli64

# การใช้ **MegaCLI/StorCLI** 

เพื่อลบข้อมูลบนดิสก์ (clear disk) โดยใช้คำสั่ง `storcli64 /c0/fall del force` 
แยกเป็น **สองส่วนหลัก** คือ 

**การดาวน์โหลด/ติดตั้ง**
**การใช้งานคำสั่งอย่างปลอดภัย**

---

## **1️⃣ ดาวน์โหลดและติดตั้ง StorCLI**

StorCLI เป็นเครื่องมือของ Broadcom/LSI สำหรับจัดการ RAID controllers (LSI MegaRAID)

1. ไปที่เว็บ Broadcom (เดิม LSI) สำหรับ MegaRAID StorCLI:

   * URL ล่าสุด (เช็คจาก Broadcom): [https://www.broadcom.com/support/download-search](https://www.broadcom.com/support/download-search)
   * ค้นหา `StorCLI` หรือ `MegaRAID Storage Manager (MSM)`

2. เลือกเวอร์ชันที่ตรงกับระบบปฏิบัติการของคุณ

   * Linux: จะได้ไฟล์ `storcli-x.x.x.tar.gz` หรือ `storcli-xx.x.x-linux-x86_64.tar.gz`
   * Windows: จะได้ไฟล์ `.zip`

3. ติดตั้งบน Linux:

   ```bash
   tar -zxvf storcli-xx.x.x-linux-x86_64.tar.gz
   cd storcli-xx.x.x-linux-x86_64
   chmod +x storcli64
   sudo mv storcli64 /usr/local/bin/
   ```

   ตอนนี้สามารถเรียกใช้งานด้วย `storcli64` ได้จากทุกที่

---

## **2️⃣ ตรวจสอบ Controller และ Disk ก่อนลบ**

⚠️ **สำคัญมาก:** คำสั่ง `del force` จะลบ **ข้อมูลทั้งหมดบน disk** ซึ่งไม่สามารถกู้คืนได้

1. ตรวจสอบว่า controller ของคุณคืออะไร

   ```bash
   storcli64 show
   ```

   ตัวอย่าง output:

   ```
   Controller = 0
   Model = LSI MegaRAID 9361-8i
   ```

2. ตรวจสอบ disk ทั้งหมดบน controller 0

   ```bash
   storcli64 /c0 /eall /sall show
   ```

   * `/c0` คือ controller 0
   * `/eall` คือทุก enclosure
   * `/sall` คือทุก slot

3. ระบุ disk ที่ต้องการลบ

   * ตัวอย่าง: `/c0/e32/s1` หมายถึง controller 0, enclosure 32, slot 1

---

## **3️⃣ ลบข้อมูลทั้งหมดบน disk**

1. ลบทุก disk ใน controller 0

   ```bash
   storcli64 /c0 /fall del force
   ```

   * `/fall` = all foreign/physical disks in controller 0
   * `del force` = บังคับลบข้อมูลทั้งหมด

2. ลบ disk เฉพาะ slot หรือ enclosure

   ```bash
   storcli64 /c0 /e32 /s1 del force
   ```

   * `/c0` = controller 0
   * `/e32` = enclosure 32
   * `/s1` = slot 1

3. ตรวจสอบผลลัพธ์หลังลบ

   ```bash
   storcli64 /c0 /e32 /s1 show
   ```

   * ถ้า disk กลายเป็น `Unconfigured(good)` แสดงว่าลบเรียบร้อย

---

## **4️⃣ ข้อควรระวัง**

* ทำการ **backup ข้อมูลสำคัญ** ก่อนทุกครั้ง
* ใช้ `show` ตรวจสอบ disk และ controller ให้ชัดเจนก่อนรัน `del force`
* คำสั่งนี้ **ไม่สามารถ undo** ได้

---


## Example: 

ต้องการ **ลบข้อมูลของ disk 4 และ 5** ด้วย StorCLI อย่างละเอียดและปลอดภัย ผมจะอธิบาย **step-by-step พร้อมคำสั่งจริง**

---

## **1️⃣ ตรวจสอบ Controller และ Disk ก่อนลบ**

ก่อนทำอะไร เราต้องแน่ใจว่า disk ที่จะลบคือ disk 4 และ 5 จริง ๆ

1. เช็ค controller ทั้งหมด:

```bash
storcli64 show
```

Output ตัวอย่าง:

```
Controller = 0
Model = LSI MegaRAID 9361-8i
```

* จากตัวอย่างนี้ controller ของคุณคือ `0`

2. แสดง disk ทั้งหมดใน controller 0:

```bash
storcli64 /c0 /eall /sall show
```

* `/eall` = ทุก enclosure
* `/sall` = ทุก slot

Output ตัวอย่างจะมีลักษณะประมาณนี้:

```
EID:Slt DID State
32:0   0   Online
32:1   1   Online
32:2   2   Online
32:3   3   Online
32:4   4   Online  <-- Disk 4
32:5   5   Online  <-- Disk 5
```

* จำว่า Disk 4 = `/c0/e32/s4`
* Disk 5 = `/c0/e32/s5`

> ⚠️ ตรวจสอบดี ๆ ก่อนรันคำสั่งลบ เพราะ `del force` **ลบข้อมูลทั้งหมดทันทีไม่สามารถกู้คืนได้**

---

## **2️⃣ ลบข้อมูล Disk 4 และ 5**

### ลบทีละ disk

```bash
storcli64 /c0 /e32 /s4 del force
storcli64 /c0 /e32 /s5 del force
```

* `/c0` = Controller 0
* `/e32` = Enclosure 32
* `/s4` = Slot 4 (disk 4)
* `/s5` = Slot 5 (disk 5)
* `del force` = ลบข้อมูลทั้งหมดอย่างบังคับ

### ลบพร้อมกัน (ถ้าต้องการ)

StorCLI รองรับการลบหลาย disk ในคำสั่งเดียว:

```bash
storcli64 /c0 /e32 /s4-5 del force
```

* `/s4-5` = Disk slot 4 ถึง 5

---

## **3️⃣ ตรวจสอบผลลัพธ์หลังลบ**

หลังรันคำสั่งลบ ให้ตรวจสอบว่า disk กลายเป็น `Unconfigured(good)`

```bash
storcli64 /c0 /e32 /s4 show
storcli64 /c0 /e32 /s5 show
```

Output ตัวอย่าง:

```
EID:Slt DID State
32:4   4   Unconfigured(good)
32:5   5   Unconfigured(good)
```

* แสดงว่าลบข้อมูลเรียบร้อย พร้อมใช้งานสร้าง RAID ใหม่

---

## **4️⃣ ข้อควรระวัง**

1. **Backup ข้อมูลสำคัญ** ทุกครั้ง
2. ตรวจสอบ disk number ให้ชัดเจนก่อนลบ
3. คำสั่ง `del force` **ไม่สามารถ undo** ได้

---

ถ้าคุณต้องการ ผมสามารถทำ **ภาพจำลองขั้นตอนและคำสั่ง** พร้อมแสดง output หลังลบ disk 4 และ 5 ให้ดูเหมือนจริง เพื่อให้ follow ง่ายขึ้น

คุณอยากให้ผมทำภาพจำลองแบบนั้นไหม?


