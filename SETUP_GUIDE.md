# Complete Setup Guide: Next.js TypeScript Expense Tracker

## Overview

This is a modern expense tracking application built with:

-   **Next.js 15** with TypeScript and App Router
-   **Clerk** for authentication and user management
-   **Prisma** as ORM (Object-Relational Mapping)
-   **Neon Database** (PostgreSQL in the cloud)
-   **Tailwind CSS** for styling
-   **ESLint** for code quality

## Prerequisites

Before starting, ensure you have:

-   **Node.js 18+** installed ([Download here](https://nodejs.org/))
-   **Git** installed ([Download here](https://git-scm.com/))
-   **A Neon Database account** ([Sign up here](https://neon.tech/))
-   **A Clerk account** ([Sign up here](https://clerk.com/))
-   **A code editor** (VS Code recommended)

---

## Step 1: Initialize the Next.js Project

### 1.1 Create New Project

```powershell
# Create a new Next.js project with all the modern features
npx create-next-app@latest expense-tracker --typescript --tailwind --eslint --app --src-dir=false --import-alias="@/*"

# Navigate to the project directory
Set-Location "expense-tracker"
```

### 1.2 Verify Project Structure

Your project should have this structure:

```
expense-tracker/
├── app/
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── public/
├── package.json
├── tsconfig.json
├── tailwind.config.ts
└── next.config.ts
```

---

## Step 2: Install Required Dependencies

### 2.1 Install Main Dependencies

```powershell
# Authentication and database dependencies
npm install @clerk/nextjs @prisma/client

# Development dependencies for database and styling
npm install -D prisma @tailwindcss/postcss
```

### 2.2 Verify Installation

Check your `package.json` to ensure these packages are installed:

-   `@clerk/nextjs` - Authentication
-   `@prisma/client` - Database client
-   `prisma` - Database toolkit
-   `@tailwindcss/postcss` - CSS processing

---

## Step 3: Set Up Neon Database

### 3.1 Create Neon Database Account

1. Go to [Neon Console](https://console.neon.tech/)
2. Sign up with your email or GitHub account
3. Verify your email if required

### 3.2 Create New Database Project

1. Click "Create Project"
2. Choose a project name (e.g., "expense-tracker-db")
3. Select a region close to your users (e.g., US East for North America)
4. Choose PostgreSQL version (latest recommended)
5. Click "Create Project"

### 3.3 Get Connection String

1. In your Neon dashboard, go to "Connection Details"
2. Copy the connection string (it looks like):
    ```
    postgresql://username:password@host/database?sslmode=require
    ```
3. Keep this safe - you'll need it in the next step

---

## Step 4: Configure Environment Variables

### 4.1 Create Environment File

Create a `.env` file in your project root:

```env
# Database Configuration
DATABASE_URL="your_neon_database_connection_string_here"

# Clerk Authentication Keys (get these from Step 5)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key_here
CLERK_SECRET_KEY=your_clerk_secret_key_here

# Clerk URL Configuration
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL=/
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL=/
```

### 4.2 Add .env to .gitignore

Ensure your `.gitignore` file includes:

```
# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
```

---

## Step 5: Set Up Clerk Authentication

### 5.1 Create Clerk Application

1. Go to [Clerk Dashboard](https://dashboard.clerk.com/)
2. Sign up or log in
3. Click "Create Application"
4. Choose a name (e.g., "Expense Tracker")
5. Select authentication methods:
    - ✅ Email
    - ✅ Google (recommended)
    - ✅ GitHub (optional)
6. Click "Create Application"

### 5.2 Get API Keys

1. In your Clerk dashboard, go to "API Keys"
2. Copy the "Publishable Key" and "Secret Key"
3. Add them to your `.env` file:
    ```env
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
    CLERK_SECRET_KEY=sk_test_...
    ```

### 5.3 Create Authentication Pages

```powershell
# Create sign-in page directory structure
New-Item -ItemType Directory -Path "app\sign-in\[[...sign-in]]" -Force

# Create sign-up page directory structure
New-Item -ItemType Directory -Path "app\sign-up\[[...sign-up]]" -Force
```

### 5.4 Create Sign-In Page

Create `app/sign-in/[[...sign-in]]/page.tsx`:

```typescript
import { SignIn } from "@clerk/nextjs";

export default function Page() {
    return (
        <div className="flex items-center justify-center min-h-screen">
            <SignIn />
        </div>
    );
}
```

### 5.5 Create Sign-Up Page

Create `app/sign-up/[[...sign-up]]/page.tsx`:

```typescript
import { SignUp } from "@clerk/nextjs";

export default function Page() {
    return (
        <div className="flex items-center justify-center min-h-screen">
            <SignUp />
        </div>
    );
}
```

---

## Step 6: Set Up Prisma Database

### 6.1 Initialize Prisma

```powershell
# Initialize Prisma in your project
npx prisma init
```

This creates:

-   `prisma/` directory
-   `prisma/schema.prisma` file
-   Updates your `.env` file

### 6.2 Configure Database Schema

Replace the content of `prisma/schema.prisma`:

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id             String         @id @default(uuid())
  clerkUserId    String         @unique
  email          String         @unique
  name           String?
  imageUrl       String?
  createdAt      DateTime       @default(now())
  updatedAt      DateTime       @updatedAt
  ExpenseEntries ExpenseEntry[]

  @@map("users")
}

model ExpenseEntry {
  id        String   @id @default(uuid())
  text      String
  amount    Float
  category  String   @default("Other")
  date      DateTime @default(now())
  userId    String
  user      User     @relation(fields: [userId], references: [clerkUserId], onDelete: Cascade)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([userId])
  @@index([date])
  @@map("expense_entries")
}
```

### 6.3 Generate Prisma Client

```powershell
# Generate the Prisma client
npx prisma generate
```

### 6.4 Create and Run Migration

```powershell
# Create your first migration
npx prisma migrate dev --name init
```

This will:

-   Create the database tables
-   Generate migration files
-   Update your Prisma client

---

## Step 7: Create Database Connection

### 7.1 Create Database Utility

Create `lib/db.ts`:

```typescript
import { PrismaClient } from "@prisma/client";

declare global {
    var prisma: PrismaClient | undefined;
}

export const db = globalThis.prisma || new PrismaClient();

if (process.env.NODE_ENV !== "production") {
    globalThis.prisma = db;
}
```

### 7.2 Why This Pattern?

This pattern prevents multiple Prisma Client instances in development due to hot reloading, which can cause connection issues.

---

## Step 8: Create User Management System

### 8.1 Create User Check Utility

Create `lib/checkUser.ts`:

```typescript
import { currentUser } from "@clerk/nextjs/server";
import { db } from "./db";

export const checkUser = async () => {
    const user = await currentUser();

    // If no user is logged in, return null
    if (!user) {
        return null;
    }

    // Check if user exists in our database
    const loggedInUser = await db.user.findUnique({
        where: {
            clerkUserId: user.id,
        },
    });

    // If user exists, return them
    if (loggedInUser) {
        return loggedInUser;
    }

    // If user doesn't exist, create them
    const newUser = await db.user.create({
        data: {
            clerkUserId: user.id,
            name: `${user.firstName} ${user.lastName}`,
            imageUrl: user.imageUrl,
            email: user.emailAddresses[0]?.emailAddress || "",
        },
    });

    return newUser;
};
```

---

## Step 9: Create Components

### 9.1 Create Components Directory

```powershell
# Create components directory if it doesn't exist
New-Item -ItemType Directory -Path "components" -Force
```

### 9.2 Create Navbar Component

Create `components/Navbar.tsx`:

```typescript
import { UserButton } from "@clerk/nextjs";
import { checkUser } from "@/lib/checkUser";

const Navbar = async () => {
    const user = await checkUser();

    return (
        <nav className="bg-white shadow-sm border-b">
            <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div className="flex justify-between h-16">
                    <div className="flex items-center">
                        <h1 className="text-xl font-semibold text-gray-900">
                            Expense Tracker
                        </h1>
                    </div>
                    <div className="flex items-center space-x-4">
                        {user && (
                            <>
                                <span className="text-sm text-gray-700">
                                    Welcome, {user.name}!
                                </span>
                                <UserButton afterSignOutUrl="/" />
                            </>
                        )}
                    </div>
                </div>
            </div>
        </nav>
    );
};

export default Navbar;
```

---

## Step 10: Update Application Layout

### 10.1 Update Root Layout

Update `app/layout.tsx`:

```typescript
import Navbar from "@/components/Navbar";
import { ClerkProvider } from "@clerk/nextjs";
import type { Metadata } from "next";
import { Geist, Geist_Mono } from "next/font/google";
import "./globals.css";

const geistSans = Geist({
    variable: "--font-geist-sans",
    subsets: ["latin"],
});

const geistMono = Geist_Mono({
    variable: "--font-geist-mono",
    subsets: ["latin"],
});

export const metadata: Metadata = {
    title: "Expense Tracker",
    description:
        "Track your expenses efficiently with our modern expense tracker",
};

export default function RootLayout({
    children,
}: Readonly<{
    children: React.ReactNode;
}>) {
    return (
        <ClerkProvider>
            <html lang="en">
                <body
                    className={`${geistSans.variable} ${geistMono.variable} antialiased bg-gray-50`}
                >
                    <Navbar />
                    <main className="min-h-screen">{children}</main>
                </body>
            </html>
        </ClerkProvider>
    );
}
```

### 10.2 Update Home Page

Update `app/page.tsx`:

```typescript
import { checkUser } from "@/lib/checkUser";
import { redirect } from "next/navigation";

export default async function Home() {
    const user = await checkUser();

    if (!user) {
        redirect("/sign-in");
    }

    return (
        <div className="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
            <div className="px-4 py-6 sm:px-0">
                <div className="border-4 border-dashed border-gray-200 rounded-lg h-96 flex items-center justify-center">
                    <div className="text-center">
                        <h1 className="text-3xl font-bold text-gray-900 mb-4">
                            Welcome to Expense Tracker!
                        </h1>
                        <p className="text-gray-600">
                            Start tracking your expenses efficiently.
                        </p>
                    </div>
                </div>
            </div>
        </div>
    );
}
```

---

## Step 11: Create Middleware for Route Protection

### 11.1 Update Middleware

Update `middleware.ts` in the root directory:

```typescript
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isProtectedRoute = createRouteMatcher([
    "/",
    "/dashboard(.*)",
    "/expenses(.*)",
]);

export default clerkMiddleware(async (auth, req) => {
    if (isProtectedRoute(req)) await auth.protect();
});

export const config = {
    matcher: [
        // Skip Next.js internals and all static files, unless found in search params
        "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
        // Always run for API routes
        "/(api|trpc)(.*)",
    ],
};
```

---

## Step 12: Update Package.json Scripts

### 12.1 Add Useful Scripts

Update your `package.json` scripts section:

```json
{
    "scripts": {
        "dev": "next dev",
        "build": "next build",
        "start": "next start",
        "lint": "next lint",
        "db:generate": "prisma generate",
        "db:migrate": "prisma migrate dev",
        "db:studio": "prisma studio",
        "db:push": "prisma db push",
        "db:reset": "prisma migrate reset",
        "db:seed": "tsx prisma/seed.ts"
    }
}
```

---

## Step 13: Test Your Setup

### 13.1 Start Development Server

```powershell
# Start the development server
npm run dev
```

### 13.2 Test the Application

1. Open your browser to `http://localhost:3000`
2. You should be redirected to the sign-in page
3. Create a new account or sign in
4. You should see the welcome page after authentication

### 13.3 Check Database

```powershell
# Open Prisma Studio to view your database
npm run db:studio
```

---

## Step 14: Additional Development Tools

### 14.1 Database Management Commands

```powershell
# View your database in a web interface
npm run db:studio

# Reset database (⚠️ This deletes all data!)
npm run db:reset

# Push schema changes without creating migration files
npm run db:push

# Generate Prisma client after schema changes
npm run db:generate

# Create a new migration
npx prisma migrate dev --name your_migration_name
```

### 14.2 Useful Development Commands

```powershell
# Check for linting errors
npm run lint

# Build for production (test if everything works)
npm run build

# Clear Next.js cache if you encounter issues
Remove-Item -Recurse -Force .next

# Reinstall all dependencies
Remove-Item -Recurse -Force node_modules
Remove-Item package-lock.json
npm install
```

---

## Step 15: Environment Variables Checklist

### 15.1 Required Environment Variables

Ensure your `.env` file contains all these variables:

```env
# ✅ Database
DATABASE_URL="postgresql://username:password@host/database?sslmode=require"

# ✅ Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="pk_test_..."
CLERK_SECRET_KEY="sk_test_..."

# ✅ Clerk URLs
NEXT_PUBLIC_CLERK_SIGN_IN_URL="/sign-in"
NEXT_PUBLIC_CLERK_SIGN_UP_URL="/sign-up"
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL="/"
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL="/"
```

### 15.2 Verification Steps

1. ✅ Database connection works (test with `npm run db:studio`)
2. ✅ Clerk authentication works (can sign up/sign in)
3. ✅ User creation works (check database after first sign-in)
4. ✅ Protected routes work (redirects to sign-in when not authenticated)

---

## Troubleshooting Guide

### Common Issues and Solutions

#### 1. Database Connection Errors

**Error**: `Can't reach database server`
**Solution**:

-   Verify your Neon database URL is correct
-   Check if your Neon database is active (not paused)
-   Ensure the connection string includes `?sslmode=require`

#### 2. Clerk Authentication Issues

**Error**: `Clerk: Missing publishable key`
**Solution**:

-   Check your `.env` file has the correct Clerk keys
-   Ensure keys start with `pk_test_` and `sk_test_`
-   Restart your development server after adding keys

#### 3. Prisma Client Errors

**Error**: `PrismaClient is unable to run in this browser environment`
**Solution**:

-   Run `npm run db:generate` after schema changes
-   Ensure you're not importing Prisma client in client-side components

#### 4. TypeScript Errors

**Error**: Various TypeScript compilation errors
**Solution**:

-   Ensure all dependencies are installed: `npm install`
-   Check your `tsconfig.json` is properly configured
-   Restart your TypeScript server in VS Code

#### 5. Middleware Issues

**Error**: Authentication not working on protected routes
**Solution**:

-   Check your `middleware.ts` file is in the root directory
-   Verify the route matchers are correct
-   Ensure Clerk environment variables are set

### Getting Help

If you encounter issues:

1. Check the [Next.js Documentation](https://nextjs.org/docs)
2. Check the [Clerk Documentation](https://clerk.com/docs)
3. Check the [Prisma Documentation](https://www.prisma.io/docs)
4. Check the [Neon Documentation](https://neon.tech/docs)

---

## Next Steps

After completing this setup, you can:

1. **Add Expense Management Features**:

    - Create expense forms
    - List expenses
    - Edit/delete expenses
    - Filter and search expenses

2. **Add Categories**:

    - Create category management
    - Assign expenses to categories
    - Category-based reporting

3. **Add Dashboard**:

    - Expense summaries
    - Charts and graphs
    - Monthly/yearly reports

4. **Add Export Features**:

    - Export to CSV
    - PDF reports
    - Email summaries

5. **Deploy Your Application**:
    - Deploy to Vercel
    - Set up production environment variables
    - Configure production database

---

## Project Structure Overview

Your final project structure should look like this:

```
expense-tracker/
├── app/
│   ├── globals.css
│   ├── layout.tsx
│   ├── page.tsx
│   ├── sign-in/
│   │   └── [[...sign-in]]/
│   │       └── page.tsx
│   └── sign-up/
│       └── [[...sign-up]]/
│           └── page.tsx
├── components/
│   └── Navbar.tsx
├── lib/
│   ├── checkUser.ts
│   └── db.ts
├── prisma/
│   ├── migrations/
│   └── schema.prisma
├── public/
├── .env
├── .gitignore
├── middleware.ts
├── next.config.ts
├── package.json
├── tailwind.config.ts
└── tsconfig.json
```

Congratulations! You now have a fully functional expense tracker application with authentication, database integration, and a modern tech stack. 🎉
