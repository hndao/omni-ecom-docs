# AI Documentation - Maintenance Guide

**Purpose:** Guide for maintaining AI-friendly documentation  
**Last Updated:** January 22, 2026

---

## 📚 Documentation Files

This folder contains **5 comprehensive AI-friendly documentation files** designed for AI assistants (Gemini, Claude, GPT, etc.) to quickly understand the OmniStock project:

| File | Lines | Purpose |
|------|-------|---------|
| **AI_PROJECT_OVERVIEW.md** | 387 | Business context, architecture, features, design patterns |
| **AI_TECH_STACK.md** | 433 | Complete technology stack, versions, dependencies |
| **AI_DATABASE_GUIDE.md** | 552 | Database schema, relationships, 43 tables |
| **AI_API_REFERENCE.md** | 399 | REST API endpoints (23 total) |
| **AI_CODE_CONVENTIONS.md** | 781 | Coding standards, best practices, patterns |
| **TOTAL** | **2,552** | **Complete project documentation** |

---

## ⚠️ CRITICAL: Documentation Maintenance Rules

### **Rule #1: Documentation MUST Stay in Sync with Code**

**Every code change that affects project structure, features, or architecture MUST be accompanied by documentation updates.**

### **Rule #2: Update Before Merging**

Documentation updates should be part of the same PR as the code changes, not done separately later.

### **Rule #3: AI Assistants Should Update Docs**

When an AI assistant makes changes to the codebase, it should automatically update the relevant documentation files.

---

## 🔄 When to Update Each File

### **AI_PROJECT_OVERVIEW.md**

Update when:
- ✅ New feature is added or removed
- ✅ Business domain changes (new entities, workflows)
- ✅ Architecture changes (new services, patterns)
- ✅ Multilingual support changes (new locales)
- ✅ Permission/role system changes
- ✅ Project status changes (completed features, new planned features)
- ✅ Design patterns are introduced or changed

**Sections to check:**
- Key Features (lines 60-90)
- Business Domain (lines 35-58)
- Multilingual Support (lines 92-110)
- Security & Permissions (lines 112-135)
- Current Status (lines 180-210)
- Key Design Patterns (lines 240-280)

---

### **AI_TECH_STACK.md**

Update when:
- ✅ Technology version upgraded (Laravel, PHP, Next.js, React, Node.js, etc.)
- ✅ New dependency added or removed
- ✅ Development tool added or changed
- ✅ Infrastructure changes (cloud services, CI/CD)
- ✅ Environment variables added or changed
- ✅ Performance optimization techniques added

**Sections to check:**
- Core Framework versions (lines 15-25)
- Complete Dependency List (lines 130-190)
- Database Technology (lines 192-210)
- Development Environment (lines 250-290)
- Environment Variables (lines 320-360)
- Technology Maturity & Support table (lines 400-410)

---

### **AI_DATABASE_GUIDE.md**

Update when:
- ✅ New table created
- ✅ Table schema modified (columns added/removed/changed)
- ✅ New relationship added
- ✅ Index added or removed
- ✅ Foreign key constraint changed
- ✅ Migration created or modified
- ✅ Database pattern changed

**Sections to check:**
- Table Groups Summary (lines 10-20)
- Core Tables (lines 22-150)
- EAV System (lines 152-200)
- E-Commerce Tables (lines 202-250)
- Multi-Channel Tables (lines 252-320)
- Content Management Tables (lines 322-360)
- Permission System (lines 362-400)
- Important Database Patterns (lines 520-552)

---

### **AI_API_REFERENCE.md**

Update when:
- ✅ New API endpoint added
- ✅ Existing endpoint modified (parameters, response format)
- ✅ Endpoint removed or deprecated
- ✅ Authentication method changed
- ✅ Rate limiting changed
- ✅ Error response format changed
- ✅ New query parameters added

**Sections to check:**
- API Overview (lines 10-15)
- Authentication Endpoints (lines 17-190)
- Products API (lines 192-280)
- Orders API (lines 282-350)
- Customer Addresses API (lines 352-380)
- Error Responses (lines 382-395)
- API Summary Table (lines 397-399)

---

### **AI_CODE_CONVENTIONS.md**

Update when:
- ✅ New coding pattern introduced
- ✅ Naming convention changed
- ✅ New security best practice added
- ✅ Testing strategy changed
- ✅ Git workflow changed
- ✅ Project structure changed
- ✅ UI/UX design system updated
- ✅ Performance optimization technique added

**Sections to check:**
- Laravel Naming Conventions (lines 80-150)
- Security Best Practices (lines 152-220)
- Database Best Practices (lines 222-260)
- TypeScript/Next.js Conventions (lines 262-360)
- Testing Conventions (lines 362-420)
- Git Commit Conventions (lines 422-460)
- UI/UX Conventions (lines 462-500)
- Project Structure (lines 720-781)

---

## 📝 How to Update Documentation

### **Step 1: Identify What Changed**

Review your code changes and identify which documentation files are affected.

### **Step 2: Update Relevant Sections**

Open the affected documentation files and update the relevant sections:
- Update version numbers if significant changes
- Update "Last Updated" date
- Add new sections if needed
- Update code examples to match current implementation
- Update counts (e.g., "23 endpoints" → "25 endpoints")

### **Step 3: Verify Accuracy**

- Read through the updated sections
- Ensure code examples are accurate
- Check that all cross-references are still valid
- Verify that the documentation is still AI-friendly (clear, comprehensive, no ambiguity)

### **Step 4: Update Version Numbers**

At the top of each modified file, update:
```markdown
**Last Updated:** January 22, 2026  
**Version:** 1.1  # Increment appropriately
```

**Version Increment Rules:**
- **Patch (1.0 → 1.1):** Minor updates, clarifications, typo fixes
- **Minor (1.1 → 1.2):** New sections, significant content additions
- **Major (1.0 → 2.0):** Complete restructuring or major architectural changes

---

## 🤖 AI Assistant Guidelines

When you (an AI assistant) make changes to the OmniStock codebase:

1. **Before making code changes:**
   - Read the relevant AI documentation files to understand current architecture
   - Follow the patterns and conventions documented

2. **After making code changes:**
   - Identify which documentation files need updates
   - Update the documentation in the same session
   - Increment version numbers appropriately
   - Update the "Last Updated" date

3. **When creating new features:**
   - Add documentation for the new feature in AI_PROJECT_OVERVIEW.md
   - Document any new API endpoints in AI_API_REFERENCE.md
   - Document any new database tables in AI_DATABASE_GUIDE.md
   - Document any new patterns in AI_CODE_CONVENTIONS.md

4. **When upgrading dependencies:**
   - Update version numbers in AI_TECH_STACK.md
   - Update the Technology Maturity & Support table
   - Update the Complete Dependency List

---

## ✅ Documentation Quality Checklist

Before considering documentation updates complete:

- [ ] All affected files have been updated
- [ ] Version numbers and dates are current
- [ ] Code examples match current implementation
- [ ] No broken cross-references
- [ ] Counts are accurate (endpoints, tables, permissions, etc.)
- [ ] New features are documented
- [ ] Deprecated features are removed or marked as deprecated
- [ ] Documentation is still AI-friendly (clear, comprehensive, structured)
- [ ] No sensitive information (API keys, passwords, etc.)

---

## 📊 Documentation Metrics

Track these metrics to ensure documentation stays comprehensive:

| Metric | Current Value | Notes |
|--------|---------------|-------|
| Total Documentation Lines | 2,552 | Across 5 files |
| API Endpoints Documented | 23 | Update when endpoints change |
| Database Tables Documented | 43 | Update when schema changes |
| Permissions Documented | 39 | Update when permissions change |
| Roles Documented | 4 | Update when roles change |
| Supported Locales | 2 (en, fr) | Update when locales added |
| Last Full Review | 2026-01-22 | Review quarterly |

---

## 🔍 Regular Maintenance Schedule

### **Weekly**
- Review recent PRs for documentation updates
- Verify documentation matches current codebase

### **Monthly**
- Check for outdated version numbers
- Review and update "Current Status" section in AI_PROJECT_OVERVIEW.md
- Update metrics in this README

### **Quarterly**
- Full documentation review
- Update all "Last Updated" dates
- Verify all code examples still work
- Check for new patterns or conventions to document

---

## 📞 Questions?

If you're unsure whether a change requires documentation updates, **err on the side of updating**. It's better to have over-documented than under-documented code.

---

**Remember: Good documentation is as important as good code!** 📚✨

