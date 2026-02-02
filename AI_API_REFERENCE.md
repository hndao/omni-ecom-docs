# AI API Reference - OmniStock REST API

**Document Purpose:** Complete API endpoint reference for AI assistants
**Last Updated:** January 24, 2026
**Version:** 1.1
**Base URL:** `https://api.example.com/api`

---

## 🎯 API Overview

The OmniStock API is a RESTful API that provides access to product catalog, customer management, and order processing.

**Authentication:** Laravel Sanctum (Token-based)  
**Response Format:** JSON  
**Character Encoding:** UTF-8  
**Rate Limiting:** 60 requests per minute per IP

---

## 🔐 Authentication

### **Register Customer**
```
POST /api/register
```

**Request Body:**
```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "password_confirmation": "SecurePass123!"
}
```

**Response (201 Created):**
```json
{
  "message": "Registration successful",
  "customer": {
    "id": 1,
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "created_at": "2026-01-22T10:00:00.000000Z"
  },
  "token": "1|abc123xyz..."
}
```

---

### **Login**
```
POST /api/login
```

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```

**Response (200 OK):**
```json
{
  "message": "Login successful",
  "customer": {
    "id": 1,
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com"
  },
  "token": "2|def456uvw..."
}
```

---

### **Logout**
```
POST /api/logout
```

**Headers:**
```
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "message": "Logged out successfully"
}
```

---

### **Get Profile**
```
GET /api/profile
```

**Headers:**
```
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "email_verified_at": "2026-01-22T10:05:00.000000Z",
  "created_at": "2026-01-22T10:00:00.000000Z",
  "updated_at": "2026-01-22T10:00:00.000000Z"
}
```

---

### **Update Profile**
```
PUT /api/profile
```

**Headers:**
```
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "first_name": "John",
  "last_name": "Smith",
  "phone": "+1234567890"
}
```

**Response (200 OK):**
```json
{
  "message": "Profile updated successfully",
  "customer": {
    "id": 1,
    "first_name": "John",
    "last_name": "Smith",
    "email": "john@example.com",
    "phone": "+1234567890"
  }
}
```

---

### **Change Password**
```
POST /api/change-password
```

**Headers:**
```
Authorization: Bearer {token}
```

**Request Body:**
```json
{
  "current_password": "OldPass123!",
  "password": "NewPass456!",
  "password_confirmation": "NewPass456!"
}
```

**Response (200 OK):**
```json
{
  "message": "Password changed successfully"
}
```

---

## 🏪 Listings API (Customer-Facing Shop)

### **List Listings**
```
GET /api/listings
```

**Purpose:** Get listings for the customer-facing shop. Listings are what customers see and purchase, not raw products.

**Headers:**
```
X-API-Key: {channel_api_key}
X-Locale: en
```

**Query Parameters:**
- `page` (integer) - Page number (default: 1)
- `per_page` (integer) - Items per page (default: 15, max: 100)
- `category` (string) - Filter by category slug
- `search` (string) - Search in title and description
- `status` (string) - Filter by status: active, inactive (default: active)
- `sort` (string) - Sort field: title, price, created_at (default: created_at)
- `order` (string) - Sort order: asc, desc (default: desc)

**Example Request:**
```
GET /api/listings?category=bracelets&search=amethyst&page=1&per_page=12
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": 19,
      "title": "ABC ABC",
      "slug": "abc-abc",
      "description": "Beautiful handmade product...",
      "price": "49.99",
      "status": "active",
      "mapping_type": "variant",
      "calculated_quantity": 169,
      "url": null,
      "external_id": null,
      "products": [
        {
          "id": 1,
          "sku": "WSDRS01",
          "name": "Women's Summer Dress",
          "quantity_ratio": 1,
          "inventory": {
            "on_hand": 74,
            "reserved": 0,
            "available": 74
          },
          "images": [...],
          "categories": [...]
        }
      ],
      "images": [...],
      "categories": [...],
      "external_purchase_url": null,
      "created_at": "2026-01-24T10:00:00.000000Z",
      "updated_at": "2026-01-24T10:00:00.000000Z"
    }
  ],
  "links": {...},
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 1,
    "per_page": 12,
    "to": 3,
    "total": 3
  }
}
```

**Listing Mapping Types:**
- `simple`: 1 product = 1 listing (1:1 ratio)
- `bundle`: Multiple products = 1 listing (e.g., gift set with 3 items)
- `variant`: 1 listing with multiple product options (e.g., different sizes/colors)

---

### **Get Single Listing**
```
GET /api/listings/{id}
GET /api/listings/{slug}
```

**Purpose:** Get detailed information about a specific listing. Supports both numeric ID and slug.

**Headers:**
```
X-API-Key: {channel_api_key}
X-Locale: en
```

**Response (200 OK):**
```json
{
  "data": {
    "id": 19,
    "title": "ABC ABC",
    "slug": "abc-abc",
    "description": "Beautiful handmade product...",
    "price": "49.99",
    "status": "active",
    "mapping_type": "variant",
    "calculated_quantity": 169,
    "products": [
      {
        "id": 1,
        "sku": "WSDRS01",
        "name": "Women's Summer Dress",
        "description": "Elegant summer dress...",
        "price": "39.99",
        "quantity_ratio": 1,
        "inventory": {
          "on_hand": 74,
          "reserved": 0,
          "available": 74
        },
        "images": [
          {
            "id": 10,
            "storage_url": "https://cdn.example.com/images/dress.jpg",
            "thumbnail_url": "https://cdn.example.com/images/dress-thumb.jpg",
            "is_cover": true,
            "sort_order": 0
          }
        ],
        "categories": [...]
      },
      {
        "id": 2,
        "sku": "WEARB01",
        "name": "Wireless Earbuds",
        "quantity_ratio": 1,
        "inventory": {
          "available": 95
        },
        "images": [...],
        "categories": [...]
      }
    ],
    "images": [...],
    "categories": [...],
    "external_purchase_url": null,
    "created_at": "2026-01-24T10:00:00.000000Z",
    "updated_at": "2026-01-24T10:00:00.000000Z"
  }
}
```

---

## 📦 Products API (Internal/Admin)

### **List Products**
```
GET /api/products
```

**Purpose:** Get raw product inventory data. Used internally for admin purposes. For customer-facing shop, use `/api/listings` instead.

**Headers:**
```
X-API-Key: {channel_api_key}
X-Locale: en
```

**Query Parameters:**
- `page` (integer) - Page number (default: 1)
- `per_page` (integer) - Items per page (default: 15, max: 100)
- `category` (string) - Filter by category slug
- `search` (string) - Search in name and description
- `min_price` (decimal) - Minimum price filter
- `max_price` (decimal) - Maximum price filter
- `sort` (string) - Sort field: name, price, created_at (default: created_at)
- `order` (string) - Sort order: asc, desc (default: desc)

**Example Request:**
```
GET /api/products?category=5&min_price=10&max_price=100&sort=price&order=asc&page=1&per_page=20
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": 1,
      "sku": "AMETHYST-BRACELET-7IN",
      "name": "Amethyst Crystal Bracelet",
      "description": "Beautiful 7-inch amethyst bracelet...",
      "price": "49.99",
      "status": "active",
      "cover_image": {
        "url": "https://cdn.example.com/images/product-1.jpg",
        "thumbnail_url": "https://cdn.example.com/images/product-1-thumb.jpg"
      },
      "categories": [
        {
          "id": 5,
          "name": "Bracelets",
          "slug": "bracelets"
        }
      ],
      "created_at": "2026-01-15T08:00:00.000000Z"
    }
  ],
  "links": {
    "first": "https://api.example.com/api/products?page=1",
    "last": "https://api.example.com/api/products?page=10",
    "prev": null,
    "next": "https://api.example.com/api/products?page=2"
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 10,
    "per_page": 15,
    "to": 15,
    "total": 150
  }
}
```

---

### **Get Single Product**
```
GET /api/products/{id}
```

**Response (200 OK):**
```json
{
  "id": 1,
  "sku": "AMETHYST-BRACELET-7IN",
  "name": "Amethyst Crystal Bracelet",
  "description": "Beautiful 7-inch amethyst bracelet made with natural amethyst beads...",
  "price": "49.99",
  "cost_price": "25.00",
  "status": "active",
  "images": [
    {
      "id": 10,
      "url": "https://cdn.example.com/images/product-1.jpg",
      "thumbnail_url": "https://cdn.example.com/images/product-1-thumb.jpg",
      "is_cover": true,
      "sort_order": 0
    }
  ],
  "categories": [
    {
      "id": 5,
      "name": "Bracelets",
      "slug": "bracelets"
    }
  ],
  "attributes": [
    {
      "name": "Gemstone Type",
      "value": "Amethyst"
    }
  ],
  "inventory": {
    "on_hand": 25,
    "reserved": 3,
    "available": 22
  },
  "created_at": "2026-01-15T08:00:00.000000Z"
}
```

---

## 🛒 Orders API

### **Create Order (Guest)**
```
POST /api/orders/guest
```

**Request Body:**
```json
{
  "items": [
    {
      "product_id": 1,
      "quantity": 2
    }
  ],
  "customer": {
    "email": "guest@example.com",
    "first_name": "Jane",
    "last_name": "Smith"
  },
  "shipping_address": {
    "first_name": "Jane",
    "last_name": "Smith",
    "address_line_1": "456 Oak Ave",
    "city": "Los Angeles",
    "state": "CA",
    "postal_code": "90001",
    "country": "US"
  },
  "payment_method": "paypal"
}
```

**Response (201 Created):**
```json
{
  "message": "Order created successfully",
  "order": {
    "id": 102,
    "order_number": "ORD-20260122-003",
    "status": "pending",
    "total": "109.98"
  }
}
```

---

## ⚠️ Error Responses

### **Validation Error (422)**
```json
{
  "message": "The given data was invalid.",
  "errors": {
    "email": [
      "The email field is required."
    ]
  }
}
```

---

### **Unauthorized (401)**
```json
{
  "message": "Unauthenticated."
}
```

---

### **Not Found (404)**
```json
{
  "message": "Resource not found."
}
```

---

## 📊 API Summary

**Total Endpoints:** 25+
**Authentication:** Laravel Sanctum (Token-based) + Channel API Keys
**Rate Limiting:** 60 requests/minute
**Response Format:** JSON

### **Channel-Based Filtering**

All API endpoints use channel-based filtering via the `X-API-Key` header:
- Each channel (Main Website, Etsy, Shopify, etc.) has a unique API key
- Products and listings are filtered to show only items available on that channel
- This ensures customers only see products available for purchase on their current platform

### **Key Concepts**

**Products vs Listings:**
- **Products** = Master inventory items with SKUs (internal)
- **Listings** = What customers see and purchase (customer-facing)
- A listing can contain multiple products (bundles) or represent product variants

**Mapping Types:**
- **Simple (1:1):** 1 product = 1 listing
- **Bundle (many:1):** Multiple products = 1 listing (e.g., gift set)
- **Variant (1:many):** 1 listing with multiple product options (e.g., sizes/colors)

**Stock Calculation:**
- **Bundle listings:** Available quantity = MIN(all product quantities)
- **Variant listings:** Available quantity = MAX(all product quantities)
- **Simple listings:** Available quantity = product inventory

---

## 🛒 Cart API

### **Get Cart**
```
GET /api/cart
```

**Headers:**
```
X-Session-ID: {session_id}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 1,
        "product_id": 5,
        "product_name": "Premium Headphones",
        "sku": "HDPH-001",
        "price": "99.99",
        "quantity": 2,
        "subtotal": "199.98",
        "image_url": "https://example.com/images/headphones.jpg"
      }
    ],
    "subtotal": "199.98",
    "item_count": 2
  }
}
```

---

### **Add Item to Cart**
```
POST /api/cart/items
```

**Headers:**
```
X-Session-ID: {session_id}
```

**Request Body:**
```json
{
  "product_id": 5,
  "quantity": 2
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Item added to cart",
  "data": {
    "id": 1,
    "product_id": 5,
    "quantity": 2,
    "subtotal": "199.98"
  }
}
```

---

### **Update Cart Item**
```
PUT /api/cart/items/{item_id}
```

**Request Body:**
```json
{
  "quantity": 3
}
```

---

### **Remove Cart Item**
```
DELETE /api/cart/items/{item_id}
```

---

### **Clear Cart**
```
DELETE /api/cart
```

---

## ❤️ Wishlist API

### **Get Wishlist**
```
GET /api/wishlist
```

**Headers:**
```
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 1,
        "product_id": 10,
        "product_name": "Wireless Mouse",
        "sku": "MSE-002",
        "price": "29.99",
        "image_url": "https://example.com/images/mouse.jpg",
        "in_stock": true,
        "added_at": "2026-01-25T14:30:00Z"
      }
    ],
    "item_count": 1
  }
}
```

---

### **Add Item to Wishlist**
```
POST /api/wishlist/items
```

**Request Body:**
```json
{
  "product_id": 10
}
```

---

### **Remove Item from Wishlist**
```
DELETE /api/wishlist/items/{item_id}
```

---

### **Move Item to Cart**
```
POST /api/wishlist/items/{item_id}/move-to-cart
```

**Headers:**
```
Authorization: Bearer {token}
X-Session-ID: {session_id}
```

---

## 📦 Order Tracking

### **Track Order (Guest)**
```
POST /api/orders/track
```

**Request Body:**
```json
{
  "order_number": "ORD-2026-0001",
  "email": "customer@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "order_number": "ORD-2026-0001",
    "status": "shipped",
    "total": "338.97",
    "items": [...],
    "shipping_address": {...}
  }
}
```

---

**Document Status:** ✅ Complete and Ready for AI Consumption
**Last Updated:** January 27, 2026
**Version:** 1.2


