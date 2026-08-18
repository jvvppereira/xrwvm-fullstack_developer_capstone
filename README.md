# Car Dealership Platform - Full Stack Developer Capstone

A microservices-based car dealership platform featuring dealer management, vehicle inventory search, review system with sentiment analysis, and user authentication.

## Project Objective

Build a scalable, full-stack car dealership application using a microservices architecture. The platform enables users to:
- Browse dealerships by state
- Search vehicle inventory with multiple filters (year, make, model, mileage, price)
- Read and submit dealership reviews with automated sentiment analysis
- Register and authenticate securely

## Tech Stack

| Service | Language/Framework | Database | Port | Key Dependencies |
|---------|-------------------|----------|------|------------------|
| **Django Main App** | Python 3.12 / Django 3.2 / DRF | SQLite (dev) | 8000 | gunicorn, python-dotenv, Pillow, requests |
| **Database Microservice** | Node.js 18 / Express | MongoDB | 3030 | mongoose, cors |
| **Cars Inventory** | Node.js 18 / Express | MongoDB | 3050 | mongoose, cors |
| **Sentiment Analysis** | Python 3.9 / Flask | — | 5050 | nltk (VADER) |
| **React Frontend** | React 18 / React Router 6 | — | 3000 (dev) | react-scripts, testing-library |

## Project Structure

```
xrwvm-fullstack_developer_capstone/
├── server/
│   ├── djangoproj/           # Django project config (settings, urls, wsgi, asgi)
│   ├── djangoapp/            # Main Django app
│   │   ├── views.py          # API endpoints (auth, dealers, reviews, cars)
│   │   ├── restapis.py       # HTTP clients for microservices
│   │   ├── models.py         # CarMake, CarModel
│   │   ├── populate.py       # Seed data for car makes/models
│   │   ├── admin.py          # Django admin config
│   │   ├── apps.py           # App configuration
│   │   └── microservices/    # Sentiment Analysis (Flask)
│   │       ├── app.py        # Flask app with VADER sentiment
│   │       ├── requirements.txt
│   │       ├── Dockerfile
│   │       └── vader_lexicon.zip
│   ├── database/             # Dealerships/Reviews microservice (Node/Express)
│   │   ├── app.js            # Express server
│   │   ├── dealership.js     # Dealership model & routes
│   │   ├── review.js         # Review model & routes
│   │   ├── inventory.js      # Inventory routes
│   │   ├── data/             # Seed JSON data
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   └── package.json
│   ├── carsInventory/        # Car search microservice (Node/Express)
│   │   ├── app.js            # Express server
│   │   ├── inventory.js      # Car search logic
│   │   ├── data/car_records.json
│   │   ├── Dockerfile
│   │   ├── docker-compose.yaml
│   │   └── package.json
│   ├── frontend/             # React app (served by Django in prod)
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── Header/
│   │   │   │   ├── Dealers/ (Dealers, Dealer, SearchCar, PostReview)
│   │   │   │   ├── Login/
│   │   │   │   └── Register/
│   │   │   ├── App.js
│   │   │   └── index.js
│   │   ├── public/
│   │   ├── package.json
│   │   └── README.md
│   ├── Dockerfile            # Django + Gunicorn
│   ├── entrypoint.sh         # Migrations + collectstatic
│   ├── requirements.txt      # Python dependencies
│   ├── deployment.yaml       # K8s deployment for Django
│   └── package.json          # Root package.json
└── README.md
```

## Services Breakdown

### 1. Django Main App (`server/`)
**Port:** 8000 | **Role:** API Gateway & Main Backend

**Responsibilities:**
- User authentication (registration, login, logout)
- Proxy requests to microservices
- Serve React frontend (production)
- Car make/model data management
- Static file serving

**Key Endpoints:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/login` | User login |
| POST | `/api/register` | User registration |
| GET | `/api/logout` | User logout |
| GET | `/api/dealerships` | List all dealerships |
| GET | `/api/dealerships/<state>` | List dealerships by state |
| GET | `/api/dealer/<id>` | Get dealer details |
| GET | `/api/reviews/dealer/<id>` | Get reviews for dealer (with sentiment) |
| POST | `/api/review` | Submit review (auth required) |
| GET | `/api/cars` | List car makes/models |
| GET | `/api/inventory/<dealer_id>` | Search dealer inventory |

**Environment Variables** (`.env` in `server/djangoapp/`):
```env
backend_url=http://localhost:3030
sentiment_analyzer_url=http://localhost:5050/
searchcars_url=http://localhost:3050/
```

---

### 2. Database Microservice (`server/database/`)
**Port:** 3030 | **Role:** Dealerships & Reviews CRUD

**Tech:** Node.js 18, Express, MongoDB, Mongoose

**Endpoints:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/fetchDealers` | All dealerships |
| GET | `/fetchDealers/:state` | Dealerships by state |
| GET | `/fetchDealer/:id` | Single dealership |
| GET | `/fetchReviews/dealer/:id` | Reviews for dealer |
| POST | `/insert_review` | Create review |
| GET | `/cars/:dealerId` | Dealer's inventory |

**Data Models:** Dealership, Review

**Docker:** `docker-compose.yml` includes MongoDB on port 27017

---

### 3. Cars Inventory Microservice (`server/carsInventory/`)
**Port:** 3050 | **Role:** Vehicle Search & Filtering

**Tech:** Node.js 18, Express, MongoDB, Mongoose

**Endpoints:**
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/cars/:dealerId` | All cars for dealer |
| GET | `/carsbyyear/:dealerId/:year` | Filter by year |
| GET | `/carsbymake/:dealerId/:make` | Filter by make |
| GET | `/carsbymodel/:dealerId/:model` | Filter by model |
| GET | `/carsbymaxmileage/:dealerId/:mileage` | Max mileage |
| GET | `/carsbyprice/:dealerId/:price` | Max price |

**Data:** `car_records.json` with 1000+ vehicle records

**Docker:** `docker-compose.yaml` includes MongoDB on port 27018

---

### 4. Sentiment Analysis (`server/djangoapp/microservices/`)
**Port:** 5050 | **Role:** Review Sentiment Classification

**Tech:** Python 3.9, Flask, NLTK (VADER)

**Endpoints:**
| Method | Endpoint | Response |
|--------|----------|----------|
| GET | `/` | Welcome message |
| GET | `/analyze/<text>` | `{"sentiment": "positive|negative|neutral"}` |

**Algorithm:** VADER (Valence Aware Dictionary and sEntiment Reasoner)

---

### 5. React Frontend (`server/frontend/`)
**Port:** 3000 (dev) | **Role:** Single Page Application

**Tech:** React 18, React Router 6, React Scripts 5

**Pages/Components:**
- **Header** - Navigation, auth state
- **Dealers** - List, search by state, dealer details
- **SearchCar** - Inventory filtering (year, make, model, mileage, price)
- **PostReview** - Submit review with sentiment preview
- **Login/Register** - Authentication forms

**Production:** Built assets served by Django (`npm run build` → `frontend/build/`)

## Running Locally

### Prerequisites
- Docker & Docker Compose
- Node.js 18+
- Python 3.12+
- MongoDB (if running without Docker)

---

### Option A: Docker Compose (Recommended)

Run each service in its own terminal:

```bash
# 1. Database Microservice (port 3030 + MongoDB 27017)
cd server/database
docker-compose up -d

# 2. Cars Inventory Microservice (port 3050 + MongoDB 27018)
cd ../carsInventory
docker-compose up -d

# 3. Sentiment Analysis (port 5050)
cd ../djangoapp/microservices
docker build -t sentiment .
docker run -d -p 5050:5000 sentiment

# 4. Django Main App (port 8000)
cd ../../..
docker build -t dealership .
docker run -d -p 8000:8000 \
  -e backend_url=http://host.docker.internal:3030 \
  -e sentiment_analyzer_url=http://host.docker.internal:5050/ \
  -e searchcars_url=http://host.docker.internal:3050/ \
  dealership

# 5. React Frontend Dev Server (port 3000)
cd server/frontend
npm install
npm start
```

**Access:**
- Frontend: http://localhost:3000
- Django API: http://localhost:8000
- Database API: http://localhost:3030
- Cars Inventory: http://localhost:3050
- Sentiment: http://localhost:5050

> **Note:** On Linux, use `host.docker.internal` or `--network host`. On Mac/Windows, `host.docker.internal` works by default.

---

### Option B: Local Development (Without Docker)

**Terminal 1 - MongoDB:**
```bash
mongod
```

**Terminal 2 - Database Microservice:**
```bash
cd server/database
npm install
node app.js
```

**Terminal 3 - Cars Inventory:**
```bash
cd server/carsInventory
npm install
node app.js
```

**Terminal 4 - Sentiment Analysis:**
```bash
cd server/djangoapp/microservices
pip install -r requirements.txt
python3 -m flask run --host=0.0.0.0 --port=5050
```

**Terminal 5 - Django Main App:**
```bash
cd server
pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py populate  # optional: seed car data
python manage.py runserver
```

**Terminal 6 - React Frontend:**
```bash
cd server/frontend
npm install
npm start
```

## Docker Instructions

### Build Images

```bash
# Django Main App
cd server
docker build -t dealership .

# Database Microservice
cd server/database
docker build -t db-service .

# Cars Inventory
cd ../carsInventory
docker build -t cars-inv .

# Sentiment Analysis
cd ../djangoapp/microservices
docker build -t sentiment .
```

### Run Containers

```bash
# Database (requires MongoDB)
docker run -d -p 3030:3030 \
  --name db-service \
  --link mongodb:mongo \
  db-service

# Cars Inventory (requires MongoDB)
docker run -d -p 3050:3050 \
  --name cars-inv \
  --link mongodb:mongo \
  cars-inv

# Sentiment Analysis
docker run -d -p 5050:5000 --name sentiment sentiment

# Django App
docker run -d -p 8000:8000 \
  -e backend_url=http://host.docker.internal:3030 \
  -e sentiment_analyzer_url=http://host.docker.internal:5050/ \
  -e searchcars_url=http://host.docker.internal:3050/ \
  --name dealership dealership
```

### Environment Variables for Django Container

| Variable | Description | Default |
|----------|-------------|---------|
| `backend_url` | Database microservice URL | `http://localhost:3030` |
| `sentiment_analyzer_url` | Sentiment service URL | `http://localhost:5050/` |
| `searchcars_url` | Cars inventory URL | `http://localhost:3050/` |

## Kubernetes Deployment

### Current State

The repository includes a basic `deployment.yaml` for the Django main app only:

```yaml
# server/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: dealership
  name: dealership
spec:
  replicas: 1
  selector:
    matchLabels:
      run: dealership
  template:
    metadata:
      labels:
        run: dealership
    spec:
      containers:
      - image: us.icr.io/sn-labs-joaovitorpae/dealership:latest
        imagePullPolicy: Always
        name: dealership
        ports:
        - containerPort: 8000
          protocol: TCP
      restartPolicy: Always
```

**Deploys to:** IBM Cloud Container Registry (`us.icr.io/sn-labs-joaovitorpae/dealership:latest`)

### ⚠️ Note: Incomplete K8s Configuration

The current `deployment.yaml` **only deploys the Django main app**. For a production-ready full-stack deployment, you need additional manifests:

**Missing Components:**
- **MongoDB** - StatefulSet with PersistentVolumeClaim for data persistence
- **Database Microservice** - Deployment + Service (ClusterIP)
- **Cars Inventory Microservice** - Deployment + Service (ClusterIP)
- **Sentiment Analysis** - Deployment + Service (ClusterIP)
- **ConfigMaps/Secrets** - For inter-service URLs and credentials
- **Ingress** - For external access (NGINX/Traefik)
- **Namespace** - Resource isolation
- **HorizontalPodAutoscaler** - For scaling under load

**Suggested K8s Structure:**
```
k8s/
├── namespace.yaml
├── configmap.yaml          # Service URLs
├── secrets.yaml            # DB credentials, SECRET_KEY
├── mongodb/
│   ├── statefulset.yaml
│   ├── pvc.yaml
│   └── service.yaml
├── database-service/
│   ├── deployment.yaml
│   └── service.yaml
├── cars-inventory/
│   ├── deployment.yaml
│   └── service.yaml
├── sentiment-analysis/
│   ├── deployment.yaml
│   └── service.yaml
├── django-app/
│   ├── deployment.yaml     # (existing, needs updates)
│   └── service.yaml
└── ingress.yaml
```

**To deploy current Django-only:**
```bash
kubectl apply -f server/deployment.yaml
kubectl expose deployment dealership --type=LoadBalancer --port=8000
```

## API Quick Reference

### Django Main App (`localhost:8000`)
```
POST   /api/login              # {userName, password}
POST   /api/register           # {userName, password, firstName, lastName, email}
GET    /api/logout
GET    /api/dealerships        # ?state=CA
GET    /api/dealer/:id
GET    /api/reviews/dealer/:id
POST   /api/review             # Auth required
GET    /api/cars
GET    /api/inventory/:dealerId?year=2020&make=Toyota
```

### Database Microservice (`localhost:3030`)
```
GET    /fetchDealers
GET    /fetchDealers/:state
GET    /fetchDealer/:id
GET    /fetchReviews/dealer/:id
POST   /insert_review
GET    /cars/:dealerId
```

### Cars Inventory (`localhost:3050`)
```
GET    /cars/:dealerId
GET    /carsbyyear/:dealerId/:year
GET    /carsbymake/:dealerId/:make
GET    /carsbymodel/:dealerId/:model
GET    /carsbymaxmileage/:dealerId/:mileage
GET    /carsbyprice/:dealerId/:price
```

### Sentiment Analysis (`localhost:5050`)
```
GET    /analyze/:text          # Returns {"sentiment": "positive|negative|neutral"}
```

## Environment Variables Summary

### Django App (`server/djangoapp/.env`)
```env
backend_url=http://localhost:3030
sentiment_analyzer_url=http://localhost:5050/
searchcars_url=http://localhost:3050/
```

### Database & Cars Inventory
Configured via Docker Compose or MongoDB connection string in code.

## License

ISC License - See LICENSE file for details.