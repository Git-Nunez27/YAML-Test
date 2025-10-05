on: push

ทำให้ Workflow รัน ทุกครั้งที่มีการ Push โค้ดไปที่ branch main

ช่วยให้ทีมสามารถตรวจสอบโค้ดที่ถูกส่งขึ้น repository ได้ทันที

on: pull_request

ทำให้ Workflow รัน เมื่อมีการสร้างหรืออัปเดต Pull Request ไปที่ branch main

ช่วยให้มั่นใจว่าโค้ดที่จะ Merge เข้าสู่ main ผ่าน การตรวจสอบ CI/CD แล้ว

## Docker Compose — คำอธิบาย (services, ports, volumes, environment)

- services:
  - หมายถึงกลุ่ม containers ที่เราต้องการรันร่วมกันเป็นส่วนหนึ่งของสแต็ก เช่น `web` และ `db` ในตัวอย่างเราใช้ `nginx` และ `mysql` แต่ละ service ระบุ image, คำสั่งรัน, network และ volumes ได้
  - ประโยชน์: จัดการหลาย container เป็นกลุ่มเดียวกัน สร้าง network ระหว่างกันได้อัตโนมัติ และทำให้สาธิตสภาพแวดล้อมการทำงานได้อย่างรวดเร็ว

- ports:
  - กำหนดการแม็ปพอร์ตระหว่าง host กับ container เช่น `8080:80` หมายถึง พอร์ต 8080 บนเครื่อง host จะถูกส่งต่อไปยังพอร์ต 80 ของ container
  - ประโยชน์: ช่วยให้บริการภายใน container เข้าถึงจากเครื่องภายนอกหรือเครื่องผู้พัฒนาได้

- volumes:
  - ใช้สำหรับเก็บข้อมูลถาวรหรือแชร์โฟลเดอร์ระหว่าง host กับ container เช่น บันทึกฐานข้อมูลหรือไฟล์ log
  - ประโยชน์: ข้อมูลยังคงอยู่แม้ container จะถูกลบ/สร้างใหม่ ช่วยให้การพัฒนาและทดสอบมีข้อมูลคงที่

- environment:
  - กำหนดตัวแปรแวดล้อม (environment variables) ให้กับ container เช่น `MYSQL_ROOT_PASSWORD` หรือ `MYSQL_DATABASE`
  - ประโยชน์: ตั้งค่าการทำงานของแอป/ฐานข้อมูล โดยไม่ต้องแก้ไข image หรือโค้ด (สามารถเก็บค่าในไฟล์ `.env` หรือใช้ secrets ใน CI)

## Docker Compose ทำงานร่วมกับ Project ยังไง

- บทบาทหลัก:
  - Docker Compose ช่วยรันสแต็กของบริการ (services) แบบโลคัลเพื่อใช้ในการพัฒนา ทดสอบ หรือสาธิตก่อน deploy จริง
  - มักใช้สำหรับสร้างสภาพแวดล้อมที่ใกล้เคียง production เช่น รัน web server, database, cache พร้อมกัน

- วงจรการใช้งานร่วมกับ Project:
  1. นักพัฒนาสร้าง/แก้ไข `docker-compose.yml` เพื่อกำหนด service ที่ต้องการ
  2. รัน `docker-compose up` บนเครื่องพัฒนา (หรือ CI runner ที่รองรับ Docker) เพื่อเริ่ม containers ทั้งหมด
  3. ทดสอบแอป locally โดยเข้าถึงผ่านพอร์ตที่แม็ปไว้ (เช่น http://localhost:8080)
  4. เมื่อผ่านการทดสอบบน Compose แล้ว ขั้นตอน CI/CD อาจสร้าง Docker image และ push ขึ้น registry
  5. จากนั้น CD จะนำ image เหล่านั้นมา deploy บน Kubernetes cluster โดยใช้ `k8s/*.yml` (หรือ Helm)

- ข้อดีเพิ่มเติม:
  - ให้สภาพแวดล้อมทดสอบที่รวดเร็วและสม่ำเสมอสำหรับทีม
  - ลดความต่างของ environment ระหว่างเครื่องพัฒนาและ CI
  - ง่ายต่อการอ่านและแชร์ setup ด้วยไฟล์ YAML เดียว

(เพิ่มได้: ถ้าต้องการ ผมสามารถเพิ่มตัวอย่าง `volumes` สำหรับ `db` หรือตัวอย่าง `.env` file เพื่อแสดง pattern การจัดการ secret/credential แบบปลอดภัย)

## Kubernetes — คำอธิบาย (Deployment, replicas, selector, template)

- Deployment:
  - เป็น controller ที่ใช้จัดการการรันชุดของ Pods แบบ declarative (กำหนด desired state แล้ว Kubernetes จะพยายามทำให้ตรง)
  - มีหน้าที่เช่น สร้าง/ลบ/อัปเดต Pods, ทำ rolling update, และทำ self-healing (restart/replace เมื่อพ็อดล้ม)
  - ในโปรเจคนี้ใช้ `k8s/deployment.yml` เพื่อประกาศ Deployment ของ `nginx`

- replicas:
  - ระบุจำนวนสำเนาของ Pod ที่ต้องการให้รันพร้อมกัน เช่น `replicas: 2` หมายถึงต้องการให้มี 2 พ็อดของ nginx เพื่อเพิ่ม availability และรองรับโหลด

- selector:
  - เป็นกลไกการจับคู่ (label selector) ที่บอกว่า Deployment หรือ Service ควรทำงานกับ Pods ใด เช่น `matchLabels: { app: nginx }`
  - เมื่อ Deployment สร้าง Pods จะใส่ labels ตามที่กำหนดไว้ใน `template.metadata.labels` เพื่อให้ selector จับคู่ได้

- template:
  - คือ Pod template (metadata + spec) ที่ Deployment ใช้เป็นต้นแบบในการสร้าง Pods ใหม่
  - ภายใน template จะกำหนด container image, ports, env, volume mounts เป็นต้น

## Service — คำอธิบาย (Service, port, targetPort)

- Service:
  - เป็น abstraction ที่ให้การเข้าถึงกลุ่ม Pods โดยมี IP และพอร์ตที่คงที่ (virtual) แม้ Pods ภายในจะขึ้นลงหรือเปลี่ยนแปลง
  - Service จะใช้ `selector` เพื่อเลือก Pods ที่จะเป็น backend (Endpoints)
  - มีชนิดต่างๆ เช่น `ClusterIP` (ภายใน cluster), `NodePort` (เปิดพอร์ตบนทุก Node), `LoadBalancer` (ขอ LB จาก cloud provider)

- port:
  - พอร์ตที่ Service เปิดให้ผู้เรียกติดต่อ (เช่น Service port = 80)
  - ผู้เรียกภายนอกหรือภายใน cluster จะติดต่อไปที่พอร์ตนี้ของ Service

- targetPort:
  - พอร์ตบน container/pod ที่ Service จะส่ง traffic เข้าไป (เช่น containerPort = 80)
  - Service จะ map `port` → `targetPort` เพื่อส่งคำขอไปยัง container ที่แท้จริง

- nodePort (เมื่อใช้ NodePort):
  - พอร์ตบน Node ที่เปิดให้เข้าถึง Service จากภายนอก (เช่น ใน `k8s/service.yml` เรากำหนด `nodePort: 30080`)
  - การเรียก http://<node-ip>:30080 จะถูกส่งไปยัง Service แล้วกระจายไปยัง Pods ที่ถูก selector จับ

## ความสัมพันธ์ระหว่าง Deployment และ Service

- Deployment สร้างและจัดการ Pods ตาม `template` และ `replicas` ที่กำหนด โดยใส่ `labels` ให้กับ Pods เหล่านั้น
- Service ใช้ `selector` เดียวกันกับ label ของ Pods เพื่อชี้ไปยัง Pods ที่ต้องการรับ traffic
- ผลลัพธ์: Service ให้ endpoint คงที่ (IP:port) สำหรับเข้าถึงแอป แม้จำนวนหรือตัวตนของ Pods จะเปลี่ยนไปจากการ scale หรือ rollout
- การไหลของ traffic (สรุป):
  1. ผู้เรียกติดต่อไปที่ Service (ClusterIP / NodePort / LoadBalancer)
  2. Service ใช้ selector หาค่า Endpoints (IPs ของ Pods ที่ match)
  3. Service ส่ง traffic ไปยังหนึ่งใน Pods (ผ่าน kube-proxy หรือ IPVS) โดย map `port` → `targetPort`

(เพิ่มเติม: ถ้าต้องการ ผมสามารถเพิ่มตัวอย่าง readiness/liveness probe ใน `k8s/deployment.yml` เพื่อแสดงการควบคุม lifecycle และการทำ health check ได้)