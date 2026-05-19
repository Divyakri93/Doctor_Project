<div align="center">

# 🏥 Prescripto — Doctor Appointment System

### A full-stack healthcare appointment booking platform with Admin, Doctor & Patient portals

[![React](https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=white)](https://react.dev/)
[![Node.js](https://img.shields.io/badge/Node.js-18-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-5-000000?style=for-the-badge&logo=express&logoColor=white)](https://expressjs.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?style=for-the-badge&logo=jenkins&logoColor=white)](https://www.jenkins.io/)

---

### 🌐 Live Demo

[![Frontend](https://img.shields.io/badge/🟢%20Patient%20App-Live-success?style=for-the-badge)](https://doctor-appointment-frontend-f8hl.onrender.com)
[![Admin](https://img.shields.io/badge/🟡%20Admin%20Panel-Live-yellow?style=for-the-badge)](https://doctor-appointment-admin-zdnm.onrender.com)
[![Backend](https://img.shields.io/badge/🔵%20Backend%20API-Live-blue?style=for-the-badge)](https://doctor-appointment-backend-e1wn.onrender.com)

| Portal | Live URL |
|---|---|
| 🟢 **Patient Frontend** | https://doctor-appointment-frontend-f8hl.onrender.com |
| 🟡 **Admin Panel** | https://doctor-appointment-admin-zdnm.onrender.com |
| 🔵 **Backend API** | https://doctor-appointment-backend-e1wn.onrender.com |

> ⚠️ Hosted on Render free tier — first load may take ~30 seconds to wake up.

</div>


---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [Docker Deployment](#-docker-deployment)
- [Jenkins CI/CD](#-jenkins-cicd)
- [API Endpoints](#-api-endpoints)
- [Project Structure](#-project-structure)

---

## 🌟 Overview

**Prescripto** is a production-ready, multi-portal doctor appointment booking system. It enables patients to search for doctors, book appointments, and make payments — while giving doctors and administrators dedicated dashboards to manage their workflow.

The project is fully containerised with Docker and has an automated CI/CD pipeline powered by Jenkins.

---

## ✨ Features

### 👤 Patient Portal (Frontend — Port 5173)
- 🔍 Browse and search doctors by speciality
- 📅 Book, view, and cancel appointments
- 💳 Online payments via **Razorpay**
- 👤 Personal profile management
- 🔔 Toast notifications for real-time feedback

### 🩺 Admin Panel (Port 5174)
- ➕ Add / manage doctors and appointments
- 📊 Dashboard with live statistics
- 🖼️ Image uploads via **Cloudinary**
- 🔐 Secure admin authentication with JWT

### 🖥️ Backend API (Port 5000)
- RESTful API built with Express 5
- JWT-based authentication for all portals
- MongoDB Atlas for cloud database
- Multer for multipart file handling
- Razorpay integration for payments

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 19, Vite, Tailwind CSS, Axios |
| **Admin** | React 19, Vite, Tailwind CSS |
| **Backend** | Node.js 18, Express 5, Mongoose |
| **Database** | MongoDB Atlas |
| **Storage** | Cloudinary |
| **Payments** | Razorpay |
| **Auth** | JSON Web Tokens (JWT), bcrypt |
| **Container** | Docker, Docker Compose |
| **CI/CD** | Jenkins Pipeline |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Jenkins CI/CD                      │
│  Checkout → .env Inject → Docker Build → Deploy      │
└──────────────────────┬──────────────────────────────┘
                       │
          ┌────────────▼────────────┐
          │    Docker Compose       │
          │  (docker-compose.yml)   │
          └────────────┬────────────┘
                       │
     ┌─────────────────┼─────────────────┐
     │                 │                 │
┌────▼────┐      ┌─────▼─────┐    ┌─────▼─────┐
│Frontend │      │  Backend  │    │   Admin   │
│:5173    │      │  API:5000 │    │  :5174    │
│React+   │◄────►│ Express+  │◄───│React+     │
│Vite     │      │ Mongoose  │    │Vite       │
└─────────┘      └─────┬─────┘    └───────────┘
                       │
            ┌──────────┴──────────┐
            │                     │
      ┌─────▼─────┐       ┌───────▼──────┐
      │  MongoDB  │       │  Cloudinary  │
      │   Atlas   │       │  (Images)    │
      └───────────┘       └──────────────┘
```

---

## 🚀 Getting Started

### Prerequisites

Make sure you have the following installed:

```
Node.js >= 18
Docker Desktop
Git
```

### 1. Clone the Repository

```bash
git clone https://github.com/Divyakri93/DoctorAppointment_jenkins.git
cd DoctorAppointment_jenkins
```

### 2. Set Up Environment Variables

Create a `.env` file inside the `backend/` folder:

```bash
cp backend/.env.example backend/.env
```

Fill in your credentials (see [Environment Variables](#-environment-variables) section).

### 3. Run Locally (without Docker)

```bash
# Backend
cd backend
npm install
npm start        # → http://localhost:5000

# Frontend (new terminal)
cd frontend
npm install
npm run dev      # → http://localhost:5173

# Admin (new terminal)
cd admin
npm install
npm run dev      # → http://localhost:5174
```

---

## 🔐 Environment Variables

Create `backend/.env` with the following keys:

```env
# Database
MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net

# Cloudinary (Image Storage)
CLOUDINARY_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_SECRET_KEY=your_secret_key

# Auth
JWT_SECRET=your_jwt_secret

# Admin Credentials
ADMIN_EMAIL=admin@prescripto.com
ADMIN_PASSWORD=your_password

# Payments
RAZORPAY_KEY_ID=rzp_test_xxxxx
RAZORPAY_KEY_SECRET=your_razorpay_secret

# Currency
CURRENCY=INR
```

> ⚠️ **Never commit `.env` to Git.** It is already listed in `.gitignore`.

---

## 🐳 Docker Deployment

Run the entire application stack with a single command:

```bash
docker-compose up -d --build
```

| Service | Container | Port |
|---|---|---|
| Backend API | `workspace-backend-1` | `5000` |
| Frontend | `workspace-frontend-1` | `5173` |
| Admin Panel | `workspace-admin-1` | `5174` |

**Stop all containers:**
```bash
docker-compose down
```

**View running containers:**
```bash
docker ps
```

---

## ⚙️ Jenkins CI/CD

This project uses a fully automated **Jenkins Pipeline** defined in `Jenkinsfile`. It builds the Docker images locally and then triggers the live Render deployments.

### Pipeline Stages

```
┌──────────┐   ┌──────────────┐   ┌───────────────┐   ┌─────────┐   ┌──────────────┐   ┌─────────────────┐
│ Checkout │ → │ Create .env  │ → │ Build Docker  │ → │ Deploy  │ → │ Health Check │ → │ Deploy to Render│
│  (Git)   │   │ (Credentials)│   │   Images      │   │ Compose │   │  (3 Ports)   │   │ (Trigger Hooks) │
└──────────┘   └──────────────┘   └───────────────┘   └─────────┘   └──────────────┘   └─────────────────┘
```

### Jenkins Credentials Required

Add these as **Secret Text** credentials in Jenkins:

| Credential ID | Description |
|---|---|
| `MONGODB_URI` | MongoDB Atlas connection string |
| `CLOUDINARY_NAME` | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_SECRET_KEY` | Cloudinary secret key |
| `JWT_SECRET` | JWT signing secret |
| `RAZORPAY_KEY_ID` | Razorpay key ID |
| `RAZORPAY_KEY_SECRET` | Razorpay key secret |
| `RENDER_BACKEND_HOOK` | Render Deploy Hook URL for Backend |
| `RENDER_FRONTEND_HOOK` | Render Deploy Hook URL for Frontend |
| `RENDER_ADMIN_HOOK` | Render Deploy Hook URL for Admin |

### Setup Jenkins Job & GitHub Webhook

1. Create a new **Pipeline** job in Jenkins.
2. Set **SCM** → Git → your repository URL.
3. Set **Branch** → `*/main`.
4. Set **Script Path** → `Jenkinsfile`.
5. Check **"GitHub hook trigger for GITScm polling"** in Build Triggers.
6. Use a tool like `localtunnel` or `ngrok` to expose your local Jenkins to the internet (e.g., `npx localtunnel --port 8080`).
7. Go to your GitHub Repository **Settings** → **Webhooks** → **Add webhook**.
8. Set the **Payload URL** to your tunnel URL + `/github-webhook/` (e.g., `https://your-url.loca.lt/github-webhook/`).
9. Set **Content type** to `application/json` and select **Just the push event**.
10. Click **Add webhook**. Now, every `git push` will automatically trigger a full CI/CD deployment! 🚀

---

## 📡 API Endpoints

### Admin Routes — `/api/admin`
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/login` | Admin login |
| `POST` | `/addDoctor` | Add a new doctor |
| `GET` | `/allDoctors` | List all doctors |
| `GET` | `/appointments` | All appointments |
| `POST` | `/cancelAppointment` | Cancel appointment |
| `GET` | `/dashboard` | Dashboard stats |

### Doctor Routes — `/api/doctor`
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/login` | Doctor login |
| `GET` | `/appointments` | Doctor's appointments |
| `POST` | `/completeAppointment` | Mark as complete |
| `GET` | `/dashboard` | Doctor dashboard |
| `GET` | `/profile` | Get profile |
| `POST` | `/updateProfile` | Update profile |

### User Routes — `/api/user`
| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/register` | Patient register |
| `POST` | `/login` | Patient login |
| `GET` | `/getDoctors` | Browse doctors |
| `POST` | `/bookAppointment` | Book appointment |
| `GET` | `/appointments` | My appointments |
| `POST` | `/cancelAppointment` | Cancel appointment |
| `POST` | `/makePayment` | Razorpay payment |

---

## 📁 Project Structure

```
DoctorAppointment/
├── 📄 Jenkinsfile              # CI/CD pipeline definition
├── 📄 docker-compose.yml       # Multi-container orchestration
├── 📄 .gitignore
│
├── 📂 backend/                 # Node.js + Express API
│   ├── 📄 server.js
│   ├── 📄 Dockerfile
│   ├── 📄 package.json
│   ├── 📂 config/              # DB & Cloudinary setup
│   ├── 📂 controllers/         # Route handlers
│   ├── 📂 middlewares/         # Auth middleware
│   ├── 📂 models/              # Mongoose schemas
│   └── 📂 routes/              # API routes
│
├── 📂 frontend/                # Patient portal (React)
│   ├── 📄 Dockerfile
│   ├── 📄 vite.config.js
│   └── 📂 src/
│
└── 📂 admin/                   # Admin portal (React)
    ├── 📄 Dockerfile
    ├── 📄 vite.config.js
    └── 📂 src/
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit changes: `git commit -m 'Add your feature'`
4. Push to branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

<div align="center">

**Made with ❤️ | Prescripto Doctor Appointment System**

[![GitHub](https://img.shields.io/badge/GitHub-Divyakri93-181717?style=flat-square&logo=github)](https://github.com/Divyakri93/DoctorAppointment_jenkins)

</div>