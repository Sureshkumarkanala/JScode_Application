# Inventory Management System - Implementation Plan

## Project Overview

A full-featured inventory management system built with Next.js, featuring JWT authentication, Google OAuth, 2-factor authentication, and comprehensive inventory tracking capabilities.

## Technology Stack

### Frontend
- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **UI Libraries**:
  - Radix UI (unstyled, accessible components)
  - shadcn/ui (styled components built on Radix UI)
- **Styling**: Tailwind CSS (integrated with shadcn/ui)
- **State Management**: React Context + Server Components
- **Forms**: React Hook Form + Zod validation

### Backend
- **API**: Next.js API Routes (App Router)
- **Database**: PostgreSQL
- **ORM**: Prisma
- **Authentication**: NextAuth.js v5 (Auth.js)
- **2FA**: TOTP (Time-based One-Time Password) using `otplib`
- **Session Management**: JWT tokens

### Other Tools
- **Barcode**: HTML5 QR Code scanner library
- **Charts**: Recharts for analytics
- **Icons**: Lucide React
- **Date Handling**: date-fns
- **PDF Export**: jsPDF / react-pdf
- **CSV Export**: papaparse

## System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        UI[Next.js Frontend]
        AuthUI[Auth Pages]
        Dashboard[Dashboard]
        Inventory[Inventory Management]
        Reports[Reports & Analytics]
    end

    subgraph "API Layer"
        API[Next.js API Routes]
        AuthAPI[Auth Endpoints]
        InventoryAPI[Inventory Endpoints]
        ReportAPI[Report Endpoints]
    end

    subgraph "Auth Layer"
        NextAuth[NextAuth.js]
        JWT[JWT Tokens]
        GoogleOAuth[Google OAuth]
        TOTP[2FA TOTP]
    end

    subgraph "Data Layer"
        Prisma[Prisma ORM]
        PostgreSQL[(PostgreSQL Database)]
    end

    subgraph "External Services"
        Google[Google OAuth Provider]
        Email[Email Service for Alerts]
    end

    UI --> API
    AuthUI --> AuthAPI
    Dashboard --> API
    Inventory --> InventoryAPI
    Reports --> ReportAPI

    AuthAPI --> NextAuth
    NextAuth --> JWT
    NextAuth --> GoogleOAuth
    NextAuth --> TOTP
    NextAuth --> Google

    API --> Prisma
    Prisma --> PostgreSQL

    API --> Email
```

## Database Schema Design

### Core Models

```mermaid
erDiagram
    User ||--o{ Product : creates
    User ||--o{ PurchaseOrder : creates
    User ||--o{ Sale : creates
    User ||--o{ InventoryMovement : creates
    User ||--o{ AuditLog : creates
    
    Category ||--o{ Product : contains
    
    Supplier ||--o{ Product : supplies
    Supplier ||--o{ PurchaseOrder : receives
    
    Location ||--o{ Stock : has
    Location ||--o{ InventoryMovement : tracks
    
    Product ||--o{ Stock : has
    Product ||--o{ PurchaseOrderItem : included_in
    Product ||--o{ SaleItem : included_in
    Product ||--o{ InventoryMovement : tracked_in
    Product ||--o{ Barcode : has
    
    PurchaseOrder ||--o{ PurchaseOrderItem : contains
    
    Sale ||--o{ SaleItem : contains
    
    User {
        uuid id PK
        string name
        string email UK
        string password
        string role
        string twoFactorSecret
        boolean twoFactorEnabled
        datetime createdAt
        datetime updatedAt
    }

    Category {
        uuid id PK
        string name
        string description
        uuid parentId FK
        datetime createdAt
        datetime updatedAt
    }

    Supplier {
        uuid id PK
        string name
        string email
        string phone
        string address
        datetime createdAt
        datetime updatedAt
    }

    Location {
        uuid id PK
        string name
        string address
        string type
        datetime createdAt
        datetime updatedAt
    }

    Product {
        uuid id PK
        string name
        string sku UK
        string description
        decimal costPrice
        decimal sellingPrice
        uuid categoryId FK
        uuid supplierId FK
        string barcode
        string imageUrl
        boolean isActive
        datetime createdAt
        datetime updatedAt
    }

    Stock {
        uuid id PK
        uuid productId FK
        uuid locationId FK
        int quantity
        int reorderLevel
        datetime lastUpdated
    }

    PurchaseOrder {
        uuid id PK
        string orderNumber
        uuid supplierId FK
        uuid userId FK
        string status
        decimal totalAmount
        datetime orderDate
        datetime expectedDeliveryDate
        datetime createdAt
        datetime updatedAt
    }

    PurchaseOrderItem {
        uuid id PK
        uuid purchaseOrderId FK
        uuid productId FK
        int quantity
        decimal unitPrice
        decimal total
    }

    Sale {
        uuid id PK
        string invoiceNumber
        uuid userId FK
        string status
        decimal totalAmount
        datetime saleDate
        datetime createdAt
        datetime updatedAt
    }

    SaleItem {
        uuid id PK
        uuid saleId FK
        uuid productId FK
        int quantity
        decimal unitPrice
        decimal total
    }

    InventoryMovement {
        uuid id PK
        uuid productId FK
        uuid locationId FK
        uuid userId FK
        string type
        int quantity
        string reference
        string notes
        datetime createdAt
    }

    Barcode {
        uuid id PK
        uuid productId FK
        string barcode UK
        string type
        datetime createdAt
    }

    AuditLog {
        uuid id PK
        uuid userId FK
        string entityType
        uuid entityId
        string action
        json changes
        datetime createdAt
    }
```

## Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant NextAuth
    participant Google
    participant DB
    participant TOTP

    User->>UI: Login with Google
    UI->>NextAuth: Initiate OAuth
    NextAuth->>Google: Redirect to Google
    Google->>User: Show consent screen
    User->>Google: Authorize
    Google->>NextAuth: Return auth code
    NextAuth->>DB: Check/Create user
    DB-->>NextAuth: User data
    NextAuth->>NextAuth: Generate JWT
    NextAuth->>User: Check 2FA enabled?
    
    alt 2FA Enabled
        NextAuth->>User: Prompt for TOTP code
        User->>TOTP: Enter code from app
        TOTP->>NextAuth: Validate code
        NextAuth->>UI: Return JWT token
    else 2FA Not Enabled
        NextAuth->>UI: Return JWT token
    end
    
    UI->>UI: Store JWT in session
    UI->>UI: Redirect to dashboard
```

## UI Design Theme

### Color Palette (Warm Tones)

```css
/* Primary Colors */
--primary-50: #fff7ed;  /* Light cream */
--primary-100: #ffedd5; /* Light orange */
--primary-200: #fed7aa; /* Soft amber */
--primary-300: #fdba74; /* Warm orange */
--primary-400: #fb923c; /* Bright orange */
--primary-500: #f97316; /* Orange */
--primary-600: #ea580c; /* Deep orange */
--primary-700: #c2410c; /* Dark orange */
--primary-800: #9a3412; /* Burnt orange */
--primary-900: #7c2d12; /* Dark amber */

/* Neutral Colors */
--neutral-50: #fafaf9;   /* Warm white */
--neutral-100: #f5f5f4;  /* Light gray */
--neutral-200: #e7e5e4;  /* Gray */
--neutral-300: #d6d3d1;  /* Medium gray */
--neutral-400: #a8a29e;  /* Gray */
--neutral-500: #78716c;  /* Gray */
--neutral-600: #57534e;  /* Dark gray */
--neutral-700: #44403c;  /* Darker gray */
--neutral-800: #292524;  /* Dark gray */
--neutral-900: #1c1917;  /* Almost black */

/* Accent Colors */
--success: #16a34a;  /* Green */
--warning: #ca8a04;  /* Yellow/amber */
--danger: #dc2626;   /* Red */
--info: #0ea5e9;     /* Sky blue (minimal use) */
```

### Design Principles

1. **No purple/deep blue gradients** - strictly warm tones
2. **Clean, professional appearance** with cream/beige backgrounds
3. **Orange/amber accents** for CTAs and highlights
4. **High contrast** for readability
5. **Subtle shadows** for depth
6. **Consistent spacing** using 4px grid system

## Project Structure

```
inventory-management/
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── 2fa/
│   │   ├── (dashboard)/
│   │   │   ├── dashboard/
│   │   │   ├── products/
│   │   │   ├── categories/
│   │   │   ├── suppliers/
│   │   │   ├── locations/
│   │   │   ├── purchase-orders/
│   │   │   ├── sales/
│   │   │   ├── reports/
│   │   │   └── settings/
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   ├── products/
│   │   │   ├── categories/
│   │   │   ├── suppliers/
│   │   │   ├── locations/
│   │   │   ├── purchase-orders/
│   │   │   ├── sales/
│   │   │   └── reports/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   ├── ui/              # Radix UI + shadcn/ui components
│   │   ├── auth/
│   │   ├── inventory/
│   │   ├── dashboard/
│   │   ├── reports/
│   │   └── shared/
│   ├── lib/
│   │   ├── auth.ts
│   │   ├── prisma.ts
│   │   ├── validations.ts
│   │   ├── utils.ts
│   │   └── constants.ts
│   ├── hooks/
│   ├── types/
│   └── styles/
├── public/
├── tests/
└── package.json
```

## Implementation Phases

### Phase 1: Project Setup & Foundation
- Initialize Next.js with TypeScript
- Configure Tailwind CSS with warm theme
- Install core dependencies (Prisma, NextAuth, Radix UI, shadcn/ui)
- Set up PostgreSQL connection
- Create base Prisma schema
- Configure NextAuth.js with JWT

### Phase 2: Authentication System
- Implement Google OAuth provider
- Set up JWT token management
- Implement 2FA with TOTP
- Create authentication pages (login, register, 2FA setup)
- Add session management middleware

### Phase 3: Core Database & API
- Complete Prisma schema with all models
- Create database migrations
- Build CRUD API endpoints for all entities
- Implement validation with Zod
- Add error handling middleware

### Phase 4: UI Components & Layout
- Set up Radix UI components
- Configure shadcn/ui styling
- Create base layout with navigation
- Build shared components (buttons, inputs, modals, tables)
- Implement loading and error states

### Phase 5: Product Management
- Product listing with search and filters
- Product creation and edit forms
- Product detail view
- Category management
- Barcode generation and display

### Phase 6: Inventory Tracking
- Stock management per location
- Inventory movement tracking
- Low stock alerts system
- Multi-location support
- Real-time stock updates

### Phase 7: Supplier & Purchase Orders
- Supplier management CRUD
- Purchase order creation
- Purchase order status tracking
- Receive items from purchase orders
- Supplier analytics

### Phase 8: Sales & Orders
- Sales order creation
- Sales order management
- Invoice generation
- Customer management (optional)
- Sales analytics

### Phase 9: Reporting & Analytics
- Dashboard with key metrics
- Inventory reports
- Sales reports
- Purchase reports
- Movement history reports
- Export functionality (CSV, PDF)

### Phase 10: Advanced Features
- Barcode scanning functionality
- User roles and permissions
- Audit logging
- Data export/import
- Notifications system

### Phase 11: Testing & Deployment
- Write unit tests
- Write integration tests
- Performance optimization
- Security audit
- Deployment configuration
- Documentation

## Key Features Implementation Details

### 1. JWT Authentication with NextAuth.js

```typescript
// lib/auth.ts
import NextAuth from "next-auth"
import GoogleProvider from "next-auth/providers/google"
import { PrismaAdapter } from "@auth/prisma-adapter"
import { prisma } from "./prisma"

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  session: { strategy: "jwt" },
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    })
  ],
  callbacks: {
    async jwt({ token, user, account }) {
      // Add custom claims to JWT
      if (user) {
        token.id = user.id
        token.role = user.role
        token.twoFactorEnabled = user.twoFactorEnabled
      }
      return token
    },
    async session({ session, token }) {
      session.user.id = token.id
      session.user.role = token.role
      session.user.twoFactorEnabled = token.twoFactorEnabled
      return session
    }
  }
})
```

### 2. Two-Factor Authentication (TOTP)

```typescript
// lib/totp.ts
import { authenticator } from 'otplib'

export function generateTOTPSecret() {
  return authenticator.generateSecret()
}

export function generateTOTPQRCode(secret: string, email: string) {
  const otpauth = authenticator.keyuri(email, 'Inventory Management', secret)
  return QRCode.toDataURL(otpauth)
}

export function verifyTOTP(token: string, secret: string) {
  return authenticator.verify({
    token,
    secret
  })
}
```

### 3. Low Stock Alerts

```typescript
// lib/notifications.ts
export async function checkLowStockAlerts() {
  const lowStockItems = await prisma.stock.findMany({
    where: {
      quantity: {
        lte: prisma.stock.fields.reorderLevel
      }
    },
    include: {
      product: true,
      location: true
    }
  })

  // Send email notifications
  for (const item of lowStockItems) {
    await sendLowStockEmail(item)
  }
}
```

### 4. Barcode Scanning

```typescript
// components/barcode-scanner.tsx
'use client'
import { Html5QrcodeScanner } from 'html5-qrcode'

export function BarcodeScanner({ onScan }: { onScan: (code: string) => void }) {
  useEffect(() => {
    const scanner = new Html5QrcodeScanner('reader', {
      fps: 10,
      qrbox: { width: 250, height: 250 }
    })

    scanner.render(
      (decodedText) => {
        onScan(decodedText)
      },
      (error) => {
        console.error(error)
      }
    )

    return () => {
      scanner.clear()
    }
  }, [onScan])

  return <div id="reader" />
}
```

## Security Considerations

1. **JWT Security**:
   - Use secure, httpOnly cookies for session storage
   - Implement token rotation
   - Set appropriate expiration times

2. **2FA Implementation**:
   - Require 2FA for sensitive operations
   - Allow backup codes for account recovery
   - Rate limit TOTP verification attempts

3. **API Security**:
   - Implement rate limiting
   - Validate all inputs
   - Use parameterized queries (Prisma handles this)
   - Implement CORS properly

4. **Data Protection**:
   - Encrypt sensitive data at rest
   - Use HTTPS in production
   - Implement proper access controls
   - Regular security audits

## Performance Optimization

1. **Database**:
   - Add appropriate indexes
   - Use connection pooling
   - Implement query optimization
   - Consider read replicas for reporting

2. **Frontend**:
   - Use Next.js server components where possible
   - Implement lazy loading for heavy components
   - Optimize images with next/image
   - Use React.memo for expensive components

3. **Caching**:
   - Implement Redis for session storage
   - Cache frequently accessed data
   - Use Next.js built-in caching

## Deployment Checklist

- [ ] Set up PostgreSQL database (production)
- [ ] Configure environment variables
- [ ] Set up Google OAuth app
- [ ] Configure domain and SSL
- [ ] Set up email service for alerts
- [ ] Configure backup strategy
- [ ] Set up monitoring and logging
- [ ] Configure CI/CD pipeline
- [ ] Performance testing
- [ ] Security audit

## Environment Variables

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/inventory_db"

# NextAuth
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-key"

# Google OAuth
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# Email (for alerts)
SMTP_HOST="smtp.example.com"
SMTP_PORT="587"
SMTP_USER="your-email"
SMTP_PASSWORD="your-password"

# App
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

## Notes & Considerations

1. **shadcn/ui**: A collection of reusable components built using Radix UI and Tailwind CSS. It provides beautiful, accessible components that can be copied and pasted into your project.

2. **Barcode Scanning**: Will use `html5-qrcode` library for web-based scanning. For production, consider integrating with hardware scanners.

3. **2FA**: Using TOTP with Google Authenticator or similar apps. Consider adding SMS backup option.

4. **Multi-location**: Each location will have its own stock levels. Transfers between locations will be tracked as inventory movements.

5. **Reporting**: Consider using a dedicated reporting tool like Cube or Metabase for complex analytics if needed.

6. **Scalability**: The architecture supports horizontal scaling. Consider using a message queue (RabbitMQ/Redis) for background jobs like email notifications.

7. **Testing**: Aim for 80%+ code coverage. Focus on critical paths like authentication and inventory updates.
