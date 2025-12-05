# Bina Analytics - Quick Reference Guide

**Quick links for developers and agents working on the customization project.**

## 📚 Core Documentation

- **[Main Blueprint](./BINA_CUSTOMIZATION_BLUEPRINT.md)** - Complete customization roadmap (50KB, 1666 lines)
- **[API Inventory](./API_ENDPOINT_INVENTORY.md)** - All API endpoints and dependencies
- **[Environment Config](../.env.example)** - Environment variable reference

## 🎯 Critical File Locations

### Configuration Files
```
fincept-terminal-desktop/src-tauri/tauri.conf.json    # App metadata
fincept-terminal-desktop/package.json                  # Package name
fincept-terminal-desktop/src-tauri/Cargo.toml         # Rust package
fincept-terminal-desktop/src/App.css                  # Theme & colors
```

### Core Application Files
```
src/App.tsx                                            # Main app router (350 lines)
src/components/dashboard/DashboardScreen.tsx          # Dashboard (800+ lines)
src/contexts/AuthContext.tsx                          # Auth state (420 lines)
```

### Services to Replace
```
src/services/authApi.tsx                              # Auth API (800 lines) → Supabase
src/services/paymentApi.tsx                           # Payment → Remove
src/services/userApi.tsx                              # User API → Supabase
src/services/forumApi.tsx                             # Forum → Remove
src/services/supportApi.tsx                           # Support → Supabase + n8n
src/services/marketplaceApi.tsx                       # Marketplace → Remove
```

## 🔍 Quick Search Commands

### Find Fincept References
```bash
cd fincept-terminal-desktop
grep -r "Fincept" --include="*.tsx" --include="*.ts" --include="*.json"
grep -r "finceptbackend" --include="*.tsx" --include="*.ts"
```

### Find Hardcoded URLs
```bash
grep -r "https://" --include="*.ts" --include="*.tsx" | grep -v "node_modules"
```

### Find Color References
```bash
grep -r "#[0-9A-Fa-f]\{6\}" --include="*.tsx" --include="*.css"
grep -r "rgba(" --include="*.tsx" --include="*.css"
```

### Count Components
```bash
find src/components/tabs -name "*.tsx" | wc -l        # 112 tabs
find src/components/auth -name "*.tsx" | wc -l        # 5 auth screens
find src/components/payment -name "*.tsx" | wc -l     # 4 payment screens
```

## 📊 Stats at a Glance

- **Total TypeScript Code:** 122,662 lines
- **Tab Components:** 112
- **Auth Screens:** 5
- **Payment Screens:** 4 (to remove)
- **Service Files:** 40+
- **Python Scripts:** 30+
- **Rust Commands:** 20+

## 🚀 Quick Start Tasks

### Phase 1: Easy Wins (Day 1)
1. Update branding text in UI components
2. Replace logo assets in `src-tauri/icons/`
3. Update `tauri.conf.json` metadata
4. Update `package.json` and `Cargo.toml` names
5. Create `.env` from `.env.example`

### Phase 2: Critical Path (Week 1-2)
1. Set up Supabase project
2. Create configuration system (`src/config/`)
3. Document current API contracts
4. Set up feature flags

### Phase 3: Backend Migration (Week 3-6)
1. Replace authentication
2. Migrate user data
3. Remove payment features
4. Update data services

## 🔑 Key Environment Variables

**Required for Basic Operation:**
```bash
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_APP_NAME=Bina Analytics Control Center
```

**Feature Flags to Disable:**
```bash
VITE_ENABLE_MARKETPLACE=false
VITE_ENABLE_PAYMENT=false
VITE_ENABLE_FORUM=false
VITE_ENABLE_PRICING_SCREEN=false
```

## 🎨 Branding Quick Reference

### Colors to Change
```css
/* Current Fincept Orange */
--accent: oklch(...);                    /* App.css line 59 */
rgba(255, 165, 0, 0.2)                   /* App.css line 135 */

/* Replace with Bina Colors */
--primary: #0052A3    /* Bina Blue */
--accent: #FF6B00     /* Bina Orange */
--secondary: #00B894  /* Bina Teal */
```

### Assets to Replace
```
src-tauri/icons/icon.png      # 128x128 app icon
src-tauri/icons/icon.ico      # Windows icon
src-tauri/icons/icon.icns     # macOS icon
public/favicon.ico            # Browser favicon
```

## 🗑️ Safe to Delete

### Complete Features
- `src/components/payment/` (all 4 files)
- `src/services/paymentApi.tsx`
- `src/services/forumApi.tsx`
- `src/services/marketplaceApi.tsx`
- `src/components/tabs/ForumTab.tsx`
- `src/components/tabs/MarketplaceTab.tsx`

### Code Sections in App.tsx
- Lines 45-66: Payment window state
- Lines 233-274: Payment screen renderer
- All payment navigation logic

## 🧪 Testing Checklist

### After Auth Migration
- [ ] User registration works
- [ ] Login works
- [ ] Password reset works
- [ ] Session persists
- [ ] Logout works

### After Branding Update
- [ ] Logo displays correctly
- [ ] No "Fincept" text visible
- [ ] Colors match brand
- [ ] Dark mode works
- [ ] App title correct

### After Feature Removal
- [ ] No payment references
- [ ] No forum references
- [ ] No marketplace references
- [ ] Navigation works
- [ ] No broken links

## 🔒 Security Checklist

- [ ] No API keys in code
- [ ] All secrets in environment variables
- [ ] Supabase RLS policies configured
- [ ] CORS properly configured
- [ ] No hardcoded credentials
- [ ] Auth tokens properly managed

## 📈 Progress Tracking

Use this checklist to track migration progress:

### Configuration (Week 1-2)
- [ ] Environment variables set up
- [ ] Feature flags implemented
- [ ] Supabase project created
- [ ] Configuration files created

### Branding (Week 3)
- [ ] Logo assets replaced
- [ ] Colors updated
- [ ] Text updated
- [ ] Legal pages updated

### Backend (Week 4-6)
- [ ] Auth migrated to Supabase
- [ ] User data migrated
- [ ] Payment features removed
- [ ] API services updated

### Data Layer (Week 8-11)
- [ ] Database schema created
- [ ] Data migration complete
- [ ] Services updated
- [ ] Sync implemented

### Architecture (Week 15-18)
- [ ] Folder structure updated
- [ ] Large components split
- [ ] Imports updated
- [ ] Performance optimized

### Testing (Week 19-20)
- [ ] Test suite added
- [ ] All features tested
- [ ] Security audit done
- [ ] Documentation complete

## 🆘 Common Issues & Solutions

### Issue: Import errors after restructure
**Solution:** Update `tsconfig.json` path aliases

### Issue: Environment variables not loading
**Solution:** Restart dev server, check `.env` file name

### Issue: Supabase connection fails
**Solution:** Verify URL and anon key, check CORS settings

### Issue: Build fails after removing features
**Solution:** Search for all imports of removed files, clean up

### Issue: Dark mode colors wrong
**Solution:** Update both `:root` and `.dark` CSS variables

## 📞 Getting Help

1. Check main blueprint: [BINA_CUSTOMIZATION_BLUEPRINT.md](./BINA_CUSTOMIZATION_BLUEPRINT.md)
2. Review API inventory: [API_ENDPOINT_INVENTORY.md](./API_ENDPOINT_INVENTORY.md)
3. Check environment template: [.env.example](../.env.example)
4. Search codebase for examples
5. Consult Supabase documentation

---

**Last Updated:** December 2025  
**Maintained By:** Engineering Team

