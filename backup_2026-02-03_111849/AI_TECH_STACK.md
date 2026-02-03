# AI Tech Stack Reference - OmniStock & Omni E-Commerce

**Document Purpose:** Complete technology stack reference for AI assistants  
**Last Updated:** January 22, 2026  
**Version:** 1.0

---

## 🎯 Technology Overview

This document provides a comprehensive reference of all technologies, versions, and dependencies used in the OmniStock platform.

---

## 🔧 Backend Stack (Laravel - OmniStock)

### **Core Framework**
- **Laravel:** 12.0 (Latest version, released 2026)
- **PHP:** 8.4.6 (Latest stable)
  - JIT compilation enabled
  - OPcache enabled for production
  - Typed properties and attributes
  - Modern error handling

### **Database**
- **PostgreSQL:** 16+ (Production and CI/CD)
  - ACID compliance
  - JSON/JSONB support
  - Full-text search capabilities
  - Advanced indexing
  - Foreign key constraints

### **Authentication & Authorization**
- **Laravel Sanctum:** Built-in (API token authentication)
- **Spatie Laravel Permission:** 6.24
  - Role-based access control (RBAC)
  - Multi-guard support (admin, web)
  - 39+ granular permissions
  - 4 default roles: System Administrator, Owner, Ops, Viewer

### **Cloud Storage**
- **AWS SDK for PHP:** 3.369
- **League Flysystem AWS S3:** 3.0
  - Cloudflare R2 (S3-compatible)
  - Amazon S3 support
  - Multi-region support
  - Public CDN URLs

### **Image Processing**
- **Intervention Image:** 3.11
  - Image manipulation and optimization
  - Thumbnail generation
  - Format conversion (JPEG, PNG, WebP)
  - Resize, crop, watermark support

### **Caching & Queue**
- **Predis:** 3.3 (Redis client for PHP)
  - Session storage
  - Cache management
  - Queue backend
  - Real-time data

### **Error Tracking**
- **Bugsnag Laravel:** 2.29
  - Real-time error monitoring
  - Exception tracking
  - Release tracking
  - User context capture

### **Development Tools**
- **Laravel Tinker:** 2.10.1 (REPL)
- **Laravel Pail:** 1.2.2 (Log viewer)
- **Laravel Pint:** 1.24 (Code style fixer - PSR-12)
- **Laravel Sail:** 1.41 (Docker development environment)

### **Testing**
- **PHPUnit:** 11.5.3
- **Mockery:** 1.6
- **Faker PHP:** 1.23

---

## 🎨 Frontend Stack (Next.js - Omni E-Commerce)

### **Core Framework**
- **Next.js:** 16.1.4 (Latest with App Router)
- **React:** 19.2.3 (Latest)
- **TypeScript:** 5.x
- **Node.js:** v22.22.0 (LTS - support until April 2027)
- **npm:** 10.9.4

### **Styling**
- **Tailwind CSS:** 4.1.18 (Latest v4)
  - JIT (Just-In-Time) compilation
  - Custom design system
  - Dark mode support
  - Responsive utilities
- **@tailwindcss/forms:** 0.5.11
- **@tailwindcss/vite:** 4.0.0

### **Internationalization**
- **next-intl:** 4.4.0
  - Next.js 16 compatible
  - Locale-based routing
  - Server-side translations
  - Type-safe messages

### **UI & Animations**
- **Framer Motion:** Latest
  - Page transitions
  - Component animations
  - Scroll-based animations
  - Gesture support
- **Lucide React:** Latest
  - Icon library (1000+ icons)
  - Tree-shakeable
  - Customizable
- **react-confetti:** Latest
  - Confetti effects for interactions

### **Fonts**
- **Google Fonts:**
  - **Playfair Display** (Serif): weights 400, 700, 900; styles normal, italic
  - **Montserrat** (Sans-serif): weights 200, 300, 400, 500, 600, 700, 800
- **next/font:** Built-in font optimization

### **Build Tools**
- **Vite:** 7.0.7 (Laravel frontend assets)
- **Next.js Built-in:** Webpack/Turbopack (Next.js)
- **laravel-vite-plugin:** 2.0.0
- **concurrently:** 9.0.1 (Run multiple commands)

---

## 📦 Complete Dependency List

### **Laravel Backend (composer.json)**

```json
{
  "require": {
    "php": "^8.4",
    "laravel/framework": "^12.0",
    "aws/aws-sdk-php": "^3.369",
    "bugsnag/bugsnag-laravel": "^2.29",
    "intervention/image": "^3.11",
    "league/flysystem-aws-s3-v3": "^3.0",
    "predis/predis": "^3.3",
    "spatie/laravel-permission": "^6.24"
  },
  "require-dev": {
    "fakerphp/faker": "^1.23",
    "laravel/pail": "^1.2",
    "laravel/pint": "^1.24",
    "laravel/sail": "^1.41",
    "laravel/tinker": "^2.10",
    "mockery/mockery": "^1.6",
    "phpunit/phpunit": "^11.5"
  }
}
```

### **Next.js Frontend (package.json)**

```json
{
  "dependencies": {
    "next": "16.1.4",
    "react": "19.2.3",
    "react-dom": "19.2.3",
    "next-intl": "^4.4.0",
    "framer-motion": "latest",
    "lucide-react": "latest",
    "react-confetti": "latest"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "typescript": "^5",
    "tailwindcss": "^4.1.18",
    "@tailwindcss/forms": "^0.5.11",
    "@tailwindcss/vite": "^4.0.0"
  },
  "engines": {
    "node": ">=22.0.0",
    "npm": ">=10.0.0"
  }
}
```

---

## 🗄️ Database Technology

### **PostgreSQL 16 Features Used**
- **JSONB columns:** For flexible metadata storage
- **Generated columns:** For calculated fields (e.g., inventory.available)
- **Full-text search:** For product and content search
- **Foreign key constraints:** For data integrity
- **Indexes:** B-tree, GIN, partial indexes
- **Soft deletes:** Using deleted_at timestamp
- **Transactions:** For multi-table operations

### **Database Statistics**
- **Total Tables:** 43
- **Core System:** 6 tables
- **Permissions:** 5 tables
- **Products:** 8 tables
- **Channels:** 5 tables
- **E-Commerce:** 5 tables
- **Content:** 7 tables
- **Attributes (EAV):** 5 tables
- **Translations:** 2 tables

---

## 🔐 Security Technologies

### **Authentication Methods**
1. **Session-based:** For admin panel (Laravel sessions)
2. **Token-based:** For API (Laravel Sanctum)
3. **Multi-guard:** Separate guards for admin and web

### **Security Features**
- **CSRF Protection:** Laravel built-in
- **XSS Prevention:** Blade template escaping
- **SQL Injection Prevention:** Eloquent ORM parameter binding
- **Password Hashing:** bcrypt (12 rounds)
- **Rate Limiting:** Laravel throttle middleware
- **CORS:** Configured for API endpoints
- **Encryption:** Laravel encryption for sensitive settings

---

## 🚀 Development Environment

### **Required Software**
- **PHP:** 8.4.6 or higher
- **Node.js:** 22.22.0 or higher (LTS)
- **PostgreSQL:** 16 or higher
- **Composer:** 2.x
- **npm:** 10.x or higher
- **Git:** Latest version

### **Optional Software**
- **Redis:** For caching and queues (optional, can use database driver)
- **Docker:** For Laravel Sail development environment
- **Bugsnag:** For error tracking (optional)

### **Development Commands**

#### **Laravel (omnistock/)**
```bash
# Install dependencies
composer install

# Setup environment
cp .env.example .env
php artisan key:generate

# Run migrations
php artisan migrate

# Seed database
php artisan db:seed

# Start development server
composer run dev  # Runs server + queue + logs + vite

# Run tests
composer run test

# Format code
./vendor/bin/pint
```

#### **Next.js (omni-ecom-nextjs/)**
```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check
```

---

## 🌐 Infrastructure & Deployment

### **Cloud Services**
- **Storage:** Cloudflare R2 or Amazon S3
- **CDN:** Cloudflare (for images and static assets)
- **Error Tracking:** Bugsnag
- **Version Control:** GitHub

### **CI/CD Pipeline (GitHub Actions)**
- **Workflow File:** `.github/workflows/laravel.yml`
- **Triggers:** Push to main/develop, pull requests
- **Steps:**
  1. Checkout code
  2. Setup PHP 8.4 and Node.js 22
  3. Install Composer dependencies
  4. Install npm dependencies
  5. Create .env file
  6. Run migrations (PostgreSQL 16 service)
  7. Run PHPUnit tests
  8. Build frontend assets

### **Environment Variables**

#### **Laravel (.env)**
```env
APP_NAME=OmniStock
APP_ENV=production
APP_KEY=base64:...
APP_DEBUG=false
APP_URL=https://admin.example.com

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=omnistock
DB_USERNAME=postgres
DB_PASSWORD=

CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=database

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=auto
AWS_BUCKET=
AWS_ENDPOINT=https://...r2.cloudflarestorage.com
AWS_URL=https://cdn.example.com

BUGSNAG_API_KEY=
```

#### **Next.js (.env.local)**
```env
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_CDN_URL=https://cdn.example.com
```

---

## 📊 Performance Optimizations

### **Backend Optimizations**
- **OPcache:** Enabled for PHP bytecode caching
- **Eloquent Eager Loading:** Prevents N+1 queries
- **Database Indexing:** All foreign keys and search columns
- **Query Optimization:** Using `select()`, `chunk()`, pagination
- **Redis Caching:** For frequently accessed data
- **Queue Workers:** For background processing
- **Image Optimization:** Automatic resizing and compression

### **Frontend Optimizations**
- **Next.js App Router:** Server-side rendering
- **Code Splitting:** Automatic by Next.js
- **Image Optimization:** next/image component
- **Font Optimization:** next/font with Google Fonts
- **CSS Purging:** Tailwind JIT removes unused styles
- **Tree Shaking:** Removes unused JavaScript
- **CDN Delivery:** Static assets served from CDN

---

## 🧪 Testing Stack

### **Backend Testing**
- **Framework:** PHPUnit 11.5.3
- **Database:** RefreshDatabase trait (in-memory SQLite or PostgreSQL)
- **Mocking:** Mockery 1.6
- **Factories:** Laravel factories for test data
- **Coverage:** Feature tests + Unit tests

### **Test Categories**
- Authentication tests
- Permission tests
- Product CRUD tests
- Category tests
- Attribute system tests
- API endpoint tests
- Translation tests

---

## 🔮 Technology Maturity & Support

| Technology | Version | Maturity | LTS Support | Notes |
|------------|---------|----------|-------------|-------|
| Laravel | 12.0 | ⭐⭐⭐⭐⭐ | Until 2028 | Latest stable |
| PHP | 8.4.6 | ⭐⭐⭐⭐⭐ | Until 2027 | Latest stable |
| PostgreSQL | 16 | ⭐⭐⭐⭐⭐ | Until 2028 | Production-ready |
| Next.js | 16.1.4 | ⭐⭐⭐⭐⭐ | Active | Latest stable |
| React | 19.2.3 | ⭐⭐⭐⭐⭐ | Active | Latest stable |
| Node.js | 22.22.0 | ⭐⭐⭐⭐⭐ | Until 2027 | LTS |
| Tailwind CSS | 4.1.18 | ⭐⭐⭐⭐⭐ | Active | Latest v4 |
| TypeScript | 5.x | ⭐⭐⭐⭐⭐ | Active | Industry standard |

---

## 📚 Technology Documentation Links

### **Backend**
- [Laravel 12 Documentation](https://laravel.com/docs/12.x)
- [PHP 8.4 Documentation](https://www.php.net/manual/en/)
- [PostgreSQL 16 Documentation](https://www.postgresql.org/docs/16/)
- [Spatie Laravel Permission](https://spatie.be/docs/laravel-permission/v6)
- [Intervention Image](https://image.intervention.io/v3)

### **Frontend**
- [Next.js 16 Documentation](https://nextjs.org/docs)
- [React 19 Documentation](https://react.dev/)
- [Tailwind CSS v4 Documentation](https://tailwindcss.com/docs)
- [next-intl Documentation](https://next-intl-docs.vercel.app/)
- [Framer Motion Documentation](https://www.framer.com/motion/)

---

**Document Status:** ✅ Complete and Ready for AI Consumption


