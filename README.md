# Microsoft SLA Outage Guide

เว็บแอปแบบหน้าเดียวสำหรับช่วยประเมินผลกระทบจาก Microsoft service outage และคำนวณค่า **Monthly Uptime Percentage** เพื่อใช้เป็นข้อมูลประกอบการขอ SLA Service Credit

> แอปนี้เป็นเครื่องมือช่วยประเมินเบื้องต้นเท่านั้น ผลการคำนวณไม่ใช่การยืนยันสิทธิ์รับเครดิต โดยต้องตรวจสอบเอกสาร SLA ของบริการและเงื่อนไขสัญญาฉบับล่าสุดเสมอ

## ฟีเจอร์

- อธิบายขั้นตอนตรวจสอบเหตุการณ์ Outage, การรวบรวมหลักฐาน และการยื่นคำขอเครดิต
- แสดงหลักการคำนวณ Monthly Uptime Percentage
- คำนวณ Downtime หลังหัก Planned Maintenance
- แสดงผล Uptime, SLA threshold, ระดับเครดิตโดยประมาณ และจำนวนเครดิตจากค่าบริการรายเดือน
- สรุปช่วงเวลาสำหรับยื่นคำขอ: Azure ภายใน 2 เดือน และบริการอื่นภายใน 1 เดือน นับจากสิ้นสุด billing month ที่เกิดเหตุการณ์
- รวมลิงก์เอกสารอ้างอิงจาก Microsoft และ Microsoft Learn

## เริ่มใช้งาน

โปรเจกต์นี้ไม่มี dependency หรือขั้นตอน build เพราะเป็น static web page

1. ดาวน์โหลดหรือ clone repository นี้
2. เปิดไฟล์ `index.html` ด้วยเว็บเบราว์เซอร์สมัยใหม่ เช่น Microsoft Edge, Google Chrome หรือ Firefox
3. เลื่อนลงไปที่หัวข้อ **SLA Calculator — คำนวณ Uptime และ Credit**

## ขั้นตอนใช้เครื่องคำนวณ

1. เลือกประเภทบริการ Microsoft ที่ได้รับผลกระทบ
2. เลือกจำนวนวันของ billing month ที่เกิด Outage
3. กรอกเวลา Downtime ที่ Microsoft ยืนยันแล้ว เป็นชั่วโมงและนาที
4. กรอกเวลา Planned Maintenance ที่ได้รับแจ้งล่วงหน้า หากมี
5. กรอกค่าบริการรายเดือนเป็น USD หากต้องการประเมินวงเงินเครดิต
6. กดปุ่ม **คำนวณ SLA**
7. ตรวจสอบผลลัพธ์ ได้แก่ Uptime, Net Downtime, เวลาที่ยังอยู่ใน SLA 99.9%, ระดับเครดิต และช่วงเวลายื่นคำขอ
8. กด **รีเซ็ต** เพื่อล้างค่าที่กรอกและเริ่มคำนวณใหม่

## สูตรคำนวณ

```text
Net Downtime = Raw Downtime - Planned Maintenance

Monthly Uptime Percentage =
  (Total Billable Minutes - Net Downtime) / Total Billable Minutes × 100
```

โดย `Total Billable Minutes` ในแอปคำนวณจากจำนวนวันในเดือน × 24 ชั่วโมง × 60 นาที

## ข้อควรระวัง

- SLA ของแต่ละ Microsoft service, service resource, service tier และรูปแบบการซื้ออาจมีนิยาม Downtime, เงื่อนไข, exclusions และอัตรา Service Credit แตกต่างกัน
- บางบริการใช้การวัดจาก valid requests, กำหนดให้มี retry หรือมีเงื่อนไขด้าน configuration ก่อนจึงจะเข้าเกณฑ์ SLA
- เครดิตอาจคิดเฉพาะบริการหรือทรัพยากรที่ได้รับผลกระทบ ไม่ใช่ค่าบริการทั้งชุด
- ควรเก็บ Incident ID, เวลาเริ่มและสิ้นสุดพร้อม timezone, error logs, หลักฐาน retry และ Support Ticket ก่อนส่งคำขอ
- ตรวจสอบกำหนดเวลายื่นเรื่องและเอกสาร SLA ฉบับล่าสุดก่อนดำเนินการ

## แหล่งอ้างอิง

- [Service Level Agreements (SLA) for Online Services](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services)
- [How to read a Service-Level Agreement (SLA)](https://learn.microsoft.com/azure/reliability/concept-service-level-agreements)
- [Request a credit from Microsoft](https://learn.microsoft.com/en-us/partner-center/request-credit)
- [Report a service problem on behalf of a customer](https://learn.microsoft.com/en-us/partner-center/customers/report-problems-on-behalf-of-a-customer)

## โครงสร้างโปรเจกต์

```text
.
├── index.html   # หน้าเว็บและ logic ของเครื่องคำนวณ
└── README.md    # เอกสารแนะนำการใช้งาน
```
