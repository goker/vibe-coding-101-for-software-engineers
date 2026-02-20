# PRD Example 23: Multi-tenant SaaS Starter

> **Difficulty:** Advanced | **Project Type:** Full Stack Web App | **Time:** 3-4 weeks

---

## Overview

| | |
|---|---|
| **What** | A multi-tenant SaaS boilerplate with auth, billing, and team management |
| **Who** | Developers who want to quickly launch a SaaS product |
| **Why** | Provides production-ready foundation so developers can focus on their unique features |

---

## Core Features (MVP)

1. **Authentication:** Email/password signup, login, password reset
2. **Multi-tenancy:** Organizations with isolated data
3. **Team Management:** Invite members, assign roles (owner/admin/member)
4. **Subscription Billing:** Stripe integration with free/pro/enterprise tiers
5. **Settings:** Organization settings, user profile

---

## Non-Goals

**Will NOT build:**
- SSO or OAuth providers
- Usage-based billing
- White-labeling or custom domains
- API access for customers
- Audit logging
- Two-factor authentication
- Mobile app
- Admin super-dashboard

**Will NOT use:**
- Complex permission libraries (simple role-based)
- Multiple payment providers (Stripe only)
- Email marketing platforms
- Analytics platforms

---

## Technical Constraints

| | |
|---|---|
| **Frontend** | Next.js 14 (App Router), TypeScript, Tailwind CSS |
| **Backend** | Next.js API Routes + Server Actions |
| **Database** | PostgreSQL with Prisma |
| **Auth** | NextAuth.js v5 |
| **Payments** | Stripe (Checkout + Customer Portal) |
| **Email** | Resend (transactional only) |
| **Testing** | Playwright for E2E |
| **Deploy** | Vercel |

---

## Success Criteria

- [ ] Users can sign up and create organization
- [ ] Organization data is isolated (multi-tenant)
- [ ] Owners can invite team members
- [ ] Stripe checkout works for upgrades
- [ ] Customer portal for managing subscription
- [ ] Role-based access control works
- [ ] All E2E tests pass
- [ ] Deploys to Vercel successfully

---

## Implementation Phases

### Phase 1: Project Setup & Auth
**Goal:** Set up Next.js with authentication

**Tasks:**
1. Create Next.js project with TypeScript
2. Set up Prisma with PostgreSQL
3. Create User model (email, name, passwordHash)
4. Implement NextAuth with credentials provider
5. Create signup, login, forgot password pages
6. Set up Resend for password reset emails

**Verification:**
```
1. Sign up with email → Account created
2. Login → Redirects to dashboard
3. Forgot password → Email received
4. Reset password → Can login with new password
```

**Deliverables:** Auth system working

---

### Phase 2: Multi-tenancy & Organizations
**Goal:** Implement organization isolation

**Tasks:**
1. Create Organization model (name, slug, owner_id)
2. Create Membership model (user_id, org_id, role)
3. Create organization on first signup
4. Add org context to all queries
5. Build organization settings page

**Verification:**
```
1. Sign up → Organization "My Company" created
2. All data queries include org_id filter
3. Can't access other org's data
4. Settings page shows org name, can update
```

**Deliverables:** Multi-tenant data isolation

---

### Phase 3: Team Management
**Goal:** Invite and manage team members

**Tasks:**
1. Build team members list page
2. Implement invite by email
3. Send invite email via Resend
4. Accept invite flow (signup or existing user)
5. Implement role management (owner/admin/member)

**Verification:**
```
1. Invite colleague@example.com as Admin
2. Email sent with invite link
3. Colleague accepts → Joins organization
4. Owner can change roles
5. Members can't access admin settings
```

**Deliverables:** Team management working

---

### Phase 4: Stripe Billing
**Goal:** Implement subscription billing

**Tasks:**
1. Set up Stripe products (Free, Pro $29/mo, Enterprise $99/mo)
2. Create Subscription model linked to Organization
3. Implement Stripe Checkout for upgrades
4. Handle webhooks (subscription created/updated/canceled)
5. Implement Customer Portal for management
6. Gate features by plan (e.g., member limit)

**Verification:**
```
1. Free tier → 3 member limit
2. Click "Upgrade to Pro" → Stripe Checkout
3. Complete payment → Pro features unlocked
4. Click "Manage Subscription" → Stripe portal
5. Cancel subscription → Downgrades at period end
```

**Deliverables:** Billing fully working

---

### Phase 5: Polish & Testing
**Goal:** Polish UI and comprehensive testing

**Tasks:**
1. Build dashboard layout with navigation
2. Add loading states and error handling
3. Write Playwright E2E tests
4. Test complete user journey
5. Set up Vercel deployment with environment variables

**Verification:**
```bash
npx playwright test
# All E2E tests pass

# Deploy to Vercel
vercel --prod
# App accessible at production URL

# Complete journey:
# Sign up → Create org → Invite member → Upgrade → Use features
```

**Deliverables:** Production-ready SaaS starter

---

## Database Schema

```prisma
model User {
  id            String       @id @default(uuid())
  email         String       @unique
  name          String
  passwordHash  String
  memberships   Membership[]
  createdAt     DateTime     @default(now())
}

model Organization {
  id            String       @id @default(uuid())
  name          String
  slug          String       @unique
  memberships   Membership[]
  subscription  Subscription?
  createdAt     DateTime     @default(now())
}

model Membership {
  userId        String
  orgId         String
  role          Role         @default(MEMBER)
  user          User         @relation(fields: [userId], references: [id])
  organization  Organization @relation(fields: [orgId], references: [id])
  createdAt     DateTime     @default(now())
  @@id([userId, orgId])
}

enum Role {
  OWNER
  ADMIN
  MEMBER
}

model Subscription {
  id            String       @id @default(uuid())
  orgId         String       @unique
  organization  Organization @relation(fields: [orgId], references: [id])
  stripeCustomerId    String @unique
  stripeSubscriptionId String? @unique
  plan          Plan         @default(FREE)
  status        String       @default("active")
  currentPeriodEnd DateTime?
}

enum Plan {
  FREE
  PRO
  ENTERPRISE
}

model Invite {
  id            String       @id @default(uuid())
  email         String
  orgId         String
  role          Role         @default(MEMBER)
  token         String       @unique
  expiresAt     DateTime
  createdAt     DateTime     @default(now())
}
```

---

## Agent Rules

| Always | Ask First | Never |
|--------|-----------|-------|
| Filter all queries by organization_id | Before changing database schema | Expose data across organizations |
| Validate user has permission for action | Before adding new Stripe products | Store Stripe keys in code |
| Hash passwords with bcrypt | Before adding new roles | Allow non-owners to delete org |
| Handle Stripe webhooks idempotently | | Skip email verification for invites |
