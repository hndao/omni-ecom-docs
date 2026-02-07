# Home Page Enhancement - Technical Specification

**GitHub Issue:** [#18 - Home Page Enhancement - API Integration](https://github.com/hndao/omni-ecom-nextjs/issues/18)  
**Priority:** Medium - Marketing  
**Estimated Effort:** 3-5 days  
**Created:** February 7, 2026  
**Status:** Ready for Development

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Current State Analysis](#current-state-analysis)
3. [Requirements](#requirements)
4. [Technical Architecture](#technical-architecture)
5. [Backend API Requirements](#backend-api-requirements)
6. [Frontend Implementation](#frontend-implementation)
7. [Development Workflow](#development-workflow)
8. [Task Breakdown](#task-breakdown)
9. [Testing Requirements](#testing-requirements)
10. [Acceptance Criteria](#acceptance-criteria)

---

## 📖 Overview

### Objective
Transform the current static home page into a dynamic, API-driven page that displays real-time content from the backend CMS and product catalog.

### Current Implementation
- **File:** `omni-ecom-nextjs/app/[locale]/page.tsx`
- **Type:** Client-side component with hardcoded content
- **Content:** Static hero section, hardcoded product cards, static newsletter form

### Target Implementation
- **Type:** Server-side component with client components for interactive sections
- **Content:** Dynamic hero banner, featured products from API, real blog posts, functional newsletter signup
- **Data Source:** OmniStock Laravel API

---

## 🔍 Current State Analysis

### Existing Sections (Static)
1. **Hero Section** - Hardcoded title, description, and image
2. **Trust Bar** - Static icons and text (GIA Certified, Hand-crafted, Etsy Star Seller)
3. **Rare Acquisitions** - 4 hardcoded product cards with static images
4. **Ethical Sourcing** - Static content section with image
5. **Newsletter Section** - Non-functional form

### What Needs to Change
- ✅ Keep: Trust Bar (static branding elements)
- ✅ Keep: Ethical Sourcing section (static marketing content)
- 🔄 Update: Hero Section → Dynamic from CMS
- 🔄 Update: Rare Acquisitions → Featured Products from API
- ➕ Add: Categories Showcase section
- ➕ Add: Testimonials/Reviews section
- ➕ Add: Blog Posts Preview section
- 🔄 Update: Newsletter → Functional API integration

---

## 📝 Requirements

### Functional Requirements

#### FR-1: Dynamic Hero Banner
- Display hero content from CMS (Pages table)
- Support for title, subtitle, description, CTA button text/link
- Background image from CMS
- Fallback to default content if CMS data unavailable

#### FR-2: Featured Products Section
- Display 4-6 featured products from API
- Show product image, title, price
- Link to product detail page
- Responsive grid layout (1 col mobile, 3 cols desktop)

#### FR-3: Categories Showcase
- Display 4-6 main product categories
- Show category image, name, product count
- Link to category/shop page with filter applied

#### FR-4: Testimonials/Reviews Section
- Display 3-4 customer testimonials
- Show customer name, rating, review text, date
- Carousel/slider for mobile, grid for desktop

#### FR-5: Blog Posts Preview
- Display 3 latest published blog posts
- Show featured image, title, excerpt, publish date
- Link to blog post detail page
- "View All" link to blog list page

#### FR-6: Newsletter Signup
- Functional email input and submit
- API integration to save email subscription
- Success/error message display
- Email validation

---

## 🏗️ Technical Architecture

### Component Structure
```
app/[locale]/page.tsx (Server Component)
├── HeroSection (Client Component)
├── TrustBar (Static Component)
├── FeaturedProducts (Server Component)
│   └── ProductCard (Client Component with hover effects)
├── CategoriesShowcase (Server Component)
│   └── CategoryCard (Client Component)
├── TestimonialsSection (Client Component - carousel)
├── EthicalSourcing (Static Component)
├── BlogPreview (Server Component)
│   └── BlogCard (Client Component)
└── NewsletterSection (Client Component - form)
```

### Data Flow
```
Server Component (page.tsx)
  ↓
Fetch data from API (server-side)
  ↓
Pass data as props to Client Components
  ↓
Client Components handle interactivity
```

---

## 🔌 Backend API Requirements

### Required Endpoints

#### 1. Home Page Data Endpoint
**NEW ENDPOINT REQUIRED**

```
GET /api/home
```

**Response:**
```json
{
  "success": true,
  "data": {
    "hero": {
      "title": "Ethereal Earth Gems",
      "subtitle": "Discover Rare Gemstones",
      "description": "Curated world of rare, ethically sourced gemstones...",
      "cta_text": "Discover Collection",
      "cta_link": "/shop",
      "background_image": {
        "id": 1,
        "url": "https://...",
        "storage_url": "https://...",
        "alt": "Hero background"
      }
    },
    "featured_section_title": "Rare Acquisitions",
    "featured_section_subtitle": "Curation No. 1"
  }
}
```

**Implementation Notes:**
- Create new `HomeController` in `app/Http/Controllers/Api/`
- Hero content stored in `pages` table with slug `home-hero`
- Use channel detection to return channel-specific content

---

#### 2. Featured Products Endpoint
**EXISTING ENDPOINT - NEEDS ENHANCEMENT**

```
GET /api/listings?featured=true&limit=6
```

**Current Status:** Endpoint exists but needs `featured` filter support

**Required Changes:**
- Add `is_featured` boolean column to `listings` table (migration needed)
- Update `ListingController@index` to support `featured` query parameter
- Add `limit` parameter support

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "Rare Padparadscha Sapphire",
      "slug": "rare-padparadscha-sapphire",
      "price": "12400.00",
      "sale_price": null,
      "images": [
        {
          "id": 1,
          "storage_url": "https://...",
          "is_cover": true
        }
      ],
      "is_featured": true
    }
  ],
  "meta": {
    "total": 6
  }
}
```

---

#### 3. Categories Endpoint
**EXISTING ENDPOINT**

```
GET /api/categories?show_on_website=true&limit=6
```

**Current Status:** ✅ Already implemented

---

#### 4. Testimonials Endpoint
**NEW ENDPOINT REQUIRED**

```
GET /api/testimonials?status=published&limit=4
```

**Database Requirements:**
- Create new `testimonials` table (migration needed)
- Fields: `id`, `channel_id`, `customer_name`, `customer_email`, `rating`, `review_text`, `status`, `featured`, `created_at`, `updated_at`

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "customer_name": "Sarah Johnson",
      "rating": 5,
      "review_text": "Absolutely stunning gemstone! The quality exceeded my expectations...",
      "created_at": "2026-01-15T10:00:00Z"
    }
  ]
}
```

---

#### 5. Blog Posts Preview Endpoint
**EXISTING ENDPOINT**

```
GET /api/blog/posts?status=published&limit=3&sort=published_at:desc
```

**Current Status:** ✅ Already implemented

---

#### 6. Newsletter Subscription Endpoint
**NEW ENDPOINT REQUIRED**

```
POST /api/newsletter/subscribe
```

**Request Body:**
```json
{
  "email": "customer@example.com",
  "source": "home_page"
}
```

**Database Requirements:**
- Create new `newsletter_subscriptions` table (migration needed)
- Fields: `id`, `channel_id`, `email`, `status`, `source`, `subscribed_at`, `unsubscribed_at`, `created_at`, `updated_at`

**Response:**
```json
{
  "success": true,
  "message": "Successfully subscribed to newsletter"
}
```

---

## 💻 Frontend Implementation

### File Structure
```
omni-ecom-nextjs/
├── app/[locale]/
│   └── page.tsx (Main home page - Server Component)
├── components/
│   ├── home/
│   │   ├── HeroSection.tsx (Client Component)
│   │   ├── FeaturedProducts.tsx (Server Component)
│   │   ├── ProductCard.tsx (Client Component)
│   │   ├── CategoriesShowcase.tsx (Server Component)
│   │   ├── CategoryCard.tsx (Client Component)
│   │   ├── TestimonialsSection.tsx (Client Component)
│   │   ├── BlogPreview.tsx (Server Component)
│   │   ├── BlogCard.tsx (Client Component)
│   │   └── NewsletterSection.tsx (Client Component)
│   ├── Header.tsx (Existing)
│   └── Footer.tsx (Existing)
├── lib/
│   ├── home.ts (NEW - API functions for home page)
│   ├── products.ts (Existing)
│   ├── blog.ts (Existing)
│   ├── testimonials.ts (NEW - Testimonials API functions)
│   └── newsletter.ts (NEW - Newsletter API functions)
└── types/
    └── index.ts (Add new types)
```

### New Type Definitions

Add to `types/index.ts`:

```typescript
export interface HeroContent {
  title: string;
  subtitle?: string;
  description: string;
  cta_text: string;
  cta_link: string;
  background_image?: ProductImage;
}

export interface HomePageData {
  hero: HeroContent;
  featured_section_title: string;
  featured_section_subtitle: string;
}

export interface Testimonial {
  id: number;
  customer_name: string;
  rating: number;
  review_text: string;
  created_at: string;
}

export interface NewsletterSubscription {
  email: string;
  source: string;
}
```

### New API Library Files

#### `lib/home.ts`
```typescript
import { get } from './api';
import type { HomePageData } from '@/types';

export async function getHomePageData(locale: string = 'en'): Promise<HomePageData> {
  return get('/home', locale);
}
```

#### `lib/newsletter.ts`
```typescript
import { post } from './api';
import type { NewsletterSubscription } from '@/types';

export async function subscribeToNewsletter(
  data: NewsletterSubscription,
  locale: string = 'en'
): Promise<{ message: string }> {
  return post('/newsletter/subscribe', data, { locale });
}
```

#### `lib/testimonials.ts`
```typescript
import { get } from './api';
import type { Testimonial } from '@/types';

export async function getTestimonials(
  limit: number = 4,
  locale: string = 'en'
): Promise<Testimonial[]> {
  return get(`/testimonials?status=published&limit=${limit}`, locale);
}
```

---

## 🔄 Development Workflow

### Step 1: Create Feature Branch

```bash
# Make sure you're on develop branch
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/18-home-page-enhancement

# Verify you're on the correct branch
git branch
```

### Step 2: Backend Development (Laravel)

**Location:** `omnistock/` repository

#### Task 2.1: Database Migrations

Create migrations:
```bash
cd omnistock
php artisan make:migration add_is_featured_to_listings_table
php artisan make:migration create_testimonials_table
php artisan make:migration create_newsletter_subscriptions_table
```

Run migrations:
```bash
php artisan migrate
```

#### Task 2.2: Create Models

```bash
php artisan make:model Testimonial
php artisan make:model NewsletterSubscription
```

#### Task 2.3: Create Controllers

```bash
php artisan make:controller Api/HomeController
php artisan make:controller Api/TestimonialController
php artisan make:controller Api/NewsletterController
```

#### Task 2.4: Update Routes

Add to `routes/api.php`:
```php
Route::get('/home', [HomeController::class, 'index']);
Route::get('/testimonials', [TestimonialController::class, 'index']);
Route::post('/newsletter/subscribe', [NewsletterController::class, 'subscribe']);
```

#### Task 2.5: Update Existing ListingController

Add featured filter support to `ListingController@index`

### Step 3: Frontend Development (Next.js)

**Location:** `omni-ecom-nextjs/` repository

#### Task 3.1: Create Type Definitions

Update `types/index.ts` with new interfaces

#### Task 3.2: Create API Library Files

Create:
- `lib/home.ts`
- `lib/newsletter.ts`
- `lib/testimonials.ts`

#### Task 3.3: Create Components

Create all components in `components/home/` directory

#### Task 3.4: Update Home Page

Refactor `app/[locale]/page.tsx` to use new components and API data

### Step 4: Testing

```bash
# Backend tests
cd omnistock
php artisan test

# Frontend build
cd omni-ecom-nextjs
npm run build
npm run dev
```

### Step 5: Create Pull Request

```bash
# Add all changes
git add .

# Commit with descriptive message
git commit -m "feat: Implement home page enhancement with API integration (#18)

- Add dynamic hero section from CMS
- Integrate featured products API
- Add categories showcase
- Implement testimonials section
- Add blog posts preview
- Create functional newsletter signup
- Add backend endpoints for home, testimonials, newsletter
- Create database migrations for new tables
- Update ListingController with featured filter

Closes #18"

# Push to remote
git push origin feature/18-home-page-enhancement
```

### Step 6: Create Pull Request on GitHub

1. Go to https://github.com/hndao/omni-ecom-nextjs
2. Click "Pull requests" → "New pull request"
3. Base: `develop` ← Compare: `feature/18-home-page-enhancement`
4. Title: `feat: Home Page Enhancement - API Integration (#18)`
5. Description: Link to this spec document and list all changes
6. Assign reviewers
7. Add labels: `frontend`, `backend`, `enhancement`
8. Link to issue #18
9. Create pull request

---

## 📋 Task Breakdown

### Backend Tasks (Laravel - omnistock)

#### B1: Database Schema Changes
- [ ] **B1.1** Create migration: `add_is_featured_to_listings_table`
  - Add `is_featured` boolean column (default: false)
  - Add index on `is_featured`
- [ ] **B1.2** Create migration: `create_testimonials_table`
  - All fields as specified in API requirements
  - Indexes on `channel_id`, `status`, `featured`
- [ ] **B1.3** Create migration: `create_newsletter_subscriptions_table`
  - All fields as specified in API requirements
  - Unique index on `channel_id, email`
  - Index on `status`
- [ ] **B1.4** Run migrations and verify schema

#### B2: Models
- [ ] **B2.1** Create `Testimonial` model with fillable fields and relationships
- [ ] **B2.2** Create `NewsletterSubscription` model with fillable fields
- [ ] **B2.3** Update `Listing` model to include `is_featured` in fillable

#### B3: Controllers
- [ ] **B3.1** Create `HomeController` with `index()` method
  - Fetch hero content from pages table (slug: 'home-hero')
  - Return structured JSON response
- [ ] **B3.2** Create `TestimonialController` with `index()` method
  - Support `status`, `limit`, `featured` query parameters
  - Return paginated testimonials
- [ ] **B3.3** Create `NewsletterController` with `subscribe()` method
  - Validate email format
  - Check for duplicate subscriptions
  - Create subscription record
  - Return success/error response
- [ ] **B3.4** Update `ListingController@index`
  - Add support for `featured=true` query parameter
  - Add support for `limit` query parameter

#### B4: Routes & Middleware
- [ ] **B4.1** Add routes to `routes/api.php`
- [ ] **B4.2** Apply `DetectChannel` middleware to all new routes
- [ ] **B4.3** Test routes with Postman/Insomnia

#### B5: Seeders (Optional but Recommended)
- [ ] **B5.1** Create seeder for sample testimonials
- [ ] **B5.2** Create seeder for home hero content page
- [ ] **B5.3** Update existing listings to mark some as featured

---

### Frontend Tasks (Next.js - omni-ecom-nextjs)

#### F1: Type Definitions
- [ ] **F1.1** Add `HeroContent` interface to `types/index.ts`
- [ ] **F1.2** Add `HomePageData` interface
- [ ] **F1.3** Add `Testimonial` interface
- [ ] **F1.4** Add `NewsletterSubscription` interface

#### F2: API Library Files
- [ ] **F2.1** Create `lib/home.ts` with `getHomePageData()`
- [ ] **F2.2** Create `lib/newsletter.ts` with `subscribeToNewsletter()`
- [ ] **F2.3** Create `lib/testimonials.ts` with `getTestimonials()`
- [ ] **F2.4** Update `lib/products.ts` to support `featured` parameter

#### F3: Components - Hero Section
- [ ] **F3.1** Create `components/home/HeroSection.tsx`
  - Client component with framer-motion animations
  - Props: `heroData: HeroContent`
  - Responsive layout
  - CTA button with link

#### F4: Components - Featured Products
- [ ] **F4.1** Create `components/home/ProductCard.tsx`
  - Client component with hover effects
  - Props: `product: Product`
  - Link to product detail page
- [ ] **F4.2** Create `components/home/FeaturedProducts.tsx`
  - Server component
  - Fetch featured products
  - Responsive grid layout
  - Pass data to ProductCard components

#### F5: Components - Categories Showcase
- [ ] **F5.1** Create `components/home/CategoryCard.tsx`
  - Client component with hover effects
  - Props: `category: Category`
  - Link to shop page with category filter
- [ ] **F5.2** Create `components/home/CategoriesShowcase.tsx`
  - Server component
  - Fetch categories
  - Grid layout

#### F6: Components - Testimonials
- [ ] **F6.1** Create `components/home/TestimonialsSection.tsx`
  - Client component with carousel/slider
  - Props: `testimonials: Testimonial[]`
  - Star rating display
  - Responsive: carousel on mobile, grid on desktop

#### F7: Components - Blog Preview
- [ ] **F7.1** Create `components/home/BlogCard.tsx`
  - Client component
  - Props: `post: BlogPost`
  - Link to blog post detail
- [ ] **F7.2** Create `components/home/BlogPreview.tsx`
  - Server component
  - Fetch latest 3 blog posts
  - Grid layout
  - "View All" link

#### F8: Components - Newsletter
- [ ] **F8.1** Create `components/home/NewsletterSection.tsx`
  - Client component with form state
  - Email validation
  - API integration
  - Success/error message display
  - Loading state

#### F9: Main Page Integration
- [ ] **F9.1** Refactor `app/[locale]/page.tsx`
  - Convert to Server Component
  - Fetch all data server-side
  - Pass data to client components
  - Maintain existing static sections (Trust Bar, Ethical Sourcing)
  - Proper error handling and loading states

#### F10: Styling & Responsiveness
- [ ] **F10.1** Ensure all components match existing design system
- [ ] **F10.2** Test responsive layout on mobile, tablet, desktop
- [ ] **F10.3** Verify animations and transitions
- [ ] **F10.4** Accessibility checks (ARIA labels, keyboard navigation)

---

## 🧪 Testing Requirements

### Backend Testing

#### Unit Tests
- [ ] Test `HomeController@index` returns correct structure
- [ ] Test `TestimonialController@index` filters by status
- [ ] Test `NewsletterController@subscribe` validates email
- [ ] Test `NewsletterController@subscribe` prevents duplicates
- [ ] Test `ListingController@index` filters by featured

#### Integration Tests
- [ ] Test channel detection works for all new endpoints
- [ ] Test locale support for all endpoints
- [ ] Test pagination for testimonials

### Frontend Testing

#### Manual Testing Checklist
- [ ] Hero section displays correctly with API data
- [ ] Featured products load and display properly
- [ ] Categories showcase works with links
- [ ] Testimonials carousel/slider functions smoothly
- [ ] Blog preview shows latest posts
- [ ] Newsletter form validates email
- [ ] Newsletter form shows success message
- [ ] Newsletter form shows error message for duplicates
- [ ] All links navigate correctly
- [ ] Responsive layout works on all screen sizes
- [ ] Animations perform smoothly
- [ ] Loading states display properly
- [ ] Error states handle gracefully

#### Build Testing
```bash
npm run build
# Should complete without errors
```

---

## ✅ Acceptance Criteria

### Must Have (P0)
1. ✅ Hero section displays dynamic content from CMS API
2. ✅ Featured products section shows 6 products from API
3. ✅ Newsletter signup form is functional and saves to database
4. ✅ All API endpoints return correct data structure
5. ✅ Page is responsive on mobile, tablet, and desktop
6. ✅ Build completes without errors
7. ✅ No console errors in browser

### Should Have (P1)
8. ✅ Categories showcase displays 6 categories with images
9. ✅ Blog preview shows 3 latest posts
10. ✅ Testimonials section displays customer reviews
11. ✅ All animations work smoothly
12. ✅ Loading states for async operations
13. ✅ Error handling for failed API calls

### Nice to Have (P2)
14. ⭐ Testimonials carousel auto-plays
15. ⭐ Newsletter form has email verification
16. ⭐ Hero section supports multiple slides
17. ⭐ Analytics tracking for CTA clicks
18. ⭐ A/B testing support for hero content

---

## 📚 Reference Documentation

- **GitHub Issue:** https://github.com/hndao/omni-ecom-nextjs/issues/18
- **API Documentation:** `docs/AI_API_REFERENCE.md`
- **Database Schema:** `omnistock/docs/DATABASE_SCHEMA.md`
- **Code Conventions:** `docs/AI_CODE_CONVENTIONS.md`
- **Tech Stack:** `docs/AI_TECH_STACK.md`

---

## 🚨 Important Notes

### For Backend Developers
1. **Channel Detection:** All new endpoints MUST use `DetectChannel` middleware
2. **Locale Support:** All endpoints should respect `Accept-Language` header
3. **Response Format:** Follow existing API response structure with `success`, `data`, `message`
4. **Validation:** Use Laravel Form Requests for input validation
5. **Testing:** Write tests for all new endpoints

### For Frontend Developers
1. **Server Components:** Use Server Components for data fetching where possible
2. **Client Components:** Only use "use client" for interactive components
3. **Error Handling:** Always handle API errors gracefully with fallback UI
4. **Loading States:** Show loading indicators for async operations
5. **Type Safety:** Use TypeScript interfaces for all API responses
6. **Accessibility:** Ensure all interactive elements are keyboard accessible

### Git Workflow
1. **Branch Naming:** `feature/18-home-page-enhancement`
2. **Commit Messages:** Follow conventional commits format
3. **PR Target:** Create PR to `develop` branch, NOT `main` or `sit`
4. **Code Review:** Request review before merging
5. **Testing:** Ensure all tests pass before creating PR

---

## 📞 Questions or Issues?

If you encounter any issues or have questions during implementation:

1. Check this specification document first
2. Review related documentation in `docs/` folder
3. Check existing similar implementations in the codebase
4. Ask for clarification in the GitHub issue comments
5. Update this spec document if requirements change

---

**Document Version:** 1.0
**Last Updated:** February 7, 2026
**Author:** AI Assistant
**Status:** Ready for Development


