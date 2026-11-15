# Microsoft SLA Outage Guide

เว็บแอปแบบหน้าเดียวสำหรับช่วยประเมินผลกระทบจาก Microsoft service outage และคำนวณค่า **Monthly Uptime Percentage** เพื่อใช้เป็นข้อมูลประกอบการขอ SLA Service Credit

> แอปนี้เป็นเครื่องมือช่วยประเมินเบื้องต้นเท่านั้น ผลการคำนวณไม่ใช่การยืนยันสิทธิ์รับเครดิต โดยต้องตรวจสอบเอกสาร SLA ของบริการและเงื่อนไขสัญญาฉบับล่าสุดเสมอ

## ฟีเจอร์

- อธิบายขั้นตอนตรวจสอบเหตุการณ์ Outage, การรวบรวมหลักฐาน และการยื่นคำขอเครดิต
- แสดงหลักการคำนวณ Monthly Uptime Percentage
- คำนวณ Downtime หลังหัก Planned Maintenance
- แสดงผล Uptime, SLA threshold, ระดับเครดิตโดยประมาณ และจำนวนเครดิตจากค่าบริการรายเดือน
- มีตารางและตัวตรวจสอบเกณฑ์ SLA เดือนตุลาคม (31 วัน) สำหรับ Microsoft 365, Dynamics 365, Intune และ Azure SLA profiles
- สร้างร่างภาษาอังกฤษสำหรับเปิด Support Ticket / ขอ SLA Service Credit พร้อมรายการรายละเอียดที่ควรระบุและปุ่มคัดลอก
- คู่มือเฉพาะสำหรับ **CSP Partner** ยื่น Credit Request ผ่าน Partner Center: prerequisites, GDAP, ตารางเลือกเส้นทาง ticket, 8 ขั้นตอนโดยละเอียด, Required Information และ Evidence ของผลกระทบ
- สรุปช่วงเวลาสำหรับยื่นคำขอ: Azure ภายใน 60 วันนับจาก Incident; บริการอื่นภายในสิ้นสุด Applicable Period ถัดจากเดือนที่เกิด Incident
- รวมลิงก์เอกสารอ้างอิงจาก Microsoft และ Microsoft Learn

### ประสบการณ์ใช้งาน (UX / UI)

- **ปรับขนาดอัตโนมัติ** รองรับทั้งมือถือ แท็บเล็ต และเดสก์ท็อป ด้วย fluid typography (`clamp()`) และ grid แบบ `auto-fit/minmax()`
- **Dark Mode** พร้อมปุ่มสลับที่มุมบนขวา (จำค่าใน `localStorage` และ auto-detect `prefers-color-scheme`)
- **Sticky Nav** แบบ glassmorphism (backdrop-filter) ที่เลื่อนแนวนอนบนมือถือและ wrap ตรงกลางบนเดสก์ท็อป
- **Accessibility**: `:focus-visible` ring ชัดเจน, รองรับ `prefers-reduced-motion`, `aria-label` บนปุ่ม toggle
- **Print-friendly**: ปุ่มและ toggle ซ่อนอัตโนมัติเมื่อสั่งพิมพ์

## เริ่มใช้งาน

โปรเจกต์นี้ไม่มี dependency หรือขั้นตอน build เพราะเป็น static web page

1. ดาวน์โหลดหรือ clone repository นี้
2. เปิดไฟล์ `index.html` ด้วยเว็บเบราว์เซอร์สมัยใหม่ เช่น Microsoft Edge, Google Chrome หรือ Firefox
3. เลื่อนลงไปที่หัวข้อ **SLA Calculator — คำนวณ Uptime และ Credit**

## ขั้นตอนใช้เครื่องคำนวณ

ทำตามขั้นตอนต่อไปนี้หลังจากเปิด `index.html` แล้ว

### 1. เตรียมข้อมูลของเหตุการณ์

ก่อนเริ่มคำนวณ ให้รวบรวม Incident ID หรือ Support Ticket, วันที่และเวลาเริ่ม/สิ้นสุดเหตุการณ์ (พร้อม timezone), ชื่อบริการหรือ resource ที่ได้รับผลกระทบ และหลักฐานจาก Microsoft Service Health หรือการแจ้งเหตุที่ได้รับ ข้อมูลนี้ช่วยให้ตรวจสอบตัวเลขและใช้ประกอบการยื่นคำขอเครดิตได้

### 2. เลือกประเภทบริการ

ที่หัวข้อ **SLA Calculator — คำนวณ Uptime และ Credit** ให้เลือกบริการ Microsoft ที่ได้รับผลกระทบจากรายการ **ประเภทบริการ**

- เลือกให้ตรงกับบริการหรือรูปแบบการซื้อที่ใช้งานจริงมากที่สุด
- หากมีหลายบริการได้รับผลกระทบ ให้คำนวณแยกกัน เพราะ SLA, อัตราเครดิต และค่าบริการของแต่ละบริการอาจต่างกัน

### 3. ระบุจำนวนวันในรอบบิล

เลือกจำนวนวันของเดือนที่เกิดเหตุการณ์จากช่อง **Billing month**

- เลือก 28, 29, 30 หรือ 31 วันให้ตรงกับเดือนในรอบบิลจริง
- หาก outage เกิดข้ามเดือน ให้แยกคำนวณตามแต่ละ billing month

จำนวนวันนี้ใช้หาเวลารวมที่นำไปคำนวณ SLA (`จำนวนวัน × 24 × 60 นาที`)

### 4. กรอกเวลา Downtime

กรอกระยะเวลาที่บริการใช้งานไม่ได้ในช่อง **Downtime** โดยแยกเป็นชั่วโมงและนาที

- ใช้ช่วงเวลาที่ Microsoft ยืนยัน หรือช่วงเวลาที่เข้าเกณฑ์ตาม SLA ของบริการนั้น
- ตัวอย่าง: ระบบใช้งานไม่ได้ 2 ชั่วโมง 15 นาที ให้กรอก `2` ชั่วโมง และ `15` นาที
- หากมี outage หลายช่วงในเดือนเดียวกัน ให้รวมระยะเวลาทุกช่วงก่อนกรอก

### 5. กรอก Planned Maintenance (ถ้ามี)

หากเหตุการณ์มีช่วงเวลาบำรุงรักษาที่ Microsoft แจ้งล่วงหน้าและ SLA ของบริการอนุญาตให้หักออกได้ ให้กรอกในช่อง **Planned Maintenance**

- กรอกเป็นชั่วโมงและนาที เช่น 30 นาที ให้กรอก `0` ชั่วโมง และ `30` นาที
- หากไม่มี หรือไม่แน่ใจว่าเข้าเงื่อนไข ให้ใส่ `0`
- เครื่องมือจะนำค่านี้ไปหักจาก Downtime เพื่อได้ **Net Downtime**

### 6. กรอกค่าบริการรายเดือน (ไม่บังคับ)

กรอกค่าบริการรายเดือนของบริการหรือ resource ที่ได้รับผลกระทบในหน่วย USD ที่ช่อง **Monthly service charge** หากต้องการให้ระบบประเมินมูลค่าเครดิต

- ใช้เฉพาะค่าบริการส่วนที่เกี่ยวข้องกับเหตุการณ์ ไม่ใช่ค่าใช้จ่ายรวมทุกบริการ
- เว้นช่องนี้ไว้ได้ หากต้องการดูเพียงเปอร์เซ็นต์ Uptime และระดับเครดิต

### 7. คำนวณผล

กดปุ่ม **คำนวณ SLA** ระบบจะคำนวณและแสดงผลทันที ตรวจสอบตัวเลขที่กรอกอีกครั้งหากผลลัพธ์ไม่ตรงกับที่คาดไว้

### 8. อ่านผลลัพธ์

ผลลัพธ์หลักมีความหมายดังนี้

- **Monthly Uptime Percentage**: เปอร์เซ็นต์เวลาที่บริการพร้อมใช้งานในเดือนนั้นหลังหัก Net Downtime
- **Net Downtime**: Downtime ที่เหลือหลังหัก Planned Maintenance
- **SLA threshold / เวลาที่ยังอยู่ใน SLA**: ค่าเป้าหมาย SLA และเวลาที่ downtime ยังไม่ทำให้ต่ำกว่าค่าเป้าหมาย
- **ระดับเครดิตโดยประมาณ**: ระดับ Service Credit ที่เครื่องมือประเมินจากค่า Uptime
- **เครดิตโดยประมาณ**: มูลค่าเครดิตจากระดับเครดิตและค่าบริการรายเดือนที่กรอกไว้
- **ช่วงเวลายื่นคำขอ**: กำหนดเวลาสำหรับยื่นคำขอเครดิต โดย Azure ภายใน 60 วันนับจาก Incident; บริการอื่นภายในสิ้นสุด Applicable Period ถัดจากเดือนที่เกิด Incident

### 9. ตรวจสอบก่อนยื่นคำขอ

ใช้ผลลัพธ์เป็นข้อมูลประกอบ แล้วตรวจสอบ SLA ฉบับล่าสุดของบริการ, exclusions, นิยาม downtime, อัตราเครดิต และกำหนดเวลาจากเอกสารสัญญาจริงก่อนยื่นคำขอ พร้อมแนบ Incident ID, หลักฐานเวลาเกิดเหตุ และ Support Ticket ตามที่กำหนด

### 10. เริ่มการคำนวณใหม่

กดปุ่ม **รีเซ็ต** เพื่อล้างข้อมูลทั้งหมด แล้วกรอกข้อมูลชุดใหม่สำหรับเหตุการณ์หรือบริการอื่น

### ตัวอย่างย่อ

หากบริการมี outage 2 ชั่วโมง 15 นาทีในเดือน 30 วัน และมี Planned Maintenance ที่หักได้ 30 นาที ให้เลือก 30 วัน กรอก Downtime เป็น `2` ชั่วโมง `15` นาที กรอก Planned Maintenance เป็น `0` ชั่วโมง `30` นาที แล้วกด **คำนวณ SLA** ระบบจะใช้ Net Downtime เท่ากับ 1 ชั่วโมง 45 นาทีในการคำนวณ Uptime

## สร้างข้อความสำหรับเปิด Ticket / ขอ SLA Credit

หลังคำนวณ SLA แล้ว ให้เปิดหัวข้อ **สร้าง Ticket** จากแถบเมนู และทำตามขั้นตอนนี้

1. กรอกชื่อองค์กร, Tenant ID หรือ tenant domain และ Subscription ID / Billing account (ถ้ามี)
2. กรอก Microsoft Incident ID, ชื่อบริการหรือ resource ที่ได้รับผลกระทบ และ region หรือ location
3. ระบุเวลาเริ่มและสิ้นสุดเหตุพร้อม timezone รวมถึงจำนวนผู้ใช้หรือ workloads ที่ได้รับผลกระทบ
4. อธิบายอาการที่พบและผลกระทบทางธุรกิจที่ตรวจสอบได้ เช่น error message, timeout หรือการทำงานที่หยุดชะงัก
5. ระบุหลักฐานที่แนบและการตรวจสอบที่ทำแล้ว เช่น Service Health screenshot, error logs, request IDs, retry logs, Post Incident Review และ Support Case เดิม
6. กรอกชื่อกับอีเมลธุรกิจของผู้ติดต่อ แล้วกด **สร้างข้อความสำหรับ Ticket**
7. ตรวจทานร่างภาษาอังกฤษ โดยแทนที่ข้อความในวงเล็บเหลี่ยมที่ยังไม่มีข้อมูล และกด **คัดลอกข้อความ** เพื่อนำไปวางใน Microsoft Support Portal

ร่างข้อความจะดึงรายละเอียดการคำนวณที่ทำไว้ ได้แก่ billing month, downtime, planned maintenance, net downtime, monthly uptime และเครดิตโดยประมาณมาใส่ให้อัตโนมัติ ทั้งนี้ต้องตรวจสอบความถูกต้องและเงื่อนไข SLA ฉบับล่าสุดก่อนส่งเสมอ

## CSP Partner: ขอ Credit ผ่าน Partner Center

ในหน้าเว็บมีหัวข้อ **ขอเคลม** → การ์ด *CSP Partner: ขอ Credit ผ่าน Partner Center (ฉบับละเอียด)* ซึ่งสรุปขั้นตอนตามเอกสาร Microsoft Learn ไว้ครบ ใช้เป็น checklist ก่อนเปิด ticket จริง

### ใครยื่นได้ / ยื่นไม่ได้

- ✅ **CSP Direct Provider** และ **CSP Indirect Provider** ยื่นได้
- ❌ **Indirect Reseller** ยื่นเองไม่ได้ — ต้องให้ Indirect Provider เป็นผู้ยื่น
- ❌ ลูกค้าปลายทางไม่สามารถยื่น Credit ผ่าน Partner Center ได้ (ต้องยื่นผ่าน portal ของบริการที่ตนซื้อโดยตรง)

### Prerequisites ก่อนเปิด Ticket

- บัญชีของคุณต้องมี **Admin agent role** ใน Partner Center
- มี **GDAP relationship** ที่ active กับ tenant ลูกค้า (เช่น `Service support administrator`)
- สำหรับ Azure: ต้องเป็นผู้ให้บริการ Azure Subscription + มี **RBAC** บน subscription นั้น
- มี **email จากลูกค้า** ที่ส่งจาก domain ของ tenant ที่ได้รับผลกระทบจริง (ไม่รับ personal email)
- ยืนยันว่า Incident ไม่ใช่ **Advisory** เพียงอย่างเดียว

### เส้นทาง Ticket

| บริการ | เส้นทาง | Topic |
| --- | --- | --- |
| Microsoft 365 / Office 365 / Dynamics 365 / Intune / EMS | Partner Center → Help + support → Create request | *Cancel, Credit or suspend (non-Azure) subscriptions* |
| Microsoft Azure | Partner Center → Administer customer → New support request (เปิดใน Azure Portal context ของ partner) | Issue type: *Billing* → Problem type: *Refund Request* |

### เอกสาร 2 ชุดที่ต้องแนบ

**ชุดที่ 1 — Required Information**

1. Customer Tenant GUID
2. Outage Incident Identifier (เช่น `EX25194`)
3. หลักฐานว่าลูกค้าได้รับผลกระทบและขอ Credit
4. ระบุว่า subscription ซื้อผ่าน CSP หรือไม่ (Yes/No)

**ชุดที่ 2 — Evidence of Customer Impact**

1. Time & duration ของ downtime พร้อม timezone
2. จำนวนและที่ตั้งของ users ที่ได้รับผลกระทบ
3. Troubleshooting ที่ทำระหว่างเกิดเหตุ (retry, failover, workaround)
4. Email จากลูกค้าจาก domain ของ tenant ที่ได้รับผลกระทบ (ขอ support + credit)
5. Support ticket number และบันทึก contact กับลูกค้า

> ⚠️ ข้อมูลเหล่านี้ **ไม่เพียงพอ** สำหรับการ process: ชื่อลูกค้า + Tenant ID + Partner Ticket Number + Ticket created timestamp *เพียงเท่านี้* Microsoft จะ reject หรือทวงหลักฐานเพิ่ม

### หลังได้รับอนุมัติ

Credit จะถูก prorate ตามบริการที่กระทบและ duration ของ outage แล้วปรากฏใน invoice ถัดไปและ **Reconciliation File** ที่ Partner Center → *Billing → Invoices & reconciliation* ใช้ไฟล์นี้ map กลับไปยัง customer/subscription เพื่อออก credit note ให้ลูกค้าปลายทาง

## สูตรคำนวณ

```text
Net Downtime = Raw Downtime - Planned Maintenance

Monthly Uptime Percentage =
  (Total Billable Minutes - Net Downtime) / Total Billable Minutes × 100
```

โดย `Total Billable Minutes` ในแอปคำนวณจากจำนวนวันในเดือน × 24 ชั่วโมง × 60 นาที

## เกณฑ์ SLA เดือนตุลาคม

ในหน้าเว็บเปิดเมนู **SLA ตุลาคม** เพื่อดูว่า Downtime ต้องเกินเท่าใดจึงเริ่มต่ำกว่า SLA และใช้ตัวตรวจสอบเบื้องต้นได้ทันที

1. เลือกโปรไฟล์ SLA ที่ตรงกับบริการและรูปแบบ deployment ของคุณ
2. กรอก Net Downtime เป็นหน่วยนาทีหลังหัก Planned / Scheduled Maintenance ที่เข้าเงื่อนไข
3. กดปุ่มตรวจเกณฑ์ เพื่อเปรียบเทียบ Monthly Uptime กับ commitment ของเดือนตุลาคม
4. ก่อนยื่นเคลม ให้ยืนยัน service, tier, architecture, วิธีวัด และ exclusions จาก SLA ของ resource นั้น

- เดือนตุลาคมมี 44,640 นาที; SLA 99.9% จะต่ำกว่า commitment เมื่อ Downtime เกิน 44 นาที 38.4 วินาที (หากวัดเป็นจำนวนนาที ให้ใช้ 45 นาทีขึ้นไป)
- Microsoft 365, Dynamics 365 และ Intune ในโปรไฟล์มาตรฐานใช้ 99.9% และระดับเครดิต 25% / 50% / 100% เมื่อ Uptime ต่ำกว่า 99.9% / 99% / 95% ตามลำดับ
- Azure ไม่มีค่า SLA เดียวสำหรับทุกบริการ; ตารางนี้ใช้ตัวอย่างโปรไฟล์ 99.9%, 99.95% และ 99.99% จากบริการที่ระบุในเอกสาร ซึ่งขึ้นกับ service, tier และ architecture

ข้อมูลในตารางอ้างอิงจากเอกสาร Microsoft Online Services SLA (WW) ฉบับ October 2026 ต้องตรวจสอบหัวข้อ SLA ของ service/resource ที่ใช้จริงก่อนยื่นคำขอ เนื่องจากบางบริการวัดด้วย request/error rate หรือมีเงื่อนไขเฉพาะ ค่า 99.999% พร้อมเครดิต 5% ที่พบในเอกสารเป็นเงื่อนไขเฉพาะของ Teams Calling/Phone/Audio Conferencing จึงไม่รวมเป็น Azure profile ทั่วไป

## ข้อควรระวัง

- SLA ของแต่ละ Microsoft service, service resource, service tier และรูปแบบการซื้ออาจมีนิยาม Downtime, เงื่อนไข, exclusions และอัตรา Service Credit แตกต่างกัน
- บางบริการใช้การวัดจาก valid requests, กำหนดให้มี retry หรือมีเงื่อนไขด้าน configuration ก่อนจึงจะเข้าเกณฑ์ SLA
- เครดิตอาจคิดเฉพาะบริการหรือทรัพยากรที่ได้รับผลกระทบ ไม่ใช่ค่าบริการทั้งชุด
- ควรเก็บ Incident ID, เวลาเริ่มและสิ้นสุดพร้อม timezone, error logs, หลักฐาน retry และ Support Ticket ก่อนส่งคำขอ
- ตรวจสอบกำหนดเวลายื่นเรื่องและเอกสาร SLA ฉบับล่าสุดก่อนดำเนินการ

## แหล่งอ้างอิง

- [Service Level Agreements (SLA) for Online Services — Microsoft Online Services SLA (WW), October 2026](https://www.microsoft.com/licensing/docs/view/Service-Level-Agreements-SLA-for-Online-Services)
- [How to read a Service-Level Agreement (SLA)](https://learn.microsoft.com/azure/reliability/concept-service-level-agreements)
- [How and when to request a credit from Microsoft — Partner Center](https://learn.microsoft.com/partner-center/billing/request-credit)
- [Report a service problem on behalf of a customer — Partner Center](https://learn.microsoft.com/partner-center/customers/report-problems-on-behalf-of-a-customer)
- [GDAP Least Privileged Roles by Task](https://learn.microsoft.com/partner-center/gdap-least-privileged-roles-by-task)
- [Partner Center — Create Credit Request (non-Azure) โดยตรง](https://partner.microsoft.com/dashboard/support/servicerequests/create?stage=2&topicid=c65fe2c7-105c-aafc-2f8c-3d3acecb2e4a)

## โครงสร้างโปรเจกต์

```text
.
├── index.html   # หน้าเว็บและ logic ของเครื่องคำนวณ
└── README.md    # เอกสารแนะนำการใช้งาน
```
