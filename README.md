# WhatsApp Automation Platform

A modern, production-ready WhatsApp Automation Platform built with Python (FastAPI), Next.js, PostgreSQL, Redis, and Celery. It leverages the official WhatsApp Business Cloud API to allow businesses to automate, schedule, and analyze their WhatsApp messaging.

## Features

- **Dashboard**: Track sent, delivered, failed, and scheduled messages.
- **Contact Management**: Import/Export, group, and tag contacts.
- **WhatsApp Messaging**: Send text, media, and template messages.
- **Bulk Campaigns**: Batch sending with API rate-limit handling.
- **Scheduling**: One-time, recurring, and event-based scheduling.
- **AI Auto Reply**: Integrate with LLMs to automatically answer queries.
- **Live Chat Inbox**: Real-time messaging via WebSockets.
- **Templates**: Manage and sync WhatsApp approved templates.

---

## 🛠️ Prerequisites

Before you start, make sure you have the following installed:
- [Docker & Docker Compose](https://www.docker.com/)
- [Node.js 18+](https://nodejs.org/) (for local frontend development)
- [Python 3.11+](https://www.python.org/) (for local backend development)
- [ngrok](https://ngrok.com/) (for exposing local webhooks to Meta)

---

## 📱 WhatsApp Cloud API Integration Guide

To connect the application to WhatsApp, you must create and configure a Meta Developer App.

### 1. Create a Meta Developer App
1. Go to the [Meta Developer Dashboard](https://developers.facebook.com/).
2. Click **Create App** -> Select **Other** -> Select **Business**.
3. Name your app and connect it to a Meta Business Account.

### 2. Configure WhatsApp
1. On the App Dashboard, click **Set Up** on the **WhatsApp** product.
2. Navigate to **API Setup**.
3. Copy the following credentials:
   - **Temporary Access Token** (or generate a permanent system user token).
   - **Phone Number ID**.
   - **WhatsApp Business Account ID**.

### 3. Configure Webhooks (For Receiving Messages)
Webhooks are required for receiving incoming messages, delivery statuses, and read receipts.
1. Run `ngrok http 8000` (assuming your backend runs on port 8000).
2. Copy your public ngrok URL (e.g., `https://<your-id>.ngrok.io`).
3. In the Meta Developer Dashboard, go to **WhatsApp > Configuration**.
4. Under **Webhook**, click **Edit**.
5. Set the **Callback URL** to `https://<your-id>.ngrok.io/api/v1/webhook/whatsapp`.
6. Enter a secure **Verify Token** (this should match `WHATSAPP_WEBHOOK_VERIFY_TOKEN` in your `.env`).
7. Subscribe to the `messages` field.

---

## ⚙️ Environment Configuration

Create a `.env` file in the `backend` and `frontend` directories based on the `.env.example`.

**Backend `.env`:**
```env
# Database
POSTGRES_SERVER=db
POSTGRES_USER=wa_user
POSTGRES_PASSWORD=wa_password
POSTGRES_DB=wa_db

# Security
SECRET_KEY=your_super_secret_jwt_key
ACCESS_TOKEN_EXPIRE_MINUTES=11520

# WhatsApp API
WHATSAPP_API_TOKEN=your_whatsapp_access_token
WHATSAPP_PHONE_ID=your_phone_number_id
WHATSAPP_BUSINESS_ACCOUNT_ID=your_business_account_id
WHATSAPP_WEBHOOK_VERIFY_TOKEN=your_secure_verify_token

# Redis / Celery
REDIS_URL=redis://redis:6379/0
```

---

## 🚀 Local Deployment (Docker Compose)

The easiest way to run the full stack locally is via Docker Compose.

1. **Build and start the containers:**
   ```bash
   docker-compose up --build -d
   ```
2. **Run Database Migrations (Alembic):**
   ```bash
   docker-compose exec backend alembic upgrade head
   ```
3. **Access the Application:**
   - **Frontend**: http://localhost:3000
   - **Backend API**: http://localhost:8000
   - **API Docs (Swagger)**: http://localhost:8000/api/v1/openapi.json

---

## 🌍 Production Deployment

For deploying the platform to a production server (e.g., AWS EC2, DigitalOcean Droplet, Ubuntu Server):

### 1. Server Setup
1. Install Docker and Docker Compose on your server.
2. Clone this repository onto the server.
3. Configure the `.env` variables with production credentials.

### 2. Nginx & HTTPS Configuration
You must use HTTPS in production for Meta's webhooks to work. 

1. Install **Nginx** and **Certbot**.
2. Configure a reverse proxy pointing to your Docker containers.
   - Map `/api` traffic to the FastAPI backend (port 8000).
   - Map all other traffic to the Next.js frontend (port 3000).
3. Run `certbot --nginx` to automatically generate and apply SSL certificates.

### 3. Run Production Containers
1. Ensure the `docker-compose.prod.yml` (if applicable) is configured for production (e.g., setting `NODE_ENV=production`, disabling debug logs).
2. Start the services:
   ```bash
   docker-compose -f docker-compose.yml up -d
   ```
3. Scale Celery workers if required for high throughput:
   ```bash
   docker-compose up -d --scale celery_worker=3
   ```

### 4. CI/CD (GitHub Actions)
You can configure a GitHub Actions workflow to:
1. Run `pytest` on push.
2. Build Docker images and push them to a registry (e.g., Docker Hub, AWS ECR).
3. SSH into the production server, pull the latest images, and restart the containers.
