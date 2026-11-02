# 🎮 Questify — Project Overview

> **A Gamified Life & Skill-Tree Tracker built with React Native / React and Java Spring Boot Microservices.**

---

## 📌 Executive Summary

**Questify** is a productivity and habit-tracking application designed around **RPG (Role-Playing Game) gamification mechanics**. Instead of traditional, boring to-do lists, Questify turns personal development, health, learning, and productivity into an interactive game where the user is the main character. 

The primary goal of building Questify is dual-fold:
1. **Product Value:** Create an engaging, self-rewarding productivity tool that developers can actually use daily.
2. **Technical Mastery:** Provide a realistic domain for implementing **Microservice Architecture, Event-Driven Systems, and Modern DevOps Pipelines** using **Java Spring Boot**, **React Native / React**, and **Kubernetes**.

---

## 🎯 Core Product Features

### 1. ⚔️ Quest System (Habits & Tasks)
* **Quests:** Habits or tasks defined with difficulty levels (Easy, Medium, Hard).
* **Categories:** Quests tied to specific domains (e.g., Coding, Fitness, Finance).
* **Recurrence:** Daily, weekly, or custom scheduled tasks.

### 2. 🌳 Skill-Tree & Leveling (XP System)
* **Experience Points (XP):** Earned upon completing quests.
* **Level Progression:** Dynamic leveling formula ($MaxXP = 100 \times Level^{1.5}$).
* **Skill Trees:** Visual progression node graph representing growth in specific life areas (e.g., Java Proficiency, Running Distance).

### 3. 🛍️ Custom Reward Shop (Gold Currency)
* **Gold Earnings:** Earned alongside XP when finishing quests.
* **Self-Reward System:** Users define their own real-life rewards and "Gold costs" (e.g., 500 Gold = *1 Movie Night*).
* **Redemption Engine:** Transactional exchange of virtual currency for real-life breaks/rewards.

### 4. 🛡️ Guilds & Leaderboards (Social Features)
* **Guilds:** Join groups with friends for mutual accountability.
* **Weekly Boss Fights:** Shared guild goals where missed personal habits damage the team.
* **Leaderboards:** Real-time ranking of top XP earners among friends/guild members.

---

## 🏗️ Architecture & Microservices Breakdown

```
                         +-------------------+
                         | React / React Native |
                         +---------+---------+
                                   |
                         +---------v---------+
                         |  Spring Cloud     |
                         |  API Gateway      |
                         +----+----+----+----+
                              |    |    |
        +---------------------+    |    +----------------------+
        |                          |                           |
+-------v-------+          +-------v-------+           +-------v-------+
|  User & XP    |          |  Quest & Event|           | Social & Guild|
|  Service      |          |  Engine       |           | Service       |
|  (PostgreSQL) |          |  (PostgreSQL) |           | (Redis/Postgres)|
+-------+-------+          +-------+-------+           +-------+-------+
        |                          |                           |
        +--------------------+     |     +---------------------+
                             |     |     |
                          +--v-----v-----v--+
                          |   Apache Kafka  |
                          +--------+--------+
                                   |
                          +--------v--------+
                          | Notification    |
                          | Service         |
                          +-----------------+
```

### Microservices

1. **User & Gamification Service**
   * Manages User profiles, XP calculations, Level progression logic, and Gold balances.
   * *Database:* PostgreSQL

2. **Quest Engine Service**
   * Handles quest creation, completion workflows, and recurring schedules.
   * *Database:* PostgreSQL

3. **Social & Guild Service**
   * Manages friendships, guild chat, and real-time leaderboards.
   * *Database:* Redis (for leaderboard caching) & PostgreSQL

4. **Notification Service**
   * Asynchronously consumes events (e.g., Quest Completed, Level Up) and dispatches push/email notifications.

---

## 🛠️ DevOps & Infrastructure Objectives

* **Containerization:** Dockerize all Spring Boot services and Frontend applications.
* **Event-Driven Architecture:** Use **Apache Kafka** or **RabbitMQ** for decoupling inter-service communications (e.g., `QuestCompletedEvent`).
* **API Gateway & Security:** **Spring Cloud Gateway** with **OAuth2 / JWT** security pattern.
* **Orchestration:** Deploy using **Kubernetes** (Minikube/k3s) with Horizontal Pod Autoscaling (HPA).
* **CI/CD Pipeline:** Automate testing, Docker building, vulnerability scanning (Trivy), and deployment via **GitHub Actions**.
* **Observability:** Centralized logging with **ELK Stack / Loki** and metrics monitoring via **Prometheus & Grafana**.

---

## 🚀 Suggested Implementation Roadmap

1. **Phase 1: Modular Monolith MVP**
   * Create the core domain (Quests, XP, Gold, Rewards) inside a single Spring Boot application.
   * Build a basic React or React Native screen to interact with the API.

2. **Phase 2: Microservice Extraction & Dockerization**
   * Split the monolith into `User/XP Service` and `Quest Service`.
   * Set up Spring Cloud Gateway and Docker Compose.

3. **Phase 3: Event-Driven & Async Logic**
   * Integrate Kafka for asynchronous events (`QuestCompletedEvent` $\rightarrow$ `User XP Update` & `Notification`).

4. **Phase 4: Kubernetes & DevOps Infrastructure**
   * Write Kubernetes manifests / Helm charts.
   * Set up Prometheus, Grafana, and CI/CD automation pipelines.