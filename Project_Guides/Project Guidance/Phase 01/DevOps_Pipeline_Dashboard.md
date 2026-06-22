# DevOps Pipeline Dashboard — Complete Production-Ready MERN Stack Application

> **Authored by:** Senior MERN Stack Developer, Software Architect, UI/UX Designer & Technical Writer
> **Version:** 1.0.0 | **Stack:** MongoDB · Express.js · React.js (Vite) · Node.js
> **Architecture:** MVC · REST API · JWT Auth · Docker/K8s-Ready · CI/CD-Ready

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture Overview](#2-architecture-overview)
3. [Complete Folder Structure](#3-complete-folder-structure)
4. [Prerequisites & Installation](#4-prerequisites--installation)
5. [Backend — Server Setup](#5-backend--server-setup)
6. [Frontend — Client Setup](#6-frontend--client-setup)
7. [Environment Variables](#7-environment-variables)
8. [Backend Source Code](#8-backend-source-code)
9. [Frontend Source Code](#9-frontend-source-code)
10. [Running the Application](#10-running-the-application)
11. [API Documentation](#11-api-documentation)
12. [Docker & Kubernetes Readiness](#12-docker--kubernetes-readiness)
13. [CI/CD Pipeline Readiness](#13-cicd-pipeline-readiness)
14. [Dependency Reference](#14-dependency-reference)
15. [Troubleshooting](#15-troubleshooting)

---

## 1. Project Overview

**DevOps Pipeline Dashboard** is a full-stack web application that provides a centralized, real-time dashboard for monitoring CI/CD pipelines, deployment statuses, server health metrics, and team activity. It is architected from day one to be containerized (Docker), orchestrated (Kubernetes), and integrated into automated CI/CD pipelines without requiring architectural refactoring.

### Core Features

| Feature | Description |
|---|---|
| JWT Authentication | Secure login/register with httpOnly cookies |
| Pipeline Monitor | Track build, test, and deploy statuses |
| Server Health Metrics | CPU, Memory, Uptime, Request charts via Recharts |
| Deployment History | Paginated log of all deployments |
| User Management | Role-based (Admin / Developer / Viewer) |
| REST API | Fully documented, versioned API (`/api/v1`) |
| Secure Headers | Helmet.js security headers |
| CORS Configured | Fine-grained origin control |
| Request Logging | Morgan HTTP request logger |
| Error Handling | Centralized async error handler |
| Input Validation | Server-side with custom validators |

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT (React + Vite)                     │
│  React Router DOM → Pages → Components → Recharts Charts    │
│  Axios (with interceptors) → API Layer                       │
│  Tailwind CSS → UI Styling                                   │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/HTTPS REST API
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   SERVER (Express.js)                        │
│  Routes → Controllers → Services → Models                   │
│  Middleware: Auth · Helmet · CORS · Morgan · CookieParser   │
│  JWT (httpOnly Cookies) · bcryptjs Password Hashing         │
└──────────────────────────┬──────────────────────────────────┘
                           │ Mongoose ODM
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   DATABASE (MongoDB)                         │
│  Collections: users · pipelines · deployments · metrics     │
└─────────────────────────────────────────────────────────────┘
```

### Design Principles

- **MVC Architecture** — Models, Controllers, Routes are separated cleanly
- **12-Factor App** — Config via environment variables, stateless processes
- **API Versioning** — All routes under `/api/v1/` for backward compatibility
- **Container-First** — No hardcoded paths, ports from env, health check endpoint built-in
- **Security-First** — Helmet, CORS, JWT in httpOnly cookies, input sanitization

---

## 3. Complete Folder Structure

```
devops-pipeline-dashboard/
│
├── client/                          # React.js Frontend (Vite)
│   ├── public/
│   │   └── vite.svg
│   ├── src/
│   │   ├── api/
│   │   │   └── axios.js             # Axios instance with interceptors
│   │   ├── assets/
│   │   │   └── logo.svg
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── Navbar.jsx       # Top navigation bar
│   │   │   │   ├── Sidebar.jsx      # Side navigation
│   │   │   │   └── Layout.jsx       # Page layout wrapper
│   │   │   ├── charts/
│   │   │   │   ├── CpuChart.jsx     # CPU usage line chart
│   │   │   │   ├── MemoryChart.jsx  # Memory usage area chart
│   │   │   │   └── DeployChart.jsx  # Deployments bar chart
│   │   │   ├── ui/
│   │   │   │   ├── StatCard.jsx     # Metric stat card
│   │   │   │   ├── Badge.jsx        # Status badge
│   │   │   │   ├── Spinner.jsx      # Loading spinner
│   │   │   │   └── Alert.jsx        # Alert/notification
│   │   │   └── pipeline/
│   │   │       ├── PipelineCard.jsx # Individual pipeline card
│   │   │       └── PipelineList.jsx # Pipeline listing
│   │   ├── context/
│   │   │   └── AuthContext.jsx      # Global auth state (React Context)
│   │   ├── hooks/
│   │   │   ├── useAuth.js           # Auth hook
│   │   │   └── useFetch.js          # Generic data-fetching hook
│   │   ├── pages/
│   │   │   ├── LoginPage.jsx        # Login screen
│   │   │   ├── RegisterPage.jsx     # Registration screen
│   │   │   ├── DashboardPage.jsx    # Main dashboard
│   │   │   ├── PipelinesPage.jsx    # Pipelines list
│   │   │   ├── DeploymentsPage.jsx  # Deployments history
│   │   │   ├── MetricsPage.jsx      # Server metrics & charts
│   │   │   ├── ProfilePage.jsx      # User profile
│   │   │   └── NotFoundPage.jsx     # 404 page
│   │   ├── routes/
│   │   │   ├── AppRoutes.jsx        # Centralized route definitions
│   │   │   └── ProtectedRoute.jsx   # Auth guard component
│   │   ├── utils/
│   │   │   ├── formatDate.js        # Date formatting utility
│   │   │   └── statusColors.js      # Status → color mapping
│   │   ├── App.jsx                  # Root application component
│   │   ├── main.jsx                 # React entry point
│   │   └── index.css                # Tailwind CSS imports
│   ├── .env                         # Frontend environment variables
│   ├── .env.example
│   ├── index.html
│   ├── package.json
│   ├── tailwind.config.js
│   └── vite.config.js
│
├── server/                          # Node.js + Express.js Backend
│   ├── config/
│   │   └── db.js                    # MongoDB connection logic
│   ├── controllers/
│   │   ├── authController.js        # Register, Login, Logout, Me
│   │   ├── pipelineController.js    # CRUD for pipelines
│   │   ├── deploymentController.js  # CRUD for deployments
│   │   └── metricsController.js     # Server metrics
│   ├── middleware/
│   │   ├── authMiddleware.js        # JWT verification middleware
│   │   ├── errorMiddleware.js       # Global error handler
│   │   ├── roleMiddleware.js        # Role-based access control
│   │   └── validateMiddleware.js    # Request body validation
│   ├── models/
│   │   ├── User.js                  # User Mongoose schema
│   │   ├── Pipeline.js              # Pipeline Mongoose schema
│   │   ├── Deployment.js            # Deployment Mongoose schema
│   │   └── Metric.js                # Metric Mongoose schema
│   ├── routes/
│   │   ├── authRoutes.js            # /api/v1/auth
│   │   ├── pipelineRoutes.js        # /api/v1/pipelines
│   │   ├── deploymentRoutes.js      # /api/v1/deployments
│   │   └── metricsRoutes.js         # /api/v1/metrics
│   ├── services/
│   │   ├── authService.js           # Auth business logic
│   │   ├── pipelineService.js       # Pipeline business logic
│   │   └── metricsService.js        # Metrics aggregation logic
│   ├── utils/
│   │   ├── generateToken.js         # JWT token generator
│   │   ├── asyncHandler.js          # Async error wrapper
│   │   └── ApiResponse.js           # Standardized API response
│   ├── .env                         # Backend environment variables
│   ├── .env.example
│   ├── .gitignore
│   ├── package.json
│   └── server.js                    # Express app entry point
│
├── .gitignore                       # Root gitignore
└── README.md                        # This document
```

---

## 4. Prerequisites & Installation

### 4.1 System Requirements

| Tool | Minimum Version | Recommended | Check Command |
|---|---|---|---|
| Node.js | 18.x | 20.x LTS | `node --version` |
| npm | 9.x | 10.x | `npm --version` |
| MongoDB | 6.x | 7.x | `mongod --version` |
| Git | 2.x | Latest | `git --version` |

### 4.2 Install Node.js

#### macOS (using Homebrew)
```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js LTS
brew install node@20

# Verify
node --version   # v20.x.x
npm --version    # 10.x.x
```

#### Ubuntu / Debian Linux
```bash
# Update package index
sudo apt update && sudo apt upgrade -y

# Install curl
sudo apt install -y curl

# Add NodeSource repository for Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js and npm
sudo apt install -y nodejs

# Verify
node --version
npm --version
```

#### Windows
```powershell
# Using winget (Windows 11/10)
winget install OpenJS.NodeJS.LTS

# Or download installer from https://nodejs.org/en/download

# Verify in PowerShell
node --version
npm --version
```

#### Using NVM (Recommended — all platforms)
```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Reload shell
source ~/.bashrc   # or source ~/.zshrc on macOS

# Install and use Node.js 20 LTS
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version
npm --version
```

### 4.3 Install MongoDB

#### macOS
```bash
brew tap mongodb/brew
brew install mongodb-community@7.0
brew services start mongodb-community@7.0
```

#### Ubuntu / Debian
```bash
# Import MongoDB public key
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Add repository
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
  https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Install
sudo apt update
sudo apt install -y mongodb-org

# Start service
sudo systemctl start mongod
sudo systemctl enable mongod

# Verify
mongod --version
```

### 4.4 Clone / Initialize the Project

```bash
# Create project root directory
mkdir devops-pipeline-dashboard
cd devops-pipeline-dashboard

# Initialize git repository
git init
```

---

## 5. Backend — Server Setup

### 5.1 Initialize Node.js Project

```bash
# From the project root
mkdir server
cd server

# Initialize Node.js project with default package.json
npm init -y
```

**What `npm init -y` does:** Creates a `package.json` with default values, skipping all interactive prompts. This is the manifest file that tracks project metadata, scripts, and dependencies.

### 5.2 Install Backend Dependencies

```bash
# Production dependencies
npm install express mongoose cors dotenv jsonwebtoken bcryptjs cookie-parser axios helmet morgan

# Development dependencies
npm install -D nodemon
```

### 5.3 Backend Dependency Explanation

| Package | Version | Purpose |
|---|---|---|
| `express` | ^4.x | Web framework — handles HTTP routing, middleware, and responses |
| `mongoose` | ^8.x | MongoDB ODM — provides schema-based models and query API |
| `cors` | ^2.x | Cross-Origin Resource Sharing — allows browser clients from different origins |
| `dotenv` | ^16.x | Loads environment variables from `.env` file into `process.env` |
| `jsonwebtoken` | ^9.x | Creates and verifies JSON Web Tokens for stateless authentication |
| `bcryptjs` | ^2.x | Password hashing with bcrypt algorithm (pure JavaScript, no native deps) |
| `cookie-parser` | ^1.x | Parses `Cookie` header and populates `req.cookies` |
| `axios` | ^1.x | HTTP client (used in server for internal service calls if needed) |
| `helmet` | ^7.x | Sets security-related HTTP headers (XSS, CSP, HSTS, etc.) |
| `morgan` | ^1.x | HTTP request logger middleware (logs method, URL, status, response time) |
| `nodemon` *(dev)* | ^3.x | Watches files and auto-restarts the server on code changes |

### 5.4 Add npm Scripts to `server/package.json`

After installation, your `package.json` should look like this:

```json
{
  "name": "devops-pipeline-dashboard-server",
  "version": "1.0.0",
  "description": "DevOps Pipeline Dashboard — Express.js REST API",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "engines": {
    "node": ">=18.0.0"
  },
  "keywords": ["devops", "dashboard", "mern", "api"],
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {
    "axios": "^1.7.2",
    "bcryptjs": "^2.4.3",
    "cookie-parser": "^1.4.6",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "helmet": "^7.1.0",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.4.1",
    "morgan": "^1.10.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.4"
  }
}
```

---

## 6. Frontend — Client Setup

### 6.1 Create Vite React Application

```bash
# From the project root (not inside server/)
# Go back to devops-pipeline-dashboard/
cd ..

# Create the React app with Vite using the official React template
npm create vite@latest client -- --template react
```

**What this command does:**
- `npm create vite@latest` — uses npm's `create` command to run the latest Vite scaffolding tool
- `client` — the directory name for the frontend app
- `-- --template react` — the `--` separates npm args from Vite args; `--template react` selects the React + JSX template

```bash
# Enter the client directory
cd client

# Install all Vite-scaffolded dependencies
npm install
```

### 6.2 Install Frontend Dependencies

```bash
# Core UI and routing libraries
npm install react-router-dom axios recharts react-icons

# Tailwind CSS as a dev dependency
npm install -D tailwindcss @tailwindcss/vite
```

### 6.3 Frontend Dependency Explanation

| Package | Version | Purpose |
|---|---|---|
| `react-router-dom` | ^6.x | Client-side routing — enables SPA navigation without full page reloads |
| `axios` | ^1.x | HTTP client — makes API calls to Express backend, supports interceptors |
| `recharts` | ^2.x | Composable React chart library built on D3 — for pipeline and metrics charts |
| `react-icons` | ^5.x | 40,000+ SVG icons (FontAwesome, Material, etc.) as React components |
| `tailwindcss` *(dev)* | ^4.x | Utility-first CSS framework — styles components via class names |

### 6.4 Frontend Dependency Explanation

| Package | Version | Purpose |
|---|---|---|
| `react` | ^18.x | Core React library — component model, state, hooks |
| `react-dom` | ^18.x | React DOM renderer — mounts React into the browser DOM |
| `vite` *(dev)* | ^5.x | Build tool — extremely fast HMR dev server and production bundler |
| `@vitejs/plugin-react` *(dev)* | ^4.x | Vite plugin that enables React JSX, Fast Refresh, and Babel transforms |

### 6.5 Configure Tailwind CSS

```bash
# In client/ directory - Initialize Tailwind config
npx tailwindcss init
```

This creates `tailwind.config.js` in the client root.

---

## 7. Environment Variables

### 7.1 Backend — `server/.env`

```bash
# Copy from example
cp server/.env.example server/.env
```

**`server/.env`**
```env
# Server Configuration
NODE_ENV=development
PORT=5000

# MongoDB Connection
MONGO_URI=mongodb://localhost:27017/devops_dashboard

# JWT Configuration
JWT_SECRET=your_super_secret_jwt_key_change_in_production_min_32_chars
JWT_EXPIRES_IN=7d
JWT_COOKIE_EXPIRES=7

# CORS — comma-separated allowed origins
CORS_ORIGIN=http://localhost:5173

# Rate Limiting (for future use)
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=100
```

**`server/.env.example`** (commit this, not `.env`)
```env
NODE_ENV=development
PORT=5000
MONGO_URI=mongodb://localhost:27017/devops_dashboard
JWT_SECRET=CHANGE_THIS_TO_A_RANDOM_32_CHAR_STRING
JWT_EXPIRES_IN=7d
JWT_COOKIE_EXPIRES=7
CORS_ORIGIN=http://localhost:5173
```

### 7.2 Frontend — `client/.env`

**`client/.env`**
```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_APP_NAME=DevOps Pipeline Dashboard
VITE_APP_VERSION=1.0.0
```

**`client/.env.example`**
```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_APP_NAME=DevOps Pipeline Dashboard
VITE_APP_VERSION=1.0.0
```

> **Important:** Vite only exposes env vars prefixed with `VITE_` to the client. Never put secrets in the frontend `.env`.

### 7.3 Root `.gitignore`

```gitignore
# Dependencies
node_modules/
client/node_modules/
server/node_modules/

# Environment Variables — NEVER commit these
.env
client/.env
server/.env

# Build outputs
client/dist/
client/build/

# Logs
*.log
npm-debug.log*

# OS files
.DS_Store
Thumbs.db

# Editor
.vscode/
.idea/

# Docker (local only)
*.env.local
```

---

## 8. Backend Source Code

### 8.1 `server/server.js` — Application Entry Point

```javascript
// server/server.js
// ─────────────────────────────────────────────────────────────
// Application Entry Point — Wires together Express, middleware,
// routes, and starts the HTTP server.
// ─────────────────────────────────────────────────────────────

import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import cookieParser from 'cookie-parser';
import dotenv from 'dotenv';
import { connectDB } from './config/db.js';

// Route imports
import authRoutes from './routes/authRoutes.js';
import pipelineRoutes from './routes/pipelineRoutes.js';
import deploymentRoutes from './routes/deploymentRoutes.js';
import metricsRoutes from './routes/metricsRoutes.js';

// Middleware imports
import { notFound, errorHandler } from './middleware/errorMiddleware.js';

// Load environment variables from .env file
dotenv.config();

// Connect to MongoDB
connectDB();

const app = express();
const PORT = process.env.PORT || 5000;

// ─── Security Middleware ───────────────────────────────────────
// helmet: Sets 14+ security HTTP headers automatically
// Prevents clickjacking, XSS, MIME sniffing, etc.
app.use(helmet());

// ─── CORS Configuration ───────────────────────────────────────
// Allows the React frontend to make requests to this API
// In production, restrict to specific domains
app.use(cors({
  origin: process.env.CORS_ORIGIN?.split(',') || 'http://localhost:5173',
  credentials: true,               // Allow cookies to be sent cross-origin
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));

// ─── Request Parsing Middleware ────────────────────────────────
app.use(express.json({ limit: '10kb' }));           // Parse JSON bodies (max 10KB)
app.use(express.urlencoded({ extended: true }));    // Parse URL-encoded bodies
app.use(cookieParser());                            // Parse Cookie header → req.cookies

// ─── HTTP Request Logger ──────────────────────────────────────
// morgan: Logs every request: METHOD URL STATUS RESPONSE_TIME
// 'dev' format: colored, compact — great for development
// Use 'combined' in production for Apache-style logs
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
} else {
  app.use(morgan('combined'));
}

// ─── Health Check Endpoint ────────────────────────────────────
// Used by Docker HEALTHCHECK, Kubernetes liveness probes,
// and load balancers to verify the service is alive
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV,
    version: process.env.npm_package_version || '1.0.0',
  });
});

// ─── API Routes ───────────────────────────────────────────────
// All routes are versioned under /api/v1 for backward compatibility
app.use('/api/v1/auth', authRoutes);
app.use('/api/v1/pipelines', pipelineRoutes);
app.use('/api/v1/deployments', deploymentRoutes);
app.use('/api/v1/metrics', metricsRoutes);

// ─── Error Handling Middleware ────────────────────────────────
// These must be LAST — after all routes
app.use(notFound);      // 404 handler for unmatched routes
app.use(errorHandler);  // Global error handler

// ─── Start Server ─────────────────────────────────────────────
const server = app.listen(PORT, () => {
  console.log(`
  ╔═══════════════════════════════════════════╗
  ║   DevOps Pipeline Dashboard — API Server  ║
  ║   Mode     : ${process.env.NODE_ENV?.padEnd(27)}║
  ║   Port     : ${String(PORT).padEnd(27)}║
  ║   DB       : MongoDB Connected            ║
  ╚═══════════════════════════════════════════╝
  `);
});

// ─── Graceful Shutdown ────────────────────────────────────────
// Handles SIGTERM (sent by Docker/Kubernetes on shutdown)
// and SIGINT (Ctrl+C) to close connections cleanly
process.on('SIGTERM', () => {
  console.log('SIGTERM received. Shutting down gracefully...');
  server.close(() => {
    console.log('HTTP server closed.');
    process.exit(0);
  });
});

process.on('SIGINT', () => {
  console.log('\nSIGINT received. Shutting down...');
  server.close(() => process.exit(0));
});

// Handle unhandled promise rejections (e.g., DB connection failure)
process.on('unhandledRejection', (err) => {
  console.error('UNHANDLED REJECTION:', err.message);
  server.close(() => process.exit(1));
});

export default app;
```

### 8.2 `server/config/db.js` — MongoDB Connection

```javascript
// server/config/db.js
// MongoDB connection using Mongoose.
// Exported as a function so it can be called from server.js
// and easily mocked in tests.

import mongoose from 'mongoose';

export const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGO_URI, {
      // These options ensure a stable connection pool
      maxPoolSize: 10,                // Max simultaneous connections
      serverSelectionTimeoutMS: 5000, // Timeout if no server found in 5s
      socketTimeoutMS: 45000,         // Close socket after 45s of inactivity
    });

    console.log(`✅ MongoDB Connected: ${conn.connection.host}`);

    // Log when MongoDB connection is lost
    mongoose.connection.on('disconnected', () => {
      console.warn('⚠️  MongoDB disconnected. Attempting to reconnect...');
    });

    mongoose.connection.on('reconnected', () => {
      console.log('✅ MongoDB reconnected.');
    });

  } catch (error) {
    console.error(`❌ MongoDB Connection Error: ${error.message}`);
    // Exit process with failure — Kubernetes will restart the pod
    process.exit(1);
  }
};
```

### 8.3 `server/models/User.js` — User Schema

```javascript
// server/models/User.js
// Defines the User document structure in MongoDB.
// Handles password hashing via Mongoose pre-save hook.

import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Name is required'],
      trim: true,
      minlength: [2, 'Name must be at least 2 characters'],
      maxlength: [50, 'Name cannot exceed 50 characters'],
    },
    email: {
      type: String,
      required: [true, 'Email is required'],
      unique: true,          // Creates a unique index in MongoDB
      lowercase: true,       // Auto-converts to lowercase before saving
      trim: true,
      match: [
        /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/,
        'Please provide a valid email address',
      ],
    },
    password: {
      type: String,
      required: [true, 'Password is required'],
      minlength: [8, 'Password must be at least 8 characters'],
      select: false, // Never return password in queries by default
    },
    role: {
      type: String,
      enum: {
        values: ['admin', 'developer', 'viewer'],
        message: 'Role must be admin, developer, or viewer',
      },
      default: 'developer',
    },
    avatar: {
      type: String,
      default: '',
    },
    isActive: {
      type: Boolean,
      default: true,
    },
    lastLogin: {
      type: Date,
    },
    passwordChangedAt: {
      type: Date,
    },
  },
  {
    timestamps: true,   // Adds createdAt and updatedAt automatically
    versionKey: false,  // Removes __v field from documents
  }
);

// ─── Pre-Save Hook: Hash Password ─────────────────────────────
// Runs before every .save() call.
// Only re-hashes if the password field was actually modified.
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();

  // bcrypt salt rounds: 12 is secure; each round doubles hashing time
  this.password = await bcrypt.hash(this.password, 12);

  // If password was changed (not on first creation), set timestamp
  if (!this.isNew) {
    this.passwordChangedAt = Date.now() - 1000; // 1s buffer for JWT timing
  }

  next();
});

// ─── Instance Method: Compare Passwords ───────────────────────
// Used in authController during login to validate the password.
userSchema.methods.comparePassword = async function (candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// ─── Instance Method: Check if Password Changed After Token ───
userSchema.methods.passwordChangedAfter = function (jwtIssuedAt) {
  if (this.passwordChangedAt) {
    const changedTimestamp = parseInt(this.passwordChangedAt.getTime() / 1000, 10);
    return jwtIssuedAt < changedTimestamp;
  }
  return false;
};

// ─── JSON Transform: Remove Sensitive Fields ──────────────────
userSchema.set('toJSON', {
  transform: (doc, ret) => {
    delete ret.password;
    delete ret.passwordChangedAt;
    return ret;
  },
});

const User = mongoose.model('User', userSchema);
export default User;
```

### 8.4 `server/models/Pipeline.js` — Pipeline Schema

```javascript
// server/models/Pipeline.js
// Represents a CI/CD pipeline with its stages and current status.

import mongoose from 'mongoose';

const stageSchema = new mongoose.Schema({
  name: { type: String, required: true },
  status: {
    type: String,
    enum: ['pending', 'running', 'success', 'failed', 'skipped'],
    default: 'pending',
  },
  duration: { type: Number, default: 0 }, // seconds
  startedAt: { type: Date },
  finishedAt: { type: Date },
  logs: { type: String, default: '' },
});

const pipelineSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Pipeline name is required'],
      trim: true,
      maxlength: [100, 'Name cannot exceed 100 characters'],
    },
    description: {
      type: String,
      trim: true,
      maxlength: [500, 'Description cannot exceed 500 characters'],
    },
    repository: {
      type: String,
      required: [true, 'Repository URL is required'],
      trim: true,
    },
    branch: {
      type: String,
      default: 'main',
      trim: true,
    },
    status: {
      type: String,
      enum: ['idle', 'pending', 'running', 'success', 'failed', 'cancelled'],
      default: 'idle',
    },
    stages: [stageSchema],
    triggeredBy: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    environment: {
      type: String,
      enum: ['development', 'staging', 'production'],
      default: 'development',
    },
    duration: { type: Number, default: 0 }, // total seconds
    lastRunAt: { type: Date },
    runCount: { type: Number, default: 0 },
    tags: [{ type: String, trim: true }],
    isActive: { type: Boolean, default: true },
  },
  {
    timestamps: true,
    versionKey: false,
  }
);

// Index for fast queries on status and environment
pipelineSchema.index({ status: 1, environment: 1 });
pipelineSchema.index({ triggeredBy: 1 });

const Pipeline = mongoose.model('Pipeline', pipelineSchema);
export default Pipeline;
```

### 8.5 `server/models/Deployment.js` — Deployment Schema

```javascript
// server/models/Deployment.js
// Records each deployment event — linked to a Pipeline.

import mongoose from 'mongoose';

const deploymentSchema = new mongoose.Schema(
  {
    pipeline: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Pipeline',
      required: [true, 'Pipeline reference is required'],
    },
    version: {
      type: String,
      required: [true, 'Deployment version is required'],
      trim: true,
    },
    environment: {
      type: String,
      enum: ['development', 'staging', 'production'],
      required: [true, 'Environment is required'],
    },
    status: {
      type: String,
      enum: ['pending', 'in_progress', 'success', 'failed', 'rolled_back'],
      default: 'pending',
    },
    deployedBy: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    commitHash: {
      type: String,
      trim: true,
      maxlength: [40, 'Commit hash cannot exceed 40 characters'],
    },
    commitMessage: { type: String, trim: true },
    duration: { type: Number, default: 0 },   // seconds
    startedAt: { type: Date },
    finishedAt: { type: Date },
    notes: { type: String, trim: true, maxlength: [1000] },
    rollbackOf: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Deployment',
      default: null,
    },
  },
  {
    timestamps: true,
    versionKey: false,
  }
);

deploymentSchema.index({ pipeline: 1, environment: 1 });
deploymentSchema.index({ status: 1 });
deploymentSchema.index({ createdAt: -1 }); // Latest first

const Deployment = mongoose.model('Deployment', deploymentSchema);
export default Deployment;
```

### 8.6 `server/models/Metric.js` — Metric Schema

```javascript
// server/models/Metric.js
// Stores time-series server health data snapshots.

import mongoose from 'mongoose';

const metricSchema = new mongoose.Schema(
  {
    server: {
      type: String,
      required: true,
      default: 'primary',
    },
    cpu: {
      usage: { type: Number, min: 0, max: 100 },       // percentage
      cores: { type: Number },
    },
    memory: {
      used: { type: Number },    // bytes
      total: { type: Number },   // bytes
      usagePercent: { type: Number, min: 0, max: 100 },
    },
    disk: {
      used: { type: Number },
      total: { type: Number },
      usagePercent: { type: Number, min: 0, max: 100 },
    },
    network: {
      bytesIn: { type: Number, default: 0 },
      bytesOut: { type: Number, default: 0 },
    },
    uptime: { type: Number },          // seconds
    requestsPerMinute: { type: Number, default: 0 },
    activeConnections: { type: Number, default: 0 },
    recordedAt: {
      type: Date,
      default: Date.now,
      index: true,
    },
  },
  {
    versionKey: false,
    // Auto-expire metrics after 30 days (TTL index)
    // Keeps the collection from growing indefinitely
  }
);

// TTL index: MongoDB will automatically delete documents 30 days after recordedAt
metricSchema.index({ recordedAt: 1 }, { expireAfterSeconds: 2592000 });

const Metric = mongoose.model('Metric', metricSchema);
export default Metric;
```

### 8.7 `server/utils/generateToken.js` — JWT Utilities

```javascript
// server/utils/generateToken.js
// Creates JWT tokens and sends them as httpOnly cookies.
// httpOnly cookies are inaccessible to JavaScript (XSS-safe).

import jwt from 'jsonwebtoken';

// Generate JWT token string
export const generateToken = (userId) => {
  return jwt.sign(
    { id: userId },                       // Payload: only store user ID
    process.env.JWT_SECRET,               // Secret key from environment
    { expiresIn: process.env.JWT_EXPIRES_IN || '7d' } // Token expiry
  );
};

// Attach JWT to response as httpOnly cookie
export const sendTokenCookie = (res, userId) => {
  const token = generateToken(userId);

  const cookieOptions = {
    expires: new Date(
      Date.now() + (parseInt(process.env.JWT_COOKIE_EXPIRES) || 7) * 24 * 60 * 60 * 1000
    ),
    httpOnly: true,       // Cannot be accessed by client JavaScript
    secure: process.env.NODE_ENV === 'production', // HTTPS only in production
    sameSite: process.env.NODE_ENV === 'production' ? 'strict' : 'lax',
  };

  res.cookie('jwt', token, cookieOptions);
  return token;
};

// Clear the JWT cookie (used on logout)
export const clearTokenCookie = (res) => {
  res.cookie('jwt', 'loggedout', {
    expires: new Date(Date.now() + 10 * 1000), // Expires in 10 seconds
    httpOnly: true,
  });
};
```

### 8.8 `server/utils/asyncHandler.js` — Async Error Wrapper

```javascript
// server/utils/asyncHandler.js
// Wraps async route handlers to catch errors and pass them
// to Express's next() — eliminating repetitive try/catch blocks.
//
// Usage: router.get('/route', asyncHandler(async (req, res) => { ... }))

const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

export default asyncHandler;
```

### 8.9 `server/utils/ApiResponse.js` — Standardized Response

```javascript
// server/utils/ApiResponse.js
// Provides consistent JSON response structure across all endpoints.
// Every API response follows: { success, message, data, meta }

export class ApiResponse {
  // 200 OK — successful request with data
  static success(res, message = 'Success', data = null, statusCode = 200) {
    return res.status(statusCode).json({
      success: true,
      message,
      data,
    });
  }

  // 201 Created — resource created successfully
  static created(res, message = 'Created successfully', data = null) {
    return res.status(201).json({
      success: true,
      message,
      data,
    });
  }

  // Paginated list response
  static paginated(res, message = 'Success', data = [], pagination = {}) {
    return res.status(200).json({
      success: true,
      message,
      data,
      pagination: {
        total: pagination.total || 0,
        page: pagination.page || 1,
        limit: pagination.limit || 10,
        totalPages: Math.ceil((pagination.total || 0) / (pagination.limit || 10)),
      },
    });
  }

  // Error response
  static error(res, message = 'An error occurred', statusCode = 500, errors = null) {
    const response = { success: false, message };
    if (errors) response.errors = errors;
    return res.status(statusCode).json(response);
  }
}
```

### 8.10 `server/middleware/authMiddleware.js` — JWT Auth Guard

```javascript
// server/middleware/authMiddleware.js
// Verifies JWT from httpOnly cookie or Authorization header.
// Attaches the authenticated user to req.user.

import jwt from 'jsonwebtoken';
import asyncHandler from '../utils/asyncHandler.js';
import User from '../models/User.js';

export const protect = asyncHandler(async (req, res, next) => {
  let token;

  // 1. Check httpOnly cookie (preferred — XSS-safe)
  if (req.cookies?.jwt && req.cookies.jwt !== 'loggedout') {
    token = req.cookies.jwt;
  }
  // 2. Fallback: Check Authorization header (for API clients like Postman)
  else if (req.headers.authorization?.startsWith('Bearer ')) {
    token = req.headers.authorization.split(' ')[1];
  }

  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access denied. Please log in to continue.',
    });
  }

  // Verify the token signature and expiry
  const decoded = jwt.verify(token, process.env.JWT_SECRET);

  // Check if the user still exists in the database
  const currentUser = await User.findById(decoded.id).select('+passwordChangedAt');

  if (!currentUser) {
    return res.status(401).json({
      success: false,
      message: 'The user belonging to this token no longer exists.',
    });
  }

  // Check if the user changed password after the token was issued
  if (currentUser.passwordChangedAfter(decoded.iat)) {
    return res.status(401).json({
      success: false,
      message: 'Password was recently changed. Please log in again.',
    });
  }

  // Attach user to request — available in all subsequent middleware/controllers
  req.user = currentUser;
  next();
});
```

### 8.11 `server/middleware/roleMiddleware.js` — RBAC

```javascript
// server/middleware/roleMiddleware.js
// Role-Based Access Control middleware.
// Use after protect middleware: router.delete('/', protect, restrictTo('admin'), handler)

export const restrictTo = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        message: `Access forbidden. Required role: ${roles.join(' or ')}.`,
      });
    }
    next();
  };
};
```

### 8.12 `server/middleware/errorMiddleware.js` — Global Error Handler

```javascript
// server/middleware/errorMiddleware.js
// Centralized error handling for the entire Express app.
// Catches errors thrown anywhere in routes or middleware.

// 404 handler — called when no route matches
export const notFound = (req, res, next) => {
  const error = new Error(`Route not found: ${req.originalUrl}`);
  error.statusCode = 404;
  next(error);
};

// Global error handler — Express recognizes it by the (err, req, res, next) signature
export const errorHandler = (err, req, res, next) => {
  let statusCode = err.statusCode || err.status || 500;
  let message = err.message || 'Internal Server Error';
  let errors = null;

  // ─── Mongoose Validation Error ────────────────────────────
  if (err.name === 'ValidationError') {
    statusCode = 400;
    message = 'Validation failed';
    errors = Object.values(err.errors).map((e) => ({
      field: e.path,
      message: e.message,
    }));
  }

  // ─── Mongoose Duplicate Key Error ─────────────────────────
  if (err.code === 11000) {
    statusCode = 409;
    const field = Object.keys(err.keyValue)[0];
    message = `${field.charAt(0).toUpperCase() + field.slice(1)} already exists`;
  }

  // ─── Mongoose Cast Error (invalid ObjectId) ───────────────
  if (err.name === 'CastError') {
    statusCode = 400;
    message = `Invalid ${err.path}: ${err.value}`;
  }

  // ─── JWT Errors ───────────────────────────────────────────
  if (err.name === 'JsonWebTokenError') {
    statusCode = 401;
    message = 'Invalid token. Please log in again.';
  }

  if (err.name === 'TokenExpiredError') {
    statusCode = 401;
    message = 'Token expired. Please log in again.';
  }

  // Log error in development for debugging
  if (process.env.NODE_ENV === 'development') {
    console.error(`[ERROR] ${statusCode} — ${message}`, err.stack);
  }

  const response = { success: false, message };
  if (errors) response.errors = errors;
  if (process.env.NODE_ENV === 'development') response.stack = err.stack;

  res.status(statusCode).json(response);
};
```

### 8.13 `server/middleware/validateMiddleware.js` — Input Validation

```javascript
// server/middleware/validateMiddleware.js
// Lightweight request body validators.
// Returns 422 Unprocessable Entity if validation fails.

export const validateRegister = (req, res, next) => {
  const { name, email, password } = req.body;
  const errors = [];

  if (!name || name.trim().length < 2)
    errors.push({ field: 'name', message: 'Name must be at least 2 characters' });

  if (!email || !/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(email))
    errors.push({ field: 'email', message: 'Valid email is required' });

  if (!password || password.length < 8)
    errors.push({ field: 'password', message: 'Password must be at least 8 characters' });

  if (errors.length > 0) {
    return res.status(422).json({ success: false, message: 'Validation failed', errors });
  }
  next();
};

export const validateLogin = (req, res, next) => {
  const { email, password } = req.body;
  const errors = [];

  if (!email) errors.push({ field: 'email', message: 'Email is required' });
  if (!password) errors.push({ field: 'password', message: 'Password is required' });

  if (errors.length > 0) {
    return res.status(422).json({ success: false, message: 'Validation failed', errors });
  }
  next();
};

export const validatePipeline = (req, res, next) => {
  const { name, repository } = req.body;
  const errors = [];

  if (!name || name.trim().length < 2)
    errors.push({ field: 'name', message: 'Pipeline name must be at least 2 characters' });

  if (!repository || !repository.startsWith('http'))
    errors.push({ field: 'repository', message: 'Valid repository URL is required' });

  if (errors.length > 0) {
    return res.status(422).json({ success: false, message: 'Validation failed', errors });
  }
  next();
};
```

### 8.14 `server/controllers/authController.js` — Auth Controller

```javascript
// server/controllers/authController.js
// Handles: Register, Login, Logout, Get Current User, Update Profile

import User from '../models/User.js';
import asyncHandler from '../utils/asyncHandler.js';
import { sendTokenCookie, clearTokenCookie } from '../utils/generateToken.js';
import { ApiResponse } from '../utils/ApiResponse.js';

// ─── POST /api/v1/auth/register ───────────────────────────────
export const register = asyncHandler(async (req, res) => {
  const { name, email, password, role } = req.body;

  // Check if user already exists
  const existingUser = await User.findOne({ email: email.toLowerCase() });
  if (existingUser) {
    return ApiResponse.error(res, 'Email is already registered', 409);
  }

  // Create user — password is hashed by the pre-save hook in User model
  const user = await User.create({
    name: name.trim(),
    email: email.toLowerCase().trim(),
    password,
    role: role || 'developer',
  });

  // Send JWT as httpOnly cookie
  sendTokenCookie(res, user._id);

  ApiResponse.created(res, 'Account created successfully', {
    user: {
      _id: user._id,
      name: user.name,
      email: user.email,
      role: user.role,
    },
  });
});

// ─── POST /api/v1/auth/login ──────────────────────────────────
export const login = asyncHandler(async (req, res) => {
  const { email, password } = req.body;

  // Find user and include password (excluded by default via select: false)
  const user = await User.findOne({ email: email.toLowerCase() }).select('+password');

  if (!user || !(await user.comparePassword(password))) {
    // Generic message prevents user enumeration attacks
    return ApiResponse.error(res, 'Invalid email or password', 401);
  }

  if (!user.isActive) {
    return ApiResponse.error(res, 'Your account has been deactivated. Contact support.', 403);
  }

  // Update last login timestamp
  user.lastLogin = new Date();
  await user.save({ validateBeforeSave: false });

  sendTokenCookie(res, user._id);

  ApiResponse.success(res, 'Logged in successfully', {
    user: {
      _id: user._id,
      name: user.name,
      email: user.email,
      role: user.role,
      lastLogin: user.lastLogin,
    },
  });
});

// ─── POST /api/v1/auth/logout ─────────────────────────────────
export const logout = asyncHandler(async (req, res) => {
  clearTokenCookie(res);
  ApiResponse.success(res, 'Logged out successfully');
});

// ─── GET /api/v1/auth/me ──────────────────────────────────────
export const getMe = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id);
  ApiResponse.success(res, 'User profile fetched', { user });
});

// ─── PATCH /api/v1/auth/update-profile ───────────────────────
export const updateProfile = asyncHandler(async (req, res) => {
  const { name, avatar } = req.body;

  const updatedUser = await User.findByIdAndUpdate(
    req.user._id,
    { name: name?.trim(), avatar },
    { new: true, runValidators: true }
  );

  ApiResponse.success(res, 'Profile updated successfully', { user: updatedUser });
});

// ─── PATCH /api/v1/auth/change-password ──────────────────────
export const changePassword = asyncHandler(async (req, res) => {
  const { currentPassword, newPassword } = req.body;

  const user = await User.findById(req.user._id).select('+password');

  if (!(await user.comparePassword(currentPassword))) {
    return ApiResponse.error(res, 'Current password is incorrect', 401);
  }

  user.password = newPassword;
  await user.save(); // Triggers pre-save hook to hash new password

  sendTokenCookie(res, user._id);
  ApiResponse.success(res, 'Password changed successfully');
});
```

### 8.15 `server/controllers/pipelineController.js` — Pipeline Controller

```javascript
// server/controllers/pipelineController.js
// Full CRUD for Pipeline documents.

import Pipeline from '../models/Pipeline.js';
import asyncHandler from '../utils/asyncHandler.js';
import { ApiResponse } from '../utils/ApiResponse.js';

// ─── GET /api/v1/pipelines ────────────────────────────────────
export const getPipelines = asyncHandler(async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const filter = { isActive: true };
  if (req.query.status) filter.status = req.query.status;
  if (req.query.environment) filter.environment = req.query.environment;

  const [pipelines, total] = await Promise.all([
    Pipeline.find(filter)
      .populate('triggeredBy', 'name email role')
      .sort({ updatedAt: -1 })
      .skip(skip)
      .limit(limit),
    Pipeline.countDocuments(filter),
  ]);

  ApiResponse.paginated(res, 'Pipelines fetched', pipelines, { total, page, limit });
});

// ─── GET /api/v1/pipelines/:id ────────────────────────────────
export const getPipelineById = asyncHandler(async (req, res) => {
  const pipeline = await Pipeline.findById(req.params.id)
    .populate('triggeredBy', 'name email');

  if (!pipeline) {
    return ApiResponse.error(res, 'Pipeline not found', 404);
  }

  ApiResponse.success(res, 'Pipeline fetched', { pipeline });
});

// ─── POST /api/v1/pipelines ───────────────────────────────────
export const createPipeline = asyncHandler(async (req, res) => {
  const { name, description, repository, branch, environment, stages, tags } = req.body;

  const pipeline = await Pipeline.create({
    name,
    description,
    repository,
    branch: branch || 'main',
    environment: environment || 'development',
    stages: stages || [
      { name: 'Build', status: 'pending' },
      { name: 'Test', status: 'pending' },
      { name: 'Deploy', status: 'pending' },
    ],
    tags: tags || [],
    triggeredBy: req.user._id,
  });

  const populated = await pipeline.populate('triggeredBy', 'name email');
  ApiResponse.created(res, 'Pipeline created successfully', { pipeline: populated });
});

// ─── PUT /api/v1/pipelines/:id ────────────────────────────────
export const updatePipeline = asyncHandler(async (req, res) => {
  const pipeline = await Pipeline.findByIdAndUpdate(
    req.params.id,
    { ...req.body, updatedAt: new Date() },
    { new: true, runValidators: true }
  ).populate('triggeredBy', 'name email');

  if (!pipeline) {
    return ApiResponse.error(res, 'Pipeline not found', 404);
  }

  ApiResponse.success(res, 'Pipeline updated', { pipeline });
});

// ─── PATCH /api/v1/pipelines/:id/trigger ─────────────────────
export const triggerPipeline = asyncHandler(async (req, res) => {
  const pipeline = await Pipeline.findById(req.params.id);

  if (!pipeline) return ApiResponse.error(res, 'Pipeline not found', 404);

  if (pipeline.status === 'running') {
    return ApiResponse.error(res, 'Pipeline is already running', 400);
  }

  // Simulate triggering the pipeline
  pipeline.status = 'running';
  pipeline.lastRunAt = new Date();
  pipeline.runCount += 1;
  pipeline.stages = pipeline.stages.map((s) => ({ ...s.toObject(), status: 'pending' }));
  await pipeline.save();

  ApiResponse.success(res, 'Pipeline triggered successfully', { pipeline });
});

// ─── DELETE /api/v1/pipelines/:id ────────────────────────────
export const deletePipeline = asyncHandler(async (req, res) => {
  const pipeline = await Pipeline.findByIdAndUpdate(
    req.params.id,
    { isActive: false },
    { new: true }
  );

  if (!pipeline) return ApiResponse.error(res, 'Pipeline not found', 404);

  ApiResponse.success(res, 'Pipeline deleted');
});

// ─── GET /api/v1/pipelines/stats ─────────────────────────────
export const getPipelineStats = asyncHandler(async (req, res) => {
  const stats = await Pipeline.aggregate([
    { $match: { isActive: true } },
    {
      $group: {
        _id: '$status',
        count: { $sum: 1 },
        avgDuration: { $avg: '$duration' },
      },
    },
  ]);

  const formatted = stats.reduce((acc, s) => {
    acc[s._id] = { count: s.count, avgDuration: Math.round(s.avgDuration || 0) };
    return acc;
  }, {});

  ApiResponse.success(res, 'Pipeline stats fetched', { stats: formatted });
});
```

### 8.16 `server/controllers/deploymentController.js` — Deployment Controller

```javascript
// server/controllers/deploymentController.js
// CRUD operations for Deployment records.

import Deployment from '../models/Deployment.js';
import asyncHandler from '../utils/asyncHandler.js';
import { ApiResponse } from '../utils/ApiResponse.js';

// ─── GET /api/v1/deployments ──────────────────────────────────
export const getDeployments = asyncHandler(async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const filter = {};
  if (req.query.environment) filter.environment = req.query.environment;
  if (req.query.status) filter.status = req.query.status;
  if (req.query.pipeline) filter.pipeline = req.query.pipeline;

  const [deployments, total] = await Promise.all([
    Deployment.find(filter)
      .populate('pipeline', 'name repository')
      .populate('deployedBy', 'name email')
      .sort({ createdAt: -1 })
      .skip(skip)
      .limit(limit),
    Deployment.countDocuments(filter),
  ]);

  ApiResponse.paginated(res, 'Deployments fetched', deployments, { total, page, limit });
});

// ─── GET /api/v1/deployments/:id ─────────────────────────────
export const getDeploymentById = asyncHandler(async (req, res) => {
  const deployment = await Deployment.findById(req.params.id)
    .populate('pipeline', 'name repository branch')
    .populate('deployedBy', 'name email role');

  if (!deployment) return ApiResponse.error(res, 'Deployment not found', 404);

  ApiResponse.success(res, 'Deployment fetched', { deployment });
});

// ─── POST /api/v1/deployments ─────────────────────────────────
export const createDeployment = asyncHandler(async (req, res) => {
  const { pipeline, version, environment, commitHash, commitMessage, notes } = req.body;

  const deployment = await Deployment.create({
    pipeline,
    version,
    environment,
    commitHash,
    commitMessage,
    notes,
    deployedBy: req.user._id,
    status: 'pending',
    startedAt: new Date(),
  });

  const populated = await deployment.populate([
    { path: 'pipeline', select: 'name repository' },
    { path: 'deployedBy', select: 'name email' },
  ]);

  ApiResponse.created(res, 'Deployment created', { deployment: populated });
});

// ─── PATCH /api/v1/deployments/:id/status ────────────────────
export const updateDeploymentStatus = asyncHandler(async (req, res) => {
  const { status, notes } = req.body;
  const validStatuses = ['pending', 'in_progress', 'success', 'failed', 'rolled_back'];

  if (!validStatuses.includes(status)) {
    return ApiResponse.error(res, `Invalid status. Must be: ${validStatuses.join(', ')}`, 400);
  }

  const update = { status };
  if (notes) update.notes = notes;
  if (status === 'success' || status === 'failed') {
    update.finishedAt = new Date();
  }

  const deployment = await Deployment.findByIdAndUpdate(req.params.id, update, {
    new: true,
    runValidators: true,
  });

  if (!deployment) return ApiResponse.error(res, 'Deployment not found', 404);

  ApiResponse.success(res, 'Deployment status updated', { deployment });
});

// ─── GET /api/v1/deployments/stats ────────────────────────────
export const getDeploymentStats = asyncHandler(async (req, res) => {
  const thirtyDaysAgo = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);

  const [byEnvironment, byStatus, recentCount] = await Promise.all([
    Deployment.aggregate([
      { $group: { _id: '$environment', count: { $sum: 1 } } },
    ]),
    Deployment.aggregate([
      { $group: { _id: '$status', count: { $sum: 1 } } },
    ]),
    Deployment.countDocuments({ createdAt: { $gte: thirtyDaysAgo } }),
  ]);

  ApiResponse.success(res, 'Deployment stats fetched', {
    stats: { byEnvironment, byStatus, last30Days: recentCount },
  });
});
```

### 8.17 `server/controllers/metricsController.js` — Metrics Controller

```javascript
// server/controllers/metricsController.js
// Returns server health metrics — either simulated or from Metric model.

import os from 'os';
import Metric from '../models/Metric.js';
import asyncHandler from '../utils/asyncHandler.js';
import { ApiResponse } from '../utils/ApiResponse.js';

// ─── GET /api/v1/metrics/live ─────────────────────────────────
// Returns live server metrics using Node.js 'os' module
export const getLiveMetrics = asyncHandler(async (req, res) => {
  const totalMem = os.totalmem();
  const freeMem = os.freemem();
  const usedMem = totalMem - freeMem;
  const memPercent = ((usedMem / totalMem) * 100).toFixed(1);

  const cpuUsage = os.loadavg()[0]; // 1-minute load average

  const metrics = {
    server: os.hostname(),
    cpu: {
      usage: Math.min(parseFloat((cpuUsage * 10).toFixed(1)), 100),
      cores: os.cpus().length,
      model: os.cpus()[0]?.model || 'Unknown',
    },
    memory: {
      used: usedMem,
      total: totalMem,
      free: freeMem,
      usagePercent: parseFloat(memPercent),
    },
    uptime: Math.floor(os.uptime()),
    platform: os.platform(),
    arch: os.arch(),
    nodeVersion: process.version,
    recordedAt: new Date().toISOString(),
  };

  // Save snapshot to DB for historical tracking
  await Metric.create({
    server: os.hostname(),
    cpu: { usage: metrics.cpu.usage, cores: metrics.cpu.cores },
    memory: {
      used: usedMem,
      total: totalMem,
      usagePercent: parseFloat(memPercent),
    },
    uptime: metrics.uptime,
  });

  ApiResponse.success(res, 'Live metrics fetched', { metrics });
});

// ─── GET /api/v1/metrics/history ─────────────────────────────
// Returns historical metrics for charting
export const getMetricsHistory = asyncHandler(async (req, res) => {
  const hours = parseInt(req.query.hours) || 24;
  const since = new Date(Date.now() - hours * 60 * 60 * 1000);

  const history = await Metric.find({ recordedAt: { $gte: since } })
    .select('cpu.usage memory.usagePercent uptime recordedAt')
    .sort({ recordedAt: 1 })
    .limit(100);

  // If no real data, generate mock data for demonstration
  const data = history.length > 0 ? history : generateMockHistory(24);

  ApiResponse.success(res, 'Metrics history fetched', { history: data });
});

// ─── GET /api/v1/metrics/summary ──────────────────────────────
// Returns aggregate stats
export const getMetricsSummary = asyncHandler(async (req, res) => {
  const oneDayAgo = new Date(Date.now() - 24 * 60 * 60 * 1000);

  const summary = await Metric.aggregate([
    { $match: { recordedAt: { $gte: oneDayAgo } } },
    {
      $group: {
        _id: null,
        avgCpu: { $avg: '$cpu.usage' },
        maxCpu: { $max: '$cpu.usage' },
        avgMem: { $avg: '$memory.usagePercent' },
        maxMem: { $max: '$memory.usagePercent' },
        count: { $sum: 1 },
      },
    },
  ]);

  ApiResponse.success(res, 'Metrics summary fetched', {
    summary: summary[0] || { avgCpu: 0, maxCpu: 0, avgMem: 0, maxMem: 0 },
  });
});

// Helper: Generate mock time-series data for demo purposes
const generateMockHistory = (hours) => {
  return Array.from({ length: hours }, (_, i) => ({
    recordedAt: new Date(Date.now() - (hours - i) * 60 * 60 * 1000),
    'cpu.usage': Math.floor(20 + Math.random() * 60),
    'memory.usagePercent': Math.floor(40 + Math.random() * 40),
  }));
};
```

### 8.18 `server/routes/authRoutes.js`

```javascript
// server/routes/authRoutes.js
import express from 'express';
import {
  register, login, logout, getMe, updateProfile, changePassword,
} from '../controllers/authController.js';
import { protect } from '../middleware/authMiddleware.js';
import { validateRegister, validateLogin } from '../middleware/validateMiddleware.js';

const router = express.Router();

// Public routes — no authentication required
router.post('/register', validateRegister, register);
router.post('/login', validateLogin, login);
router.post('/logout', logout);

// Protected routes — JWT required
router.get('/me', protect, getMe);
router.patch('/update-profile', protect, updateProfile);
router.patch('/change-password', protect, changePassword);

export default router;
```

### 8.19 `server/routes/pipelineRoutes.js`

```javascript
// server/routes/pipelineRoutes.js
import express from 'express';
import {
  getPipelines, getPipelineById, createPipeline,
  updatePipeline, deletePipeline, triggerPipeline, getPipelineStats,
} from '../controllers/pipelineController.js';
import { protect } from '../middleware/authMiddleware.js';
import { restrictTo } from '../middleware/roleMiddleware.js';
import { validatePipeline } from '../middleware/validateMiddleware.js';

const router = express.Router();

// All pipeline routes require authentication
router.use(protect);

router.get('/stats', getPipelineStats);
router.get('/', getPipelines);
router.get('/:id', getPipelineById);
router.post('/', validatePipeline, createPipeline);
router.put('/:id', validatePipeline, updatePipeline);
router.patch('/:id/trigger', triggerPipeline);
router.delete('/:id', restrictTo('admin'), deletePipeline);

export default router;
```

### 8.20 `server/routes/deploymentRoutes.js`

```javascript
// server/routes/deploymentRoutes.js
import express from 'express';
import {
  getDeployments, getDeploymentById, createDeployment,
  updateDeploymentStatus, getDeploymentStats,
} from '../controllers/deploymentController.js';
import { protect } from '../middleware/authMiddleware.js';
import { restrictTo } from '../middleware/roleMiddleware.js';

const router = express.Router();

router.use(protect);

router.get('/stats', getDeploymentStats);
router.get('/', getDeployments);
router.get('/:id', getDeploymentById);
router.post('/', createDeployment);
router.patch('/:id/status', updateDeploymentStatus);

export default router;
```

### 8.21 `server/routes/metricsRoutes.js`

```javascript
// server/routes/metricsRoutes.js
import express from 'express';
import {
  getLiveMetrics, getMetricsHistory, getMetricsSummary,
} from '../controllers/metricsController.js';
import { protect } from '../middleware/authMiddleware.js';

const router = express.Router();

router.use(protect);

router.get('/live', getLiveMetrics);
router.get('/history', getMetricsHistory);
router.get('/summary', getMetricsSummary);

export default router;
```

---

## 9. Frontend Source Code

### 9.1 `client/vite.config.js`

```javascript
// client/vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
  server: {
    port: 5173,
    // Proxy API requests to Express backend during development
    // Eliminates CORS issues in development mode
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
        secure: false,
      },
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: false,
    rollupOptions: {
      output: {
        // Code splitting: vendor chunk for React libs
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          charts: ['recharts'],
        },
      },
    },
  },
})
```

### 9.2 `client/tailwind.config.js`

```javascript
// client/tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        // Brand colors — consistent across the dashboard
        brand: {
          50:  '#eff6ff',
          100: '#dbeafe',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          900: '#1e3a8a',
        },
        // Status colors for pipeline badges
        success: '#22c55e',
        warning: '#f59e0b',
        danger:  '#ef4444',
        info:    '#06b6d4',
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      animation: {
        'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
        'fade-in': 'fadeIn 0.3s ease-in-out',
      },
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0', transform: 'translateY(4px)' },
          '100%': { opacity: '1', transform: 'translateY(0)' },
        },
      },
    },
  },
  plugins: [],
}
```

### 9.3 `client/src/index.css`

```css
/* client/src/index.css */
@import "tailwindcss";

/* Google Fonts — Inter for UI, JetBrains Mono for code */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

/* ─── Base Styles ─────────────────────────────────────────── */
*, *::before, *::after {
  box-sizing: border-box;
}

html {
  font-family: 'Inter', system-ui, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  @apply bg-gray-950 text-gray-100 min-h-screen;
}

/* ─── Scrollbar Styling ───────────────────────────────────── */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}
::-webkit-scrollbar-track { @apply bg-gray-900; }
::-webkit-scrollbar-thumb { @apply bg-gray-700 rounded-full; }
::-webkit-scrollbar-thumb:hover { @apply bg-gray-500; }

/* ─── Custom Component Classes ────────────────────────────── */
.card {
  @apply bg-gray-900 border border-gray-800 rounded-xl p-5;
}

.btn-primary {
  @apply bg-blue-600 hover:bg-blue-700 text-white font-medium px-4 py-2
         rounded-lg transition-colors duration-200 focus:outline-none
         focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
         focus:ring-offset-gray-900 disabled:opacity-50 disabled:cursor-not-allowed;
}

.btn-secondary {
  @apply bg-gray-700 hover:bg-gray-600 text-gray-100 font-medium px-4 py-2
         rounded-lg transition-colors duration-200;
}

.btn-danger {
  @apply bg-red-600 hover:bg-red-700 text-white font-medium px-4 py-2
         rounded-lg transition-colors duration-200;
}

.input-field {
  @apply w-full bg-gray-800 border border-gray-700 text-gray-100
         placeholder-gray-500 rounded-lg px-4 py-2.5 text-sm
         focus:outline-none focus:border-blue-500 focus:ring-1
         focus:ring-blue-500 transition-colors duration-200;
}

.label {
  @apply block text-sm font-medium text-gray-300 mb-1.5;
}
```

### 9.4 `client/src/api/axios.js` — Axios Instance

```javascript
// client/src/api/axios.js
// Configured Axios instance with base URL, interceptors, and
// automatic token refresh / error handling.

import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:5000/api/v1',
  withCredentials: true,   // Send httpOnly cookies with every request
  timeout: 15000,          // 15-second request timeout
  headers: {
    'Content-Type': 'application/json',
  },
});

// ─── Request Interceptor ──────────────────────────────────────
// Runs before every request — can add auth headers here if needed
api.interceptors.request.use(
  (config) => {
    // In cookie-based auth, no Authorization header needed
    // But if you switch to localStorage tokens, add it here:
    // const token = localStorage.getItem('token');
    // if (token) config.headers.Authorization = `Bearer ${token}`;
    return config;
  },
  (error) => Promise.reject(error)
);

// ─── Response Interceptor ─────────────────────────────────────
// Runs on every response — handles global errors
api.interceptors.response.use(
  (response) => response,
  (error) => {
    const status = error.response?.status;

    if (status === 401) {
      // Token expired or invalid — redirect to login
      // Don't redirect if already on login page
      if (!window.location.pathname.includes('/login')) {
        window.location.href = '/login';
      }
    }

    if (status === 403) {
      console.warn('Access forbidden');
    }

    if (status >= 500) {
      console.error('Server error:', error.response?.data?.message);
    }

    return Promise.reject(error);
  }
);

export default api;
```

### 9.5 `client/src/context/AuthContext.jsx` — Auth Context

```jsx
// client/src/context/AuthContext.jsx
// Global authentication state using React Context.
// Wraps the entire app so any component can access auth state.

import { createContext, useContext, useState, useEffect, useCallback } from 'react';
import api from '../api/axios';

const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);  // True during initial auth check
  const [error, setError] = useState(null);

  // Check if user is already authenticated on app load
  const checkAuth = useCallback(async () => {
    try {
      const { data } = await api.get('/auth/me');
      setUser(data.data.user);
    } catch {
      setUser(null);  // 401 = not logged in
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    checkAuth();
  }, [checkAuth]);

  const login = async (email, password) => {
    setError(null);
    const { data } = await api.post('/auth/login', { email, password });
    setUser(data.data.user);
    return data;
  };

  const register = async (name, email, password) => {
    setError(null);
    const { data } = await api.post('/auth/register', { name, email, password });
    setUser(data.data.user);
    return data;
  };

  const logout = async () => {
    await api.post('/auth/logout');
    setUser(null);
  };

  const updateUser = (updatedUser) => {
    setUser((prev) => ({ ...prev, ...updatedUser }));
  };

  return (
    <AuthContext.Provider value={{ user, loading, error, login, register, logout, updateUser, checkAuth }}>
      {children}
    </AuthContext.Provider>
  );
};

// Custom hook — usage: const { user, login, logout } = useAuth();
export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
};

export default AuthContext;
```

### 9.6 `client/src/routes/ProtectedRoute.jsx`

```jsx
// client/src/routes/ProtectedRoute.jsx
// Redirects unauthenticated users to the login page.
// Renders a spinner while auth status is being determined.

import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
import Spinner from '../components/ui/Spinner';

const ProtectedRoute = ({ children, roles }) => {
  const { user, loading } = useAuth();
  const location = useLocation();

  if (loading) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-gray-950">
        <Spinner size="lg" />
      </div>
    );
  }

  if (!user) {
    // Save the attempted URL to redirect back after login
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  // Role-based access control check
  if (roles && !roles.includes(user.role)) {
    return <Navigate to="/dashboard" replace />;
  }

  return children;
};

export default ProtectedRoute;
```

### 9.7 `client/src/routes/AppRoutes.jsx`

```jsx
// client/src/routes/AppRoutes.jsx
// Central route configuration for the entire application.

import { Routes, Route, Navigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
import ProtectedRoute from './ProtectedRoute';
import Layout from '../components/layout/Layout';

// Pages
import LoginPage from '../pages/LoginPage';
import RegisterPage from '../pages/RegisterPage';
import DashboardPage from '../pages/DashboardPage';
import PipelinesPage from '../pages/PipelinesPage';
import DeploymentsPage from '../pages/DeploymentsPage';
import MetricsPage from '../pages/MetricsPage';
import ProfilePage from '../pages/ProfilePage';
import NotFoundPage from '../pages/NotFoundPage';

const AppRoutes = () => {
  const { user } = useAuth();

  return (
    <Routes>
      {/* Public routes */}
      <Route
        path="/login"
        element={user ? <Navigate to="/dashboard" replace /> : <LoginPage />}
      />
      <Route
        path="/register"
        element={user ? <Navigate to="/dashboard" replace /> : <RegisterPage />}
      />

      {/* Protected routes — wrapped in Layout (Navbar + Sidebar) */}
      <Route
        path="/"
        element={
          <ProtectedRoute>
            <Layout />
          </ProtectedRoute>
        }
      >
        <Route index element={<Navigate to="/dashboard" replace />} />
        <Route path="dashboard" element={<DashboardPage />} />
        <Route path="pipelines" element={<PipelinesPage />} />
        <Route path="deployments" element={<DeploymentsPage />} />
        <Route path="metrics" element={<MetricsPage />} />
        <Route path="profile" element={<ProfilePage />} />
      </Route>

      {/* 404 */}
      <Route path="*" element={<NotFoundPage />} />
    </Routes>
  );
};

export default AppRoutes;
```

### 9.8 `client/src/App.jsx`

```jsx
// client/src/App.jsx
import { BrowserRouter } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import AppRoutes from './routes/AppRoutes';

const App = () => {
  return (
    <BrowserRouter>
      <AuthProvider>
        <AppRoutes />
      </AuthProvider>
    </BrowserRouter>
  );
};

export default App;
```

### 9.9 `client/src/main.jsx`

```jsx
// client/src/main.jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

### 9.10 `client/src/components/layout/Layout.jsx`

```jsx
// client/src/components/layout/Layout.jsx
import { Outlet } from 'react-router-dom';
import { useState } from 'react';
import Navbar from './Navbar';
import Sidebar from './Sidebar';

const Layout = () => {
  const [sidebarOpen, setSidebarOpen] = useState(true);

  return (
    <div className="flex h-screen bg-gray-950 overflow-hidden">
      {/* Sidebar */}
      <Sidebar isOpen={sidebarOpen} onToggle={() => setSidebarOpen(!sidebarOpen)} />

      {/* Main content area */}
      <div className="flex flex-col flex-1 overflow-hidden">
        <Navbar onMenuClick={() => setSidebarOpen(!sidebarOpen)} />
        <main className="flex-1 overflow-y-auto p-6">
          <div className="max-w-7xl mx-auto animate-fade-in">
            <Outlet />
          </div>
        </main>
      </div>
    </div>
  );
};

export default Layout;
```

### 9.11 `client/src/components/layout/Sidebar.jsx`

```jsx
// client/src/components/layout/Sidebar.jsx
import { NavLink } from 'react-router-dom';
import {
  MdDashboard, MdAccountTree, MdRocketLaunch,
  MdShowChart, MdPerson, MdChevronLeft, MdChevronRight,
} from 'react-icons/md';

const navItems = [
  { path: '/dashboard',   label: 'Dashboard',   icon: MdDashboard },
  { path: '/pipelines',   label: 'Pipelines',   icon: MdAccountTree },
  { path: '/deployments', label: 'Deployments', icon: MdRocketLaunch },
  { path: '/metrics',     label: 'Metrics',     icon: MdShowChart },
  { path: '/profile',     label: 'Profile',     icon: MdPerson },
];

const Sidebar = ({ isOpen, onToggle }) => {
  return (
    <aside
      className={`flex flex-col bg-gray-900 border-r border-gray-800 transition-all duration-300 ${
        isOpen ? 'w-60' : 'w-16'
      }`}
    >
      {/* Logo */}
      <div className="flex items-center justify-between h-16 px-4 border-b border-gray-800">
        {isOpen && (
          <div className="flex items-center gap-2">
            <span className="text-blue-500 text-xl font-bold">⬡</span>
            <span className="text-white font-semibold text-sm whitespace-nowrap">
              DevOps Hub
            </span>
          </div>
        )}
        <button
          onClick={onToggle}
          className="p-1.5 rounded-lg text-gray-400 hover:text-white hover:bg-gray-800 transition-colors ml-auto"
        >
          {isOpen ? <MdChevronLeft size={20} /> : <MdChevronRight size={20} />}
        </button>
      </div>

      {/* Navigation */}
      <nav className="flex-1 py-4 space-y-1 px-2">
        {navItems.map(({ path, label, icon: Icon }) => (
          <NavLink
            key={path}
            to={path}
            className={({ isActive }) =>
              `flex items-center gap-3 px-3 py-2.5 rounded-lg transition-all duration-150 group ${
                isActive
                  ? 'bg-blue-600 text-white'
                  : 'text-gray-400 hover:bg-gray-800 hover:text-white'
              }`
            }
          >
            <Icon size={20} className="flex-shrink-0" />
            {isOpen && (
              <span className="text-sm font-medium whitespace-nowrap">{label}</span>
            )}
          </NavLink>
        ))}
      </nav>

      {/* Version badge at bottom */}
      {isOpen && (
        <div className="p-4 border-t border-gray-800">
          <span className="text-xs text-gray-600">v{import.meta.env.VITE_APP_VERSION}</span>
        </div>
      )}
    </aside>
  );
};

export default Sidebar;
```

### 9.12 `client/src/components/layout/Navbar.jsx`

```jsx
// client/src/components/layout/Navbar.jsx
import { MdMenu, MdNotifications, MdPerson, MdLogout } from 'react-icons/md';
import { useAuth } from '../../context/AuthContext';
import { useNavigate } from 'react-router-dom';

const Navbar = ({ onMenuClick }) => {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = async () => {
    await logout();
    navigate('/login');
  };

  return (
    <header className="flex items-center justify-between h-16 px-6 bg-gray-900 border-b border-gray-800 flex-shrink-0">
      {/* Left: Hamburger + Title */}
      <div className="flex items-center gap-4">
        <button
          onClick={onMenuClick}
          className="p-1.5 rounded-lg text-gray-400 hover:text-white hover:bg-gray-800 transition-colors"
        >
          <MdMenu size={22} />
        </button>
        <h1 className="text-base font-semibold text-white hidden sm:block">
          DevOps Pipeline Dashboard
        </h1>
      </div>

      {/* Right: User info */}
      <div className="flex items-center gap-3">
        {/* Notification bell */}
        <button className="p-1.5 rounded-lg text-gray-400 hover:text-white hover:bg-gray-800 relative">
          <MdNotifications size={20} />
          <span className="absolute top-1 right-1 w-2 h-2 bg-blue-500 rounded-full" />
        </button>

        {/* User avatar + name */}
        <div className="flex items-center gap-2 px-3 py-1.5 rounded-lg bg-gray-800">
          <div className="w-7 h-7 rounded-full bg-blue-600 flex items-center justify-center">
            <MdPerson size={16} className="text-white" />
          </div>
          <div className="hidden sm:block">
            <p className="text-xs font-medium text-white">{user?.name}</p>
            <p className="text-xs text-gray-400 capitalize">{user?.role}</p>
          </div>
        </div>

        {/* Logout */}
        <button
          onClick={handleLogout}
          className="p-1.5 rounded-lg text-gray-400 hover:text-red-400 hover:bg-gray-800 transition-colors"
          title="Logout"
        >
          <MdLogout size={20} />
        </button>
      </div>
    </header>
  );
};

export default Navbar;
```

### 9.13 `client/src/components/ui/StatCard.jsx`

```jsx
// client/src/components/ui/StatCard.jsx
// Reusable metric/stat card for the dashboard overview.

const StatCard = ({ title, value, subtitle, icon: Icon, color = 'blue', trend }) => {
  const colorMap = {
    blue:   'text-blue-400 bg-blue-500/10',
    green:  'text-green-400 bg-green-500/10',
    yellow: 'text-yellow-400 bg-yellow-500/10',
    red:    'text-red-400 bg-red-500/10',
    purple: 'text-purple-400 bg-purple-500/10',
    cyan:   'text-cyan-400 bg-cyan-500/10',
  };

  return (
    <div className="card hover:border-gray-700 transition-colors">
      <div className="flex items-start justify-between">
        <div>
          <p className="text-xs font-medium text-gray-400 uppercase tracking-wider">{title}</p>
          <p className="text-3xl font-bold text-white mt-1">{value}</p>
          {subtitle && <p className="text-xs text-gray-500 mt-1">{subtitle}</p>}
        </div>
        {Icon && (
          <div className={`p-2.5 rounded-lg ${colorMap[color] || colorMap.blue}`}>
            <Icon size={22} className={colorMap[color]?.split(' ')[0]} />
          </div>
        )}
      </div>
      {trend !== undefined && (
        <div className="mt-3 flex items-center gap-1">
          <span className={`text-xs font-medium ${trend >= 0 ? 'text-green-400' : 'text-red-400'}`}>
            {trend >= 0 ? '▲' : '▼'} {Math.abs(trend)}%
          </span>
          <span className="text-xs text-gray-500">vs last period</span>
        </div>
      )}
    </div>
  );
};

export default StatCard;
```

### 9.14 `client/src/components/ui/Badge.jsx`

```jsx
// client/src/components/ui/Badge.jsx
// Status badge with color-coded appearance.

const statusStyles = {
  success:     'bg-green-500/15 text-green-400 border-green-500/30',
  failed:      'bg-red-500/15 text-red-400 border-red-500/30',
  running:     'bg-blue-500/15 text-blue-400 border-blue-500/30',
  pending:     'bg-yellow-500/15 text-yellow-400 border-yellow-500/30',
  idle:        'bg-gray-500/15 text-gray-400 border-gray-500/30',
  cancelled:   'bg-orange-500/15 text-orange-400 border-orange-500/30',
  in_progress: 'bg-cyan-500/15 text-cyan-400 border-cyan-500/30',
  rolled_back: 'bg-purple-500/15 text-purple-400 border-purple-500/30',
  skipped:     'bg-gray-500/15 text-gray-400 border-gray-500/30',
};

// Animated pulse dot for active statuses
const activeDot = (status) => ['running', 'in_progress', 'pending'].includes(status);

const Badge = ({ status, label }) => {
  const styles = statusStyles[status] || statusStyles.idle;
  const text = label || status?.replace(/_/g, ' ') || 'unknown';

  return (
    <span className={`inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-medium border ${styles}`}>
      {activeDot(status) && (
        <span className="relative flex h-1.5 w-1.5">
          <span className={`animate-ping absolute inline-flex h-full w-full rounded-full opacity-75 ${styles.split(' ')[1]?.replace('text', 'bg')}`} />
          <span className={`relative inline-flex rounded-full h-1.5 w-1.5 ${styles.split(' ')[1]?.replace('text', 'bg')}`} />
        </span>
      )}
      {text}
    </span>
  );
};

export default Badge;
```

### 9.15 `client/src/components/ui/Spinner.jsx`

```jsx
// client/src/components/ui/Spinner.jsx
const sizeMap = { sm: 'w-4 h-4', md: 'w-8 h-8', lg: 'w-12 h-12' };

const Spinner = ({ size = 'md', label = 'Loading...' }) => (
  <div className="flex flex-col items-center gap-3">
    <div
      className={`${sizeMap[size]} rounded-full border-2 border-gray-700 border-t-blue-500 animate-spin`}
      role="status"
      aria-label={label}
    />
    {size === 'lg' && <p className="text-sm text-gray-400">{label}</p>}
  </div>
);

export default Spinner;
```

### 9.16 `client/src/components/ui/Alert.jsx`

```jsx
// client/src/components/ui/Alert.jsx
import { MdCheckCircle, MdError, MdWarning, MdInfo, MdClose } from 'react-icons/md';

const variants = {
  success: { icon: MdCheckCircle, styles: 'bg-green-500/10 border-green-500/30 text-green-400' },
  error:   { icon: MdError,       styles: 'bg-red-500/10 border-red-500/30 text-red-400' },
  warning: { icon: MdWarning,     styles: 'bg-yellow-500/10 border-yellow-500/30 text-yellow-400' },
  info:    { icon: MdInfo,        styles: 'bg-blue-500/10 border-blue-500/30 text-blue-400' },
};

const Alert = ({ type = 'info', message, onClose }) => {
  const { icon: Icon, styles } = variants[type] || variants.info;

  return (
    <div className={`flex items-start gap-3 p-4 rounded-lg border ${styles} animate-fade-in`}>
      <Icon size={18} className="flex-shrink-0 mt-0.5" />
      <p className="text-sm flex-1">{message}</p>
      {onClose && (
        <button onClick={onClose} className="hover:opacity-70 transition-opacity">
          <MdClose size={16} />
        </button>
      )}
    </div>
  );
};

export default Alert;
```

### 9.17 `client/src/components/charts/CpuChart.jsx`

```jsx
// client/src/components/charts/CpuChart.jsx
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer, ReferenceLine,
} from 'recharts';

const CustomTooltip = ({ active, payload, label }) => {
  if (!active || !payload?.length) return null;
  return (
    <div className="bg-gray-800 border border-gray-700 rounded-lg p-3 text-xs shadow-xl">
      <p className="text-gray-400 mb-1">{label}</p>
      <p className="text-blue-400 font-bold">{payload[0]?.value}% CPU</p>
    </div>
  );
};

const CpuChart = ({ data = [] }) => {
  const chartData = data.map((d, i) => ({
    time: new Date(d.recordedAt || d.time).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
    cpu: d['cpu.usage'] || d.cpu || Math.floor(20 + Math.random() * 60),
  }));

  return (
    <div className="card">
      <div className="flex items-center justify-between mb-4">
        <h3 className="text-sm font-semibold text-white">CPU Usage</h3>
        <span className="text-xs text-gray-500">Last 24 hours</span>
      </div>
      <ResponsiveContainer width="100%" height={200}>
        <LineChart data={chartData} margin={{ top: 5, right: 5, bottom: 5, left: -20 }}>
          <CartesianGrid strokeDasharray="3 3" stroke="#1f2937" />
          <XAxis dataKey="time" tick={{ fill: '#6b7280', fontSize: 10 }} tickLine={false} />
          <YAxis domain={[0, 100]} tick={{ fill: '#6b7280', fontSize: 10 }} tickLine={false} />
          <Tooltip content={<CustomTooltip />} />
          <ReferenceLine y={80} stroke="#ef4444" strokeDasharray="4 4" opacity={0.5} />
          <Line
            type="monotone"
            dataKey="cpu"
            stroke="#3b82f6"
            strokeWidth={2}
            dot={false}
            activeDot={{ r: 4, fill: '#3b82f6' }}
          />
        </LineChart>
      </ResponsiveContainer>
      <p className="text-xs text-gray-500 mt-1">Red line = 80% threshold</p>
    </div>
  );
};

export default CpuChart;
```

### 9.18 `client/src/components/charts/MemoryChart.jsx`

```jsx
// client/src/components/charts/MemoryChart.jsx
import {
  AreaChart, Area, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer,
} from 'recharts';

const CustomTooltip = ({ active, payload }) => {
  if (!active || !payload?.length) return null;
  return (
    <div className="bg-gray-800 border border-gray-700 rounded-lg p-3 text-xs shadow-xl">
      <p className="text-purple-400 font-bold">{payload[0]?.value}% Memory</p>
    </div>
  );
};

const MemoryChart = ({ data = [] }) => {
  const chartData = data.map((d) => ({
    time: new Date(d.recordedAt || d.time).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
    memory: d['memory.usagePercent'] || d.memory || Math.floor(40 + Math.random() * 40),
  }));

  return (
    <div className="card">
      <div className="flex items-center justify-between mb-4">
        <h3 className="text-sm font-semibold text-white">Memory Usage</h3>
        <span className="text-xs text-gray-500">Last 24 hours</span>
      </div>
      <ResponsiveContainer width="100%" height={200}>
        <AreaChart data={chartData} margin={{ top: 5, right: 5, bottom: 5, left: -20 }}>
          <defs>
            <linearGradient id="memGradient" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor="#a855f7" stopOpacity={0.3} />
              <stop offset="95%" stopColor="#a855f7" stopOpacity={0} />
            </linearGradient>
          </defs>
          <CartesianGrid strokeDasharray="3 3" stroke="#1f2937" />
          <XAxis dataKey="time" tick={{ fill: '#6b7280', fontSize: 10 }} tickLine={false} />
          <YAxis domain={[0, 100]} tick={{ fill: '#6b7280', fontSize: 10 }} tickLine={false} />
          <Tooltip content={<CustomTooltip />} />
          <Area
            type="monotone"
            dataKey="memory"
            stroke="#a855f7"
            strokeWidth={2}
            fill="url(#memGradient)"
          />
        </AreaChart>
      </ResponsiveContainer>
    </div>
  );
};

export default MemoryChart;
```

### 9.19 `client/src/components/charts/DeployChart.jsx`

```jsx
// client/src/components/charts/DeployChart.jsx
import {
  BarChart, Bar, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer, Legend,
} from 'recharts';

const CustomTooltip = ({ active, payload, label }) => {
  if (!active || !payload?.length) return null;
  return (
    <div className="bg-gray-800 border border-gray-700 rounded-lg p-3 text-xs shadow-xl">
      <p className="text-gray-300 mb-2 font-medium">{label}</p>
      {payload.map((p) => (
        <p key={p.name} style={{ color: p.fill }}>{p.name}: {p.value}</p>
      ))}
    </div>
  );
};

// Sample deployment data (replace with API data)
const sampleData = [
  { month: 'Jan', success: 12, failed: 2 },
  { month: 'Feb', success: 18, failed: 1 },
  { month: 'Mar', success: 15, failed: 3 },
  { month: 'Apr', success: 22, failed: 0 },
  { month: 'May', success: 19, failed: 2 },
  { month: 'Jun', success: 25, failed: 1 },
];

const DeployChart = ({ data = sampleData }) => (
  <div className="card">
    <div className="flex items-center justify-between mb-4">
      <h3 className="text-sm font-semibold text-white">Deployments by Month</h3>
      <span className="text-xs text-gray-500">Last 6 months</span>
    </div>
    <ResponsiveContainer width="100%" height={200}>
      <BarChart data={data} margin={{ top: 5, right: 5, bottom: 5, left: -20 }}>
        <CartesianGrid strokeDasharray="3 3" stroke="#1f2937" />
        <XAxis dataKey="month" tick={{ fill: '#6b7280', fontSize: 10 }} tickLine={false} />
        <YAxis tick={{ fill: '#6b7280', fontSize: 10 }} tickLine={false} />
        <Tooltip content={<CustomTooltip />} />
        <Legend
          wrapperStyle={{ fontSize: '11px', paddingTop: '8px' }}
          formatter={(v) => <span className="text-gray-400">{v}</span>}
        />
        <Bar dataKey="success" name="Success" fill="#22c55e" radius={[3, 3, 0, 0]} />
        <Bar dataKey="failed" name="Failed" fill="#ef4444" radius={[3, 3, 0, 0]} />
      </BarChart>
    </ResponsiveContainer>
  </div>
);

export default DeployChart;
```

### 9.20 `client/src/components/pipeline/PipelineCard.jsx`

```jsx
// client/src/components/pipeline/PipelineCard.jsx
import { MdPlayArrow, MdBranch, MdAccessTime } from 'react-icons/md';
import { VscGithubAlt } from 'react-icons/vsc';
import Badge from '../ui/Badge';

const formatDuration = (seconds) => {
  if (!seconds) return '—';
  const m = Math.floor(seconds / 60);
  const s = seconds % 60;
  return m > 0 ? `${m}m ${s}s` : `${s}s`;
};

const PipelineCard = ({ pipeline, onTrigger }) => {
  const { name, repository, branch, status, environment, stages, runCount, duration, lastRunAt } = pipeline;

  return (
    <div className="card hover:border-gray-700 transition-all duration-200 group">
      {/* Header */}
      <div className="flex items-start justify-between gap-4">
        <div className="flex-1 min-w-0">
          <div className="flex items-center gap-2 flex-wrap">
            <h3 className="font-semibold text-white text-sm truncate">{name}</h3>
            <Badge status={status} />
            <Badge status={environment} label={environment} />
          </div>
          <div className="flex items-center gap-3 mt-1.5 text-xs text-gray-500">
            <span className="flex items-center gap-1">
              <VscGithubAlt size={13} />
              <span className="truncate max-w-[140px]">{repository}</span>
            </span>
            <span className="flex items-center gap-1">
              <MdBranch size={13} />
              {branch}
            </span>
          </div>
        </div>

        {/* Trigger button */}
        <button
          onClick={() => onTrigger?.(pipeline._id)}
          disabled={status === 'running'}
          className="flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-medium
                     bg-blue-600 hover:bg-blue-700 text-white transition-colors
                     disabled:opacity-40 disabled:cursor-not-allowed flex-shrink-0"
        >
          <MdPlayArrow size={15} />
          Run
        </button>
      </div>

      {/* Stage pipeline visualization */}
      {stages?.length > 0 && (
        <div className="flex items-center gap-1.5 mt-4">
          {stages.map((stage, i) => (
            <div key={i} className="flex items-center gap-1.5 flex-1">
              <div className={`flex-1 h-1.5 rounded-full ${
                stage.status === 'success' ? 'bg-green-500' :
                stage.status === 'failed'  ? 'bg-red-500'   :
                stage.status === 'running' ? 'bg-blue-500 animate-pulse' :
                'bg-gray-700'
              }`} />
              {i < stages.length - 1 && <div className="w-2 h-px bg-gray-700" />}
            </div>
          ))}
        </div>
      )}

      {stages?.length > 0 && (
        <div className="flex gap-3 mt-1">
          {stages.map((stage, i) => (
            <span key={i} className="text-xs text-gray-600 flex-1 text-center truncate">{stage.name}</span>
          ))}
        </div>
      )}

      {/* Footer stats */}
      <div className="flex items-center gap-4 mt-4 pt-3 border-t border-gray-800 text-xs text-gray-500">
        <span>Runs: <span className="text-gray-300">{runCount || 0}</span></span>
        <span className="flex items-center gap-1">
          <MdAccessTime size={12} />
          {formatDuration(duration)}
        </span>
        {lastRunAt && (
          <span className="ml-auto">
            {new Date(lastRunAt).toLocaleDateString()}
          </span>
        )}
      </div>
    </div>
  );
};

export default PipelineCard;
```

### 9.21 `client/src/pages/LoginPage.jsx`

```jsx
// client/src/pages/LoginPage.jsx
import { useState } from 'react';
import { Link, useNavigate, useLocation } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
import Alert from '../components/ui/Alert';
import Spinner from '../components/ui/Spinner';
import { MdEmail, MdLock } from 'react-icons/md';

const LoginPage = () => {
  const [form, setForm] = useState({ email: '', password: '' });
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();

  const from = location.state?.from?.pathname || '/dashboard';

  const handleChange = (e) => {
    setForm((prev) => ({ ...prev, [e.target.name]: e.target.value }));
    setError(null);
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!form.email || !form.password) {
      setError('Please fill in all fields');
      return;
    }
    setLoading(true);
    try {
      await login(form.email, form.password);
      navigate(from, { replace: true });
    } catch (err) {
      setError(err.response?.data?.message || 'Login failed. Please try again.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen bg-gray-950 flex items-center justify-center p-4">
      <div className="w-full max-w-md">
        {/* Brand */}
        <div className="text-center mb-8">
          <div className="inline-flex items-center justify-center w-14 h-14 rounded-2xl bg-blue-600 mb-4">
            <span className="text-white text-2xl font-bold">⬡</span>
          </div>
          <h1 className="text-2xl font-bold text-white">Welcome back</h1>
          <p className="text-gray-400 text-sm mt-1">Sign in to DevOps Pipeline Dashboard</p>
        </div>

        {/* Card */}
        <div className="card space-y-5">
          {error && (
            <Alert type="error" message={error} onClose={() => setError(null)} />
          )}

          <form onSubmit={handleSubmit} className="space-y-4">
            {/* Email */}
            <div>
              <label className="label" htmlFor="email">Email Address</label>
              <div className="relative">
                <MdEmail className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-500" size={16} />
                <input
                  id="email"
                  name="email"
                  type="email"
                  value={form.email}
                  onChange={handleChange}
                  placeholder="you@company.com"
                  className="input-field pl-9"
                  autoComplete="email"
                  required
                />
              </div>
            </div>

            {/* Password */}
            <div>
              <label className="label" htmlFor="password">Password</label>
              <div className="relative">
                <MdLock className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-500" size={16} />
                <input
                  id="password"
                  name="password"
                  type="password"
                  value={form.password}
                  onChange={handleChange}
                  placeholder="••••••••"
                  className="input-field pl-9"
                  autoComplete="current-password"
                  required
                />
              </div>
            </div>

            <button
              type="submit"
              disabled={loading}
              className="btn-primary w-full flex items-center justify-center gap-2 mt-2"
            >
              {loading ? <Spinner size="sm" /> : 'Sign In'}
            </button>
          </form>

          <p className="text-center text-sm text-gray-400">
            Don't have an account?{' '}
            <Link to="/register" className="text-blue-400 hover:text-blue-300 font-medium">
              Create one
            </Link>
          </p>
        </div>

        {/* Demo credentials hint */}
        <div className="mt-4 p-3 bg-gray-900 border border-gray-800 rounded-lg text-center">
          <p className="text-xs text-gray-500">
            Demo: <span className="text-gray-300 font-mono">admin@devops.com</span> / <span className="text-gray-300 font-mono">password123</span>
          </p>
        </div>
      </div>
    </div>
  );
};

export default LoginPage;
```

### 9.22 `client/src/pages/RegisterPage.jsx`

```jsx
// client/src/pages/RegisterPage.jsx
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
import Alert from '../components/ui/Alert';
import Spinner from '../components/ui/Spinner';

const RegisterPage = () => {
  const [form, setForm] = useState({ name: '', email: '', password: '', confirmPassword: '' });
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  const { register } = useAuth();
  const navigate = useNavigate();

  const handleChange = (e) => {
    setForm((prev) => ({ ...prev, [e.target.name]: e.target.value }));
    setError(null);
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (form.password !== form.confirmPassword) {
      setError('Passwords do not match');
      return;
    }
    if (form.password.length < 8) {
      setError('Password must be at least 8 characters');
      return;
    }
    setLoading(true);
    try {
      await register(form.name, form.email, form.password);
      navigate('/dashboard', { replace: true });
    } catch (err) {
      setError(err.response?.data?.message || 'Registration failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="min-h-screen bg-gray-950 flex items-center justify-center p-4">
      <div className="w-full max-w-md">
        <div className="text-center mb-8">
          <div className="inline-flex items-center justify-center w-14 h-14 rounded-2xl bg-blue-600 mb-4">
            <span className="text-white text-2xl font-bold">⬡</span>
          </div>
          <h1 className="text-2xl font-bold text-white">Create account</h1>
          <p className="text-gray-400 text-sm mt-1">Join DevOps Pipeline Dashboard</p>
        </div>

        <div className="card space-y-5">
          {error && <Alert type="error" message={error} onClose={() => setError(null)} />}

          <form onSubmit={handleSubmit} className="space-y-4">
            {[
              { id: 'name',            label: 'Full Name',       type: 'text',     placeholder: 'John Doe' },
              { id: 'email',           label: 'Email Address',   type: 'email',    placeholder: 'you@company.com' },
              { id: 'password',        label: 'Password',        type: 'password', placeholder: '8+ characters' },
              { id: 'confirmPassword', label: 'Confirm Password',type: 'password', placeholder: 'Repeat password' },
            ].map(({ id, label, type, placeholder }) => (
              <div key={id}>
                <label className="label" htmlFor={id}>{label}</label>
                <input
                  id={id}
                  name={id}
                  type={type}
                  value={form[id]}
                  onChange={handleChange}
                  placeholder={placeholder}
                  className="input-field"
                  required
                />
              </div>
            ))}

            <button
              type="submit"
              disabled={loading}
              className="btn-primary w-full flex items-center justify-center gap-2 mt-2"
            >
              {loading ? <Spinner size="sm" /> : 'Create Account'}
            </button>
          </form>

          <p className="text-center text-sm text-gray-400">
            Already have an account?{' '}
            <Link to="/login" className="text-blue-400 hover:text-blue-300 font-medium">
              Sign in
            </Link>
          </p>
        </div>
      </div>
    </div>
  );
};

export default RegisterPage;
```

### 9.23 `client/src/pages/DashboardPage.jsx`

```jsx
// client/src/pages/DashboardPage.jsx
// Main dashboard with stats overview, recent pipelines, and charts.

import { useEffect, useState } from 'react';
import { Link } from 'react-router-dom';
import {
  MdAccountTree, MdRocketLaunch, MdCheckCircle, MdError,
  MdTimeline, MdRefresh,
} from 'react-icons/md';
import api from '../api/axios';
import { useAuth } from '../context/AuthContext';
import StatCard from '../components/ui/StatCard';
import Badge from '../components/ui/Badge';
import CpuChart from '../components/charts/CpuChart';
import MemoryChart from '../components/charts/MemoryChart';
import DeployChart from '../components/charts/DeployChart';
import Spinner from '../components/ui/Spinner';

const DashboardPage = () => {
  const { user } = useAuth();
  const [stats, setStats] = useState(null);
  const [pipelines, setPipelines] = useState([]);
  const [deployments, setDeployments] = useState([]);
  const [metrics, setMetrics] = useState([]);
  const [loading, setLoading] = useState(true);

  const fetchData = async () => {
    setLoading(true);
    try {
      const [pipeRes, deployRes, metricRes] = await Promise.all([
        api.get('/pipelines?limit=5'),
        api.get('/deployments?limit=5'),
        api.get('/metrics/history'),
      ]);

      setPipelines(pipeRes.data.data || []);
      setDeployments(deployRes.data.data || []);
      setMetrics(metricRes.data.data?.history || []);

      // Compute quick stats from responses
      const allPipelines = pipeRes.data.data || [];
      setStats({
        total:   pipeRes.data.pagination?.total || 0,
        running: allPipelines.filter((p) => p.status === 'running').length,
        success: allPipelines.filter((p) => p.status === 'success').length,
        failed:  allPipelines.filter((p) => p.status === 'failed').length,
        deployments: deployRes.data.pagination?.total || 0,
      });
    } catch (err) {
      console.error('Dashboard fetch error:', err);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, []);

  if (loading) {
    return (
      <div className="flex items-center justify-center h-64">
        <Spinner size="lg" label="Loading dashboard..." />
      </div>
    );
  }

  return (
    <div className="space-y-6">
      {/* Page header */}
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-xl font-bold text-white">
            Good {new Date().getHours() < 12 ? 'morning' : 'afternoon'}, {user?.name?.split(' ')[0]} 👋
          </h2>
          <p className="text-gray-400 text-sm mt-0.5">Here's what's happening in your pipelines</p>
        </div>
        <button onClick={fetchData} className="btn-secondary flex items-center gap-2 text-sm">
          <MdRefresh size={16} />
          Refresh
        </button>
      </div>

      {/* Stats Grid */}
      <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
        <StatCard title="Total Pipelines"  value={stats?.total || 0}       icon={MdAccountTree}  color="blue"   trend={5} />
        <StatCard title="Running"          value={stats?.running || 0}     icon={MdTimeline}     color="cyan"   />
        <StatCard title="Successful"       value={stats?.success || 0}     icon={MdCheckCircle}  color="green"  />
        <StatCard title="Failed"           value={stats?.failed || 0}      icon={MdError}        color="red"    />
      </div>

      {/* Charts Row */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-4">
        <CpuChart data={metrics} />
        <MemoryChart data={metrics} />
        <DeployChart />
      </div>

      {/* Recent Activity */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-4">
        {/* Recent Pipelines */}
        <div className="card">
          <div className="flex items-center justify-between mb-4">
            <h3 className="text-sm font-semibold text-white">Recent Pipelines</h3>
            <Link to="/pipelines" className="text-xs text-blue-400 hover:text-blue-300">View all →</Link>
          </div>
          <div className="space-y-3">
            {pipelines.length === 0 ? (
              <p className="text-sm text-gray-500 text-center py-6">No pipelines yet</p>
            ) : (
              pipelines.map((p) => (
                <div key={p._id} className="flex items-center justify-between py-2 border-b border-gray-800 last:border-0">
                  <div>
                    <p className="text-sm font-medium text-white">{p.name}</p>
                    <p className="text-xs text-gray-500">{p.branch} · {p.environment}</p>
                  </div>
                  <Badge status={p.status} />
                </div>
              ))
            )}
          </div>
        </div>

        {/* Recent Deployments */}
        <div className="card">
          <div className="flex items-center justify-between mb-4">
            <h3 className="text-sm font-semibold text-white">Recent Deployments</h3>
            <Link to="/deployments" className="text-xs text-blue-400 hover:text-blue-300">View all →</Link>
          </div>
          <div className="space-y-3">
            {deployments.length === 0 ? (
              <p className="text-sm text-gray-500 text-center py-6">No deployments yet</p>
            ) : (
              deployments.map((d) => (
                <div key={d._id} className="flex items-center justify-between py-2 border-b border-gray-800 last:border-0">
                  <div>
                    <p className="text-sm font-medium text-white">{d.pipeline?.name || 'Unknown'} v{d.version}</p>
                    <p className="text-xs text-gray-500">{d.environment} · {d.deployedBy?.name}</p>
                  </div>
                  <Badge status={d.status} />
                </div>
              ))
            )}
          </div>
        </div>
      </div>
    </div>
  );
};

export default DashboardPage;
```

### 9.24 `client/src/pages/PipelinesPage.jsx`

```jsx
// client/src/pages/PipelinesPage.jsx
import { useEffect, useState } from 'react';
import { MdAdd, MdSearch, MdFilterList } from 'react-icons/md';
import api from '../api/axios';
import PipelineCard from '../components/pipeline/PipelineCard';
import Spinner from '../components/ui/Spinner';
import Alert from '../components/ui/Alert';

const PipelinesPage = () => {
  const [pipelines, setPipelines] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [search, setSearch] = useState('');
  const [statusFilter, setStatusFilter] = useState('');
  const [showModal, setShowModal] = useState(false);
  const [form, setForm] = useState({ name: '', repository: '', branch: 'main', environment: 'development', description: '' });
  const [formError, setFormError] = useState(null);
  const [creating, setCreating] = useState(false);

  const fetchPipelines = async () => {
    setLoading(true);
    try {
      const { data } = await api.get('/pipelines', {
        params: { status: statusFilter || undefined, limit: 20 },
      });
      setPipelines(data.data || []);
    } catch (err) {
      setError('Failed to load pipelines');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchPipelines();
  }, [statusFilter]);

  const handleTrigger = async (id) => {
    try {
      await api.patch(`/pipelines/${id}/trigger`);
      fetchPipelines();
    } catch (err) {
      setError(err.response?.data?.message || 'Failed to trigger pipeline');
    }
  };

  const handleCreate = async (e) => {
    e.preventDefault();
    setCreating(true);
    setFormError(null);
    try {
      await api.post('/pipelines', form);
      setShowModal(false);
      setForm({ name: '', repository: '', branch: 'main', environment: 'development', description: '' });
      fetchPipelines();
    } catch (err) {
      setFormError(err.response?.data?.message || 'Failed to create pipeline');
    } finally {
      setCreating(false);
    }
  };

  const filtered = pipelines.filter((p) =>
    p.name.toLowerCase().includes(search.toLowerCase()) ||
    p.repository?.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div className="space-y-5">
      {/* Header */}
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-xl font-bold text-white">Pipelines</h2>
          <p className="text-gray-400 text-sm">{filtered.length} pipeline{filtered.length !== 1 ? 's' : ''}</p>
        </div>
        <button onClick={() => setShowModal(true)} className="btn-primary flex items-center gap-2 text-sm">
          <MdAdd size={18} /> New Pipeline
        </button>
      </div>

      {error && <Alert type="error" message={error} onClose={() => setError(null)} />}

      {/* Filters */}
      <div className="flex gap-3 flex-wrap">
        <div className="relative flex-1 min-w-[200px]">
          <MdSearch className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-500" size={16} />
          <input
            type="text"
            placeholder="Search pipelines..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="input-field pl-9"
          />
        </div>
        <select
          value={statusFilter}
          onChange={(e) => setStatusFilter(e.target.value)}
          className="input-field w-auto min-w-[150px]"
        >
          <option value="">All Statuses</option>
          {['idle','running','success','failed','pending','cancelled'].map((s) => (
            <option key={s} value={s}>{s.charAt(0).toUpperCase() + s.slice(1)}</option>
          ))}
        </select>
      </div>

      {/* Pipeline Cards */}
      {loading ? (
        <div className="flex justify-center py-16"><Spinner size="lg" /></div>
      ) : filtered.length === 0 ? (
        <div className="card text-center py-16">
          <p className="text-gray-400">No pipelines found</p>
          <button onClick={() => setShowModal(true)} className="btn-primary mt-4 text-sm">
            Create your first pipeline
          </button>
        </div>
      ) : (
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-4">
          {filtered.map((pipeline) => (
            <PipelineCard key={pipeline._id} pipeline={pipeline} onTrigger={handleTrigger} />
          ))}
        </div>
      )}

      {/* Create Modal */}
      {showModal && (
        <div className="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 p-4">
          <div className="bg-gray-900 border border-gray-800 rounded-xl w-full max-w-lg p-6">
            <h3 className="text-lg font-semibold text-white mb-5">Create Pipeline</h3>
            {formError && <Alert type="error" message={formError} onClose={() => setFormError(null)} />}
            <form onSubmit={handleCreate} className="space-y-4 mt-4">
              {[
                { id: 'name', label: 'Pipeline Name', type: 'text', placeholder: 'my-app-pipeline' },
                { id: 'repository', label: 'Repository URL', type: 'url', placeholder: 'https://github.com/org/repo' },
                { id: 'branch', label: 'Branch', type: 'text', placeholder: 'main' },
                { id: 'description', label: 'Description', type: 'text', placeholder: 'Optional description' },
              ].map(({ id, label, type, placeholder }) => (
                <div key={id}>
                  <label className="label">{label}</label>
                  <input
                    type={type}
                    value={form[id]}
                    onChange={(e) => setForm((p) => ({ ...p, [id]: e.target.value }))}
                    placeholder={placeholder}
                    className="input-field"
                    required={id !== 'description'}
                  />
                </div>
              ))}
              <div>
                <label className="label">Environment</label>
                <select
                  value={form.environment}
                  onChange={(e) => setForm((p) => ({ ...p, environment: e.target.value }))}
                  className="input-field"
                >
                  <option value="development">Development</option>
                  <option value="staging">Staging</option>
                  <option value="production">Production</option>
                </select>
              </div>
              <div className="flex gap-3 pt-2">
                <button type="button" onClick={() => setShowModal(false)} className="btn-secondary flex-1">Cancel</button>
                <button type="submit" disabled={creating} className="btn-primary flex-1 flex items-center justify-center gap-2">
                  {creating ? <Spinner size="sm" /> : 'Create'}
                </button>
              </div>
            </form>
          </div>
        </div>
      )}
    </div>
  );
};

export default PipelinesPage;
```

### 9.25 `client/src/pages/MetricsPage.jsx`

```jsx
// client/src/pages/MetricsPage.jsx
import { useEffect, useState } from 'react';
import { MdRefresh, MdMemory, MdComputer, MdAccessTime } from 'react-icons/md';
import api from '../api/axios';
import CpuChart from '../components/charts/CpuChart';
import MemoryChart from '../components/charts/MemoryChart';
import StatCard from '../components/ui/StatCard';
import Spinner from '../components/ui/Spinner';

const formatBytes = (bytes) => {
  if (!bytes) return '0 B';
  const gb = bytes / (1024 ** 3);
  return gb >= 1 ? `${gb.toFixed(1)} GB` : `${(bytes / (1024 ** 2)).toFixed(0)} MB`;
};

const formatUptime = (seconds) => {
  const d = Math.floor(seconds / 86400);
  const h = Math.floor((seconds % 86400) / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  return `${d}d ${h}h ${m}m`;
};

const MetricsPage = () => {
  const [live, setLive] = useState(null);
  const [history, setHistory] = useState([]);
  const [loading, setLoading] = useState(true);

  const fetchMetrics = async () => {
    try {
      const [liveRes, histRes] = await Promise.all([
        api.get('/metrics/live'),
        api.get('/metrics/history'),
      ]);
      setLive(liveRes.data.data?.metrics);
      setHistory(histRes.data.data?.history || []);
    } catch (err) {
      console.error('Metrics fetch error:', err);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchMetrics();
    const interval = setInterval(fetchMetrics, 30000); // Auto-refresh every 30s
    return () => clearInterval(interval);
  }, []);

  if (loading) {
    return <div className="flex justify-center py-16"><Spinner size="lg" /></div>;
  }

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-xl font-bold text-white">Server Metrics</h2>
          <p className="text-gray-400 text-sm">Auto-refreshes every 30 seconds</p>
        </div>
        <button onClick={fetchMetrics} className="btn-secondary flex items-center gap-2 text-sm">
          <MdRefresh size={16} /> Refresh
        </button>
      </div>

      {/* Live Stats */}
      {live && (
        <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
          <StatCard
            title="CPU Usage"
            value={`${live.cpu?.usage || 0}%`}
            subtitle={`${live.cpu?.cores} cores`}
            icon={MdComputer}
            color={live.cpu?.usage > 80 ? 'red' : live.cpu?.usage > 60 ? 'yellow' : 'green'}
          />
          <StatCard
            title="Memory Used"
            value={`${live.memory?.usagePercent || 0}%`}
            subtitle={`${formatBytes(live.memory?.used)} / ${formatBytes(live.memory?.total)}`}
            icon={MdMemory}
            color={live.memory?.usagePercent > 85 ? 'red' : 'blue'}
          />
          <StatCard
            title="Uptime"
            value={formatUptime(live.uptime || 0)}
            icon={MdAccessTime}
            color="green"
          />
          <StatCard
            title="Node.js"
            value={live.nodeVersion || 'N/A'}
            subtitle={`${live.platform} · ${live.arch}`}
            icon={MdComputer}
            color="cyan"
          />
        </div>
      )}

      {/* Charts */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-4">
        <CpuChart data={history} />
        <MemoryChart data={history} />
      </div>

      {/* Server Info */}
      {live && (
        <div className="card">
          <h3 className="text-sm font-semibold text-white mb-4">Server Information</h3>
          <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
            {[
              { label: 'Hostname', value: live.server },
              { label: 'Platform', value: live.platform },
              { label: 'Architecture', value: live.arch },
              { label: 'Node.js', value: live.nodeVersion },
            ].map(({ label, value }) => (
              <div key={label} className="bg-gray-800 rounded-lg p-3">
                <p className="text-xs text-gray-500 mb-1">{label}</p>
                <p className="text-sm font-mono text-white">{value}</p>
              </div>
            ))}
          </div>
        </div>
      )}
    </div>
  );
};

export default MetricsPage;
```

### 9.26 `client/src/pages/DeploymentsPage.jsx`

```jsx
// client/src/pages/DeploymentsPage.jsx
import { useEffect, useState } from 'react';
import { MdRocketLaunch, MdSearch } from 'react-icons/md';
import api from '../api/axios';
import Badge from '../components/ui/Badge';
import Spinner from '../components/ui/Spinner';
import Alert from '../components/ui/Alert';

const DeploymentsPage = () => {
  const [deployments, setDeployments] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [envFilter, setEnvFilter] = useState('');

  const fetchDeployments = async () => {
    setLoading(true);
    try {
      const { data } = await api.get('/deployments', {
        params: { environment: envFilter || undefined, limit: 20 },
      });
      setDeployments(data.data || []);
    } catch {
      setError('Failed to load deployments');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => { fetchDeployments(); }, [envFilter]);

  return (
    <div className="space-y-5">
      <div className="flex items-center justify-between">
        <div>
          <h2 className="text-xl font-bold text-white">Deployments</h2>
          <p className="text-gray-400 text-sm">Deployment history and status</p>
        </div>
      </div>

      {error && <Alert type="error" message={error} onClose={() => setError(null)} />}

      {/* Filters */}
      <div className="flex gap-3">
        <select
          value={envFilter}
          onChange={(e) => setEnvFilter(e.target.value)}
          className="input-field w-auto"
        >
          <option value="">All Environments</option>
          <option value="development">Development</option>
          <option value="staging">Staging</option>
          <option value="production">Production</option>
        </select>
      </div>

      {/* Table */}
      {loading ? (
        <div className="flex justify-center py-16"><Spinner size="lg" /></div>
      ) : (
        <div className="card overflow-hidden p-0">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b border-gray-800">
                {['Pipeline', 'Version', 'Environment', 'Status', 'Deployed By', 'Date'].map((h) => (
                  <th key={h} className="text-left text-xs font-medium text-gray-400 uppercase tracking-wider px-5 py-3">
                    {h}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody className="divide-y divide-gray-800/50">
              {deployments.length === 0 ? (
                <tr>
                  <td colSpan={6} className="text-center text-gray-500 py-12">
                    No deployments found
                  </td>
                </tr>
              ) : (
                deployments.map((d) => (
                  <tr key={d._id} className="hover:bg-gray-800/50 transition-colors">
                    <td className="px-5 py-3">
                      <p className="font-medium text-white">{d.pipeline?.name || '—'}</p>
                      <p className="text-xs text-gray-500 font-mono truncate max-w-[120px]">
                        {d.commitHash?.slice(0, 8) || ''}
                      </p>
                    </td>
                    <td className="px-5 py-3 font-mono text-gray-300">v{d.version}</td>
                    <td className="px-5 py-3">
                      <Badge status={d.environment} label={d.environment} />
                    </td>
                    <td className="px-5 py-3">
                      <Badge status={d.status} />
                    </td>
                    <td className="px-5 py-3 text-gray-300">{d.deployedBy?.name || '—'}</td>
                    <td className="px-5 py-3 text-gray-400 text-xs">
                      {new Date(d.createdAt).toLocaleDateString()} {new Date(d.createdAt).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}
                    </td>
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>
      )}
    </div>
  );
};

export default DeploymentsPage;
```

### 9.27 `client/src/pages/ProfilePage.jsx`

```jsx
// client/src/pages/ProfilePage.jsx
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';
import api from '../api/axios';
import Alert from '../components/ui/Alert';
import Spinner from '../components/ui/Spinner';
import { MdPerson, MdEmail, MdShield, MdCalendarToday } from 'react-icons/md';

const ProfilePage = () => {
  const { user, updateUser } = useAuth();
  const [name, setName] = useState(user?.name || '');
  const [currentPassword, setCurrentPassword] = useState('');
  const [newPassword, setNewPassword] = useState('');
  const [message, setMessage] = useState(null);
  const [loading, setLoading] = useState(false);

  const handleUpdateProfile = async (e) => {
    e.preventDefault();
    setLoading(true);
    setMessage(null);
    try {
      const { data } = await api.patch('/auth/update-profile', { name });
      updateUser(data.data.user);
      setMessage({ type: 'success', text: 'Profile updated successfully' });
    } catch (err) {
      setMessage({ type: 'error', text: err.response?.data?.message || 'Update failed' });
    } finally {
      setLoading(false);
    }
  };

  const handleChangePassword = async (e) => {
    e.preventDefault();
    setLoading(true);
    setMessage(null);
    try {
      await api.patch('/auth/change-password', { currentPassword, newPassword });
      setCurrentPassword('');
      setNewPassword('');
      setMessage({ type: 'success', text: 'Password changed successfully' });
    } catch (err) {
      setMessage({ type: 'error', text: err.response?.data?.message || 'Password change failed' });
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="space-y-6 max-w-2xl">
      <div>
        <h2 className="text-xl font-bold text-white">Profile Settings</h2>
        <p className="text-gray-400 text-sm">Manage your account details and password</p>
      </div>

      {message && (
        <Alert type={message.type} message={message.text} onClose={() => setMessage(null)} />
      )}

      {/* User info card */}
      <div className="card">
        <div className="flex items-center gap-4 mb-6">
          <div className="w-16 h-16 rounded-full bg-blue-600 flex items-center justify-center text-2xl font-bold text-white">
            {user?.name?.[0]?.toUpperCase()}
          </div>
          <div>
            <p className="font-semibold text-white text-lg">{user?.name}</p>
            <p className="text-gray-400 text-sm">{user?.email}</p>
            <span className="mt-1 inline-block text-xs font-medium text-blue-400 bg-blue-500/10 border border-blue-500/30 px-2 py-0.5 rounded-full capitalize">
              {user?.role}
            </span>
          </div>
        </div>

        <div className="grid grid-cols-2 gap-4 text-sm">
          {[
            { icon: MdPerson,       label: 'Name',     value: user?.name },
            { icon: MdEmail,        label: 'Email',    value: user?.email },
            { icon: MdShield,       label: 'Role',     value: user?.role },
            { icon: MdCalendarToday,label: 'Joined',   value: new Date(user?.createdAt).toLocaleDateString() },
          ].map(({ icon: Icon, label, value }) => (
            <div key={label} className="bg-gray-800 rounded-lg p-3">
              <div className="flex items-center gap-2 text-gray-400 mb-1">
                <Icon size={14} /><span className="text-xs">{label}</span>
              </div>
              <p className="text-white font-medium truncate">{value}</p>
            </div>
          ))}
        </div>
      </div>

      {/* Update name */}
      <div className="card">
        <h3 className="text-sm font-semibold text-white mb-4">Update Profile</h3>
        <form onSubmit={handleUpdateProfile} className="space-y-4">
          <div>
            <label className="label">Full Name</label>
            <input type="text" value={name} onChange={(e) => setName(e.target.value)}
              className="input-field" required />
          </div>
          <button type="submit" disabled={loading} className="btn-primary flex items-center gap-2">
            {loading ? <Spinner size="sm" /> : 'Save Changes'}
          </button>
        </form>
      </div>

      {/* Change password */}
      <div className="card">
        <h3 className="text-sm font-semibold text-white mb-4">Change Password</h3>
        <form onSubmit={handleChangePassword} className="space-y-4">
          <div>
            <label className="label">Current Password</label>
            <input type="password" value={currentPassword} onChange={(e) => setCurrentPassword(e.target.value)}
              className="input-field" required />
          </div>
          <div>
            <label className="label">New Password</label>
            <input type="password" value={newPassword} onChange={(e) => setNewPassword(e.target.value)}
              className="input-field" required minLength={8} />
          </div>
          <button type="submit" disabled={loading} className="btn-primary flex items-center gap-2">
            {loading ? <Spinner size="sm" /> : 'Change Password'}
          </button>
        </form>
      </div>
    </div>
  );
};

export default ProfilePage;
```

### 9.28 `client/src/pages/NotFoundPage.jsx`

```jsx
// client/src/pages/NotFoundPage.jsx
import { Link } from 'react-router-dom';

const NotFoundPage = () => (
  <div className="min-h-screen bg-gray-950 flex items-center justify-center p-4">
    <div className="text-center">
      <p className="text-8xl font-bold text-gray-800 select-none">404</p>
      <h1 className="text-2xl font-bold text-white mt-4">Page Not Found</h1>
      <p className="text-gray-400 text-sm mt-2">
        The route you're looking for doesn't exist.
      </p>
      <Link to="/dashboard" className="btn-primary inline-block mt-6">
        Return to Dashboard
      </Link>
    </div>
  </div>
);

export default NotFoundPage;
```

---

## 10. Running the Application

### 10.1 Start MongoDB

```bash
# macOS
brew services start mongodb-community@7.0

# Linux
sudo systemctl start mongod

# Verify MongoDB is running
mongosh --eval "db.adminCommand({ ping: 1 })"
# Expected output: { ok: 1 }
```

### 10.2 Start the Backend Server

```bash
# Navigate to server directory
cd devops-pipeline-dashboard/server

# Ensure .env exists
cp .env.example .env
# Edit .env with your values (especially JWT_SECRET)

# Start in development mode (auto-restarts on file changes)
npm run dev

# Expected output:
# ╔═══════════════════════════════════════════╗
# ║   DevOps Pipeline Dashboard — API Server  ║
# ║   Mode     : development                  ║
# ║   Port     : 5000                         ║
# ║   DB       : MongoDB Connected            ║
# ╚═══════════════════════════════════════════╝
```

### 10.3 Start the Frontend Client

```bash
# In a NEW terminal — navigate to client directory
cd devops-pipeline-dashboard/client

# Ensure .env exists
cp .env.example .env

# Start Vite development server
npm run dev

# Expected output:
# VITE v5.x.x  ready in 300ms
# ➜  Local:   http://localhost:5173/
# ➜  Network: http://192.168.x.x:5173/
```

### 10.4 Access the Application

| Service | URL | Description |
|---|---|---|
| Frontend | http://localhost:5173 | React dashboard |
| Backend API | http://localhost:5000/api/v1 | REST API |
| Health Check | http://localhost:5000/health | Server health status |

### 10.5 Production Build

```bash
# Build the React frontend for production
cd client
npm run build
# Output: client/dist/ — static files to serve via Nginx or CDN

# Start backend in production mode
cd ../server
NODE_ENV=production npm start
```

---

## 11. API Documentation

### Base URL
```
Development: http://localhost:5000/api/v1
Production:  https://your-domain.com/api/v1
```

### Authentication
All protected endpoints require a valid JWT cookie (`jwt`) sent automatically by the browser, or an `Authorization: Bearer <token>` header.

### Response Format
```json
{
  "success": true,
  "message": "Human-readable message",
  "data": { ... },
  "pagination": {
    "total": 100,
    "page": 1,
    "limit": 10,
    "totalPages": 10
  }
}
```

### Error Format
```json
{
  "success": false,
  "message": "Error description",
  "errors": [
    { "field": "email", "message": "Valid email is required" }
  ]
}
```

---

### Auth Endpoints — `/api/v1/auth`

#### POST /register
Register a new user account.

**Request Body:**
```json
{
  "name": "Jane DevOps",
  "email": "jane@company.com",
  "password": "securePass123",
  "role": "developer"
}
```
**Response:** `201 Created`
```json
{
  "success": true,
  "message": "Account created successfully",
  "data": {
    "user": {
      "_id": "665abc...",
      "name": "Jane DevOps",
      "email": "jane@company.com",
      "role": "developer"
    }
  }
}
```

#### POST /login
Authenticate user and set JWT cookie.

**Request Body:**
```json
{ "email": "jane@company.com", "password": "securePass123" }
```
**Response:** `200 OK` + Sets `jwt` httpOnly cookie

#### POST /logout
Clear the JWT cookie.
**Response:** `200 OK`

#### GET /me 🔒
Get the authenticated user's profile.
**Response:** `200 OK` with user object

#### PATCH /update-profile 🔒
Update user's name or avatar.
**Request Body:** `{ "name": "New Name" }`

#### PATCH /change-password 🔒
Change the user's password.
**Request Body:** `{ "currentPassword": "old", "newPassword": "new123456" }`

---

### Pipelines — `/api/v1/pipelines` 🔒

| Method | Path | Description | Roles |
|---|---|---|---|
| GET | `/` | List pipelines (paginated) | All |
| GET | `/:id` | Get pipeline by ID | All |
| GET | `/stats` | Aggregated pipeline stats | All |
| POST | `/` | Create new pipeline | Developer, Admin |
| PUT | `/:id` | Update pipeline | Developer, Admin |
| PATCH | `/:id/trigger` | Trigger pipeline run | Developer, Admin |
| DELETE | `/:id` | Soft-delete pipeline | Admin only |

#### Query Parameters
```
GET /pipelines?status=running&environment=production&page=1&limit=10
```

#### POST /pipelines — Request Body
```json
{
  "name": "my-app-ci",
  "description": "Main CI pipeline",
  "repository": "https://github.com/org/my-app",
  "branch": "main",
  "environment": "production",
  "stages": [
    { "name": "Build", "status": "pending" },
    { "name": "Test",  "status": "pending" },
    { "name": "Deploy","status": "pending" }
  ],
  "tags": ["nodejs", "api"]
}
```

---

### Deployments — `/api/v1/deployments` 🔒

| Method | Path | Description |
|---|---|---|
| GET | `/` | List deployments (paginated) |
| GET | `/:id` | Get deployment by ID |
| GET | `/stats` | Deployment statistics |
| POST | `/` | Create deployment record |
| PATCH | `/:id/status` | Update deployment status |

#### POST /deployments — Request Body
```json
{
  "pipeline": "665abc...",
  "version": "2.4.1",
  "environment": "production",
  "commitHash": "a1b2c3d4e5f6",
  "commitMessage": "feat: add health check endpoint",
  "notes": "Production release v2.4.1"
}
```

#### PATCH /deployments/:id/status — Request Body
```json
{
  "status": "success",
  "notes": "Deployment completed in 3m 12s"
}
```
Valid statuses: `pending`, `in_progress`, `success`, `failed`, `rolled_back`

---

### Metrics — `/api/v1/metrics` 🔒

| Method | Path | Description |
|---|---|---|
| GET | `/live` | Real-time server metrics snapshot |
| GET | `/history` | Historical metrics for charts |
| GET | `/summary` | 24h aggregated summary |

#### GET /metrics/history — Query Parameters
```
?hours=24   # default 24, returns up to 100 data points
```

#### GET /metrics/live — Response
```json
{
  "success": true,
  "data": {
    "metrics": {
      "server": "hostname",
      "cpu": { "usage": 34.2, "cores": 8, "model": "Intel..." },
      "memory": { "used": 4294967296, "total": 8589934592, "usagePercent": 50.0 },
      "uptime": 86400,
      "platform": "linux",
      "arch": "x64",
      "nodeVersion": "v20.14.0",
      "recordedAt": "2024-06-15T10:30:00.000Z"
    }
  }
}
```

---

### Health Check — No Auth Required

#### GET /health
Used by Kubernetes liveness probes and load balancers.

**Response:** `200 OK`
```json
{
  "status": "ok",
  "timestamp": "2024-06-15T10:30:00.000Z",
  "uptime": 3600,
  "environment": "production",
  "version": "1.0.0"
}
```

---

## 12. Docker & Kubernetes Readiness

### 12.1 `server/Dockerfile`

```dockerfile
# server/Dockerfile
# Multi-stage build for a minimal production image

# ─── Stage 1: Build ───────────────────────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files first — leverages Docker layer caching
# (Only reinstalls dependencies if package.json changes)
COPY package*.json ./
RUN npm ci --omit=dev

COPY . .

# ─── Stage 2: Production ──────────────────────────────────────
FROM node:20-alpine AS production
WORKDIR /app

# Non-root user for security
RUN addgroup -g 1001 -S appgroup && \
    adduser -S -u 1001 -G appgroup appuser

# Copy only production dependencies and source
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app .

# Use non-root user
USER appuser

# Document the port (does not publish it)
EXPOSE 5000

# Health check for Docker (Kubernetes uses its own probes)
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD wget -qO- http://localhost:5000/health || exit 1

# Graceful shutdown via SIGTERM (handled in server.js)
CMD ["node", "server.js"]
```

### 12.2 `client/Dockerfile`

```dockerfile
# client/Dockerfile
# Build React app and serve via Nginx

FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
# Pass build-time env vars from CI/CD
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

RUN npm run build

# Production: Nginx to serve static files
FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html

# Custom Nginx config for React Router (SPA support)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 12.3 `client/nginx.conf`

```nginx
# client/nginx.conf
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    # React Router — serve index.html for all routes
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets aggressively
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header Referrer-Policy "strict-origin-when-cross-origin";
}
```

### 12.4 `docker-compose.yml` (Root)

```yaml
# docker-compose.yml
# Development: spins up MongoDB + Backend + Frontend together
version: '3.9'

services:
  mongodb:
    image: mongo:7.0
    container_name: devops-mongo
    restart: unless-stopped
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: changeme
      MONGO_INITDB_DATABASE: devops_dashboard
    volumes:
      - mongo_data:/data/db
    networks:
      - devops-net
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 5

  server:
    build:
      context: ./server
      dockerfile: Dockerfile
    container_name: devops-server
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      NODE_ENV: production
      PORT: 5000
      MONGO_URI: mongodb://admin:changeme@mongodb:27017/devops_dashboard?authSource=admin
      JWT_SECRET: ${JWT_SECRET:-your_secret_change_this}
      JWT_EXPIRES_IN: 7d
      JWT_COOKIE_EXPIRES: 7
      CORS_ORIGIN: http://localhost:3000
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - devops-net
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  client:
    build:
      context: ./client
      dockerfile: Dockerfile
      args:
        VITE_API_URL: http://localhost:5000/api/v1
    container_name: devops-client
    restart: unless-stopped
    ports:
      - "3000:80"
    depends_on:
      - server
    networks:
      - devops-net

volumes:
  mongo_data:

networks:
  devops-net:
    driver: bridge
```

### 12.5 Kubernetes Deployment Manifests

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: devops-dashboard
  labels:
    app.kubernetes.io/name: devops-dashboard
```

```yaml
# k8s/server-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-server
  namespace: devops-dashboard
  labels:
    app: devops-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-server
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: devops-server
    spec:
      containers:
        - name: devops-server
          image: your-registry/devops-server:latest
          ports:
            - containerPort: 5000
          envFrom:
            - secretRef:
                name: devops-secrets
            - configMapRef:
                name: devops-config
          # Liveness probe: restart container if /health fails
          livenessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 30
            periodSeconds: 30
            failureThreshold: 3
          # Readiness probe: only send traffic when ready
          readinessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 10
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: devops-server-service
  namespace: devops-dashboard
spec:
  selector:
    app: devops-server
  ports:
    - port: 80
      targetPort: 5000
  type: ClusterIP
```

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: devops-config
  namespace: devops-dashboard
data:
  NODE_ENV: "production"
  PORT: "5000"
  JWT_EXPIRES_IN: "7d"
  JWT_COOKIE_EXPIRES: "7"
---
# k8s/secret.yaml — base64-encode values: echo -n "value" | base64
apiVersion: v1
kind: Secret
metadata:
  name: devops-secrets
  namespace: devops-dashboard
type: Opaque
data:
  MONGO_URI: <base64-encoded-mongo-uri>
  JWT_SECRET: <base64-encoded-secret>
  CORS_ORIGIN: <base64-encoded-origin>
```

---

## 13. CI/CD Pipeline Readiness

### 13.1 GitHub Actions — `.github/workflows/ci.yml`

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20.x'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─── Backend Tests ──────────────────────────────────────────
  test-backend:
    name: Test Backend
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:7.0
        ports: ['27017:27017']
        options: >-
          --health-cmd "mongosh --eval 'db.adminCommand({ ping: 1 })'"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: server/package-lock.json

      - name: Install dependencies
        run: cd server && npm ci

      - name: Run tests
        run: cd server && npm test
        env:
          NODE_ENV: test
          MONGO_URI: mongodb://localhost:27017/test_db
          JWT_SECRET: test_secret_key_12345678901234567890

  # ─── Frontend Build ─────────────────────────────────────────
  build-frontend:
    name: Build Frontend
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: client/package-lock.json

      - name: Install dependencies
        run: cd client && npm ci

      - name: Build
        run: cd client && npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: client-dist
          path: client/dist

  # ─── Docker Build & Push ────────────────────────────────────
  docker:
    name: Build & Push Docker Images
    needs: [test-backend, build-frontend]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push server image
        uses: docker/build-push-action@v5
        with:
          context: ./server
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/server:latest

      - name: Build and push client image
        uses: docker/build-push-action@v5
        with:
          context: ./client
          push: true
          build-args: VITE_API_URL=${{ secrets.VITE_API_URL }}
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/client:latest

  # ─── Deploy ─────────────────────────────────────────────────
  deploy:
    name: Deploy to Kubernetes
    needs: docker
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Set up kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubeconfig
        run: echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config

      - name: Apply Kubernetes manifests
        run: kubectl apply -f k8s/ --namespace=devops-dashboard

      - name: Rollout restart (pull new image)
        run: |
          kubectl rollout restart deployment/devops-server -n devops-dashboard
          kubectl rollout restart deployment/devops-client -n devops-dashboard

      - name: Wait for rollout
        run: kubectl rollout status deployment/devops-server -n devops-dashboard --timeout=120s
```

---

## 14. Dependency Reference

### Backend Dependencies

| Package | Why It's Used | What Happens Without It |
|---|---|---|
| `express` | HTTP server and routing framework | No server |
| `mongoose` | Schema validation, ODM, MongoDB queries | Raw MongoDB driver required |
| `cors` | Cross-Origin Resource Sharing headers | Browser blocks API requests |
| `dotenv` | Load `.env` file into `process.env` | Hardcoded secrets, not deployable |
| `jsonwebtoken` | Create/verify JWT tokens | No authentication |
| `bcryptjs` | Hash passwords securely | Passwords stored in plaintext |
| `cookie-parser` | Parse `req.cookies` | JWT cookie unreadable |
| `helmet` | HTTP security headers | XSS, clickjacking vulnerabilities |
| `morgan` | HTTP request logs | No visibility into API traffic |
| `axios` | HTTP client for external calls | Manual fetch API needed |
| `nodemon` (dev) | Auto-restart on file change | Manual restart after every edit |

### Frontend Dependencies

| Package | Why It's Used | What Happens Without It |
|---|---|---|
| `react` + `react-dom` | Component rendering | No UI |
| `vite` | Dev server + production bundler | No build system |
| `react-router-dom` | Client-side routing | Full page reloads on navigation |
| `axios` | API calls with interceptors | Manual fetch with less control |
| `recharts` | SVG-based charts | No data visualization |
| `react-icons` | 40k+ icon set | No icons |
| `tailwindcss` | Utility CSS framework | Manual CSS required |

---

## 15. Troubleshooting

### Common Issues

#### `ECONNREFUSED` — MongoDB connection failed
```bash
# Check if MongoDB is running
sudo systemctl status mongod          # Linux
brew services list | grep mongodb     # macOS

# Start MongoDB
sudo systemctl start mongod           # Linux
brew services start mongodb-community # macOS

# Verify MONGO_URI in server/.env is correct
cat server/.env | grep MONGO_URI
```

#### CORS error in browser
```bash
# Ensure CORS_ORIGIN in server/.env matches your frontend URL exactly
# No trailing slash: http://localhost:5173 (not http://localhost:5173/)
CORS_ORIGIN=http://localhost:5173

# Restart the server after changing .env
```

#### `Cannot find module` error on backend startup
```bash
# Missing "type": "module" in package.json (for ES modules)
# Or ensure all imports use .js extension
# Check server/package.json has:
"type": "module"

# Reinstall dependencies
rm -rf server/node_modules
cd server && npm install
```

#### Vite `VITE_API_URL` not working
```bash
# Ensure the variable is prefixed with VITE_
# Wrong:  API_URL=http://localhost:5000
# Correct: VITE_API_URL=http://localhost:5000/api/v1

# Restart Vite dev server after changing client/.env
```

#### JWT errors — "Invalid token" or "Token expired"
```bash
# Check JWT_SECRET is the same in .env (not changed after generating tokens)
# Check system clock is accurate (JWT iat validation is time-based)

# Clear cookies in browser and log in again
# Or set JWT_EXPIRES_IN to a longer value during development
```

#### Port already in use
```bash
# Find and kill process using port 5000
lsof -ti:5000 | xargs kill -9   # macOS/Linux
netstat -ano | findstr :5000     # Windows (then taskkill /PID <pid> /F)

# Or change the port in server/.env
PORT=5001
```

#### Docker build fails
```bash
# Ensure Docker Desktop is running
docker info

# Clean Docker cache
docker builder prune -a

# Rebuild without cache
docker-compose build --no-cache
```

---

## Final Setup Checklist

```
Prerequisites
  ✅ Node.js 20.x LTS installed
  ✅ npm 10.x installed
  ✅ MongoDB 7.x installed and running
  ✅ Git initialized

Backend
  ✅ server/package.json created (npm init -y)
  ✅ All dependencies installed (npm install ...)
  ✅ server/.env created from .env.example
  ✅ JWT_SECRET set (min 32 random characters)
  ✅ MONGO_URI correct

Frontend
  ✅ client/ created (npm create vite@latest)
  ✅ All dependencies installed
  ✅ client/.env created
  ✅ tailwind.config.js configured
  ✅ vite.config.js has API proxy

Running
  ✅ MongoDB service started
  ✅ npm run dev in server/ → http://localhost:5000
  ✅ npm run dev in client/ → http://localhost:5173
  ✅ /health endpoint returns { status: "ok" }
  ✅ Register a user and log in
```

---

*DevOps Pipeline Dashboard — Production-Ready MERN Stack Application*
*Architecture designed for Docker · Kubernetes · CI/CD from day one.*

**Repository Structure Version:** 1.0.0 | **Node.js:** 20 LTS | **MongoDB:** 7.x | **React:** 18.x | **Express:** 4.x
