# Changelog - January 24, 2026

## 🎯 Summary

This changelog documents the major updates made to the OmniStock and Omni E-Commerce projects on January 24, 2026, including the listing-based shop architecture, toast notification system, and documentation updates.

---

## 🏪 Listing-Based Shop Architecture

### **Problem Solved**
The shop was displaying raw **products** (inventory items) instead of **listings** (what customers actually purchase). This caused issues:
- A listing with 2 products showed as 2 separate items in the shop
- Stock availability wasn't calculated correctly for bundles and variants
- No distinction between internal inventory and customer-facing offerings

### **Solution Implemented**

#### **Backend Changes**
1. **Created `ListingResource.php`** - API resource for transforming listings
2. **Created `ListingController.php`** - API endpoints for listings
3. **Added API Routes:**
   - `GET /api/listings` - List all listings for a channel
   - `GET /api/listings/{id}` - Get single listing (supports ID or slug)

#### **Frontend Changes**
1. **Updated `lib/api.ts`:**
   - Added `getListings()` function
   - Added `getListing()` function
2. **Updated `app/[locale]/shop/page.tsx`:**
   - Changed from `getProducts()` to `getListings()`
3. **Updated `app/[locale]/shop/[id]/page.tsx`:**
   - Changed from `getProduct()` to `getListing()`
4. **Updated TypeScript types:**
   - Added listing-specific fields: `title`, `slug`, `mapping_type`, `calculated_quantity`, `products[]`

### **Stock Calculation Logic**

Added helper functions to properly calculate stock for different listing types:

```typescript
// Bundle listings: ALL products must be in stock
if (mapping_type === 'bundle') {
  available = MIN(all product quantities)
}

// Variant listings: ANY product must be in stock
if (mapping_type === 'variant') {
  available = MAX(all product quantities)
}

// Simple listings: Direct product inventory
available = product.inventory.available
```

### **Impact**
- ✅ Shop now correctly shows 3 listings instead of 3 products
- ✅ "ABC ABC" variant listing shows as IN STOCK (was showing OUT OF STOCK)
- ✅ Bundle listings calculate stock correctly
- ✅ External purchase links work (e.g., "Buy on Etsy")

---

## 🎨 Toast Notification System

### **Problem Solved**
The admin panel used system `alert()` calls for user feedback, which:
- Block the entire UI
- Look unprofessional
- Can't be dismissed easily
- Don't support different message types

### **Solution Implemented**

#### **Created Toast Component**
- **File:** `resources/views/layouts/admin.blade.php`
- **Technology:** Alpine.js + Tailwind CSS
- **Global Function:** `window.toast(message, type)`

#### **Features**
- ✅ 4 types: `success`, `error`, `warning`, `info`
- ✅ Auto-dismiss after 8 seconds
- ✅ Manual close button
- ✅ Smooth slide-in animations
- ✅ Color-coded with icons
- ✅ Multiple toasts stack vertically
- ✅ Top-right positioning

#### **Files Updated**
Replaced 36 `alert()` calls across 10 blade templates:
1. `resources/views/admin/listings/index.blade.php` - 7 alerts
2. `resources/views/admin/products/index.blade.php` - 3 alerts
3. `resources/views/admin/categories/index.blade.php` - 7 alerts
4. `resources/views/admin/attributes/index.blade.php` - 6 alerts
5. `resources/views/admin/images/index.blade.php` - 5 alerts
6. `resources/views/admin/mappings/index.blade.php` - 3 alerts
7. `resources/views/admin/menus/edit.blade.php` - 5 alerts

### **Impact**
- ✅ Professional, modern UI feedback
- ✅ Non-blocking notifications
- ✅ Consistent user experience
- ✅ Better accessibility

---

## 🐛 Bug Fixes

### **Fixed: Double Locale in URLs**
- **Problem:** Clicking listing cards navigated to `/en/en/shop/18`
- **Cause:** Manual locale prefix when router already handles it
- **Fix:** Changed `router.push(\`/${locale}/shop/${id}\`)` to `router.push(\`/shop/${id}\`)`
- **Impact:** URLs now correctly show `/en/shop/18`

---

## 📚 Documentation Updates

### **Files Updated**
1. **`docs/AI_PROJECT_OVERVIEW.md`** (v1.0 → v1.1)
   - Added Products vs Listings distinction
   - Documented toast notification system
   - Updated completed features list
   - Added stock calculation logic
   - Added recent updates section

2. **`docs/AI_API_REFERENCE.md`** (v1.0 → v1.1)
   - Added `/api/listings` endpoint documentation
   - Added `/api/listings/{id}` endpoint documentation
   - Documented listing mapping types
   - Added stock calculation examples
   - Updated API summary

3. **`docs/AI_DATABASE_GUIDE.md`** (v1.0 → v1.1)
   - Updated `listings` table documentation
   - Updated `listing_mappings` table documentation
   - Added detailed mapping examples
   - Documented new columns: `slug`, `calculated_quantity`, `mapping_type`, `etsy_listing_id`
   - Added recent updates section

---

## 🎯 Key Concepts for Future Development

### **Products vs Listings**
- **Products** = Master inventory items (SKU-based, internal)
- **Listings** = What customers see and buy (channel-specific, customer-facing)
- **Relationship** = Many-to-many through `listing_mappings`

### **Listing Mapping Types**
1. **Simple (1:1):** 1 product = 1 listing
2. **Bundle (many:1):** Multiple products = 1 listing (gift sets)
3. **Variant (1:many):** 1 listing with multiple product options (sizes/colors)

### **Channel-Based Filtering**
- All API endpoints filter by channel using `X-API-Key` header
- Ensures customers only see products available on their platform
- Supports multi-channel inventory management

---

## ✅ Testing Checklist

- [x] Shop page shows listings instead of products
- [x] Variant listings show correct stock status
- [x] Bundle listings calculate stock correctly
- [x] External purchase links work
- [x] Toast notifications appear and dismiss
- [x] URL navigation works without double locale
- [x] API endpoints return correct data
- [x] Documentation is up to date

---

**Date:** January 24, 2026  
**Version:** 1.1  
**Status:** ✅ Complete

