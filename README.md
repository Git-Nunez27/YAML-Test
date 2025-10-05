# Project: CI/CD ด้วย YAML (GitHub Actions, Docker Compose, Kubernetes)

โครงงานตัวอย่างนี้แสดงการจัดวางไฟล์ YAML หลายไฟล์ในโปรเจคเดียว โดยประกอบด้วย:

- `.github/workflows/ci.yml` - GitHub Actions workflow สำหรับ CI (Lint → Build → Notify)
- `docker-compose.yml` - กำหนด Docker Compose สำหรับรัน nginx และ mysql
- `k8s/deployment.yml` - Kubernetes Deployment สำหรับ nginx (2 replicas)
- `k8s/service.yml` - Kubernetes Service แบบ NodePort

ส่วนถัดไปตอบคำถามตามโจทย์เป็นภาษาไทย

## โจทย์ 1: GitHub Actions Workflow

ไฟล์: `.github/workflows/ci.yml`

สรุปการทำงาน
- Trigger: `on: push` และ `on: pull_request` ทั้งคู่ถูกจำกัดให้ทำงานเฉพาะเมื่อ branch เป็น `main` เท่านั้น
- jobs:
  - Checkout Repository: ใช้ `actions/checkout` เพื่อดึงซอร์สโค้ด
  - Lint YAML: ตรวจ syntax ของทุกไฟล์ .yml/.yaml, ตรวจ Docker Compose (docker-compose config), ตรวจ Kubernetes แบบ dry-run (kubectl apply --dry-run=client)
  - Build Application: ตัวอย่างเป็น `echo "Build successful"`
  - Notify: ตัวอย่างเป็น `echo "Workflow finished"`

ทำไมต้องใช้ `on: push` และ `on: pull_request`?
- `push`: จะรันเมื่อมีการ push โค้ดไปยัง repository (เช่น commit/merge ไปยัง `main`) — ใช้สำหรับการทดสอบหลังจากโค้ดถูกรวมเข้ากับสาขาหลัก
- `pull_request`: จะรันเมื่อมีการเปิดหรืออัพเดต pull request ที่เป้าหมายเป็น `main` — ช่วยตรวจสอบก่อน merge เพื่อป้องกันข้อผิดพลาด

jobs และ steps ทำหน้าที่อะไร?
- jobs: เป็นชุดของงานที่รันแบบแยกกัน (โดยปกติจะรันขนานกันได้ ยกเว้นมี `needs:` ระบุขึ้นต่อ)
- steps: แต่ละ job ประกอบด้วยหลาย steps เป็นลำดับขั้นตอนที่รันภายใน runner เดียวกัน เช่น เช็คเอาท์, ติดตั้ง dependency, รันทดสอบ

## โจทย์ 2: Docker Compose

ไฟล์: `docker-compose.yml`

มี services:
- web: image `nginx:latest`, เปิด port ภายนอก 8080 มายัง container port 80
- db: image `mysql:5.7`, กำหนด environment variables สำหรับ `MYSQL_ROOT_PASSWORD` และ `MYSQL_DATABASE` และ expose พอร์ต 3306

อธิบายคำศัพท์:
- services: คือ containers หรือ groups ของ containers ที่ต้องการรันร่วมกัน ภายใน compose จะกำหนด image, network, volume, env ได้
- ports: แม็ปพอร์ตของโฮสต์ไปยังพอร์ตของ container (เช่น `8080:80` หมายถึง โฮสต์พอร์ต 8080 → container 80)
- volumes: (ในตัวอย่างนี้ไม่ได้ใช้) ใช้เก็บข้อมูลถาวรหรือแชร์โฟลเดอร์ระหว่าง host กับ container
- environment: กำหนดตัวแปรสภาพแวดล้อมให้ container (เช่น รหัสผ่าน, ชื่อฐานข้อมูล)

Docker Compose ทำงานร่วมกับ Project ยังไง?
- Docker Compose ช่วยรันสแต็กแบบโลคัลสำหรับการพัฒนาและทดสอบ เช่น รัน nginx + mysql พร้อมกัน เสมือนสภาพแวดล้อมขนาดเล็กก่อนจะนำไป deploy จริง

## โจทย์ 3: Kubernetes YAML

ไฟล์: `k8s/deployment.yml` และ `k8s/service.yml`

deployment.yml: สร้าง Deployment ของ nginx โดยมี `replicas: 2` เพื่อให้มี 2 พ็อดทำงานพร้อมกัน
service.yml: สร้าง Service แบบ `NodePort` โดยแม็ป `nodePort: 30080` เพื่อให้เข้าถึงเว็บจากโหนดได้

อธิบายคำศัพท์:
- Deployment: เป็น controller ใน Kubernetes ที่จัดการการรันพ็อด (Pods) และการอัพเดตแบบ declarative
- replicas: จำนวนสำเนาของพ็อดที่ต้องการให้รัน (สำหรับเพิ่ม availability)
- selector: เงื่อนไขสำหรับ Deployment/Service ในการจับคู่พ็อด (เช่น labels)
- template: เป็น template ของพ็อดที่จะถูกสร้าง (metadata + spec ของ container)

- Service: ออบเจ็กต์ที่ให้การเข้าถึงชุดพ็อดผ่าน network abstraction (ClusterIP / NodePort / LoadBalancer)
- port: พอร์ตของ Service ที่ผู้เรียกจะติดต่อ
- targetPort: พอร์ตที่ Service จะส่ง traffic เข้าไปยัง container/pod

ความสัมพันธ์ระหว่าง Deployment และ Service:
- Deployment สร้างพ็อดที่มี label (`app: nginx`) และ Service ใช้ selector เดียวกันเพื่อชี้ไปยังพ็อดเหล่านั้น Service จะส่ง traffic ไปยัง targetPort ของพ็อด

## โจทย์ 4: รวม YAML หลายไฟล์ และ Workflow ตรวจสอบทั้งหมด

การทำงานร่วมกันของไฟล์:
- `.github/workflows/ci.yml` ตรวจสอบความถูกต้องของ YAML ทั้งหมด และตรวจ Docker Compose และ Kubernetes manifests โดยใช้ `docker/compose-action` และ `kubectl --dry-run`
- `docker-compose.yml` ใช้สำหรับรันสแต็กโลคัลของ service เช่น เมื่อทดสอบบนเครื่อง developer หรือ CI ที่รองรับ Docker
- `k8s/*.yml` ใช้สำหรับ deploy จริงบน cluster (หลังจากผ่าน CI แล้ว จะมีขั้นตอน CD เพิ่มเติมเพื่อ apply manifests ไปยัง cluster)

ลำดับขั้นตอนเมื่อ Push โค้ดไปที่ GitHub (ตัวอย่าง):
1. Push → GitHub Actions ถูก trigger (push / pull_request)
2. Workflow ทำการ Checkout และ Lint YAML ทุกไฟล์
3. ตรวจ Docker Compose (ตรวจ config)
4. ตรวจ Kubernetes manifests ด้วย dry-run
5. (ถ้าเป็น workflow ที่รวม build) ทำการ Build
6. (ในระบบจริง) ถ้าผ่านทุกอย่าง จะมีขั้นตอน CD ต่อไป เช่น สร้าง Docker image → push ไป registry → apply manifests ไปยัง cluster

## โครงสร้าง Project (ไฟล์ที่สำคัญ)
- .github/workflows/ci.yml  — GitHub Actions workflow
- docker-compose.yml      — Docker Compose สำหรับ develop/local
- k8s/deployment.yml      — Kubernetes Deployment
- k8s/service.yml         — Kubernetes Service

## Diagram ความสัมพันธ์ (ASCII)

CI/CD Pipeline

  [Developer push] -> [GitHub Actions CI] -> (Lint YAML, Check Docker Compose, K8s dry-run)
                                      |
                                      v
                            [Build / Create Docker Image]
                                      |
                                      v
                         [Push image to Registry (optional)]
                                      |
                                      v
                       [Kubernetes cluster: kubectl apply k8s/*.yml]

Docker Compose (local): docker-compose up -> รัน services (web, db) เพื่อทดสอบก่อน deploy

---

หากต้องการให้ผมเพิ่มตัวอย่างขั้นตอน CD (เช่น สร้าง image, push ไป registry, apply ไป cluster) หรือเพิ่มเทสอัตโนมัติของแอป/DB บอกได้เลย ผมจะเสริมให้
# YAML Test

