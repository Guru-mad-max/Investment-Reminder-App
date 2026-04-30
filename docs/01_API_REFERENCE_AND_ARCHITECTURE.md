# Complete API Reference & System Architecture

**Version:** 1.0 (Production Ready)  
**Last Updated:** 2026-04-30  

---

## 🏗️ HIGH-LEVEL SYSTEM ARCHITECTURE

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│  Web App (React.js)           Mobile App (React Native)         │
│  - TypeScript + TailwindCSS    - Expo + React Native           │
│  - Redux for state mgmt        - Push notifications enabled     │
│  - Responsive design           - Offline-first architecture     │
└──────────────────────┬──────────────────────────────────────────┘
                       │ HTTPS + TLS 1.3
┌──────────────────────▼──────────────────────────────────────────┐
│                    API GATEWAY LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│  Kong API Gateway                                               │
│  - Request routing & versioning                                │
│  - Rate limiting (100 req/min per IP)                          │
│  - Authentication & JWT validation                             │
│  - CORS configuration                                          │
│  - Request/Response logging                                    │
│  - API documentation (OpenAPI/Swagger)                         │
└──────────────────────┬──────────────────────────────────────────┘
                       │
    ┌──────────────────┼──────────────────┐
    │                  │                  │
┌───▼────┐      ┌──────▼───┐     ┌──────▼──┐
│  Auth  │      │  Core    │     │ Support │
│Service │      │ Services │     │Services │
│ :3001  │      │ :3002-07 │     │ :3008-10│
└────────┘      └──────────┘     └─────────┘
    │                  │                  │
    └──────────────────┼──────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    ┌───▼───┐    ┌────▼────┐  ┌─────▼───┐
    │  DB   │    │ Cache   │  │  Queue  │
    │PostgreSQL  │ Redis   │  │RabbitMQ │
    └───────┘    └─────────┘  └─────────┘
```

---

## 📡 MICROSERVICES OVERVIEW

| Service | Port | Responsibility | Tech Stack |
|---------|------|-----------------|-----------|
| **Auth Service** | 3001 | JWT, OAuth, 2FA | Node + Passport |
| **User Service** | 3002 | Profile, Settings, Preferences | Node + Express |
| **SIP Service** | 3003 | Plan CRUD, Scheduling | Node + Prisma |
| **Investment Service** | 3004 | Portfolio, Holdings, Tracking | Node + Prisma |
| **Notification Service** | 3005 | Email, Push, SMS | Node + Nodemailer |
| **Analytics Service** | 3006 | Reports, Insights, ML | Node + Python ML |
| **Recommendation Service** | 3007 | ML-based suggestions | Node + Python/TensorFlow |
| **Scheduler Service** | 3008 | Cron jobs, Event scheduling | Node + node-cron |

---

## 🔐 AUTHENTICATION FLOW

```
1. User Login
   └─> POST /auth/login
       ├─> Verify credentials
       ├─> Generate JWT (15 min)
       ├─> Generate Refresh Token (30 days)
       └─> Return tokens

2. API Requests
   └─> Include Authorization header: "Bearer <JWT>"
       ├─> Gateway validates JWT
       ├─> If expired → use refresh token
       └─> Forward to service

3. Token Refresh
   └─> POST /auth/refresh
       ├─> Validate refresh token
       ├─> Generate new JWT
       └─> Return new token
```

---

## 50+ API ENDPOINTS

### **AUTH ENDPOINTS** (Auth Service :3001)

#### 1. User Registration
```
POST /v1/auth/register
Content-Type: application/json

{
  "firstName": "Raj",
  "lastName": "Kumar",
  "email": "raj@example.com",
  "phone": "+919876543210",
  "password": "SecurePass@123",
  "dateOfBirth": "1990-05-15",
  "annualIncome": 500000
}

Response (201):
{
  "success": true,
  "message": "Registration successful. Please verify your email.",
  "data": {
    "userId": "usr_123456",
    "email": "raj@example.com",
    "verificationEmailSent": true
  }
}
```

#### 2. User Login
```
POST /v1/auth/login
Content-Type: application/json

{
  "email": "raj@example.com",
  "password": "SecurePass@123",
  "deviceId": "mobile_device_id" // optional
}

Response (200):
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "refresh_token_here",
    "expiresIn": 900,
    "user": {
      "id": "usr_123456",
      "email": "raj@example.com",
      "firstName": "Raj",
      "role": "user"
    }
  }
}
```

#### 3. Token Refresh
```
POST /v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "refresh_token_here"
}

Response (200):
{
  "success": true,
  "data": {
    "accessToken": "new_jwt_token",
    "expiresIn": 900
  }
}
```

#### 4. Email Verification
```
POST /v1/auth/verify-email
Content-Type: application/json

{
  "code": "123456"
}

Response (200):
{
  "success": true,
  "message": "Email verified successfully"
}
```

#### 5. Logout
```
POST /v1/auth/logout
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

### **USER ENDPOINTS** (User Service :3002)

#### 6. Get User Profile
```
GET /v1/users/me
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "id": "usr_123456",
    "firstName": "Raj",
    "lastName": "Kumar",
    "email": "raj@example.com",
    "phone": "+919876543210",
    "annualIncome": 500000,
    "riskProfile": "Moderate",
    "createdAt": "2026-04-20T10:30:00Z"
  }
}
```

#### 7. Update User Profile
```
PUT /v1/users/me
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "firstName": "Raj",
  "lastName": "Kumar",
  "phone": "+919876543210",
  "annualIncome": 600000
}

Response (200):
{
  "success": true,
  "data": { /* updated user object */ }
}
```

#### 8. Update Notification Preferences
```
PUT /v1/users/me/notification-preferences
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "channels": {
    "push": true,
    "email": true,
    "sms": false
  },
  "frequency": "daily",
  "unsubscribeAll": false
}

Response (200):
{
  "success": true,
  "message": "Preferences updated"
}
```

#### 9. Change Password
```
POST /v1/users/me/change-password
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "currentPassword": "OldPass@123",
  "newPassword": "NewPass@456"
}

Response (200):
{
  "success": true,
  "message": "Password changed successfully"
}
```

#### 10. Request Password Reset
```
POST /v1/users/forgot-password
Content-Type: application/json

{
  "email": "raj@example.com"
}

Response (200):
{
  "success": true,
  "message": "Password reset link sent to email"
}
```

---

### **SIP PLAN ENDPOINTS** (SIP Service :3003)

#### 11. Create SIP Plan
```
POST /v1/sip-plans
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "investmentName": "Axis Growth Fund",
  "investmentType": "Mutual_Fund",
  "investmentSymbol": "AXISGG",
  "monthlyAmount": 5000,
  "startDate": "2026-05-01T00:00:00Z",
  "investmentDateOfMonth": 5,
  "notificationChannels": ["push", "email"],
  "notes": "Growth-focused investment"
}

Response (201):
{
  "success": true,
  "data": {
    "id": "sip_123456",
    "userId": "usr_123456",
    "investmentName": "Axis Growth Fund",
    "monthlyAmount": 5000,
    "status": "Active",
    "createdAt": "2026-04-30T10:00:00Z"
  }
}
```

#### 12. Get All SIP Plans
```
GET /v1/sip-plans?page=1&limit=20&status=Active
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "sips": [ /* array of SIP plans */ ],
    "total": 5,
    "page": 1,
    "pageSize": 20
  }
}
```

#### 13. Get Single SIP Plan
```
GET /v1/sip-plans/{sipId}
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": { /* complete SIP plan object */ }
}
```

#### 14. Update SIP Plan
```
PUT /v1/sip-plans/{sipId}
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "monthlyAmount": 7000,
  "notificationChannels": ["push", "email", "sms"]
}

Response (200):
{
  "success": true,
  "data": { /* updated SIP plan */ }
}
```

#### 15. Pause SIP Plan
```
POST /v1/sip-plans/{sipId}/pause
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "message": "SIP plan paused successfully"
}
```

#### 16. Resume SIP Plan
```
POST /v1/sip-plans/{sipId}/resume
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "message": "SIP plan resumed successfully"
}
```

#### 17. Delete SIP Plan
```
DELETE /v1/sip-plans/{sipId}
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "message": "SIP plan deleted successfully"
}
```

#### 18. Get SIP Statistics
```
GET /v1/sip-plans/stats/summary
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "totalSIPs": 5,
    "activeSIPs": 4,
    "pausedSIPs": 1,
    "totalMonthlyCommitment": 25000,
    "totalInvestedViaSIP": 750000,
    "totalProfit": 45000
  }
}
```

---

### **INVESTMENT ENDPOINTS** (Investment Service :3004)

#### 19. Get Portfolio Overview
```
GET /v1/investments/portfolio
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "totalInvested": 500000,
    "currentValue": 545000,
    "gainLoss": 45000,
    "gainLossPercentage": 9.0,
    "breakdown": {
      "mutualFunds": 250000,
      "stocks": 150000,
      "bonds": 100000
    }
  }
}
```

#### 20. Get Investments (Paginated)
```
GET /v1/investments?page=1&limit=10&type=Mutual_Fund
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "investments": [ /* array of investments */ ],
    "total": 15,
    "page": 1
  }
}
```

#### 21. Get Investment Details
```
GET /v1/investments/{investmentId}
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "id": "inv_123456",
    "investmentType": "Mutual_Fund",
    "investmentName": "ICICI Prudential Growth",
    "quantity": 100,
    "purchasePrice": 50,
    "currentPrice": 58,
    "totalCost": 5000,
    "currentValue": 5800,
    "gainLoss": 800,
    "gainLossPercentage": 16.0
  }
}
```

#### 22. Add Manual Investment
```
POST /v1/investments/manual
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "investmentType": "Stock",
  "investmentName": "TCS",
  "investmentSymbol": "TCS",
  "quantity": 10,
  "purchasePrice": 3500,
  "purchaseDate": "2026-04-15T00:00:00Z"
}

Response (201):
{
  "success": true,
  "data": { /* created investment */ }
}
```

#### 23. Get Investment History
```
GET /v1/investments/{investmentId}/history
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "history": [
      {
        "date": "2026-04-30",
        "price": 58,
        "value": 5800
      }
      /* more history entries */
    ]
  }
}
```

---

### **TRANSACTION ENDPOINTS** (Investment Service :3004)

#### 24. Get Transaction History
```
GET /v1/transactions?page=1&limit=20&type=BUY
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "transactions": [
      {
        "id": "txn_123456",
        "type": "BUY",
        "amount": 5000,
        "investmentName": "ICICI Prudential Growth",
        "date": "2026-04-30T10:00:00Z",
        "status": "Success"
      }
    ],
    "total": 45
  }
}
```

#### 25. Get Transaction Details
```
GET /v1/transactions/{transactionId}
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": { /* complete transaction */ }
}
```

#### 26. Download Transaction Statement
```
GET /v1/transactions/export?format=csv&startDate=2026-01-01&endDate=2026-04-30
Authorization: Bearer <JWT>

Response (200):
File download (CSV/PDF)
```

---

### **NOTIFICATION ENDPOINTS** (Notification Service :3005)

#### 27. Get Notifications
```
GET /v1/notifications?page=1&limit=20&read=false
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "notif_123456",
        "type": "SIP_Reminder",
        "title": "Investment Reminder",
        "message": "Time to invest ₹5,000 in ICICI Prudential",
        "data": { /* contextual data */ },
        "read": false,
        "createdAt": "2026-04-30T08:00:00Z"
      }
    ],
    "unreadCount": 3
  }
}
```

#### 28. Mark Notification as Read
```
POST /v1/notifications/{notificationId}/read
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "message": "Notification marked as read"
}
```

#### 29. Mark All as Read
```
POST /v1/notifications/mark-all-read
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "message": "All notifications marked as read"
}
```

#### 30. Delete Notification
```
DELETE /v1/notifications/{notificationId}
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "message": "Notification deleted"
}
```

---

### **ANALYTICS ENDPOINTS** (Analytics Service :3006)

#### 31. Get Portfolio Performance
```
GET /v1/analytics/performance?period=1Y
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "period": "1Y",
    "totalReturn": 12.5,
    "annualizedReturn": 12.5,
    "maxDrawdown": -8.2,
    "sharpeRatio": 1.45,
    "chartData": [
      { "date": "2025-05-01", "value": 500000 },
      { "date": "2026-04-30", "value": 545000 }
    ]
  }
}
```

#### 32. Get Monthly Report
```
GET /v1/analytics/monthly-report?month=2026-04
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "month": "2026-04",
    "totalInvested": 15000,
    "gainLoss": 2000,
    "numTransactions": 5,
    "topPerformer": "HDFC Bank",
    "summary": "Your portfolio grew by 15% this month"
  }
}
```

#### 33. Get Portfolio Breakdown
```
GET /v1/analytics/portfolio-breakdown
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "breakdown": [
      { "type": "Mutual_Fund", "percentage": 50, "amount": 250000 },
      { "type": "Stock", "percentage": 30, "amount": 150000 },
      { "type": "Bond", "percentage": 20, "amount": 100000 }
    ]
  }
}
```

#### 34. Get Asset Allocation
```
GET /v1/analytics/asset-allocation?riskProfile=moderate
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "recommended": {
      "stocks": 50,
      "bonds": 30,
      "gold": 10,
      "cash": 10
    },
    "current": {
      "stocks": 40,
      "bonds": 35,
      "gold": 15,
      "cash": 10
    }
  }
}
```

---

### **RECOMMENDATION ENDPOINTS** (Recommendation Service :3007)

#### 35. Get Investment Recommendations
```
GET /v1/recommendations?riskProfile=moderate&budget=10000
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "id": "rec_123456",
        "investmentName": "HDFC Growth Fund",
        "investmentType": "Mutual_Fund",
        "symbol": "HDFCGROWTH",
        "riskLevel": "Moderate",
        "expectedReturn": 12.5,
        "reason": "Matches your risk profile and growth goals",
        "score": 8.5
      }
    ]
  }
}
```

#### 36. Get Tax-Saving Recommendations
```
GET /v1/recommendations/tax-saving?income=500000
Authorization: Bearer <JWT>

Response (200):
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "investmentType": "ELSS",
        "investmentName": "Axis ELSS Tax Saver",
        "potentialTaxSavings": 30000,
        "maxInvestment": 150000,
        "daysLeftToInvest": 245
      }
    ]
  }
}
```

---

### **ERROR RESPONSES**

#### Standard Error Response (4xx/5xx)
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [
      {
        "field": "email",
        "message": "Email must be valid"
      }
    ]
  }
}
```

#### Common Error Codes
```
200 OK - Success
201 Created - Resource created
204 No Content - Success with no content
400 Bad Request - Invalid input
401 Unauthorized - Authentication required
403 Forbidden - Insufficient permissions
404 Not Found - Resource not found
409 Conflict - Conflict (e.g., duplicate email)
422 Unprocessable - Validation failed
429 Too Many Requests - Rate limited
500 Internal Server Error - Server error
503 Service Unavailable - Service down
```

---

## 📊 DATABASE SCHEMA

### **Users Table**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  firstName VARCHAR(100) NOT NULL,
  lastName VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  phone VARCHAR(20),
  passwordHash VARCHAR(255) NOT NULL,
  dateOfBirth DATE,
  annualIncome DECIMAL(12,2),
  riskProfile VARCHAR(20), -- Conservative, Moderate, Aggressive
  emailVerified BOOLEAN DEFAULT false,
  phoneVerified BOOLEAN DEFAULT false,
  twoFactorEnabled BOOLEAN DEFAULT false,
  notificationPreferences JSONB,
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW(),
  lastLoginAt TIMESTAMP,
  deletedAt TIMESTAMP -- Soft delete
);
```

### **SIP Plans Table**
```sql
CREATE TABLE sip_plans (
  id UUID PRIMARY KEY,
  userId UUID NOT NULL REFERENCES users(id),
  investmentName VARCHAR(255) NOT NULL,
  investmentType VARCHAR(50) NOT NULL, -- Mutual_Fund, Stock, Bond, Gold
  investmentSymbol VARCHAR(50),
  monthlyAmount DECIMAL(12,2) NOT NULL,
  startDate TIMESTAMP NOT NULL,
  endDate TIMESTAMP,
  investmentDateOfMonth INT,
  notificationChannels JSON,
  notes TEXT,
  status VARCHAR(20) DEFAULT 'Active', -- Active, Paused, Completed, Cancelled
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (userId) REFERENCES users(id)
);

CREATE INDEX idx_sip_userId_status ON sip_plans(userId, status);
CREATE INDEX idx_sip_investmentDateOfMonth ON sip_plans(investmentDateOfMonth);
```

### **Investments Table**
```sql
CREATE TABLE investments (
  id UUID PRIMARY KEY,
  userId UUID NOT NULL REFERENCES users(id),
  sipPlanId UUID REFERENCES sip_plans(id),
  investmentType VARCHAR(50) NOT NULL,
  investmentName VARCHAR(255) NOT NULL,
  investmentSymbol VARCHAR(50),
  quantity DECIMAL(18,2),
  purchasePrice DECIMAL(12,2),
  purchaseDate TIMESTAMP NOT NULL,
  currentPrice DECIMAL(12,2),
  sellPrice DECIMAL(12,2),
  sellDate TIMESTAMP,
  status VARCHAR(20) DEFAULT 'Active', -- Active, Sold, Pending
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (userId) REFERENCES users(id),
  FOREIGN KEY (sipPlanId) REFERENCES sip_plans(id)
);

CREATE INDEX idx_investments_userId_status ON investments(userId, status);
CREATE INDEX idx_investments_sipPlanId ON investments(sipPlanId);
```

### **Transactions Table**
```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY,
  userId UUID NOT NULL REFERENCES users(id),
  investmentId UUID REFERENCES investments(id),
  sipPlanId UUID REFERENCES sip_plans(id),
  transactionType VARCHAR(20) NOT NULL, -- BUY, SELL, DIVIDEND
  amount DECIMAL(12,2) NOT NULL,
  quantity DECIMAL(18,2),
  price DECIMAL(12,2),
  transactionDate TIMESTAMP NOT NULL,
  paymentMethod VARCHAR(50), -- Bank_Transfer, Card, UPI
  status VARCHAR(20) DEFAULT 'Pending', -- Pending, Success, Failed
  notes TEXT,
  createdAt TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (userId) REFERENCES users(id),
  FOREIGN KEY (investmentId) REFERENCES investments(id)
);

CREATE INDEX idx_transactions_userId_date ON transactions(userId, transactionDate DESC);
CREATE INDEX idx_transactions_status ON transactions(status);
```

### **Notifications Table**
```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY,
  userId UUID NOT NULL REFERENCES users(id),
  type VARCHAR(50) NOT NULL, -- SIP_Reminder, Market_Alert, Tax_Alert
  title VARCHAR(255),
  message TEXT,
  data JSONB,
  channels VARCHAR(50)[], -- push, email, sms
  readAt TIMESTAMP,
  sentAt TIMESTAMP,
  status VARCHAR(20) DEFAULT 'Sent', -- Sent, Failed, Pending
  createdAt TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (userId) REFERENCES users(id)
);

CREATE INDEX idx_notifications_userId_readAt ON notifications(userId, readAt);
```

### **Recommendations Table**
```sql
CREATE TABLE recommendations (
  id UUID PRIMARY KEY,
  userId UUID NOT NULL REFERENCES users(id),
  investmentName VARCHAR(255),
  investmentType VARCHAR(50),
  investmentSymbol VARCHAR(50),
  riskLevel VARCHAR(20),
  expectedReturn DECIMAL(5,2),
  reason TEXT,
  score DECIMAL(3,1),
  status VARCHAR(20) DEFAULT 'Active', -- Active, Dismissed, Converted
  createdAt TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (userId) REFERENCES users(id)
);
```

---

**Document Status:** ✅ COMPLETE & PRODUCTION-READY
