# 📄 Load Balancer + Auto Scaling Setup (AWS + Node.js)

---

## 🧾 1. Overview

This document explains how to set up a **Load Balanced + Auto Scaled Node.js application** using cloud infrastructure.

### Goals:

* Automatically handle increasing traffic
* Maintain high availability
* Reduce cost when traffic is low

---

## 🎯 2. Problem Statement

Without auto scaling:

```
User → Load Balancer → Fixed Servers
```

Problems:

* High traffic → servers crash ❌
* Low traffic → wasted resources ❌

---

## ✅ 3. Solution

Use **Auto Scaling with Load Balancer**

```
User → Load Balancer → Auto Scaling Group → Node.js Servers
```

---

## 🧠 4. Architecture

```
Client
 ↓
Application Load Balancer (ALB)
 ↓
Auto Scaling Group (EC2 instances)
 ↓
Node.js App (PM2)
 ↓
Database / Redis
```

---

## 🗂️ 5. Components

### 1. Load Balancer (ALB)

* Distributes traffic across instances
* Performs health checks

### 2. Auto Scaling Group (ASG)

* Automatically adds/removes servers

### 3. EC2 Instances

* Run Node.js app

### 4. AMI (Template)

* Pre-configured server image

---

## ⚙️ 6. Step-by-Step Setup

---

### Step 1: Prepare Node.js App

```js
const PORT = process.env.PORT || 3000;
app.listen(PORT);
```

---

### Step 2: Launch EC2 Instance

* OS: Ubuntu
* Open ports:

  * 22 (SSH)
  * 80 (HTTP)

---

### Step 3: Install Node + PM2

```bash
sudo apt update
sudo apt install nodejs npm -y
sudo npm install -g pm2
```

---

### Step 4: Deploy Application

```bash
git clone <your-repo>
cd project
npm install
pm2 start server.js
pm2 save
```

---

### Step 5: Create AMI

* Go to EC2 → Actions → Create Image
* Name: `node-app-template`

---

### Step 6: Create Launch Template

* Use AMI
* Select instance type
* Configure security group

---

### Step 7: Create Auto Scaling Group

* Min instances: 2
* Desired: 2
* Max: 5
* Attach to VPC + subnets

---

### Step 8: Create Load Balancer (ALB)

* Type: Application Load Balancer
* Listener: HTTP (80)
* Create target group
* Attach Auto Scaling Group

---

### Step 9: Configure Health Check

Example:

```
Path: /health
Interval: 30s
```

---

### Step 10: Add Scaling Policy

#### Scale Up:

```
If CPU > 70% → Add 1 instance
```

#### Scale Down:

```
If CPU < 30% → Remove 1 instance
```

---

## 🔁 7. Scaling Flow

```
High Traffic → CPU ↑ → New Instance Added  
Low Traffic → CPU ↓ → Instance Removed  
```

---

## ⚠️ 8. Important Considerations

---

### 1. Stateless Application

Do NOT store session in memory ❌

Use:

* JWT
* Redis

---

### 2. Shared Storage

All servers must access:

* Same database
* Same Redis

---

### 3. Cooldown Period

* Prevent rapid scaling up/down

---

### 4. Logging

* Use CloudWatch for monitoring

---

## ⚡ 9. Production Enhancements

---

### 1. Use Redis for Caching

* Faster response
* Reduced DB load

---

### 2. CI/CD Integration

* Auto deploy on code push

---

### 3. HTTPS Setup

* Use SSL (Let's Encrypt or AWS ACM)

---

## ☁️ 10. Final Production Architecture

```
User
 ↓
AWS Load Balancer (ALB)
 ↓
Auto Scaling EC2 Instances
 ↓
Node.js App (PM2)
 ↓
Redis (ElastiCache)
 ↓
Database
```

---

## 🎯 11. Interview Summary

> I use an Application Load Balancer to distribute traffic across multiple EC2 instances.
> These instances are managed by an Auto Scaling Group which dynamically adjusts capacity based on CPU utilization or request load.
> The application is stateless and uses shared services like Redis and database to ensure consistency.

---

## 🧠 12. Advantages

* High availability
* Automatic scaling
* Cost optimization
* Fault tolerance

---

## 📌 13. Use Cases

* High traffic APIs
* E-commerce systems
* Real-time applications
* SaaS platforms

---

## 🏁 14. Conclusion

This setup ensures:

* Scalability
* Reliability
* Performance

It is widely used in modern backend systems.

---
