# 💰 Inco.cash - Smart Financial Management Platform

> Multi-language, multi-currency financial tracking system serving 200+ users across 6 countries

🔗 **[inco.cash](https://inco.cash)** | 👥 200+ Users | 🌍 7 Languages | 💱 6 Currencies | 📊 Production

---

## 📸 Platform Overview

### Dashboard - Monthly Balance Overview
![Dashboard](./screenshots/dashboard.png)
*Real-time balance tracking with quick-add functionality and visual diagrams*

### Liabilities (Expenses) - Category Analysis
![Liabilities](./screenshots/liabilities.png)
*Comprehensive expense tracking with interactive charts and subcategory breakdown*

### Assets (Income) - Cash Flow Quadrants
![Assets](./screenshots/assets.png)
*Income management based on Robert Kiyosaki's Cash Flow Quadrant methodology*

### Multi-Currency Support
![Currency](./screenshots/currency.png)
*Real-time currency conversion with automatic daily exchange rate updates*

### Profile & Settings
![Settings](./screenshots/settings.png)
*Theme customization and language selection across 7 languages*

---

## ⚡ What Makes Inco.cash Special

### 🌍 **Truly International**
- **7 languages** (EN, ES, IT, FR, PL, RU, PT) with custom-built i18n system (5,000+ translations)
- **6 currencies** with daily automated exchange rates (EUR, USD, COP, PEN, PLN, RUB)
- Real-time currency conversion at transaction time

### 💡 **Behavioral Design**
- **11 strategic categories** designed to promote better financial habits
- Based on Robert Kiyosaki's Cash Flow Quadrant for income tracking
- Categories encourage awareness: "For Me" vs "Superfluous Things"

### 🎨 **User-Centric Experience**
- Theme customization
- Interactive Nivo charts (pie, bar, balance diagrams)
- Responsive design for all devices
- Unlimited custom subcategories

### 🔐 **Production-Grade Engineering**
- JWT authentication with refresh tokens
- Flyway database migrations
- Automated daily schedulers (exchange rates + token cleanup)
- Email-based password recovery
- 99.5% uptime serving 200+ users

---

## 🛠️ Tech Stack

**Frontend:** React (Vanilla JS) • Redux • Nivo • CSS Modules • Custom i18n (5,000+ translations)

**Backend:** Java 17 • Spring Boot 3.x • MySQL • Flyway Migrations • JWT • Scheduled Tasks

**Infrastructure:** Contabo VPS • SSL/HTTPS • Automated Backups • Daily Schedulers

**Integrations:** Currency Exchange API (6 pairs) • Email Service (SMTP)

---

## 💰 Category System

### Liabilities (6 Categories)
Designed to motivate improvement and financial awareness:

1. **For Me** 🏠 - Essential survival expenses (rent, transport, internet)
2. **Improve Myself** 📚 - Self-improvement investments (courses, gym, therapy)
3. **Dolce Vita** ✈️ - Pleasure expenses that make life sweet (travel, concerts, spa)
4. **Actives** 💻 - Income-generating purchases (laptop, tools, advertising)
5. **Superfluous Things** 🚬 - Unnecessary, potentially harmful (alcohol, cigarettes, gambling)
6. **For Others** 🎁 - Expenses toward others (gifts, donations, pets)

### Assets (5 Categories - Cash Flow Quadrant)
Based on Robert Kiyosaki's "The Cashflow Quadrant":

1. **Employee (E)** 💵 - Employment income (salary, bonuses, overtime)
2. **Freelancer (S)** 📝 - Self-employed income (projects, consulting, design)
3. **Entrepreneur (B)** 🛍️ - Business owner income (sales, services, subscriptions)
4. **Investor (I)** 💹 - Investment returns (dividends, rents, interest)
5. **Others** 🏆 - Extraordinary income (refunds, prizes, gifts)

**Each category supports unlimited user-defined subcategories.**

---

## 🚀 Key Features

### Financial Tracking
- Multi-currency transactions with real-time conversion (6 supported currencies)
- Three-section view per page: transaction table, visual charts, subcategory analysis
- Monthly balance tracking with interactive diagrams

### Data Visualization
- Interactive Nivo charts (pie, bar)
- Real-time balance diagrams
- Category breakdown with percentages and amounts
- Toggle between chart types

### Internationalization
- 7 languages without third-party libraries
- 5,000+ custom translations
- Modular architecture for easy expansion
- German coming soon (8th language)

### Multi-Currency System
- **6 currencies with full exchange support:**
  - 🇪🇺 EUR (Euro) - Italy, Poland
  - 🇺🇸 USD (US Dollar) - USA
  - 🇨🇴 COP (Colombian Peso) - Colombia
  - 🇵🇪 PEN (Peruvian Sol) - Peru
  - 🇵🇱 PLN (Polish Zloty) - Poland
  - 🇷🇺 RUB (Russian Ruble) - Russia
- Daily automated exchange rate synchronization
- Real-time conversion at transaction time
- Historical accuracy maintained

### Security & Authentication
- JWT with refresh tokens (15min access, 7 day refresh)
- Automated daily token cleanup
- BCrypt password hashing
- Email-based password reset
- User data isolation

---

## 🗺️ Roadmap

### ✅ Completed
- Multi-currency system with daily automated sync (6 currencies)
- Custom 7-language internationalization (5,000+ translations)
- JWT authentication with refresh token mechanism + automated cleanup
- 11 behavioral categories (6 liabilities + 5 assets) with unlimited subcategories
- Interactive data visualization with Nivo
- Flyway database migrations
- Email-based password reset
- Theme customization
- 200+ active users across 6+ countries

### 🚧 In Progress (Q1 2026)
- **Budget Management System** - Create/manage budgets, track vs actual, alerts
- **Import/Export** - CSV/Excel/PDF export, bulk transaction imports

### 📋 Coming Next (Q2 2026)
- **Stock Portfolio Tracking** - Investment tracking with real-time prices
- **Mission System** - Financial goals with gamification elements
- **German Language** - 8th language support

### 🔮 Future Vision (Q3-Q4 2026)
- Mobile app (React Native)
- AI-powered insights & predictions
- Social login (Google, Facebook, Apple)
- Bank integration & automatic imports

**[View Detailed Roadmap →](./ROADMAP.md)**

---

## 💡 Technical Highlights

### Custom Internationalization System
Built from scratch without third-party libraries. 5,000+ translations managed in custom architecture, supporting 7 languages with German in development.

### Multi-Currency Architecture
Daily automated scheduler syncs exchange rates for 6 currency pairs. Stores both original and converted values for historical accuracy.

### Production Engineering
- **Flyway** for database version control and safe migrations
- **Automated schedulers**: Exchange rate sync (2 AM) + Token cleanup (3 AM)
- **JWT refresh token strategy** balancing security with user experience
- **99.5% uptime** serving 200+ international users

### Behavioral Design Philosophy
Categories based on financial education principles (Cash Flow Quadrant) to promote awareness, encourage better habits, and motivate improvement.

---

## 🎯 Why This Project Matters

This is **not a demo or tutorial project** - it's a production application serving 200+ real users managing real finances daily across 6 countries and 7 languages.

**Demonstrates:**

✅ **Full-Stack Ownership** - Complete control from database design to UI implementation

✅ **Complex Problem-Solving** - Multi-currency conversion, custom i18n system, behavioral category design

✅ **Production Engineering** - Flyway migrations, automated schedulers, JWT security, email integration

✅ **Real-World Scale** - 200+ users, international distribution, continuous uptime

✅ **Continuous Development** - Active roadmap with Budget, Stock, Mission systems planned

---

## 📚 Deep Dive Documentation

Want to explore the technical implementation in depth?

### 📖 [Complete Technical Documentation →](./TECHNICAL.md)

Detailed coverage of:
- System architecture diagrams
- Database schema design
- API endpoint structure
- Code organization patterns
- Development challenges & solutions
- Security implementation details
- Performance metrics & optimization
- Frontend architecture (Redux, Nivo, CSS Modules)
- Backend implementation (Spring Boot, Schedulers, JWT)

### 🗺️ [Detailed Roadmap & Features →](./ROADMAP.md)

Complete feature specifications:
- Timeline and priorities
- Technical requirements
- Implementation details
- Budget Management System specs
- Stock Portfolio Tracking design
- Mission System gamification
- Import/Export functionality

### 🌐 [Try Live Application →](https://inco.cash)

Explore the platform yourself:
- Available in 7 languages
- Create account and test features
- Experience multi-currency conversion
- View all 11 categories
- Interactive charts and visualizations

---

## 💬 Let's Talk

Interested in discussing the technical implementation, architecture decisions, or seeing a detailed walkthrough?

📧 **Email:** [your-email@example.com]  
💼 **LinkedIn:** [linkedin.com/in/francesco-farfan](https://www.linkedin.com/in/francesco-farfan-88857b232/)  
🌐 **Live Demo:** [inco.cash](https://inco.cash)

---

## 📝 About This Repository

This is a **portfolio showcase** for a production financial management application that I've architected, built, and maintained independently.

**The codebase is private to:**
- Protect intellectual property
- Secure user data and business logic
- Maintain future monetization options

**During interviews, I'm happy to:**
- Walk through the live application in any of 7 languages
- Explain architecture decisions and trade-offs
- Discuss code organization and design patterns
- Share isolated code samples
- Detail production challenges and solutions

---

**Built with passion and precision by Francesco Farfan**

*From concept to production • From 1 language to 7 • From 1 currency to 6 • From 1 user to 200+*

*Still growing • Still improving • Still coding* 🚀
