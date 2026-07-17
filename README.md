# dummy-products-api

A MuleSoft 4 application that proxies product data from [dummyjson.com](https://dummyjson.com).

---

## 📦 Project Structure

```
dummy-products-api/
├── pom.xml
├── mule-artifact.json
├── settings.xml.template          ← Copy to ~/.m2/settings.xml and fill in credentials
├── .github/
│   └── workflows/
│       └── deploy.yml             ← GitHub Actions CI/CD pipeline
└── src/
    ├── main/
    │   ├── mule/
    │   │   ├── http-config.xml    ← Global HTTP Listener & Request configs
    │   │   └── products-api.xml   ← API flows
    │   └── resources/
    │       ├── config-local.properties
    │       ├── config-dev.properties
    │       └── log4j2.xml
    └── test/
        └── munit/
            └── products-api-test.xml
```

---

## 🚀 Endpoints

| Method | URL | Description |
|--------|-----|-------------|
| GET | `http://localhost:8081/dummy/products` | Fetch all products |
| GET | `http://localhost:8081/dummy/products/{id}` | Fetch product by ID |

---

## ⚙️ Running Locally (Anypoint Studio)

### Prerequisites
- Anypoint Studio 7.x (with embedded Mule Runtime 4.6.x)

### Steps
1. Import project: **File → Import → Anypoint Studio → Anypoint Studio Project from File System**
2. Select this folder
3. Right-click project → **Run As → Mule Application**
4. The app starts on `http://localhost:8081`

> The app defaults to `mule.env=local`. To use dev properties, add the JVM argument `-Dmule.env=dev`.

---

## 🔧 Configuration

All environment-specific settings live in `src/main/resources/config-{env}.properties`:

| Property | Default | Description |
|----------|---------|-------------|
| `http.listener.host` | `0.0.0.0` | Listener bind host |
| `http.listener.port` | `8081` | Listener port |
| `dummyjson.host` | `dummyjson.com` | Upstream API host |
| `dummyjson.port` | `443` | Upstream API port (HTTPS) |

---

## 🚢 Deploying to Anypoint Platform (CloudHub 2.0)

### Option A — GitHub Actions (recommended)

This repo includes a ready-made CI/CD pipeline at `.github/workflows/deploy.yml`.
Every push to `main` will build and deploy the app to CloudHub 2.0 automatically.

#### Step 1 — Push code to GitHub

```bash
git init
git add .
git commit -m "feat: initial commit"
git remote add origin https://github.com/<your-org>/<your-repo>.git
git push -u origin main
```

#### Step 2 — Add GitHub Secrets

Go to your GitHub repo → **Settings → Secrets and variables → Actions** and add:

| Secret name | Value |
|---|---|
| `ANYPOINT_USERNAME` | Your Anypoint Platform username **or** Connected App `client_id` |
| `ANYPOINT_PASSWORD` | Your Anypoint Platform password **or** Connected App `client_secret` |

> **Recommended:** Use a Connected App (Service Account) instead of your personal credentials.
> Create one at: Anypoint Platform → Access Management → Connected Apps → **Create app** (with `CloudHub Application` scope).

#### Step 3 — Add GitHub Variables (optional, defaults shown)

Go to **Settings → Secrets and variables → Actions → Variables**:

| Variable name | Default | Description |
|---|---|---|
| `CLOUDHUB_ENV` | `Sandbox` | Anypoint environment name |
| `CLOUDHUB_BUSINESS_GROUP` | _(root org)_ | Business group name if using sub-orgs |
| `CLOUDHUB_TARGET` | `Sandbox` | CloudHub 2.0 shared space name |

#### Step 4 — Push and watch it deploy

Push any commit to `main` and go to **Actions** tab in GitHub to monitor the pipeline.

---

### Option B — Deploy manually from command line

#### 1. Configure Maven credentials

Copy the template and fill in your credentials:

```bash
cp settings.xml.template ~/.m2/settings.xml
# Edit ~/.m2/settings.xml — replace YOUR_ANYPOINT_USERNAME_OR_CLIENT_ID etc.
```

#### 2. Build and deploy

```bash
mvn clean deploy -DskipTests \
  -Danypoint.username=YOUR_USERNAME \
  -Danypoint.password=YOUR_PASSWORD \
  -Dcloudhub.environment=Sandbox \
  -Dcloudhub.target=Sandbox
```

#### 3. Verify the app is running

After deployment, the app will be available at:
```
https://dummy-products-api.<your-org>.cloudhub.io/dummy/products
```

---

## 🧪 Testing

### Test with browser or Postman (once running)

```
GET http://localhost:8081/dummy/products
GET http://localhost:8081/dummy/products/1
```

### Run MUnit tests (requires Anypoint Maven credentials)

```bash
mvn clean test
```

---

## 📋 Error Responses

| HTTP Status | Code | Scenario |
|-------------|------|----------|
| 404 | `PRODUCT_NOT_FOUND` | Product ID does not exist |
| 502 | `DOWNSTREAM_ERROR` | DummyJSON service unreachable |
| 500 | `INTERNAL_SERVER_ERROR` | Unexpected runtime error |

---

## 🏗️ Built With

- **MuleSoft 4.6.0**
- **HTTP Connector 1.9.1**
- **DataWeave 2.0**
- **MUnit 3.1.0**
