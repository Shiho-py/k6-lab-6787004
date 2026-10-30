# K6 lab results

Student ID: 6787004

- Part 0: [step-0.1.png](reports/step-0.1.png), [step-0.2.png](reports/step-0.2.png), [step-0.3.png](reports/step-0.3.png), [step-0.4.png](reports/step-0.4.png), [step-0.5.png](reports/step-0.5.png), [step-0.6-1.png](reports/step-0.6-1.png), [step-0.6-2.png](reports/step-0.6-2.png), [step-0.7-1.png](reports/step-0.7-1.png), [step-0.7-2.png](reports/step-0.7-2.png), [step-0.8-1.png](reports/step-0.8-1.png), [step-0.8-2.png](reports/step-0.8-2.png), [step-0.8-explore.png](reports/step-0.8-explore.png)
- Smoke: [a1-smoke.png](reports/a1-smoke.png)
- HTML reports: [a2-average.html](reports/a2-average.html), [a3-stress.html](reports/a3-stress.html), [a4-spike.html](reports/a4-spike.html), [a5-soak.html](reports/a5-soak.html)

## Part 0

- Step 0.1: http_req_duration = **272.71 ms** จาก 1 request และ check ผ่าน 1/1
- Step 0.2: **80 requests, 3.76289 req/s, p(95)=291.69 ms** และ rate ต่ำกว่า 5 เพราะแต่ละ VU รอ response แล้วพัก 1 วินาทีก่อนรอบถัดไป
- Step 0.3: 5 VUs แบ่งกันทำครบ **20 iterations** ในประมาณ 5.8 วินาที แล้วหยุด ต่างจาก duration ที่กำหนดเวลารัน
- Step 0.4: API ตอบ **200** มีชื่อพิซซ่าและส่วนผสมเป็นมังสวิรัติทั้งหมด checks ผ่าน **3/3**, duration=339.98 ms
- Step 0.5: ไม่มี token ได้ **401** ตามที่คาดไว้และ check ผ่านโดย `expectedStatuses(401)` ทำให้ http_req_failed เป็น **0%**
- Step 0.6: เกณฑ์ 1500 ms ผ่าน (p95=401.1 ms, **exit 0**) แต่เกณฑ์ 300 ms ไม่ผ่าน (p95=383.14 ms, **exit 99**)
- Step 0.7: 3 VUs/30s ได้ **p95=399.81 ms, p99=412.88 ms, failed=0%** ผ่านครบ 3 rules แต่เปลี่ยน p95<1000 เป็น <300 ms แล้วไม่ผ่าน (384.7 ms, **exit 99**)
- Step 0.8: แก้ `pepperoni`/`knife` ให้ตรง API เป็น `Pepperoni`/`Knife` แล้ว checks ผ่าน **225/225** การส่งซ้ำพบคำตอบผิดเงื่อนไขที่ลองครั้งเดียวไม่พบ

## Part A results

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | --------- | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.422945 | 395.14 ms | 474.45 ms | 0% | 100% (20/20) |
| A2 | Average load | 3 | 1.296425 | 374.17 ms | 398.57 ms | 0% | 100% (316/316) |
| A3 | Stress | 5 | 2.165343 | 372.37 ms | 397.75 ms | 0% | 100% (528/528) |
| A4 | Spike | 5 | 1.060829 | 376.44 ms | 393.68 ms | 0% | 100% (200/200) |
| A5 | Soak | 2 | 0.985966 | 370.56 ms | 653.78 ms | 0% | 100% (868/868) |

### สรุป Part A

- A1: 1 VU/30s ทำครบ 5 journeys และ checks ผ่าน 20/20 จึงพร้อมเพิ่มโหลด
- A2: เพิ่มถึง 3 VUs ใน 30s คงไว้ 2m แล้วลดลง 30s; ผ่านครบ 4 thresholds และได้ p(95)=374.17 ms
- A3: เพิ่มถึง 5 VUs ด้วยรูปแบบเดียวกับ A2; p(95)=372.37 ms จึงไม่พบเวลาตอบสนองรวมเพิ่มขึ้นในรอบนี้
- A4: เพิ่ม 1→5 VUs ใน 5s แล้วลดกลับ รวม 2m20s; p(95) ก่อน/ช่วง spike=328.01/373.26 ms และยังระบุเวลา recovery แน่นอนไม่ได้
- A5: 2 VUs รวม 11m; p(95) ช่วงต้น/ท้าย=372.35/366.14 ms และ failed=0% ทั้งสองช่วง จึงไม่พบแนวโน้มแย่ลง

ทุกรอบ exit code=0; A2–A5 สร้าง HTML แล้ว แต่ยังไม่ได้เปิดตรวจกราฟใน HTML โดย A4–A5 วิเคราะห์ข้อมูลตามเวลาระหว่างรันแล้ว

## Part B

1. ตารางผล A1–A5: ดูตาราง Part A ด้านบน
2. แต่ละ VU รอ response และพัก 1s จึงใช้เวลาเฉลี่ย 1.31s/รอบ ทำให้ 5 VUs ส่งได้ **3.76289 req/s** ซึ่งต่ำกว่า 5 req/s
3. p(95) จาก A2 **374.17** เป็น A3 **372.37 ms** จึงไม่พบการเพิ่มขึ้น สอดคล้องกับโหลดที่ยังอยู่ใน operational envelope แต่ยังระบุขีดจำกัดสูงสุดไม่ได้
4. p(95) ก่อน/ช่วง spike=**328.01/373.26 ms**; หลังลดโหลดไม่พบช่วง 10s ที่กลับถึง baseline ภายใน 60s ที่ติดตาม จึงยังระบุ recovery แน่นอนไม่ได้และมีตัวอย่างก่อน spike เพียง 14 requests
5. Soak มี p(95) ต้น/ท้าย=**372.35/366.14 ms**, failed=0% จึงไม่พบแนวโน้มเสื่อมในรอบนี้; การรันหลายชั่วโมงอาจพบ memory leak ที่สะสมจนระบบช้าหรือเกิดข้อผิดพลาด
6. p(95) สะท้อนเวลาตอบสนองของ requests บริเวณเปอร์เซ็นไทล์ที่ 95 ซึ่งค่าเฉลี่ยอาจซ่อนไว้ เช่น A5 มี avg=292.48 ms แต่ p(95)=370.56 ms และ Max=653.78 ms
7. ผลต่างจากห้องแลปได้จาก **latency/เส้นทางอินเทอร์เน็ต** และ **ทรัพยากรหรือโหลดจากผู้ใช้อื่นบน public site** ที่ควบคุมไม่ได้
8. โหลดหลักร้อยอาจทำให้บริการช้าหรือล่มและกระทบผู้ใช้จริง จึงต้องได้รับอนุญาตและกำหนดขอบเขตก่อน; แลปนี้จำกัด QuickPizza ไม่เกิน 5 VUs
9. ตัวอย่าง LINE: จาก “เปิดแชตต้องเร็ว” เป็น “API ประวัติแชต 50 ข้อความที่ 3 VUs/30s พัก 1s/รอบ ต้อง p95<800 ms, p99<1500 ms, failed<1%”; options: `vus: 3, duration: '30s', thresholds: { http_req_duration: ['p(95)<800', 'p(99)<1500'], http_req_failed: ['rate<0.01'] }` (ตัวอย่างเสนอสำหรับระบบทดสอบที่อนุญาต ไม่ใช่ SLA จริงหรือผลทดสอบ LINE)
