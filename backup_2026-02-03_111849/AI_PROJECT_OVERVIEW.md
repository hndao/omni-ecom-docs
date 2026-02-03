# AI Project Overview - OmniStock & Omni E-Commerce

**Document Purpose:** Comprehensive project overview for AI assistants (Gemini, Claude, etc.)
**Last Updated:** January 24, 2026
**Version:** 1.1

---

## 🎯 Project Mission

**OmniStock** is a multi-channel inventory management system designed for e-commerce businesses selling gemstones, jewelry, and artisan products. The platform centralizes product data and synchronizes inventory across multiple sales channels (Etsy, Shopify, Amazon, eBay, custom websites).

**Omni E-Commerce** is the customer-facing Next.js storefront that provides a luxury shopping experience with multilingual support.

---

## 📦 Project Structure

This is a **monorepo** containing two main applications:

```
/
├── omnistock/              # Laravel 12 - Admin Portal & API
│   ├── app/                # Application code
│   ├── database/           # Migrations, seeders, factories
│   ├── resources/          # Views, CSS, JS
│   ├── routes/             # API and admin routes
│   └── docs/               # Technical documentation
│
└── omni-ecom-nextjs/       # Next.js 16 - Customer Storefront
    ├── app/                # App Router pages
    ├── components/         # React components
    ├── messages/           # i18n translation files
    └── lib/                # Utilities and API client
```

---

## 🏗️ Architecture Overview

### **Backend: Laravel 12 (OmniStock)**
- **Purpose:** Admin portal, API server, inventory management
- **Database:** PostgreSQL 16 (43 tables)
- **Authentication:** Multi-guard (admin, web, API tokens)
- **Authorization:** Spatie Laravel Permission (RBAC)
- **Storage:** Cloudflare R2 / Amazon S3 for images
- **Queue:** Laravel Queue for background jobs
- **API:** RESTful API with Laravel Sanctum

### **Frontend: Next.js 16 (Omni E-Commerce)**
- **Purpose:** Customer-facing e-commerce storefront
- **Framework:** Next.js 16 with App Router
- **Styling:** Tailwind CSS v4
- **Internationalization:** next-intl v4.4.0
- **Animations:** Framer Motion
- **Icons:** Lucide React

---

## 🎨 Business Domain

### **Target Market**
- Gemstone and jewelry retailers
- Artisan product sellers
- Multi-channel e-commerce businesses
- Businesses requiring multilingual support

### **Core Business Entities**

1. **Products** - Master product catalog with SKU-based inventory
2. **Categories** - Flexible product organization (many-to-many)
3. **Attributes** - Dynamic product specifications (EAV pattern)
4. **Inventory** - Real-time stock tracking (on-hand, reserved, available)
5. **Channels** - Sales platforms (Etsy, Shopify, Amazon, Website)
6. **Listings** - Channel-specific product listings
7. **Orders** - Customer orders with payment tracking
8. **Customers** - Registered and guest customer accounts

---

## 🌍 Multilingual Support

### **Supported Locales**
- **English (en)** - Default
- **French (fr)** - Canadian French
- Expandable to additional locales

### **Translation Strategy**
- **Backend:** Translation tables (product_translations, category_translations)
- **Frontend:** next-intl with locale-based routing (`/en`, `/fr`)
- **Fallback:** English as default for missing translations

### **Translatable Content**
- Product names and descriptions
- Category names, descriptions, and slugs
- UI labels and messages
- Blog posts and pages (future)

---

## 🔑 Key Features

### **Admin Portal (Laravel)**
✅ Product management with dynamic attributes  
✅ Multi-category assignment  
✅ Image management with deduplication  
✅ Inventory tracking and adjustments  
✅ Channel connection management  
✅ Listing creation and mapping  
✅ Role-based access control (Owner, Ops, Viewer)  
✅ Audit logging for all actions  
✅ Translation management  
✅ Customer and order management  
✅ Blog and content management  

### **Customer Storefront (Next.js)**
✅ Responsive luxury design
✅ Multilingual support (en, fr)
✅ Listing-based shop (shows what customers can buy, not raw products)
✅ Product browsing with filters and search
✅ Category filtering
✅ Stock availability checking (bundle vs variant logic)
✅ External purchase links (e.g., "Buy on Etsy")
✅ Toast notification system (modern, non-blocking alerts)
✅ Locale-aware routing (`/en/shop`, `/fr/shop`)
🚧 Shopping cart (in progress)
🚧 Customer authentication (in progress)
🚧 Order placement (in progress)
📋 Blog and content pages (planned)

---

## 🔐 Security & Permissions

### **Role Hierarchy**
1. **System Administrator** - Full system access (super user)
2. **Owner** - Full business access
3. **Ops** - Operational access (daily operations)
4. **Viewer** - Read-only access

### **Permission Categories** (39 total)
- Products: view, create, edit, delete
- Categories: view, create, edit, delete
- Attributes: view, create, edit, delete
- Inventory: view, adjust
- Listings: view, create, edit, delete, sync
- Channels: view, create, edit, delete
- Images: view, upload, delete
- Users: view, create, edit, delete
- Roles: view, create, edit, delete
- Settings: manage
- Audit Logs: view

---

## 📊 Data Flow Examples

### **Product Creation Flow**
1. Admin creates product with SKU, name, description, price
2. Assigns to categories (e.g., "Bracelets", "Gemstone Products")
3. System loads category-specific attributes
4. Admin fills attribute values (gemstone type, size, weight)
5. Uploads product images (deduplicated by SHA-256 hash)
6. System creates inventory record (initial quantity: 0)
7. Audit log entry created
8. Product available for listing creation

### **Multi-Channel Sync Flow**
1. Product inventory changes (sale, adjustment, restock)
2. System updates inventory record
3. Sync job triggered for affected listings
4. System calculates available quantity per listing
5. API requests sent to each channel (Shopify, Etsy, etc.)
6. Sync results logged
7. Admin sees sync status in dashboard

### **Customer Order Flow**
1. Customer browses products (with locale-specific translations)
2. Adds products to cart
3. Proceeds to checkout (guest or registered)
4. Enters shipping/billing information
5. Selects payment method
6. Order created with unique order number (ORD-XXXXXXXX)
7. Inventory reserved for order items
8. Payment processed
9. Order status updated (pending → processing → completed)
10. Admin receives order notification

---

## 🛠️ Technology Stack Summary

### **Backend Technologies**
- **Framework:** Laravel 12.0 (PHP 8.4.6)
- **Database:** PostgreSQL 16
- **Cache/Queue:** Redis (optional), Database driver
- **Storage:** Cloudflare R2 / Amazon S3
- **Image Processing:** Intervention Image 3.11
- **Authentication:** Laravel Sanctum
- **Permissions:** Spatie Laravel Permission 6.24
- **Error Tracking:** Bugsnag
- **Testing:** PHPUnit 11.5.3

### **Frontend Technologies**
- **Framework:** Next.js 16.1.4
- **React:** 19.2.3
- **TypeScript:** 5.x
- **Styling:** Tailwind CSS 4.1.18
- **i18n:** next-intl 4.4.0
- **Animations:** Framer Motion
- **Icons:** Lucide React
- **Fonts:** Google Fonts (Playfair Display, Montserrat)
- **Node.js:** v22.22.0 (LTS)

### **Development Tools**
- **Build Tool:** Vite 7.0.7 (Laravel), Next.js built-in (frontend)
- **Code Quality:** Laravel Pint (PSR-12)
- **Version Control:** Git
- **CI/CD:** GitHub Actions
- **Package Managers:** Composer (PHP), npm (Node.js)

---

## 📈 Current Status

### **Completed Features** ✅
- ✅ Product management with dynamic attributes (EAV)
- ✅ Multi-category support
- ✅ Image management with cloud storage (Cloudflare R2)
- ✅ Inventory tracking and adjustments
- ✅ Channel and listing management
- ✅ Listing-based shop (products vs listings architecture)
- ✅ Multi-product listings (bundles and variants)
- ✅ Stock calculation for bundles and variants
- ✅ Channel-based API filtering
- ✅ Role-based access control (RBAC)
- ✅ Audit logging
- ✅ Multilingual support (en, fr)
- ✅ Customer and order management
- ✅ Blog and content management
- ✅ API endpoints for frontend integration
- ✅ Next.js storefront with luxury design
- ✅ Responsive mobile-first design
- ✅ Toast notification system (replaced system alerts)
- ✅ External purchase links (Etsy integration)
- ✅ Locale-aware routing with next-intl

### **In Progress** 🚧
- 🚧 Multi-channel API integrations (Shopify, Etsy, Amazon)
- 🚧 Automated inventory synchronization
- 🚧 Shopping cart functionality
- 🚧 Customer authentication on frontend
- 🚧 Payment gateway integration

### **Planned Features** 📋
- 📋 Bulk product import/export (CSV, Excel)
- 📋 Product variants (size, color combinations)
- 📋 Barcode generation and scanning
- 📋 Purchase orders and supplier management
- 📋 Low stock alerts and reorder points
- 📋 Advanced analytics dashboard
- 📋 Mobile app for inventory management
- 📋 Webhook support for real-time updates
- 📋 Integration with accounting software (QuickBooks, Xero)

---

## 🎯 Key Design Patterns

### **1. Entity-Attribute-Value (EAV)**
- Flexible product attributes without schema changes
- 4-table design: attribute_definitions, attribute_options, category_attributes, product_attribute_values
- Supports text, number, boolean, and select data types

### **2. Repository Pattern**
- Service layer for business logic (AuditLogService, LocaleService)
- Controllers remain thin and focused on HTTP concerns
- Reusable service classes with dependency injection

### **3. Soft Deletes**
- Data preservation for audit and recovery
- Cascade deletes for data integrity
- Permanent delete requires explicit action

### **4. Translation Tables**
- Separate translation tables for multilingual content
- Translatable trait for reusable translation logic
- Locale-based fallback to default language

### **5. Observer Pattern**
- Model events for automatic actions (creating, updating, deleting)
- Audit logging on all changes
- Inventory adjustments on order creation

---

## 🔍 Important Concepts for AI Assistants

### **SKU (Stock Keeping Unit)**
- Unique identifier for each product
- Used across all channels and listings
- Format: alphanumeric with dashes/underscores (e.g., "AMETHYST-BRACELET-7IN")

### **Products vs Listings**
- **Products:** Master inventory items with SKUs (internal management)
- **Listings:** What customers see and purchase on different channels (customer-facing)
- **Relationship:** Many-to-many through `listing_mappings` pivot table

### **Listing Mapping Types**
1. **Simple (1:1):** 1 product = 1 listing (quantity_ratio = 1)
2. **Bundle (many:1):** Multiple products = 1 listing (e.g., gift set with 3 items)
3. **Variant (1:many):** 1 listing with multiple product options (e.g., different sizes/colors - customer chooses one)

### **Inventory Calculation**
- **On-Hand:** Physical quantity in stock
- **Reserved:** Quantity allocated to pending orders
- **Available:** On-Hand - Reserved (available for sale)

### **Stock Availability for Listings**
- **Bundle listings:** Available = MIN(all product quantities) - all products must be in stock
- **Variant listings:** Available = MAX(all product quantities) - at least one product must be in stock
- **Simple listings:** Available = product inventory quantity

### **Locale Codes**
- ISO 639-1 language codes (2 characters): `en`, `fr`, `es`, `de`
- Stored in `settings` table as JSON array
- Used in translation tables and API requests

### **Audit Logging**
- Every admin action is logged with:
  - Who (admin user)
  - What (action: created, updated, deleted)
  - When (timestamp)
  - Where (IP address, user agent)
  - Before/After values (JSON)

### **Toast Notification System**
- **Purpose:** Modern, non-blocking user feedback (replaced system `alert()` calls)
- **Implementation:** Alpine.js-based component with Tailwind CSS
- **Global Function:** `window.toast(message, type)`
- **Types:** `success`, `error`, `warning`, `info`
- **Features:**
  - Auto-dismiss after 8 seconds
  - Manual close button
  - Smooth slide-in animations
  - Color-coded with icons
  - Top-right positioning with proper z-index
  - Multiple toasts stack vertically
- **Usage:** All admin feedback messages (sync, delete, create, update operations)

---

## 📚 Documentation Structure

This documentation is organized into 5 AI-friendly files:

1. **AI_PROJECT_OVERVIEW.md** (this file) - Business context, architecture, features
2. **AI_TECH_STACK.md** - Complete technology stack, versions, dependencies
3. **AI_DATABASE_GUIDE.md** - Database schema, relationships, migrations
4. **AI_API_REFERENCE.md** - All API endpoints, request/response formats
5. **AI_CODE_CONVENTIONS.md** - Coding standards, patterns, best practices

---

## 🚀 Quick Start for AI Assistants

### **Understanding the Codebase**
1. Start with this overview to understand business context
2. Review AI_DATABASE_GUIDE.md for data structure
3. Check AI_API_REFERENCE.md for API endpoints
4. Read AI_CODE_CONVENTIONS.md before making code changes
5. Consult AI_TECH_STACK.md for technology-specific details

### **Common Tasks**
- **Adding a new feature:** Check existing patterns in CODE_CONVENTIONS.md
- **Database changes:** Review DATABASE_GUIDE.md for schema structure
- **API modifications:** Follow patterns in API_REFERENCE.md
- **Translation updates:** See multilingual support section above
- **Permission changes:** Review RBAC structure in this document

---

## 📝 Documentation Maintenance

### **IMPORTANT: Keep Documentation Updated**

⚠️ **When making changes to the project, you MUST update the corresponding documentation files:**

| Change Type | Files to Update |
|-------------|-----------------|
| **New feature added** | AI_PROJECT_OVERVIEW.md (features section), AI_CODE_CONVENTIONS.md (if new patterns) |
| **Database schema change** | AI_DATABASE_GUIDE.md (table definitions, relationships) |
| **New API endpoint** | AI_API_REFERENCE.md (endpoint documentation) |
| **Technology upgrade** | AI_TECH_STACK.md (version numbers, dependencies) |
| **New coding pattern** | AI_CODE_CONVENTIONS.md (pattern examples) |
| **Translation/locale added** | AI_PROJECT_OVERVIEW.md (multilingual section), AI_TECH_STACK.md (i18n config) |
| **Permission/role change** | AI_PROJECT_OVERVIEW.md (security section), AI_DATABASE_GUIDE.md (permissions tables) |
| **New channel type** | AI_PROJECT_OVERVIEW.md (business domain), AI_DATABASE_GUIDE.md (channels table) |

### **Documentation Update Checklist**

Before merging any PR that changes project structure, features, or architecture:

- [ ] Review all 5 AI documentation files
- [ ] Update relevant sections with new information
- [ ] Update version numbers and dates
- [ ] Verify code examples are still accurate
- [ ] Update API endpoint counts if changed
- [ ] Update table counts if database schema changed
- [ ] Test that AI assistants can understand the changes

### **Version Control**

Each documentation file has a version number and "Last Updated" date at the top. Increment the version when making significant updates:

- **Patch (1.0 → 1.1):** Minor updates, clarifications, typo fixes
- **Minor (1.0 → 2.0):** New sections, significant content additions
- **Major (1.0 → 2.0):** Complete restructuring or major architectural changes

---

**Document Status:** ✅ Complete and Ready for AI Consumption
**Last Updated:** January 24, 2026
**Version:** 1.1

---

## 📝 Recent Updates (v1.1 - January 24, 2026)

### **Listing-Based Shop Architecture**
- Implemented distinction between **Products** (inventory) and **Listings** (customer-facing)
- Created `/api/listings` endpoint for customer shop
- Added support for bundle and variant listing types
- Implemented stock calculation logic for different mapping types
- Updated Next.js shop to use listings instead of products

### **Toast Notification System**
- Replaced all system `alert()` calls with modern toast notifications
- Implemented Alpine.js-based toast component
- Added global `window.toast()` function
- Updated 10 admin blade templates (36 alert calls replaced)
- Improved user experience with non-blocking notifications

### **Frontend Improvements**
- Fixed double locale in URLs (`/en/en/shop` → `/en/shop`)
- Added stock availability helpers for bundles and variants
- Implemented external purchase URL support (Etsy links)
- Updated product/listing name display logic
- Improved TypeScript types for listings

### **API Enhancements**
- Added `ListingResource` for API responses
- Implemented channel-based filtering for listings
- Added support for slug-based listing retrieval
- Enhanced product-listing relationship loading
- Improved inventory calculation for multi-product listings


