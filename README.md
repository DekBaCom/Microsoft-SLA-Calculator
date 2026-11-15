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
- **ประเภทใน Service Health** ที่ใช้เคลมได้ / ไม่ได้ — จำแนก Incident vs Advisory, Issue Origin, Status และ Azure event types (Service issues, Planned maintenance, Health/Security advisories, Billing updates) พร้อมตารางตัดสินใจ
- **Workflow Request แยกตามประเภทสัญญา** — เส้นทาง portal, ผู้ยื่น, บทบาท, รูปแบบเครดิต และข้อควรระวังสำหรับ Direct / CSP / EA
- สรุปช่วงเวลาสำหรับยื่นคำขอ: Azure ภายใน 60 วันนับจาก Incident; บริการอื่นภายในสิ้นสุด Applicable Period ถัดจากเดือนที่เกิด Incident
- รวมลิงก์เอกสารอ้างอิงจาก Microsoft และ Microsoft Learn

### ประสบการณ์ใช้งาน (UX / UI)

- **ปรับขนาดอัตโนมัติ** รองรับทั้งมือถือ แท็บเล็ต และเดสก์ท็อป ด้วย fluid typography (`clamp()`) และ grid แบบ `auto-fit/minmax()`
- **สองภาษา (ไทย / English)** ปุ่มสลับภาษา (EN / ไทย) ที่มุมบนขวาข้างปุ่ม theme — ไทยเป็น default, จำค่าใน `localStorage` key `sla-lang` และตั้ง `html[data-ui-lang]` เพื่อสลับผ่าน CSS (`.lang-th` / `.lang-en`); ขั้นแรกแปลครอบคลุม header, top/side nav, h2 ทุก section, ปุ่มหลัก, footer และข้อความใน JS (alerts/status) — เนื้อหารายละเอียดเชิงลึกยังเป็นไทยและเพิ่ม EN ได้ incremental
- **Dark Mode** พร้อมปุ่มสลับที่มุมบนขวา (จำค่าใน `localStorage` และ auto-detect `prefers-color-scheme`)
- **Sticky Nav** แบบ glassmorphism (backdrop-filter) ที่เลื่อนแนวนอนบนมือถือและ wrap ตรงกลางบนเดสก์ท็อป
- **Floating Side-Nav** แถบนำทางด้านขวาแบบ **text pills** ที่ปรากฏเมื่อเลื่อนลงเกิน 320px — แสดงชื่อ section ทุกอันตลอดเวลา (ไม่ใช่แค่ hover) และ pill ของ section ที่กำลังดูจะเป็น **สีน้ำเงินเข้มแบบนูน** (dark gradient + layered inset/drop shadow + glow bar ด้านซ้าย + pulsing ring + bold text-shadow) sync กับ top nav อัตโนมัติผ่าน `IntersectionObserver` และซ่อนบนจอ ≤ 960px
- **Accessibility**: `:focus-visible` ring ชัดเจน, รองรับ `prefers-reduced-motion`, `aria-label` บนปุ่ม toggle, side-nav และปุ่ม social
- **Social**: ปุ่ม LinkedIn แบบ gradient ที่ footer เปิดลิงก์ไปยังโปรไฟล์ผู้พัฒนาใน tab ใหม่
- **Print-friendly**: ปุ่ม, toggle และ side-nav ซ่อนอัตโนมัติเมื่อสั่งพิมพ์

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

## ประเภทใน Service Health ที่ใช้เคลม SLA ได้

ในหน้าเว็บเปิดเมนู **ประเภทใน Service Health** เพื่อดูการจำแนกเหตุการณ์แบบละเอียด — ไม่ใช่ทุก event ที่แสดงใน Service Health จะเคลม SLA Credit ได้ ต้องแยก **ประเภท (type)**, **ต้นตอ (issue origin)** และ **สถานะ (status)** ก่อน

### Microsoft 365 / Dynamics 365 / Intune

- **Incident** — บริการหรือฟังก์ชันหลักใช้งานไม่ได้ ✅ เคลมได้ (ถ้า uptime ต่ำกว่า SLA)
- **Advisory** — รับรู้ปัญหาแต่บริการยังใช้งานได้ ⚠️ ปกติไม่เข้าเกณฑ์ ต้องพิสูจน์ user impact จริง
- **Issue Origin**: Microsoft infrastructure ✅ เคลมได้; Third-party หรือ Customer infrastructure ❌ ไม่ได้
- **Status ที่เป็นหลักฐานได้**: Service degradation, Service interruption, Restoring service, Extended recovery, Service restored, Post-incident report published
- **Status ที่ตัดสิทธิ์**: False positive
- Planned maintenance ของ M365 **ไม่แสดง**ใน Service Health แต่อยู่ใน **Message Center** หมวด *Plan for change*

### Azure Service Health — 5 ประเภท Event

| Event Type | เคลม SLA ได้? | หมายเหตุ |
| --- | --- | --- |
| **Service issues** | ✅ Error / Warning เข้าเกณฑ์ | ตรวจ `properties.EventType = ServiceIssue` |
| **Planned maintenance** | ❌ ไม่ได้ — SLA exclusion | หักออกจาก Downtime ถ้ามี |
| **Health advisories** | ❌ ไม่ได้ — เป็นคำแนะนำ | เช่น deprecation, retirement |
| **Security advisories** | ❌ ไม่ได้ — การแจ้งเตือน | อาจกระทบ availability แต่ไม่เข้า SLA |
| **Billing updates** | ❌ ไม่เกี่ยวกับ SLA | - |

ตรวจเพิ่มได้ผ่าน **Azure Resource Graph** ที่ตาราง `ServiceHealthResources` field `properties.EventType` และ `properties.Status`

### ข้อมูลที่ควรบันทึกจาก Service Health ทุก event

Tracking/Incident ID, Event Type, Issue Origin, Status, Impact start/mitigation time, Affected services & regions, Scope (subscription/tenant), ลิงก์ไปยัง PIR (ถ้ามี)

## Workflow Request — แยกตามประเภทสัญญา (Direct / CSP / EA)

ในหน้าเว็บเปิดเมนู **Workflow: Direct/CSP/EA** เพื่อดูรายละเอียดเต็ม วิธียื่น SLA Credit ขึ้นกับ **Licensing Model** ของ subscription ที่ได้รับผลกระทบ

### ตรวจประเภทสัญญาก่อน

- ซื้อตรงจาก microsoft.com / admin.microsoft.com / portal.azure.com → **Direct** (Web Direct / MCA)
- ซื้อผ่าน Partner ใบเสร็จออกโดย Partner → **CSP**
- มี Enrollment Number, สัญญา EA/EAS/SCE 3 ปี → **EA (Enterprise Agreement)**
- ไม่แน่ใจ → ตรวจที่ *Azure Portal → Cost Management + Billing → Billing scopes* ดู Account type

### เปรียบเทียบ 3 รูปแบบ

| หัวข้อ | Direct | CSP | EA |
| --- | --- | --- | --- |
| ผู้ยื่น | ลูกค้าโดยตรง (Admin) | Partner (Direct/Indirect Provider) | Enterprise Admin / Partner Admin (indirect) |
| Portal หลัก | Azure Portal / M365 Admin Center | Partner Center (+ Azure portal ผ่าน context) | Azure Portal (billing scope: EA) / VL Central |
| บทบาทที่ต้องใช้ | Billing/Global/Service Support Admin | Admin agent + GDAP (+ RBAC สำหรับ Azure) | Enterprise Admin / Account Owner |
| Topic / Category | Billing → Refund request | Cancel, Credit or suspend (non-Azure) | Billing → Refund request + ระบุ Enrollment |
| รูปแบบเครดิต | หัก invoice ถัดไป | Invoice + Reconciliation; Partner ออกให้ลูกค้าต่อ | **Future Azure usage credit** (Code O21), ไม่ใช่เงินสด |
| เครดิตหมดอายุ | ไม่ปกติ | ไม่ปกติ | **สิ้น enrollment anniversary term** |
| Deadline (Azure) | 2 เดือน | 2 เดือน | 2 เดือน |
| Deadline (M365/อื่น) | 1 เดือน | 1 เดือน | 1 เดือน |
| ครอบคลุม commerce | Legacy + NCE | Legacy + NCE | EA / EAS / SCE |

### EA — สิ่งที่ต่างจาก Direct/CSP

- **Service Credit ≠ cash refund** — เป็นเครดิตใช้กับ Azure consumption ในอนาคตเท่านั้น
- **Expire เมื่อจบ enrollment anniversary term** ปัจจุบัน (ยื่นช้า → อาจใช้ไม่ทัน)
- ปรากฏใน usage summary ด้วย **Accounting Code `O21`** — *Service Level Agreement Credit*
- **EA Indirect**: ลูกค้ายื่นเองไม่ได้ ต้องให้ **LSP / SSP Partner Administrator** เป็นผู้ยื่นผ่าน VL Central หรือ Microsoft Account Team

### Pitfalls ที่พบบ่อย

- CSP **Indirect Reseller** ยื่นเองใน Partner Center → ถูก reject (ต้องให้ Indirect Provider ยื่น)
- EA **Indirect** ลูกค้ายื่นเองใน Azure Portal → เคสถูกส่งกลับให้ติดต่อ LSP/SSP
- EA ยื่นช้าจน credit หมดอายุก่อนได้ใช้ (anniversary term ครบ)
- Direct ใช้ personal email เป็น evidence → ถูก reject ต้องเป็น email จาก tenant domain
- ใช้ Advisory incident เป็นหลักฐานหลัก → ปกติไม่เข้าเกณฑ์

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
- [How to check Microsoft 365 service health — Incident vs Advisory, statuses](https://learn.microsoft.com/microsoft-365/enterprise/view-service-health)
- [Azure Service Health portal — 5 event types](https://learn.microsoft.com/azure/service-health/service-health-portal-update)
- [Azure Service Health notifications — properties & severities](https://learn.microsoft.com/azure/service-health/service-health-notifications-properties)
- [Azure EA Pricing — SLA credit process](https://learn.microsoft.com/azure/cost-management-billing/manage/ea-pricing-overview)
- [EA Usage & credit charges — accounting codes (O21 = SLA Credit)](https://learn.microsoft.com/azure/cost-management-billing/manage/direct-ea-azure-usage-charges-invoices)
- [VL Central — Credits and transfers (EA / Volume Licensing)](https://learn.microsoft.com/volume-licensing-central/orderworkspace/creditstransfers)

## โครงสร้างโปรเจกต์

```text
.
├── index.html   # หน้าเว็บและ logic ของเครื่องคำนวณ
└── README.md    # เอกสารแนะนำการใช้งาน
```
