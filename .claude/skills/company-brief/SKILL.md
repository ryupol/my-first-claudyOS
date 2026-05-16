---
name: company-brief
description: Use when the user asks for a research brief on a public stock (e.g. "/brief AAPL", "ทำ brief NVDA ให้หน่อย", "research TSLA"). Outputs a 6-section markdown brief saved to briefs/<TICKER>.md.
---

# company-brief SOP

## When to use this

ผู้ใช้ขอ research brief ของหุ้น 1 ตัว Trigger ทั่วไป:
- `/brief <TICKER>` slash command
- "ทำ brief หุ้น X ให้หน่อย"
- "ขอข้อมูลย่อๆ ของ <ticker>"

## Inputs you need

- 1 stock ticker (เช่น AAPL, NVDA, GOOGL)
- ถ้าไม่มี ticker ให้ ask before doing anything else

## Steps

1. Confirm the ticker. ถ้า ambiguous (เช่น "META" อาจหมายถึงหลายบริษัทใน history) ask user to confirm
2. Read `CLAUDE.md` ที่ root ของ project ใน CLAUDE.md จะมีย่อหน้า investing voice ของ user, output ต้องสะท้อนสไตล์นั้น (focus, framing, kill-condition discipline)
3. Dispatch 3 named sub-agents ตามลำดับ (sequential, context isolated) แต่ละตัว return output ตรงกลับมาที่ orchestrator:

   ```
   Agent({
     name: "fundamentals-reader",
     description: "Read 10-K for <TICKER>",
     prompt: "You are fundamentals-reader. Read sources/<TICKER>/10-k-*.md and return: (1) Company snapshot, (2) Fundamentals signal, (3) Top risks. Rules: use ONLY data in the file, if file missing say so, no blockquotes with verbatim quotes, no training memory."
   })

   Agent({
     name: "call-analyst",
     description: "Read earnings call for <TICKER>",
     prompt: "You are call-analyst. Read sources/<TICKER>/q*-call.md and return: (1) Quarter numbers, (2) Guidance, (3) Management commentary. Rules: use ONLY data in the file, if file missing say so, no blockquotes with verbatim quotes, no training memory."
   })

   Agent({
     name: "market-pulse",
     description: "Search recent news for <TICKER>",
     prompt: "You are market-pulse. Use WebSearch to find news from the last 7 days for <TICKER>. Return: (1) Recent news headlines, (2) Analyst moves (upgrades/downgrades/PT changes), (3) Upcoming catalysts. Rules: report only observable signals, no market predictions, no 'market hasn't priced in', if no news found say so, no blockquotes."
   })
   ```

4. Integrate output จากทั้ง 3 agents เป็น brief 6 sections ตาม Output format ด้านล่าง:
   - fundamentals-reader → feeds Section 1 (Company snapshot) + Section 2 (Fundamentals signal) + Section 4 (Bull/Bear risks) + Section 5 (Kill conditions)
   - call-analyst → feeds Section 3 (Latest earnings)
   - market-pulse → feeds Section 4 (Bull/Bear catalysts) + Section 6 (What to ask)
   - ถ้า agent ตัวใดบอก "source not found" ให้ระบุตรงๆ ใน section ที่เกี่ยว ห้ามแต่งแทน
5. ถ้า folder `briefs/` ยังไม่มี ให้สร้าง
6. Save brief ที่ `briefs/<TICKER>.md` (uppercase ticker)
7. แสดง brief เต็มกลับใน chat ด้วย

## Output format (6 sections, required, no skipping)

ใช้ markdown headings ทั้ง 6 section ต้องมี ครบทุก brief

### 1. Company snapshot (3-4 ประโยคไทย)
บริษัททำอะไร, ขายให้ใคร, รายได้หลักมาจากไหน ภาษาคนปกติ ไม่เอาคำตลาด

### 2. Fundamentals signal (3-5 bullets)
Revenue trend, margin trend, balance sheet feel, capital allocation pattern **เน้น direction มากกว่าตัวเลข** เพราะตัวเลขเฉพาะอาจเก่า ถ้า ratio/margin specific ที่ไม่แน่ใจ ให้ใส่ "(ตัวเลข ตรวจสอบใน 10-K ล่าสุด)" ต่อท้าย

### 3. Latest earnings
3-5 bullets **Source:** อ่านทุกไฟล์ใน sources/<TICKER>/ ก่อนเขียน ถ้า folder ว่างหรือไม่มี เขียนตรงๆ: "ไม่มี earnings transcript ใน sources/<TICKER>/ skip section นี้หรือ user ใส่ source ก่อน" ห้ามแต่งตัวเลขจากความจำ ทุก bullet ในนี้ต้อง trace กลับไปที่ไฟล์ใน sources/ ได้ และระบุไฟล์ต้นทางใน parens ท้าย bullet เช่น (source: sources/AAPL/q1-2026-call.md)

### 4. Bull case / Bear case
2-3 bullets แต่ละข้าง Bear case ต้อง substantive ไม่ใช่ "เศรษฐกิจไม่ดี" ต้องเป็นเหตุผลที่ specific to บริษัทนี้

### 5. Kill conditions (สำคัญ อย่าข้าม)
2-3 bullets "ถ้าเห็นอะไรเกิดขึ้น ผม/คุณควรเลิกถือ" ตัวอย่าง: "margin ลดลง 3 quarter ติด", "ลูกค้า top-3 หายไป 1 ราย", "CEO ออก + replacement weak" Kill conditions เป็นข้อเดียวที่กันให้ thesis ไม่กลายเป็น religion

### 6. What to ask before owning it (3-5 questions)
คำถามที่ beginner ควรตอบให้ได้ก่อนกดซื้อ ไม่ใช่คำตอบ เป็น question prompt

## Voice rules

- Tone reflect investing voice ใน `CLAUDE.md` ของ project ถ้า CLAUDE.md บอก "long-term focus" ห้าม brief เน้น short-term trading angle
- **ห้าม** ออก buy/sell recommendation นี่คือ research summary ไม่ใช่คำแนะนำ
- **ห้าม** แต่ง verbatim quote ของ executive ใส่ blockquote ถ้าจะอ้างคำผู้บริหาร ใช้ indirect speech ("CEO กรอบ message ว่า...") ห้ามใส่ `>` quote ที่ verify ไม่ได้
- **ห้าม** ใช้คำว่า "moat" ตรงๆ ใช้ Helmer's 7 Powers ที่ specific (Scale Economies, Network Economies, Switching Costs, Branding, Counter-Positioning, Cornered Resource, Process Power) ถ้าจะพูดเรื่องความได้เปรียบ
- **ห้าม** บอกว่า "ตลาดยังไม่ price in" หรือทำนายว่านักลงทุนคนอื่นคิดอะไร

## When unsure

Honest > confident ถ้าข้อมูลไม่พอ พูดว่า "ผมไม่แน่ใจ ลองดูใน [source ที่ user ใช้]" ดีกว่าแต่ง
