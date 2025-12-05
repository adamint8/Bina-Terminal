# API Endpoint Inventory - Fincept Terminal

**Purpose:** Complete inventory of all API endpoints and external service calls in the codebase.

## Fincept Backend API Endpoints (TO BE REPLACED)

**Base URL:** `https://finceptbackend.share.zrok.io`

### Authentication Endpoints

| Endpoint | Method | File | Purpose | Replace With |
|----------|--------|------|---------|--------------|
| `/api/v1/auth/register` | POST | `src/services/authApi.tsx:110` | User registration | Supabase Auth |
| `/api/v1/auth/login` | POST | `src/services/authApi.tsx:140` | User login | Supabase Auth |
| `/api/v1/auth/verify-otp` | POST | `src/services/authApi.tsx:170` | OTP verification | Supabase Auth MFA |
| `/api/v1/auth/resend-otp` | POST | `src/services/authApi.tsx:190` | Resend OTP | Supabase Auth |
| `/api/v1/auth/forgot-password` | POST | `src/services/authApi.tsx:210` | Password reset request | Supabase Auth |
| `/api/v1/auth/reset-password` | POST | `src/services/authApi.tsx:230` | Password reset | Supabase Auth |
| `/api/v1/auth/logout` | POST | `src/services/authApi.tsx:250` | Logout | Supabase Auth |
| `/api/v1/auth/status` | GET | `src/services/authApi.tsx:270` | Auth status check | Supabase Session |
| `/api/v1/auth/device-register` | POST | `src/services/authApi.tsx:290` | Guest device | Remove/Replace |

### User Management Endpoints

| Endpoint | Method | File | Purpose | Replace With |
|----------|--------|------|---------|--------------|
| `/api/v1/users/profile` | GET | `src/services/userApi.tsx` | Get user profile | Supabase DB query |
| `/api/v1/users/profile` | PUT | `src/services/userApi.tsx` | Update profile | Supabase DB update |
| `/api/v1/users/api-keys` | GET | `src/services/userApi.tsx` | List API keys | Supabase Vault |
| `/api/v1/users/api-keys` | POST | `src/services/userApi.tsx` | Create API key | Supabase Vault |
| `/api/v1/users/api-keys/:id` | DELETE | `src/services/userApi.tsx` | Delete API key | Supabase Vault |

### Payment Endpoints (TO BE REMOVED)

| Endpoint | Method | File | Purpose | Action |
|----------|--------|------|---------|--------|
| `/api/v1/payments/plans` | GET | `src/services/paymentApi.tsx` | List subscription plans | Remove |
| `/api/v1/payments/create-checkout` | POST | `src/services/paymentApi.tsx` | Create checkout session | Remove |
| `/api/v1/payments/verify` | GET | `src/services/paymentApi.tsx` | Verify payment | Remove |
| `/api/v1/payments/subscription` | GET | `src/services/paymentApi.tsx` | Get subscription | Remove |
| `/api/v1/payments/cancel` | POST | `src/services/paymentApi.tsx` | Cancel subscription | Remove |

### Forum Endpoints (TO BE REMOVED)

| Endpoint | Method | File | Purpose | Action |
|----------|--------|------|---------|--------|
| `/api/v1/forum/posts` | GET | `src/services/forumApi.tsx` | List posts | Remove |
| `/api/v1/forum/posts` | POST | `src/services/forumApi.tsx` | Create post | Remove |
| `/api/v1/forum/posts/:id` | GET | `src/services/forumApi.tsx` | Get post | Remove |
| `/api/v1/forum/posts/:id/comments` | GET | `src/services/forumApi.tsx` | Get comments | Remove |

### Support Endpoints

| Endpoint | Method | File | Purpose | Replace With |
|----------|--------|------|---------|--------------|
| `/api/v1/support/tickets` | GET | `src/services/supportApi.tsx` | List tickets | Supabase DB + n8n |
| `/api/v1/support/tickets` | POST | `src/services/supportApi.tsx` | Create ticket | Supabase DB + n8n |
| `/api/v1/support/tickets/:id` | GET | `src/services/supportApi.tsx` | Get ticket | Supabase DB |

### Marketplace Endpoints (TO BE REMOVED)

| Endpoint | Method | File | Purpose | Action |
|----------|--------|------|---------|--------|
| `/api/v1/marketplace/items` | GET | `src/services/marketplaceApi.tsx` | List items | Remove |
| `/api/v1/marketplace/purchase` | POST | `src/services/marketplaceApi.tsx` | Purchase item | Remove |

## Third-Party API Endpoints (KEEP & CONFIGURE)

### Financial Data APIs

| Service | Endpoint Pattern | File | Configuration Needed |
|---------|-----------------|------|---------------------|
| Alpha Vantage | `https://www.alphavantage.co/query` | `src/services/alphaVantageService.ts` | API Key in env |
| Polygon.io | `https://api.polygon.io/v2/*` | `src/services/polygonService.ts` | API Key in env |
| Yahoo Finance | Various | `src/services/yfinanceService.ts` | Public API |
| FRED | `https://api.stlouisfed.org/*` | Rust commands | API Key in env |

### Broker APIs

| Service | Endpoint Pattern | File | Configuration Needed |
|---------|-----------------|------|---------------------|
| Fyers | `https://api-t1.fyers.in/*` | `src/services/fyersService.ts` | User credentials |
| Fyers | `https://api-t2.fyers.in/*` | `src/services/fyersAuth.ts` | User credentials |
| Zerodha Kite | `https://api.kite.trade/*` | `src/stockBrokers/india/zerodhaKite/*` | User credentials |
| Kraken | `https://api.kraken.com/*` | Implied in KrakenTab | User credentials |
| Hyperliquid | `https://api.hyperliquid.xyz/*` | `src/services/hyperliquid/*` | User credentials |

### LLM Provider APIs

| Service | Endpoint Pattern | File | Configuration Needed |
|---------|-----------------|------|---------------------|
| OpenAI | `https://api.openai.com/v1/*` | `src/services/llmApi.ts` | API Key in env |
| Anthropic | `https://api.anthropic.com/v1/*` | `src/services/llmApi.ts` | API Key in env |
| DeepSeek | `https://api.deepseek.com/*` | `src/services/llmApi.ts` | API Key in env |
| OpenRouter | `https://openrouter.ai/api/v1/*` | `src/services/llmApi.ts` | API Key in env |
| Google Gemini | `https://generativelanguage.googleapis.com/*` | `src/services/llmApi.ts` | API Key in env |
| Ollama | `http://localhost:11434/*` | `src/services/ollamaService.ts` | Local server |

### News & RSS Feeds

| Source | URL | File | Configuration Needed |
|--------|-----|------|---------------------|
| Yahoo Finance News | `https://finance.yahoo.com/news/rssindex` | `src/services/newsService.ts` | Move to config |
| Investing.com | `https://www.investing.com/rss/news.rss` | `src/services/newsService.ts` | Move to config |
| CoinDesk | `https://www.coindesk.com/arc/outboundfeeds/rss/` | `src/services/newsService.ts` | Move to config |
| CoinTelegraph | `https://cointelegraph.com/rss` | `src/services/newsService.ts` | Move to config |
| Decrypt | `https://decrypt.co/feed` | `src/services/newsService.ts` | Move to config |
| TechCrunch | `https://techcrunch.com/feed/` | `src/services/newsService.ts` | Move to config |
| The Verge | `https://www.theverge.com/rss/index.xml` | `src/services/newsService.ts` | Move to config |
| Ars Technica | `https://feeds.arstechnica.com/arstechnica/index` | `src/services/newsService.ts` | Move to config |
| OilPrice | `https://oilprice.com/rss/main` | `src/services/newsService.ts` | Move to config |

## Python Script API Dependencies

**Location:** `src-tauri/resources/scripts/`

Scripts that need API endpoint review:

| Script | Potential External Dependencies | Action Required |
|--------|-------------------------------|-----------------|
| `portfolio_analytics_service.py` | Unknown - needs audit | Review for Fincept APIs |
| `financial_report_generator.py` | Unknown - needs audit | Review for Fincept APIs |
| All other scripts | Various public APIs | Verify no Fincept dependencies |

## Rust Command API Calls

**Location:** `src-tauri/src/commands/`

Commands that may have external API calls:

| Command Module | Potential APIs | Action Required |
|---------------|---------------|-----------------|
| `ai_agents.rs` | LLM APIs | Verify endpoint configuration |
| `market_data.rs` | Market data APIs | Verify endpoint configuration |
| `analytics.rs` | Analytics services | Verify no Fincept dependencies |

## Migration Priority Matrix

### Priority 1: Critical (Blocks Application)
- [ ] All authentication endpoints
- [ ] User profile endpoints
- [ ] Session management

### Priority 2: High (Major Features)
- [ ] Support ticket endpoints
- [ ] User data storage

### Priority 3: Medium (Can Disable Feature)
- [ ] Forum endpoints (can remove feature)
- [ ] Marketplace endpoints (can remove feature)

### Priority 4: Low (Payment Related)
- [ ] Payment endpoints (can remove feature)
- [ ] Subscription endpoints (can remove feature)

## Configuration Strategy Summary

1. **Replace with Supabase:** All authentication, user management, data storage
2. **Remove Entirely:** Payment, marketplace, forum
3. **Make Configurable:** All third-party APIs (env variables)
4. **Audit & Verify:** Python scripts and Rust commands
5. **Move to Registry:** RSS feeds and data sources

---

**Document Status:** Complete  
**Last Updated:** December 2025  
**Related Documents:** BINA_CUSTOMIZATION_BLUEPRINT.md

