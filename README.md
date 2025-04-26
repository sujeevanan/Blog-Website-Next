
# 📝 Blog Website

A full-stack blog platform built with **Next.js** and **TypeScript**, supporting user authentication and blog CRUD functionalities with strict ownership rules.

---

## 📚 Features

- User **Registration** and **Login** with secure password hashing
- **Authentication** via JWT tokens
- **Anyone** can read blogs
- **Only blog owners** can **create**, **edit**, or **delete** their blogs
- Protected API routes and ownership verification
- Fully typed with **TypeScript** and clean architecture

---

## 🏗️ Folder Structure

```
/blog-website
├── /components
│    ├── BlogCard.tsx
│    ├── BlogForm.tsx
│    └── Navbar.tsx
│
├── /pages
│    ├── index.tsx                // Home page - Display all blogs
│    ├── login.tsx                 // Login page
│    ├── register.tsx              // Registration page
│    ├── create-blog.tsx           // Create blog page (protected)
│    ├── edit-blog/[id].tsx        // Edit blog page (protected, dynamic)
│    ├── /api
│    │    ├── /auth
│    │    │    ├── register.ts     // Handle registration
│    │    │    └── login.ts        // Handle login
│    │    └── /blogs
│    │         ├── index.ts        // GET all blogs / POST create blog
│    │         ├── [id].ts         // GET, PUT, DELETE blog by id
│
├── /lib
│    ├── auth.ts                   // Authentication middleware
│    └── db.ts                     // Database connection file
│
├── /types
│    ├── blog.ts                   // Blog types
│    └── user.ts                   // User types
│
├── /public
│    └── favicon.ico
│
├── .env.local                     // Environment variables
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── README.md
└── package.json
```

---

## 🔄 API Flow Diagram

```mermaid
flowchart TD
    A[User Registers] --> B[POST /api/auth/register]
    A1[User Logs in] --> C[POST /api/auth/login]
    
    C -->|Access Token| D[Frontend saves Token (cookie/localstorage)]

    D --> E[User Accesses Blogs]
    E --> F[GET /api/blogs]

    D --> G[User Creates Blog]
    G --> H[POST /api/blogs] -->|Authorization Check| I[Server Saves Blog]

    D --> J[User Edits Blog]
    J --> K[PUT /api/blogs/:id] -->|Owner Check| L[Server Updates Blog]

    D --> M[User Deletes Blog]
    M --> N[DELETE /api/blogs/:id] -->|Owner Check| O[Server Deletes Blog]
```

---

## 🛢️ Database Models

### User Model (`types/user.ts`)

```typescript
export interface User {
  id: string;
  username: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### Blog Model (`types/blog.ts`)

```typescript
export interface Blog {
  id: string;
  title: string;
  content: string;
  authorId: string;
  createdAt: Date;
  updatedAt: Date;
}
```

---

## 🔒 Authentication Middleware (`lib/auth.ts`)

```typescript
import { NextApiRequest, NextApiResponse, NextApiHandler } from "next";
import jwt from "jsonwebtoken";

const JWT_SECRET = process.env.JWT_SECRET || "your_default_secret";

export interface AuthenticatedRequest extends NextApiRequest {
  user?: {
    id: string;
    email: string;
  };
}

export const authenticate = (handler: NextApiHandler) => {
  return async (req: AuthenticatedRequest, res: NextApiResponse) => {
    try {
      const token = req.headers.authorization?.split(" ")[1];
      if (!token) {
        return res.status(401).json({ message: "No token provided" });
      }

      const decoded = jwt.verify(token, JWT_SECRET) as { id: string; email: string };
      req.user = {
        id: decoded.id,
        email: decoded.email,
      };

      return handler(req, res);
    } catch (error) {
      return res.status(401).json({ message: "Invalid or expired token" });
    }
  };
};
```

---

## 🛡️ Ownership Check Utility

You can create a helper to **check blog ownership** easily:

```typescript
export const isOwner = (blogAuthorId: string, userId: string) => {
  return blogAuthorId === userId;
};
```

In API routes:

```typescript
if (!isOwner(blog.authorId, req.user!.id)) {
  return res.status(403).json({ message: "You are not allowed to edit this blog" });
}
```

---

## 📋 Summary of CRUD Operations

| Action         | API Route             | Auth Required? | Ownership Required? |
|----------------|------------------------|----------------|----------------------|
| Register       | `POST /api/auth/register` | ❌              | N/A                  |
| Login          | `POST /api/auth/login`    | ❌              | N/A                  |
| Read Blogs     | `GET /api/blogs`         | ❌              | N/A                  |
| Create Blog    | `POST /api/blogs`        | ✅              | N/A                  |
| Edit Blog      | `PUT /api/blogs/:id`     | ✅              | ✅ (must own blog)    |
| Delete Blog    | `DELETE /api/blogs/:id`  | ✅              | ✅ (must own blog)    |

---

## ⚙️ Best Practices

- Use `getServerSideProps` to protect pages if SSR is required.
- Validate inputs server-side.
- Hash passwords before storing them.
- Validate JWT tokens on every protected API route.
- Use TypeScript types everywhere for safety.
- Handle all errors with meaningful messages and correct HTTP status codes.

---

# 📦 Tech Stack

- **Next.js** (Frontend + Backend APIs)
- **TypeScript** (strict typing)
- **JWT** for authentication
- **MongoDB** (preferred) or any database
- **TailwindCSS** (for UI, optional but recommended)

---

# 📢 Environment Variables (.env.local)

```
JWT_SECRET=your_secret_key_here
MONGODB_URI=your_mongodb_connection_string_here
```

---

# 🚀 Setup Instructions

```bash
git clone https://github.com/yourusername/blog-website.git
cd blog-website
npm install
npm run dev
```

---

# 🎯 Final Goal

Create a blog platform where:

- Anyone can **read blogs**.
- Only **registered users** can **create** blogs.
- Only **the blog owner** can **edit** or **delete** their blogs.
- APIs are **protected** and **properly validated**.
