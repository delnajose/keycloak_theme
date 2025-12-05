# Keycloak UI Customization – Technical Implementation

This repository documents the comprehensive customization of Keycloak’s UI for login, registration, and email verification workflows, including installation, configuration, and integration with frontend applications.

---

## Table of Contents
1. [Project Overview](#project-overview)  
2. [System Requirements](#system-requirements)  
3. [Installation & Setup](#installation--setup)  
4. [Realm & Client Configuration](#realm--client-configuration)  
5. [Custom Theme Development](#custom-theme-development)  
6. [Frontend Integration](#frontend-integration)  
7. [Email and SMTP Configuration](#email-and-smtp-configuration)  
8. [Token Configuration and Management](#token-configuration-and-management)  
9. [References](#references)  

---

## Project Overview

Keycloak UI customization was implemented to:  
- Align the authentication interface with corporate branding.  
- Enhance user experience during login, registration, and email verification.  
- Integrate seamlessly with Angular-based frontend applications.  

The approach involves creating a custom Keycloak theme, configuring authentication flows, email verification, and ensuring proper session management and redirect handling.

---

## System Requirements

| Component | Version / Requirement |
|-----------|---------------------|
| Operating System | Windows / macOS / Linux |
| Java Development Kit | OpenJDK 8 or higher |
| Node.js & npm | Node 20+, npm 10+ (for Angular integration) |
| Keycloak | 21.x+ |
| Tools | Git, IDE (VSCode/IntelliJ), Docker (optional) |

---

## Installation & Setup

### Docker Commands for Keycloak
```bash
docker run --name keycloak_local -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin -v C:\keycloak-themes\themes:/opt/keycloak/themes quay.io/keycloak/keycloak:latest start-dev![Uploading image.png…]()

```

> **Note (Linux/macOS):**
```bash
-v /home/user/keycloak-themes/themes:/opt/keycloak/themes
```

```bash
docker restart keycloak_local  # Restart after theme changes
```

#### Docker Images Used
- **Keycloak server image:** `quay.io/keycloak/keycloak:latest`  
- **SMTP server image:** `haravich/fake-smtp-server` or `test-smtp-local`  

---

### Java Installation
```bash
java -version
```
Ensure OpenJDK 8+ is installed and properly configured in `PATH`.

### Node.js & Angular Setup
```bash
node -v
npm -v
npm install -g @angular/cli
```

---

## Realm & Client Configuration

### Create Realm
- Navigate to Admin Console → **Add Realm**
- Provide `Realm Name` and `Display Name`.

### Configure Client
- Add new client with:
  - Client Protocol: `openid-connect`
  - Valid Redirect URIs: `http://localhost:4200/*`
  - Web Origins: `http://localhost:4200`
  - Root URL: Angular app URL

### Authentication Flows
- Customize login, registration, and email verification flows.
- Enable or disable features as per application requirements.

---

## Custom Theme Development

### Theme Directory Structure
```text
my-theme/
├─ theme.properties
├─ login/
│   ├─ login.ftl
│   ├─ register.ftl
│   ├─ messages.properties
│   ├─ resources/
│   │   ├─ css/
│   │   │   └─ styles.css
│   │   └─ js/
│   │       └─ custom.js
├─ account/
│   ├─ account.ftl
│   └─ messages.properties
└─ common/
    ├─ template.ftl
    └─ commons.ftl
```

### Theme Parent Options
| Parent      | Description | Use Case | Pros | Cons |
|------------|-------------|----------|------|------|
| `base`      | Minimal skeleton provided by Keycloak | Full custom layout | Total flexibility; can create unique designs | Must provide all CSS/JS and templates yourself |
| `keycloak`  | Default Keycloak theme | Minor branding and visual tweaks | Easy to maintain; inherits templates and styling | Limited flexibility for layout changes |
| `none`      | No inheritance | Fully custom theme | Maximum control; nothing is inherited | High maintenance; everything must be implemented manually |

---

### Examples for Each Parent Option

#### **1. Parent: `none`**
```properties
parent=none
styles=css/styles.css
scripts=js/custom.js
```
- Full control over HTML, CSS, and JS.
- Must implement all pages yourself.
- Suitable for highly branded applications.

**Simple login page (`login/login.ftl`):**
```ftl
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="resources/css/styles.css">
  <script src="resources/js/custom.js"></script>
</head>
<body>
  <h1>Welcome to My App</h1>
  <form action="${url.loginAction}" method="post">
    <input type="text" name="username" placeholder="Username"/>
    <input type="password" name="password" placeholder="Password"/>
    <button type="submit">Login</button>
  </form>
</body>
</html>
```

#### **2. Parent: `base`**
```properties
parent=base
styles=css/styles.css
scripts=js/custom.js
```
- Inherits only minimal structure.
- Provides flexibility for custom layout while keeping some base utilities.
- Suitable for mostly custom design with some reusable elements.
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/0dc1e802-1bda-461a-8b39-169eab1c11a1" />
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/2d75cc7f-10a4-4ea3-8980-9e6fcbb74682" />


#### **3. Parent: `keycloak`**
```properties
parent=keycloak
import=common/keycloak
styles=css/styles.css
scripts=js/custom.js
```
- Inherits full Keycloak templates, CSS, and JS.
- Ideal for small branding adjustments or theme tweaks.
- Easy to maintain with minimal changes.

---
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/b73796dc-3155-45ea-96d0-4fcf0e421571" />
<img width="500" height="500" alt="image (8)" src="https://github.com/user-attachments/assets/5d1abc97-e1fc-4d20-bfba-94a7de81e515" />


### Deployment
1. Copy theme to:
```
<keycloak-installation>/themes/my-theme/
```
2. Apply theme in Keycloak Admin Console:
- Login theme: `Realm Settings → Themes → Login Theme → my-theme`  
3. Restart Keycloak if necessary.

### Best Practices
- Use theme inheritance to reduce maintenance.
- Override only necessary templates, CSS, and JS.
- Test all changes in a development environment before production.
- Start with `keycloak` if new; move to `base` or `none` for more control.

### Enable Theme in Keycloak
- Admin Console → `Realm Settings → Themes → Login Theme → my-theme`

---

## Frontend Integration (Angular)

### Install Keycloak Adapter
```bash
npm install keycloak-angular keycloak-js
```

### main.ts Configuration
```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { App } from './app/app';
import { provideKeycloak } from 'keycloak-angular';

bootstrapApplication(App, {
  providers: [
    provideKeycloak({
      config: {
        url: 'http://localhost:8080',
        realm: 'myrealm',
        clientId: 'siemens-portal'
      },
      initOptions: {
        onLoad: 'login-required',
        checkLoginIframe: false
      }
    }),
    ...appConfig.providers
  ]
}).catch(err => console.error(err));
```

### Handle Login/Logout
- Redirect users to Keycloak login page.  
- Use `keycloak.logout()` to clear sessions.

---

## Email and SMTP Configuration

- SMTP ports: `1025/tcp` and `1080/tcp`  
- Use container name or `host.docker.internal` instead of fixed IP.  

### Keycloak Email Settings
- Go to `Realm Settings → Email`  
- Example:
  - Host: `localhost`
  - Port: `1025`
  - From: `noreply@example.com`
- Test connection in Keycloak console.

![Email setup](https://github.com/delnajose/Keycloak_china/blob/main/Images/email1.png)  
![Email setup](https://github.com/delnajose/Keycloak_china/blob/main/Images/email2.png)

---

## Token Configuration and Management

### OIDC Client Configuration
- Ensure `Client Protocol = OpenID Connect`  
- Enable `Standard Flow`  

### Token Lifespans
- Access Token Lifespan (~5 min)  
- Refresh Token Lifespan (~30 min)  
- Client Session Lifespan (30–60 min)

### JWK (JSON Web Keys)
- Public keys for token signature verification  
- Path: `/realms/<realm>/protocol/openid-connect/certs`

---

## References
- [Keycloak Official Documentation](https://www.keycloak.org/documentation)  
- [Keycloak Theme Development Guide](https://www.keycloak.org/docs/latest/server_development/#_themes)  
- [Keycloak GitHub Repository](https://github.com/keycloak/keycloak)

