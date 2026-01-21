# 🌐 PyNode Forum Project

A full-stack forum application built with microservices architecture.

## 🏗️ System Architecture

```


                              ┌─────────────────┐
                              │    FRONTEND     │
                              │  React + Vite   │
                              │  localhost:3000 │
                              └────────┬────────┘
                                       │
                                       │ HTTP (All requests)
                                       ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                              API GATEWAY                                     ║
║                          Express.js :8080                                    ║
║  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────────────────┐  ║
║  │   CORS   │→ │  Logger  │→ │  Router  │→ │  http-proxy-middleware       │  ║
║  └──────────┘  └──────────┘  └──────────┘  └──────────────────────────────┘  ║
╚══════════════════════════════════════════════════════════════════════════════╝
         │           │           │           │           │           │
         │           │           │           │           │           │
         ▼           ▼           ▼           ▼           ▼           ▼
┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────┬─────────────┐
│    Auth     │    User     │    Post     │   History   │   Message   │    File     │
│   Service   │   Service   │   Service   │   Service   │   Service   │   Service   │
│   (Flask)   │   (Flask)   │  (Node.js)  │  (Node.js)  │   (Flask)   │   (Flask)   │
│    :5000    │    :5001    │    :5002    │    :5003    │    :5004    │    :5005    │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ • Login     │ • Profile   │ • Posts     │ • View      │ • Contact   │ • Upload    │
│ • Register  │ • Admin     │ • Replies   │   History   │   Admin     │ • Download  │
│ • JWT       │ • Ban/Unban │ • Search    │ • Tracking  │ • Inbox     │ • S3        │
└──────┬──────┴──────┬──────┴──────┬──────┴──────┬──────┴──────┬──────┴──────┬──────┘
       │             │             │             │             │             │
       │   Internal  │             │             │             │             │
       │◄───────────►│             │             │             │             │
       │             │             │             │             │             │
       ▼             ▼             ▼             ▼             ▼             ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                              DATA LAYER                                      ║
╠══════════════════════════╦════════════════════╦══════════════════════════════╣
║       MySQL :3306        ║   MongoDB :27017   ║        External              ║
║  ┌────────────────────┐  ║  ┌──────────────┐  ║  ┌────────────────────────┐  ║
║  │  forum_user_db     │  ║  │forum_post_db │  ║  │      AWS S3            │  ║
║  │  forum_history_db  │  ║  │  • posts     │  ║  │   (File Storage)       │  ║
║  │  forum_message_db  │  ║  │  • replies   │  ║  └────────────────────────┘  ║
║  └────────────────────┘  ║  └──────────────┘  ║  ┌────────────────────────┐  ║
╠══════════════════════════╩════════════════════╣  │    RabbitMQ :5672      │  ║
║                                               ║  │   (Email Queue)        │  ║
║              Used by: User, History, Message  ║  └───────────┬────────────┘  ║
╚═══════════════════════════════════════════════╩══════════════╪═══════════════╝
                                                               │
                                                               ▼
                                                    ┌─────────────────┐
                                                    │  Email Service  │
                                                    │    (Worker)     │
                                                    │   • SendGrid    │
                                                    │   • Verify      │
                                                    └─────────────────┘
```

## 📦 Repositories

| Repository | Tech Stack | Description |
|------------|------------|-------------|
| [infrastructure](https://github.com/pynode-forum-project/infrastructure) | Docker Compose | Orchestration & shared configs |
| [frontend](https://github.com/pynode-forum-project/frontend) | React, Vite | User interface |
| [gateway](https://github.com/pynode-forum-project/gateway) | Express.js | API Gateway & routing |
| [auth-service](https://github.com/pynode-forum-project/auth-service) | Flask | Authentication & JWT |
| [user-service](https://github.com/pynode-forum-project/user-service) | Flask | User management |
| [post-reply-service](https://github.com/pynode-forum-project/post-reply-service) | Node.js | Posts & replies |
| [history-service](https://github.com/pynode-forum-project/history-service) | Flask | Browsing history |
| [message-service](https://github.com/pynode-forum-project/message-service) | Flask | Contact admin |
| [file-service](https://github.com/pynode-forum-project/file-service) | Flask | File upload (S3) |
| [email-service](https://github.com/pynode-forum-project/email-service) | Flask | Email worker |

## 🔄 Request Flow

```
┌──────────┐      ┌─────────┐      ┌──────────────┐      ┌──────────┐
│ Frontend │ ──── │ Gateway │ ──── │ Microservice │ ──── │ Database │
│  :3000   │ HTTP │  :8080  │ HTTP │  :5000-5005  │      │ MySQL/   │
│          │      │         │      │              │      │ MongoDB  │
└──────────┘      └─────────┘      └──────────────┘      └──────────┘
```



### JWT Authentication Flow

```
1. Login:     Frontend → Gateway → Auth Service → User Service (verify)
                                        ↓
                                   JWT Token ──→ Frontend (localStorage)

2. Protected: Frontend ──[Authorization: Bearer token]──→ Gateway → Service
                                                              ↓
                                                      @jwt_required()

┌─────────────────────────────────────────────────────────────────────────────┐
│                           REQUEST FLOW EXAMPLES                             │
└─────────────────────────────────────────────────────────────────────────────┘

📝 REGISTER FLOW
────────────────
Browser → Gateway → Auth Service → User Service → MySQL
                         │
                         └──→ RabbitMQ → Email Service → SendGrid → User Email

🔐 LOGIN FLOW  
─────────────
Browser → Gateway → Auth Service → User Service (verify password)
                         │
                         └──→ Return JWT Token → Browser (localStorage)

👤 GET PROFILE FLOW
───────────────────
Browser → Gateway → User Service (@jwt_required) → MySQL → Return User Data
   │                      ▲
   └── Authorization: Bearer <token>

📄 CREATE POST FLOW
───────────────────
Browser → Gateway → Post Service (@jwt_required) → MongoDB
                         │
                         └──→ History Service (record activity)
```

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | React 18, Vite, React Router |
| **Gateway** | Express.js, http-proxy-middleware |
| **Backend** | Flask (Python), Express.js (Node.js) |
| **Auth** | JWT (Flask-JWT-Extended) |
| **Databases** | MySQL 8.0, MongoDB |
| **Queue** | RabbitMQ |
| **Storage** | AWS S3 |
| **Container** | Docker, Docker Compose |

## 🚀 Quick Start

```bash
# Clone all repositories
mkdir ForumProject && cd ForumProject
git clone https://github.com/pynode-forum-project/infrastructure.git
git clone https://github.com/pynode-forum-project/frontend.git
git clone https://github.com/pynode-forum-project/gateway.git
git clone https://github.com/pynode-forum-project/auth-service.git
git clone https://github.com/pynode-forum-project/user-service.git
git clone https://github.com/pynode-forum-project/post-reply-service.git
git clone https://github.com/pynode-forum-project/history-service.git
git clone https://github.com/pynode-forum-project/message-service.git
git clone https://github.com/pynode-forum-project/file-service.git
git clone https://github.com/pynode-forum-project/email-service.git

# Start all services
cd infrastructure
docker-compose up -d --build

# Access
# Frontend:  http://localhost:3000
# Gateway:   http://localhost:8080
# RabbitMQ:  http://localhost:15672
```

## 📖 Documentation

- [Infrastructure Setup](https://github.com/pynode-forum-project/infrastructure#readme)
- [API Contracts](https://github.com/pynode-forum-project/infrastructure/tree/main/docs)
- [Postman Collection](https://github.com/pynode-forum-project/infrastructure/tree/main/postman)

## 👥 Contributors

- Jingtao Zhang
- Kshitij Taneja
- Ning Ding
- Seonho Yeom

*Beaconfire Training Program*
