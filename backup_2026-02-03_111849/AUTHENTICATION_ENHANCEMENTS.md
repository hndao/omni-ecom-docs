# Authentication & User Management Enhancements

**Date:** January 27, 2026  
**Repository:** `hndao/omni-ecom-nextjs` (Frontend) & `omnistock` (Backend)  
**Total Issues Created:** 7 (Issues #21-#27)

---

## 📋 Overview

This document outlines the planned authentication and user management enhancements for the Omni E-Commerce platform. These features will improve security, user experience, and provide modern authentication options.

---

## 🎯 GitHub Issues Created

### **Security Features** (3 issues)

| Issue # | Title | Priority | Status | URL |
|---------|-------|----------|--------|-----|
| #21 | Implement reCAPTCHA for Login and Register Pages | High | 🔄 OPEN | https://github.com/hndao/omni-ecom-nextjs/issues/21 |
| #22 | Implement Email Verification with 6-Digit Code | High | 🔄 OPEN | https://github.com/hndao/omni-ecom-nextjs/issues/22 |
| #23 | Implement Password Reset by Email | High | 🔄 OPEN | https://github.com/hndao/omni-ecom-nextjs/issues/23 |

### **Verification Tasks** (3 issues)

| Issue # | Title | Priority | Status | URL |
|---------|-------|----------|--------|-----|
| #24 | Verify Change Password Functionality | Medium | 🔄 OPEN | https://github.com/hndao/omni-ecom-nextjs/issues/24 |
| #25 | Verify Profile Update Functionality | Medium | 🔄 OPEN | https://github.com/hndao/omni-ecom-nextjs/issues/25 |
| #26 | Verify Address Management Functionality | Medium | 🔄 OPEN | https://github.com/hndao/omni-ecom-nextjs/issues/26 |

### **Enhanced Features** (1 issue)

| Issue # | Title | Priority | Status | URL |
|---------|-------|----------|--------|-----|
| #27 | Implement Social Media Authentication (OAuth) | Medium | 🔄 OPEN | https://github.com/hndao/omni-ecom-nextjs/issues/27 |

---

## 🔐 Feature Details

### 1. reCAPTCHA Protection (#21)

**Description:** Add Google reCAPTCHA v3 to login and register forms to prevent bot attacks.

**Frontend Work:**
- Install `react-google-recaptcha-v3` package
- Add reCAPTCHA component to login/register pages
- Execute reCAPTCHA only on form submission
- Send token to backend with form data

**Backend Work:**
- Create middleware to verify reCAPTCHA token
- Add settings in admin panel (site key, secret key, enable/disable, score threshold)
- Verify token with Google reCAPTCHA API
- Apply middleware to login and register endpoints

**Environment Variables:**
```env
RECAPTCHA_SITE_KEY=your-site-key
RECAPTCHA_SECRET_KEY=your-secret-key
```

---

### 2. Email Verification with 6-Digit Code (#22)

**Description:** Send verification code to user email on registration. Code expires after 5 minutes.

**Frontend Work:**
- Create email verification page
- Code input form (6 digits)
- Resend code button
- Countdown timer (5 minutes)

**Backend Work:**
- Generate random 6-digit code
- Store code in database with expiration timestamp
- Send verification email with code
- Create verification endpoint
- Handle code expiration and resend logic

**Database:**
- Create `email_verifications` table:
  - `id`, `customer_id`, `email`, `code`, `expires_at`, `verified_at`, `created_at`

---

### 3. Password Reset by Email (#23)

**Description:** Allow users to reset password via email with 6-digit code.

**Frontend Work:**
- Forgot password page (email input)
- Reset password page (code + new password)
- Resend code functionality

**Backend Work:**
- Generate 6-digit reset code
- Store code with expiration (5 minutes)
- Send password reset email
- Verify code endpoint
- Reset password endpoint

**Database:**
- Create `password_resets` table:
  - `id`, `email`, `code`, `expires_at`, `used_at`, `created_at`

---

### 4. Change Password Verification (#24)

**Description:** Verify existing change password functionality is working correctly.

**Tasks:**
- Test change password in account settings
- Verify current password validation
- Verify new password requirements
- Verify token revocation on password change
- Test error handling

**Existing Implementation:**
- Backend: `PUT /api/auth/change-password`
- Frontend: `app/[locale]/account/settings/SettingsClient.tsx`

---

### 5. Profile Update Verification (#25)

**Description:** Verify existing profile update functionality is working correctly.

**Tasks:**
- Test profile update in account settings
- Verify field validation
- Handle email change (may need verification)
- Test error handling

**Existing Implementation:**
- Backend: `PUT /api/auth/profile`
- Frontend: `app/[locale]/account/settings/SettingsClient.tsx`

---

### 6. Address Management Verification (#26)

**Description:** Verify existing address management CRUD operations are working correctly.

**Tasks:**
- Test add new address
- Test edit existing address
- Test delete address
- Test set default address (shipping/billing)
- Verify validation logic

**Existing Implementation:**
- Backend: `/api/customer/addresses/*`
- Frontend: `app/[locale]/account/addresses/AddressesClient.tsx`

---

### 7. Social Media Authentication (OAuth) (#27)

**Description:** Allow users to register and login via social media providers.

**Supported Providers:**
- Google
- Facebook
- Twitter
- GitHub

**Frontend Work:**
- Social login buttons on login/register pages
- OAuth callback handling
- Link social account to existing account

**Backend Work:**
- Install Laravel Socialite
- Create social accounts table
- Implement OAuth callback handling
- Link social accounts to customers
- Admin settings for provider configuration:
  - Enable/disable each provider
  - Client ID and secret for each provider
- Send 6-digit verification email when user registers via social media

**Database:**
- Create `social_accounts` table:
  - `id`, `customer_id`, `provider`, `provider_id`, `provider_email`, `avatar`, `created_at`

**Admin Settings:**
- Google OAuth (enable/disable, client ID, client secret)
- Facebook OAuth (enable/disable, app ID, app secret)
- Twitter OAuth (enable/disable, API key, API secret)
- GitHub OAuth (enable/disable, client ID, client secret)

---

## 📊 Summary

| Category | Issues | Priority |
|----------|--------|----------|
| Security Features | 3 | High |
| Verification Tasks | 3 | Medium |
| Enhanced Features | 1 | Medium |
| **TOTAL** | **7** | **Mixed** |

---

## 🚀 Implementation Order (Recommended)

1. **#24, #25, #26** - Verify existing functionality (Quick wins)
2. **#22** - Email verification (Foundation for other features)
3. **#21** - reCAPTCHA (Security enhancement)
4. **#23** - Password reset (Depends on email verification)
5. **#27** - Social media auth (Most complex, depends on email verification)

---

## 📝 Notes

- All email features require email service configuration (SMTP, SendGrid, etc.)
- reCAPTCHA requires Google reCAPTCHA account and API keys
- Social media OAuth requires app registration with each provider
- Admin settings panel needs to be created/updated for configuration management
- All features should support i18n (English/French)

---

**Last Updated:** January 27, 2026

