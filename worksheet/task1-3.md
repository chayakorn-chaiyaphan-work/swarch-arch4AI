# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** Chayakorn Chaiyaphan

**Student ID / รหัสนิสิต:** 6632038021

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 511 | 603 | 93.5% | 100% |
| t0-small | 22 | 49 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> large model แม่นกว่าชัดเจน (93.5% เทียบกับ 72.5%) แต่ช้ากว่ามาก และทุกรีวิวต้องเรียกโมเดลที่มีค่าใช้จ่าย

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 26.5 | 237 | 361 | 18 | 0 | 93.4% |
| t1-after | 9.3 | 398 | 1028 | 0 | 25 | 93.7% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 คิดจาก request ที่สำเร็จ เลยไม่รวม 18 request ที่ error ดูคอลัมน์ `errors` จะเห็นปัญหาชัดกว่า และ run นี้ใช้เวลาถึง 26.5 วินาที

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> request ที่ค้างยังเปิด connection และกินช่องรับงานไว้ใน gateway กับ review service คำขออื่นเลยต้องรอคิวตามไปด้วย จึงมี request อื่นช้าหรือ timeout ทั้งที่ไม่ได้เป็น call ที่ค้างเอง

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> เก็บรีวิวไว้แล้วให้ worker หรือ batch job กลับมาติด label ทีหลัง แบบนี้คือ asynchronous/batch topology (T4)

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> 0.1 วินาทีสั้นกว่าเวลาปกติของ large model ที่ราว 200 ms คำขอปกติก็อาจกลายเป็น pending ส่วน 10 วินาทีก็นานเกินไป ทำให้ connection ค้างและคิวสะสมจน client เองอาจ timeout

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 21 | 623 | 91.5% | 38% |
| t2-0.7 | 0.7 | 388 | 606 | 93.0% | 53% |
| t2-0.8 | 0.8 | 426 | 602 | 93.5% | 64% |
| t2-0.9 | 0.9 | 468 | 588 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> เลือก 0.6: accuracy ยังได้ 91.5% แต่เรียก large model 38% หรือราว 15.2 ครั้ง/วินาทีเมื่อมี 40 รีวิว/วินาที ซึ่งยังไม่เกินเพดาน 20 ครั้ง/วินาที สิ่งที่ยอมแลกคือ accuracy ที่น้อยกว่า threshold 0.8/0.9 ซึ่งได้ 93.5%

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> รีวิวส่วนใหญ่จบที่ small model เลยทำให้ p50 เหลือ 21 ms แต่กลุ่มที่ confidence ต่ำยังต้องรอ large model และคิวของมัน ค่า p95 จึงยังอยู่ที่ 623 ms

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> ในชุดทดสอบมีประโยค “Honestly the salad is not cold.” ซึ่งจริงเป็น positive แต่ small ทายเป็น negative ด้วย confidence 0.702; เพราะเกิน threshold 0.6 cascade เลยรับคำตอบนี้และติด label ผิด

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache-0.6 | 13 | 530 | 91.5% | 31% | 36% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวที่ข้อความซ้ำกันหลังปรับตัวพิมพ์กับช่องว่างจะได้ประโยชน์ ส่วนรีวิวใหม่ที่ไม่ซ้ำก็ยังต้องทำนายใหม่ รอบนี้ cache hits 36% และ large calls ลดจาก 38% เหลือ 31%; เครื่องหมายวรรคตอนที่ต่างกันยังนับเป็นคนละข้อความ

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ตอนเปลี่ยนรุ่นควรล้าง cache หรือใส่เลขรุ่นของโมเดลไว้ใน cache key ด้วย จะได้ไม่เอาคำตอบจากโมเดลเก่ามาใช้กับรุ่นใหม่ อาจตั้ง TTL ให้ข้อมูลหมดอายุเองด้วย

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> cache ช่วยมากเมื่อมีข้อความซ้ำ: รอบนี้ hits 36% ทำให้ large calls ลดจาก 38% เหลือ 31% และ p50 จาก 21 เหลือ 13 ms โดย accuracy ยัง 91.5% ถ้ารีวิวส่วนใหญ่ไม่ซ้ำ ผลที่ได้ก็คงน้อยลง
