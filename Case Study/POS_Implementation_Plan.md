# Carolina Brews POS System - Implementation Plan

## Architecture Decision: **API-First Approach**

**Why API-First?**
- Your case study already outlines this approach
- Enables parallel development (frontend can work with mock data while APIs are built)
- Easier testing and validation
- Future-proof for mobile apps or third-party integrations
- Clean separation of concerns

**Hybrid Strategy:**
- Build API endpoints first (Next.js API routes)
- Build frontend components in parallel with mock data
- Connect them incrementally as APIs are ready
- This gives you working UI quickly while ensuring solid backend foundation

---

## Phase 1: Project Setup & Foundation

### Goals
1. Initialize Next.js project with TypeScript
2. Set up database connection (Prisma + PostgreSQL)
3. Configure authentication system
4. Set up project structure and folder organization
5. Install and configure essential dependencies

### Step-by-Step Implementation

#### Step 1.1: Initialize Next.js Project
```bash
npx create-next-app@latest carolina-brews-pos --typescript --tailwind --app --no-src-dir
cd carolina-brews-pos
```

**What this gives you:**
- Next.js 14+ with App Router
- TypeScript configuration
- Tailwind CSS setup
- Modern project structure

#### Step 1.2: Install Core Dependencies
```bash
# Database & ORM
npm install prisma @prisma/client
npm install -D prisma

# Authentication (NextAuth.js)
npm install next-auth @auth/prisma-adapter bcryptjs
npm install -D @types/bcryptjs

# Form handling & validation
npm install react-hook-form zod @hookform/resolvers
npm install zustand  # State management

# UI Components (shadcn/ui - recommended)
npx shadcn-ui@latest init

# Utilities
npm install date-fns clsx tailwind-merge
npm install lucide-react  # Icons
```

#### Step 1.3: Set Up Prisma Schema
- Copy your existing Prisma schema from the case study
- Add POS-specific models (SalesOrder, SalesOrderLine, Payment, etc.)
- Run `npx prisma generate`
- Set up `.env` with `DATABASE_URL`

**New Models to Add:**
```prisma
model SalesOrder {
  id            String   @id @default(cuid())
  orderNumber   String   @unique
  locationId    String
  location      Location @relation(fields: [locationId], references: [id])
  userId        String
  user          User     @relation(fields: [userId], references: [id])
  customerId    String?
  customer      Customer? @relation(fields: [customerId], references: [id])
  status        OrderStatus @default(PENDING)
  subtotal      Decimal
  tax           Decimal
  discount      Decimal   @default(0)
  total         Decimal
  paymentMethod PaymentMethod?
  paymentStatus PaymentStatus @default(UNPAID)
  notes         String?
  lines         SalesOrderLine[]
  payments      Payment[]
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}

model SalesOrderLine {
  id            String      @id @default(cuid())
  orderId       String
  order         SalesOrder  @relation(fields: [orderId], references: [id], onDelete: Cascade)
  itemId        String
  item          Item        @relation(fields: [itemId], references: [id])
  quantity     Int
  unitPrice     Decimal
  discount      Decimal     @default(0)
  tax           Decimal
  lineTotal     Decimal
  createdAt     DateTime    @default(now())
}

model Payment {
  id            String      @id @default(cuid())
  orderId       String
  order         SalesOrder  @relation(fields: [orderId], references: [id])
  amount        Decimal
  method        PaymentMethod
  transactionId String?
  status        PaymentStatus @default(COMPLETED)
  createdAt     DateTime    @default(now())
}

model Customer {
  id            String       @id @default(cuid())
  name          String
  email         String?
  phone         String?
  address       String?
  loyaltyPoints Int          @default(0)
  orders        SalesOrder[]
  createdAt     DateTime     @default(now())
  updatedAt     DateTime     @updatedAt
}

enum OrderStatus {
  PENDING
  COMPLETED
  CANCELLED
  REFUNDED
}

enum PaymentMethod {
  CASH
  CARD
  MOBILE_PAYMENT
  GIFT_CARD
  STORE_CREDIT
}

enum PaymentStatus {
  UNPAID
  PARTIAL
  PAID
  REFUNDED
}
```

#### Step 1.4: Project Folder Structure
```
carolina-brews-pos/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── pos/
│   │   ├── inventory/
│   │   ├── reports/
│   │   └── settings/
│   ├── api/
│   │   ├── auth/
│   │   ├── products/
│   │   ├── sales/
│   │   ├── inventory/
│   │   └── customers/
│   └── layout.tsx
├── components/
│   ├── ui/          # shadcn components
│   ├── pos/         # POS-specific components
│   ├── inventory/   # Inventory components
│   └── shared/      # Shared components
├── lib/
│   ├── prisma.ts
│   ├── auth.ts
│   ├── utils.ts
│   └── validations/
├── hooks/
├── store/          # Zustand stores
├── types/
└── prisma/
    └── schema.prisma
```

#### Step 1.5: Environment Variables Setup
Create `.env.local`:
```env
DATABASE_URL="postgresql://user:password@localhost:5432/carolina_brews"
NEXTAUTH_SECRET="your-secret-key-here"
NEXTAUTH_URL="http://localhost:3000"
```

#### Step 1.6: Database Migration
```bash
npx prisma migrate dev --name init
npx prisma generate
```

---

## Phase 2: Authentication & User Management

### Goals
1. Implement user authentication (login/logout)
2. Set up role-based access control
3. Create protected routes
4. Build user management UI

### Step-by-Step Implementation

#### Step 2.1: Configure NextAuth
- Create `lib/auth.ts` with NextAuth configuration
- Set up authentication providers
- Configure session management
- Create API route: `app/api/auth/[...nextauth]/route.ts`

#### Step 2.2: Create Auth Components
- Login page (`app/(auth)/login/page.tsx`)
- Register page (if needed)
- Auth form components
- Protected route wrapper

#### Step 2.3: Middleware for Route Protection
- Create `middleware.ts` at root
- Protect dashboard routes
- Redirect unauthenticated users

#### Step 2.4: User Context/Store
- Create Zustand store for user state
- Or use NextAuth session hooks
- Display user info in navbar

---

## Phase 3: Core POS Interface (Frontend First with Mock Data)

### Goals
1. Build the main POS interface layout
2. Create product selection UI
3. Build shopping cart component
4. Design checkout flow
5. Use mock data initially, connect to APIs later

### Step-by-Step Implementation

#### Step 3.1: POS Layout Structure
Create `app/(dashboard)/pos/page.tsx`:
- Split-screen layout (products left, cart right)
- Header with user info, location selector
- Footer with totals and action buttons

**Component Structure:**
```
POS Page
├── POSHeader (location, user, time)
├── POSMain
│   ├── ProductGrid (left side)
│   └── CartPanel (right side)
└── POSFooter (totals, payment buttons)
```

#### Step 3.2: Product Grid Component
- Display products in grid/card layout
- Category filters
- Search functionality
- Product cards with image, name, price
- Click to add to cart
- **Use mock data initially**

#### Step 3.3: Shopping Cart Component
- List of selected items
- Quantity controls (increase/decrease)
- Remove item functionality
- Price calculations (subtotal, tax, total)
- Discount input
- **Use Zustand store for cart state**

#### Step 3.4: Cart State Management
Create `store/posStore.ts`:
```typescript
// Zustand store for POS cart
- cartItems: CartItem[]
- addToCart()
- removeFromCart()
- updateQuantity()
- clearCart()
- calculateTotals()
```

#### Step 3.5: Checkout Modal/Dialog
- Payment method selection
- Cash amount input
- Change calculation
- Receipt preview
- Complete transaction button

---

## Phase 4: Product & Inventory APIs

### Goals
1. Create product listing API
2. Build product search/filter API
3. Create inventory check API
4. Real-time stock level updates

### Step-by-Step Implementation

#### Step 4.1: Products API Endpoints
Create `app/api/products/route.ts`:
- `GET /api/products` - List all products
- `GET /api/products?category=beer` - Filter by category
- `GET /api/products?search=ipa` - Search products
- `GET /api/products/[id]` - Get single product

#### Step 4.2: Inventory Check API
Create `app/api/inventory/check/route.ts`:
- `POST /api/inventory/check` - Check stock for multiple items
- Returns available quantities
- Handles low stock warnings

#### Step 4.3: Connect Frontend to APIs
- Replace mock data with API calls
- Add loading states
- Error handling
- Optimistic updates

---

## Phase 5: Sales Transaction APIs

### Goals
1. Create sales order API
2. Process payments
3. Update inventory automatically
4. Generate receipts

### Step-by-Step Implementation

#### Step 5.1: Sales Order API
Create `app/api/sales/orders/route.ts`:
- `POST /api/sales/orders` - Create new order
- Validate inventory availability
- Calculate totals (subtotal, tax, discount)
- Create SalesOrder and SalesOrderLine records

#### Step 5.2: Payment Processing API
Create `app/api/sales/payments/route.ts`:
- `POST /api/sales/payments` - Process payment
- Handle different payment methods
- Update order payment status
- Create Payment records

#### Step 5.3: Inventory Deduction Service
Create `lib/services/inventory.ts`:
- Function to deduct inventory on sale
- Handle stock reservations
- Update Inventory model
- Transaction safety (rollback on error)

#### Step 5.4: Complete Transaction Flow
- Connect checkout to sales API
- Handle payment processing
- Update inventory
- Generate receipt
- Clear cart
- Show success message

---

## Phase 6: Receipt Generation

### Goals
1. Design receipt template
2. Generate PDF receipts
3. Print functionality
4. Email receipts (optional)

### Step-by-Step Implementation

#### Step 6.1: Receipt Component
- Create receipt template component
- Display order details
- Format for printing
- Include company info, items, totals

#### Step 6.2: PDF Generation
- Install `react-pdf` or `jspdf`
- Convert receipt component to PDF
- Download functionality

#### Step 6.3: Print Functionality
- Browser print dialog
- Print-optimized CSS
- Receipt printer support (future)

---

## Phase 7: Customer Management

### Goals
1. Customer lookup/search
2. Quick customer creation
3. Customer selection in POS
4. Loyalty points tracking

### Step-by-Step Implementation

#### Step 7.1: Customer API
- `GET /api/customers` - List/search customers
- `POST /api/customers` - Create customer
- `GET /api/customers/[id]` - Get customer details

#### Step 7.2: Customer Selection UI
- Customer search in POS
- Quick add customer modal
- Display customer info in cart
- Apply customer-specific pricing

---

## Phase 8: Advanced Features

### Goals
1. Discounts and promotions
2. Returns/refunds
3. Sales reports
4. Shift management

### Step-by-Step Implementation

#### Step 8.1: Discount System
- Percentage discounts
- Fixed amount discounts
- Promo codes
- Customer-specific discounts

#### Step 8.2: Returns/Refunds
- Return items to cart
- Process refunds
- Restore inventory
- Refund receipt

#### Step 8.3: Sales Reports
- Daily sales summary
- Product sales reports
- Payment method breakdown
- Sales by user/location

#### Step 8.4: Shift Management
- Open/close shifts
- Cash drawer tracking
- End-of-day reports
- Cash reconciliation

---

## Phase 9: Real-time Features & Optimizations

### Goals
1. Real-time inventory updates
2. Offline support (PWA)
3. Performance optimization
4. Barcode scanning

### Step-by-Step Implementation

#### Step 9.1: Real-time Updates
- WebSocket or Server-Sent Events
- Live inventory updates
- Multi-user conflict handling

#### Step 9.2: PWA Setup
- Service worker
- Offline mode
- Install prompt
- Cache strategies

#### Step 9.3: Barcode Scanner
- Camera-based scanning
- USB barcode scanner support
- Product lookup by barcode

---

## Development Workflow Recommendations

### Daily Development Flow:
1. **Morning**: Plan day's tasks, review previous day's work
2. **Development**: 
   - Build API endpoint
   - Test with Postman/Thunder Client
   - Build frontend component
   - Connect frontend to API
   - Test full flow
3. **End of Day**: Commit changes, update progress

### Testing Strategy:
- Test each API endpoint independently
- Test frontend components in isolation
- Integration testing for complete flows
- Manual testing on actual devices

### Version Control:
- Feature branches for each phase
- Regular commits with clear messages
- Pull requests for code review (if team)

---

## Priority Order (If Time-Constrained)

**Must Have (MVP):**
1. Phase 1: Project Setup
2. Phase 2: Authentication
3. Phase 3: POS Interface (with mock data)
4. Phase 4: Product APIs
5. Phase 5: Sales Transaction APIs

**Should Have:**
6. Phase 6: Receipt Generation
7. Phase 7: Customer Management

**Nice to Have:**
8. Phase 8: Advanced Features
9. Phase 9: Real-time & Optimizations

---

## Next Steps

1. **Start with Phase 1** - Set up the project foundation
2. **Build incrementally** - Complete each phase before moving to next
3. **Test frequently** - Don't wait until everything is built
4. **Get feedback early** - Show working prototypes to stakeholders

Would you like me to help you start with Phase 1, or do you have questions about any specific phase?

