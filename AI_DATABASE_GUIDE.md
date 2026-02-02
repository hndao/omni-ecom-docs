# AI Database Guide - OmniStock Schema

**Document Purpose:** Complete database schema reference for AI assistants
**Last Updated:** January 24, 2026
**Version:** 1.1

---

## 🎯 Database Overview

**Database Engine:** PostgreSQL 16+  
**Total Tables:** 43  
**Character Set:** UTF-8  
**Collation:** Default PostgreSQL collation

---

## 📊 Table Groups Summary

| Group | Tables | Purpose |
|-------|--------|---------|
| Core System | 6 | Users, admins, cache, jobs, audit logs, sessions |
| Permissions (RBAC) | 5 | Spatie Laravel Permission tables |
| Products | 8 | Products, inventory, images, categories, settings |
| Channels & Listings | 5 | Multi-channel management and sync |
| E-Commerce | 5 | Customers, addresses, orders, tokens |
| Content Management | 7 | Pages, blog posts, categories, tags, menus |
| Attributes (EAV) | 5 | Dynamic product attributes |
| Translations | 2 | Multilingual support |

---

## 🔑 Core Tables

### **1. products**
Master product catalog with SKU-based inventory.

**Key Columns:**
- `id` (bigint, PK) - Primary key
- `sku` (varchar, UNIQUE) - Master SKU identifier
- `name` (varchar) - Product name (default language)
- `description` (text) - Product description (default language)
- `price` (decimal 10,2) - Base selling price
- `cost_price` (decimal 10,2) - Cost for profit calculation
- `status` (varchar) - active, archived
- `metadata` (json) - Additional custom data
- `created_at`, `updated_at`, `deleted_at` (timestamps)

**Relationships:**
- Has one `inventory`
- Has many `inventory_adjustments`
- Has many `product_attribute_values`
- Has many `product_translations`
- Belongs to many `categories` (via `category_product`)
- Belongs to many `images` (via `product_images`)
- Has many `listing_mappings`

**Indexes:**
- `products_sku_unique` on `sku`
- `products_status_index` on `status`
- `products_created_at_index` on `created_at`

---

### **2. product_translations**
Multilingual translations for products.

**Key Columns:**
- `id` (bigint, PK)
- `product_id` (bigint, FK → products.id, CASCADE)
- `locale` (varchar 5) - Locale code (en, fr, etc.)
- `name` (varchar) - Translated product name
- `description` (text) - Translated description
- `created_at`, `updated_at`

**Unique Constraint:**
- `product_id` + `locale` (one translation per locale per product)

**Indexes:**
- `product_id`, `locale`

---

### **3. categories**
Product categories for organization.

**Key Columns:**
- `id` (bigint, PK)
- `name` (varchar) - Category name (default language)
- `slug` (varchar, UNIQUE) - URL-friendly slug
- `description` (text) - Category description
- `parent_id` (bigint, FK → categories.id, NULL ON DELETE) - Parent category
- `sort_order` (integer) - Display order
- `is_active` (boolean) - Active status
- `show_on_website` (boolean) - Visibility on website
- `created_at`, `updated_at`

**Relationships:**
- Belongs to many `products` (via `category_product`)
- Has many `category_attributes`
- Has many `category_translations`
- Belongs to `category` (parent)
- Has many `categories` (children)

---

### **4. category_translations**
Multilingual translations for categories.

**Key Columns:**
- `id` (bigint, PK)
- `category_id` (bigint, FK → categories.id, CASCADE)
- `locale` (varchar 5) - Locale code
- `name` (varchar) - Translated category name
- `description` (text) - Translated description
- `slug` (varchar) - Translated URL slug
- `created_at`, `updated_at`

**Unique Constraint:**
- `category_id` + `locale`

---

### **5. inventory**
Stock levels for products.

**Key Columns:**
- `id` (bigint, PK)
- `product_id` (bigint, FK → products.id, CASCADE, UNIQUE)
- `on_hand` (integer, DEFAULT 0) - Physical quantity in stock
- `reserved` (integer, DEFAULT 0) - Quantity allocated to orders
- `available` (integer, GENERATED) - Calculated: on_hand - reserved
- `created_at`, `updated_at`

**Important:** `available` is a generated column automatically calculated by PostgreSQL.

**Indexes:**
- `inventory_product_id_unique` on `product_id`
- `inventory_on_hand_index` on `on_hand`
- `inventory_available_index` on `available`

---

### **6. inventory_adjustments**
History of all inventory changes.

**Key Columns:**
- `id` (bigint, PK)
- `product_id` (bigint, FK → products.id, CASCADE)
- `admin_id` (bigint, FK → admins.id, NULL ON DELETE)
- `delta` (integer) - Change amount (+ or -)
- `on_hand_before` (integer) - Quantity before adjustment
- `on_hand_after` (integer) - Quantity after adjustment
- `reason` (varchar) - manual, sale, return, damage, recount
- `note` (text) - Optional admin note
- `created_at`, `updated_at`

**Indexes:**
- `product_id`, `admin_id`, `reason`, `created_at`

---

### **7. images**
Image metadata and storage references.

**Key Columns:**
- `id` (bigint, PK)
- `file_hash` (varchar 64, UNIQUE) - SHA-256 hash for deduplication
- `storage_path` (varchar, UNIQUE) - Path in cloud storage
- `storage_url` (varchar) - Public CDN URL
- `thumbnail_url` (varchar) - Thumbnail URL
- `filename` (varchar) - Original filename
- `mime_type` (varchar 100) - MIME type
- `size` (integer) - File size in bytes
- `width`, `height` (integer) - Image dimensions
- `uploaded_by` (bigint, FK → admins.id, NULL ON DELETE)
- `created_at`, `updated_at`

**Deduplication:** Images with the same SHA-256 hash are not uploaded twice.

**Indexes:**
- `file_hash` (unique and indexed)
- `storage_path` (unique)
- `uploaded_by`

---

### **8. product_images**
Pivot table for product-image relationships.

**Key Columns:**
- `id` (bigint, PK)
- `product_id` (bigint, FK → products.id, CASCADE)
- `image_id` (bigint, FK → images.id, CASCADE)
- `sort_order` (integer, DEFAULT 0) - Display order
- `is_cover` (boolean, DEFAULT false) - Primary image flag
- `created_at`, `updated_at`

**Unique Constraint:**
- `product_id` + `image_id`

---

## 🏷️ Entity-Attribute-Value (EAV) System

The EAV pattern allows flexible product attributes without schema changes.

### **9. attribute_definitions**
Defines available attributes.

**Key Columns:**
- `id` (bigint, PK)
- `name` (varchar, UNIQUE) - Attribute name (e.g., "Gemstone Type")
- `slug` (varchar, UNIQUE) - URL-friendly identifier
- `data_type` (varchar) - text, number, boolean, select
- `is_required` (boolean) - Required for products
- `is_filterable` (boolean) - Can be used in filters
- `is_searchable` (boolean) - Included in search
- `sort_order` (integer) - Display order
- `created_at`, `updated_at`

**Data Types:**
- `text` - Free-form text input
- `number` - Numeric values (weight, size, etc.)
- `boolean` - Yes/No values
- `select` - Dropdown with predefined options

---

### **10. attribute_options**
Predefined options for select-type attributes.

**Key Columns:**
- `id` (bigint, PK)
- `attribute_definition_id` (bigint, FK → attribute_definitions.id, CASCADE)
- `value` (varchar) - Option value (e.g., "Amethyst")
- `sort_order` (integer) - Display order
- `created_at`, `updated_at`

**Example:**
```
Attribute: "Gemstone Type" (select)
Options: "Amethyst", "Rose Quartz", "Citrine", "Clear Quartz"
```

---

### **11. category_attributes**
Links attributes to categories.

**Key Columns:**
- `id` (bigint, PK)
- `category_id` (bigint, FK → categories.id, CASCADE)
- `attribute_definition_id` (bigint, FK → attribute_definitions.id, CASCADE)
- `is_required` (boolean) - Required for this category
- `sort_order` (integer) - Display order in category
- `created_at`, `updated_at`

**Unique Constraint:**
- `category_id` + `attribute_definition_id`

**Purpose:** When a product is assigned to a category, it inherits the category's attributes.

---

### **12. product_attribute_values**
Stores actual attribute values for products.

**Key Columns:**
- `id` (bigint, PK)
- `product_id` (bigint, FK → products.id, CASCADE)
- `attribute_definition_id` (bigint, FK → attribute_definitions.id, CASCADE)
- `value_text` (text) - For text data type
- `value_number` (decimal 10,4) - For number data type
- `value_boolean` (boolean) - For boolean data type
- `attribute_option_id` (bigint, FK → attribute_options.id, NULL ON DELETE) - For select data type
- `created_at`, `updated_at`

**Unique Constraint:**
- `product_id` + `attribute_definition_id`

**Storage Strategy:**
- Only one value column is populated based on `data_type`
- For select type, `attribute_option_id` is used

---

## 🛒 E-Commerce Tables

### **13. customers**
Registered customer accounts.

**Key Columns:**
- `id` (bigint, PK)
- `email` (varchar, UNIQUE) - Customer email
- `password` (varchar) - Hashed password
- `first_name`, `last_name` (varchar)
- `phone` (varchar)
- `email_verified_at` (timestamp)
- `created_at`, `updated_at`

**Relationships:**
- Has many `customer_addresses`
- Has many `orders`
- Has many `personal_access_tokens`

---

### **14. customer_addresses**
Shipping and billing addresses.

**Key Columns:**
- `id` (bigint, PK)
- `customer_id` (bigint, FK → customers.id, CASCADE)
- `type` (varchar) - shipping, billing
- `is_default` (boolean) - Default address for type
- `first_name`, `last_name` (varchar)
- `company` (varchar)
- `address_line_1`, `address_line_2` (varchar)
- `city`, `state`, `postal_code`, `country` (varchar)
- `phone` (varchar)
- `created_at`, `updated_at`

---

### **15. orders**
Customer orders.

**Key Columns:**
- `id` (bigint, PK)
- `order_number` (varchar, UNIQUE) - Format: ORD-XXXXXXXX
- `customer_id` (bigint, FK → customers.id, NULL ON DELETE)
- `status` (varchar) - pending, processing, completed, cancelled, refunded
- `subtotal` (decimal 10,2) - Sum of item prices
- `tax` (decimal 10,2) - Tax amount
- `shipping` (decimal 10,2) - Shipping cost
- `total` (decimal 10,2) - Final total
- `payment_method` (varchar) - credit_card, paypal, etc.
- `payment_status` (varchar) - pending, paid, failed, refunded
- `shipping_address` (json) - Shipping address snapshot
- `billing_address` (json) - Billing address snapshot
- `notes` (text) - Customer notes
- `created_at`, `updated_at`

**Relationships:**
- Belongs to `customer`
- Has many `order_items`

---

### **16. order_items**
Line items in orders.

**Key Columns:**
- `id` (bigint, PK)
- `order_id` (bigint, FK → orders.id, CASCADE)
- `product_id` (bigint, FK → products.id, NULL ON DELETE)
- `sku` (varchar) - Product SKU snapshot
- `name` (varchar) - Product name snapshot
- `quantity` (integer) - Quantity ordered
- `price` (decimal 10,2) - Unit price at time of order
- `subtotal` (decimal 10,2) - quantity × price
- `created_at`, `updated_at`

**Important:** Product data is snapshotted to preserve order history even if product is deleted.

---

## 🔗 Multi-Channel Tables

### **17. channels**
Sales channel configurations.

**Key Columns:**
- `id` (bigint, PK)
- `name` (varchar) - Channel name (e.g., "Etsy Shop")
- `type` (varchar) - etsy, shopify, amazon, ebay, website
- `is_active` (boolean) - Active status
- `credentials` (text, ENCRYPTED) - API keys and tokens
- `settings` (json) - Channel-specific settings
- `last_sync_at` (timestamp) - Last successful sync
- `created_at`, `updated_at`

**Supported Channel Types:**
- `etsy` - Etsy marketplace
- `shopify` - Shopify store
- `amazon` - Amazon marketplace
- `ebay` - eBay marketplace
- `website` - Custom website (Next.js storefront)

---

### **18. listings**
Channel-specific product listings (what customers see and purchase).

**Key Columns:**
- `id` (bigint, PK)
- `channel_id` (bigint, FK → channels.id, CASCADE)
- `external_id` (varchar) - Channel's listing ID
- `title` (varchar) - Listing title
- `slug` (varchar) - URL-friendly slug
- `description` (text) - Listing description
- `price` (decimal 10,2) - Listing price
- `quantity` (integer) - Quantity available on channel
- `calculated_quantity` (integer) - Calculated from product inventories
- `status` (varchar) - active, inactive, draft
- `mapping_type` (varchar) - simple, bundle, variant
- `url` (varchar) - Public listing URL
- `etsy_listing_id` (bigint, FK → listings.id, NULL) - Link to Etsy listing for external purchase
- `metadata` (json) - Channel-specific data
- `last_synced_at` (timestamp) - Last sync time
- `created_at`, `updated_at`

**Unique Constraint:**
- `channel_id` + `external_id`

**Mapping Types:**
- `simple`: 1 product = 1 listing (1:1)
- `bundle`: Multiple products = 1 listing (e.g., gift set)
- `variant`: 1 listing with multiple product options (customer chooses one)

**Relationships:**
- Belongs to `channel`
- Belongs to many `products` (via `listing_mappings`)
- Belongs to `listing` (Etsy listing for external purchase)

---

### **19. listing_mappings**
Maps products to listings (many-to-many pivot table).

**Key Columns:**
- `id` (bigint, PK)
- `listing_id` (bigint, FK → listings.id, CASCADE)
- `product_id` (bigint, FK → products.id, CASCADE)
- `quantity_ratio` (integer, DEFAULT 1) - How many of this product per listing unit
- `status` (varchar) - active, inactive
- `created_at`, `updated_at`

**Unique Constraint:**
- `listing_id` + `product_id`

**Mapping Examples:**

1. **Simple (1:1):**
   - 1 product → 1 listing
   - `quantity_ratio = 1`
   - Example: Single bracelet product → Single bracelet listing

2. **Bundle (many:1):**
   - Multiple products → 1 listing
   - Each product has `quantity_ratio` (e.g., 2, 1, 3)
   - Example: Gift set with 2 candles + 1 soap + 3 bath bombs
   - Stock calculation: MIN(candle_qty/2, soap_qty/1, bathbomb_qty/3)

3. **Variant (1:many):**
   - 1 listing → Multiple products (customer chooses one)
   - Each product has `quantity_ratio = 1`
   - Example: T-shirt listing with Small, Medium, Large products
   - Stock calculation: MAX(small_qty, medium_qty, large_qty)

**Relationships:**
- Belongs to `listing`
- Belongs to `product`

---

### **20. sync_jobs**
Inventory sync job history.

**Key Columns:**
- `id` (bigint, PK)
- `channel_id` (bigint, FK → channels.id, CASCADE)
- `listing_id` (bigint, FK → listings.id, NULL ON DELETE)
- `status` (varchar) - pending, processing, completed, failed
- `type` (varchar) - inventory, price, listing
- `payload` (json) - Sync request data
- `response` (json) - Channel API response
- `error_message` (text) - Error details if failed
- `started_at`, `completed_at` (timestamps)
- `created_at`, `updated_at`

---

## 📝 Content Management Tables

### **21. pages**
Static pages (About, Contact, etc.).

**Key Columns:**
- `id` (bigint, PK)
- `title` (varchar) - Page title
- `slug` (varchar, UNIQUE) - URL slug
- `content` (text) - Page content (HTML/Markdown)
- `meta_title`, `meta_description` (varchar) - SEO metadata
- `is_published` (boolean) - Published status
- `published_at` (timestamp) - Publication date
- `created_at`, `updated_at`

---

### **22. blog_posts**
Blog articles.

**Key Columns:**
- `id` (bigint, PK)
- `title` (varchar) - Post title
- `slug` (varchar, UNIQUE) - URL slug
- `excerpt` (text) - Short summary
- `content` (text) - Full content
- `featured_image` (varchar) - Image URL
- `author_id` (bigint, FK → admins.id, NULL ON DELETE)
- `is_published` (boolean)
- `published_at` (timestamp)
- `created_at`, `updated_at`

**Relationships:**
- Belongs to many `blog_categories`
- Belongs to many `blog_tags` (via `blog_post_tag`)

---

## 🔐 Permission System (Spatie Laravel Permission)

### **23. permissions**
Individual permissions.

**Key Columns:**
- `id` (bigint, PK)
- `name` (varchar, UNIQUE) - Permission name (e.g., "products.create")
- `guard_name` (varchar) - Guard name (admin, web)
- `created_at`, `updated_at`

**Total Permissions:** 39 (view, create, edit, delete for each resource)

---

### **24. roles**
User roles.

**Key Columns:**
- `id` (bigint, PK)
- `name` (varchar, UNIQUE) - Role name
- `guard_name` (varchar) - Guard name
- `created_at`, `updated_at`

**Default Roles:**
1. System Administrator (super user)
2. Owner (full business access)
3. Ops (operational access)
4. Viewer (read-only)

---

### **25-27. Permission Pivot Tables**
- `model_has_permissions` - Direct permissions to users
- `model_has_roles` - Roles assigned to users
- `role_has_permissions` - Permissions assigned to roles

---

## 🔍 Important Database Patterns

### **Soft Deletes**
Tables with `deleted_at` column:
- `products`, `categories`, `admins`, `users`, `customers`, `blog_posts`, `pages`

**Behavior:**
- Records are not physically deleted
- Queries automatically exclude soft-deleted records
- Can be restored with `restore()` method

### **Timestamps**
All tables have `created_at` and `updated_at` columns automatically managed by Laravel.

### **Foreign Key Constraints**
- **CASCADE:** Child records deleted when parent is deleted
- **NULL ON DELETE:** Foreign key set to NULL when parent is deleted
- **RESTRICT:** Prevents deletion if child records exist

### **Indexes**
- All foreign keys are indexed
- Unique constraints create unique indexes
- Additional indexes on frequently queried columns (status, created_at, etc.)

---

**Document Status:** ✅ Complete and Ready for AI Consumption
**Last Updated:** January 24, 2026
**Version:** 1.1

---

## 📝 Recent Updates (v1.1 - January 24, 2026)

### **Listings Table Updates**
- Added `slug` column for URL-friendly identifiers
- Added `calculated_quantity` for computed stock levels
- Added `mapping_type` (simple, bundle, variant)
- Added `etsy_listing_id` for external purchase links

### **Listing Mappings Table Updates**
- Renamed `quantity_per_listing` to `quantity_ratio` for clarity
- Added `status` column (active, inactive)
- Added unique constraint on `listing_id` + `product_id`
- Enhanced documentation with detailed mapping examples

### **Key Architectural Changes**
- Clarified distinction between Products (inventory) and Listings (customer-facing)
- Documented stock calculation logic for bundles and variants
- Added relationship documentation for listing-product mappings


