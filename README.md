GitHub Actions Workflow

on: push → รันเมื่อมีการ Push โค้ดไป branch main

on: pull_request → รันเมื่อมี Pull Request ไป branch main

jobs → งานหลักใน Workflow (เช่น Checkout, Lint, Build, Notify)

steps → ขั้นตอนย่อยในแต่ละ job (เช่น run command หรือใช้ action)

ประโยชน์: ตรวจสอบโค้ดอัตโนมัติ ลดข้อผิดพลาดก่อน Merge

Docker Compose

services → container ที่จะรัน (เช่น Web, DB)

ports → กำหนด port mapping ระหว่าง host ↔ container

volumes → เก็บข้อมูลถาวรระหว่าง host ↔ container

environment → ตั้งค่าตัวแปรภายใน container (เช่น รหัสผ่าน DB, ชื่อ database)

การทำงานร่วมกับ Project:

รันหลาย container พร้อมกัน

ตั้งค่า centralized, portable, repeatable

เชื่อมต่อ container ได้ง่าย

เหมาะกับ development และ testing

Kubernetes Deployment & Service - Summary
Deployment

จัดการการรัน Pod แบบ declarative

replicas → จำนวน instance ของ Pod

selector → ใช้ match Pod กับ Deployment ผ่าน labels

template → กำหนดโครงสร้าง Pod (containers, ports, env)

Service

เปิด access ให้ Pod ผ่าน network

port → port ของ Service ที่ client ติดต่อ

targetPort → port ของ Pod/container ที่ Service ส่ง traffic ไป

ความสัมพันธ์

Deployment สร้างและจัดการ Pod

Service เปิดให้เข้าถึง Pod ผ่าน network

Selector ของ Service ชี้ไปยัง labels ของ Pod จาก Deployment

ผลลัพธ์: ระบบสามารถรัน, scale และเข้าถึง application ได้อย่างเสถียร
สรุป Project CI/CD แบบครบวงจร

ไฟล์ YAML ทำงานร่วมกัน

.github/workflows/ci.yml → Workflow อัตโนมัติ ตรวจสอบ Lint, Build, Notify

docker-compose.yml → รันระบบ Web + DB บน local / dev

k8s/deployment.yml → สร้างและจัดการ Pod บน Kubernetes

k8s/service.yml → เปิด access ให้ Pod ผ่าน network

ลำดับขั้นตอนเมื่อ Push โค้ด

Push โค้ดไป branch main

GitHub Actions Workflow รัน (Lint → Build → Kubernetes dry-run → Notify)

Docker Compose รัน local / dev environment

Kubernetes Deployment & Service apply → ระบบพร้อมใช้งานบน Cluster

โครงสร้าง Project

my-ci-cd-project/
├─ .github/workflows/ci.yml
├─ docker-compose.yml
├─ k8s/deployment.yml
├─ k8s/service.yml
└─ README.md


ความสัมพันธ์ Pipeline

Push Code
    │
    ▼
GitHub Actions Workflow
    │
    ▼
Docker Compose (Web + DB)
    │
    ▼
Kubernetes Deployment & Service → Production / Test Environment