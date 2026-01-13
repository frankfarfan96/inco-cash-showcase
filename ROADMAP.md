# 🗺️ Inco.cash - Detailed Roadmap

> Feature specifications, timelines, and technical requirements

[← Back to Main README](./README.md) | [Technical Documentation →](./TECHNICAL.md)

---

## Current Status (January 2026)

### ✅ Production Features
- Multi-currency system (6 currencies: EUR, USD, COP, PEN, PLN, RUB)
- Real-time currency conversion with daily automated sync
- Custom 7-language internationalization (EN, ES, IT, FR, PL, RU, PT)
- JWT authentication with refresh tokens + automated cleanup
- 11 behavioral categories (6 liabilities + 5 assets)
- Unlimited user-defined subcategories
- Interactive data visualization (Nivo charts)
- Flyway database migrations
- Email-based password reset
- Theme customization
- **200+ active users across 6+ countries**

---

## Q1 2026 (January - March)

### 🚧 Budget Management System

**Target Completion:** End of March 2026

**Features:**

#### 1. Budget Creation
- Create monthly budgets
- Create annual budgets
- Budget templates (e.g., "50/30/20 rule")
- Copy budget from previous month

#### 2. Budget Allocation
- Allocate budget per category
- Allocate budget per subcategory
- Set percentage or fixed amount
- Multi-currency budget support

#### 3. Tracking & Monitoring
- Real-time budget vs actual comparison
- Visual progress bars per category
- Overspending indicators
- Underspending indicators

#### 4. Alerts & Notifications
- Email alert when 80% spent
- Email alert when 100% spent
- Weekly budget digest
- End-of-month summary

#### 5. Budget Analytics
- Monthly budget performance
- Category-wise analysis
- Trends over time
- Savings rate calculation

**Technical Implementation:**
```sql
CREATE TABLE budgets (
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  name VARCHAR(100),
  type ENUM('MONTHLY', 'ANNUAL'),
  start_date DATE,
  end_date DATE,
  created_at TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE budget_allocations (
  id BIGINT PRIMARY KEY,
  budget_id BIGINT NOT NULL,
  category_id BIGINT,
  subcategory_id BIGINT,
  allocated_amount DECIMAL(19,4),
  currency VARCHAR(3),
  FOREIGN KEY (budget_id) REFERENCES budgets(id),
  FOREIGN KEY (category_id) REFERENCES categories(id),
  FOREIGN KEY (subcategory_id) REFERENCES subcategories(id)
);
```

**UI Mockup:**
- Budget dashboard page
- Budget creation wizard
- Budget vs actual charts (Nivo)
- Category allocation interface

---

### 🚧 Import/Export Functionality

**Target Completion:** End of March 2026

**Features:**

#### 1. Export
- **CSV Export**
  - All transactions
  - Filtered transactions (date range, category)
  - Include converted amounts
  
- **Excel Export**
  - Multiple sheets (assets, liabilities, summary)
  - Charts included
  - Formatted tables

- **PDF Report**
  - Monthly financial report
  - Charts and visualizations
  - Professional formatting

#### 2. Import
- **CSV Import**
  - Bulk transaction import
  - Column mapping interface
  - Validation and error handling
  - Preview before import

- **Excel Import**
  - Support .xlsx and .xls
  - Multiple sheets support
  - Same features as CSV

#### 3. Data Portability
- Export all user data (GDPR compliance)
- Import from other financial apps
- Backup and restore functionality

**Technical Implementation:**
```java
@Service
public class ExportService {
    
    public byte[] exportToCSV(Long userId, ExportRequest request) {
        List<Transaction> transactions = getTransactions(userId, request);
        return CSVWriter.write(transactions);
    }
    
    public byte[] exportToExcel(Long userId, ExportRequest request) {
        List<Transaction> transactions = getTransactions(userId, request);
        return ExcelWriter.write(transactions);
    }
    
    public byte[] exportToPDF(Long userId, ExportRequest request) {
        FinancialReport report = generateReport(userId, request);
        return PDFGenerator.generate(report);
    }
}

@Service
public class ImportService {
    
    public ImportResult importFromCSV(Long userId, MultipartFile file) {
        List<TransactionDTO> transactions = CSVParser.parse(file);
        ValidationResult validation = validate(transactions);
        
        if (validation.hasErrors()) {
            return ImportResult.withErrors(validation.getErrors());
        }
        
        saveTransactions(userId, transactions);
        return ImportResult.success(transactions.size());
    }
}
```

---

## Q2 2026 (April - June)

### 📊 Stock Portfolio Tracking

**Target Completion:** End of June 2026

**Features:**

#### 1. Stock Management
- Add stocks to portfolio
- Manual entry or API integration
- Track quantity and purchase price
- Multiple portfolios support

#### 2. Real-Time Prices
- Integration with stock price API
- Automatic daily price updates
- Historical price charts

#### 3. Portfolio Analytics
- Total portfolio value
- Gain/loss per stock
- Gain/loss percentage
- Diversification analysis

#### 4. Integration with Assets
- Stocks linked to "Investor" category
- Dividends tracked as income
- Capital gains/losses tracked

**Technical Stack:**
- Stock price API: Alpha Vantage or Yahoo Finance
- New scheduled task for price updates
- New database tables for stocks

**Database Schema:**
```sql
CREATE TABLE stocks (
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  symbol VARCHAR(10) NOT NULL,
  name VARCHAR(100),
  quantity DECIMAL(19,4),
  purchase_price DECIMAL(19,4),
  purchase_date DATE,
  currency VARCHAR(3),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE stock_prices (
  id BIGINT PRIMARY KEY,
  symbol VARCHAR(10) NOT NULL,
  price DECIMAL(19,4),
  date DATE,
  INDEX idx_symbol_date (symbol, date DESC)
);
```

---

### 🎯 Mission System (Financial Goals & Gamification)

**Target Completion:** End of June 2026

**Features:**

#### 1. Financial Goals
- Set savings goals
- Set debt payoff goals
- Set investment goals
- Custom goals

#### 2. Progress Tracking
- Visual progress bars
- Percentage complete
- Time remaining
- Amount remaining

#### 3. Gamification Elements
- **Achievements:**
  - "First 1000 saved"
  - "30-day streak"
  - "Budget master" (under budget 3 months)
  - "Category conqueror" (all categories used)
  
- **Levels:**
  - Level 1: Beginner (0-1000 total tracked)
  - Level 2: Novice (1000-5000)
  - Level 3: Intermediate (5000-10000)
  - Level 4: Advanced (10000-50000)
  - Level 5: Master (50000+)

- **Rewards:**
  - Badges displayed on profile
  - Unlock special themes
  - Early access to new features

#### 4. Milestones
- Celebrate when goals reached
- Share achievements (optional)
- Goal completion history

**Technical Implementation:**
```sql
CREATE TABLE goals (
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  name VARCHAR(100),
  target_amount DECIMAL(19,4),
  current_amount DECIMAL(19,4) DEFAULT 0,
  currency VARCHAR(3),
  target_date DATE,
  category_id BIGINT,
  status ENUM('ACTIVE', 'COMPLETED', 'ABANDONED'),
  created_at TIMESTAMP,
  completed_at TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (category_id) REFERENCES categories(id)
);

CREATE TABLE achievements (
  id BIGINT PRIMARY KEY,
  code VARCHAR(50) UNIQUE,
  name VARCHAR(100),
  description TEXT,
  icon_url VARCHAR(255)
);

CREATE TABLE user_achievements (
  user_id BIGINT,
  achievement_id BIGINT,
  earned_at TIMESTAMP,
  PRIMARY KEY (user_id, achievement_id),
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (achievement_id) REFERENCES achievements(id)
);
```

**Achievement Engine:**
```java
@Service
public class AchievementService {
    
    @EventListener
    public void onTransactionCreated(TransactionCreatedEvent event) {
        checkAchievements(event.getUser());
    }
    
    private void checkAchievements(User user) {
        // Check "First transaction"
        if (countTransactions(user) == 1) {
            awardAchievement(user, "FIRST_TRANSACTION");
        }
        
        // Check "30-day streak"
        if (hasConsecutiveDays(user, 30)) {
            awardAchievement(user, "STREAK_30");
        }
        
        // ... more achievement checks
    }
}
```

---

### 🇩🇪 German Language Support

**Target Completion:** End of June 2026

**Features:**
- Complete translation of 5,000+ entries
- German number formatting
- German date formatting
- German currency symbols

**Process:**
1. Extract all translation keys
2. Professional translation service
3. Native speaker review
4. QA testing
5. Deploy

---

## Q3 2026 (July - September)

### 📱 Mobile Application

**Target Completion:** End of September 2026

**Platform:** React Native (iOS + Android)

**Features:**

#### Core Functionality
- All web features available
- Offline mode support
- Sync when online
- Touch ID / Face ID authentication

#### Mobile-Specific
- Push notifications
  - Budget alerts
  - Daily reminders
  - Goal progress
- Camera integration
  - Receipt scanning (OCR)
  - Profile picture
- Native sharing
- App shortcuts

**Technical Stack:**
- React Native
- Redux (shared state logic)
- AsyncStorage (offline data)
- React Navigation
- Push notifications: Firebase Cloud Messaging

---

### 🧠 AI-Powered Insights

**Target Completion:** End of September 2026

**Features:**

#### 1. Spending Predictions
- Predict next month spending by category
- Identify unusual spending patterns
- Forecast future balance

#### 2. Smart Categorization
- Auto-suggest category for transactions
- Learn from user's past categorization
- Improve over time

#### 3. Personalized Recommendations
- "You could save X by reducing Y"
- "Consider moving money from Z to investing"
- Budget optimization suggestions

#### 4. Anomaly Detection
- Alert on unusually high spending
- Detect duplicate transactions
- Identify potential fraudulent activity

**Technical Stack:**
- Python microservice for ML models
- scikit-learn or TensorFlow
- Time series analysis
- Natural language processing (for transaction descriptions)

---

## Q4 2026 (October - December)

### 🔐 Social Login

**Target Completion:** October 2026

**Providers:**
- Google OAuth
- Facebook Login
- Apple Sign In

**Implementation:**
```java
@Service
public class SocialAuthService {
    
    public AuthenticationResponse authenticateWithGoogle(String googleToken) {
        GoogleUser googleUser = googleClient.verifyToken(googleToken);
        
        User user = userRepository
            .findByEmail(googleUser.getEmail())
            .orElseGet(() -> createUserFromGoogle(googleUser));
        
        return generateTokens(user);
    }
}
```

---

### 🏦 Bank Integration

**Target Completion:** December 2026

**Features:**

#### 1. Account Linking
- Connect bank accounts
- Secure OAuth 2.0 authentication
- Support major European banks

#### 2. Automatic Transaction Import
- Daily sync of new transactions
- Smart categorization
- Duplicate detection

#### 3. Balance Sync
- Real-time account balance
- Multiple account support
- Net worth calculation

**Technical Integration:**
- Use Open Banking APIs (PSD2 in Europe)
- Or integrate with Plaid/TrueLayer
- Encryption for sensitive data

---

### 📸 Receipt Scanning (OCR)

**Target Completion:** December 2026

**Features:**
- Take photo of receipt
- Automatically extract:
  - Amount
  - Date
  - Merchant name
  - Line items
- Pre-fill transaction form
- Attach receipt image to transaction

**Technical Stack:**
- OCR engine: Google Cloud Vision API or Tesseract
- Image preprocessing
- Text extraction and parsing

---

## 2027 & Beyond

### Advanced Features

#### 1. Cryptocurrency Tracking
- Bitcoin, Ethereum, and major altcoins
- Real-time price tracking
- Portfolio management
- Tax reporting

#### 2. Tax Preparation
- Year-end tax summary
- Deduction identification
- Export for tax software
- Multi-country support

#### 3. Investment Analysis
- Stock portfolio optimization
- Risk assessment
- Diversification recommendations
- What-if scenarios

#### 4. Collaborative Features
- Shared budgets (couples, families)
- Expense splitting
- Group goals
- Household dashboard

#### 5. Advanced Reporting
- Custom report builder
- Scheduled email reports
- Export to Google Sheets
- API for third-party integrations

---

## Feature Prioritization Framework

**How features are prioritized:**

1. **User Requests** (40%)
   - Most requested features from user feedback
   - Survey results
   - Support tickets

2. **Strategic Value** (30%)
   - Aligns with product vision
   - Competitive advantage
   - Market demand

3. **Technical Feasibility** (20%)
   - Development effort
   - Technical debt impact
   - Maintenance burden

4. **Business Impact** (10%)
   - Revenue potential (future monetization)
   - User retention
   - User acquisition

---

## How to Contribute Ideas

Have a feature suggestion? Here's how:

1. **Live Feedback:** Use the feedback form in the app
2. **Email:** [frankfarfan96@gmail.com]
3. **LinkedIn:** [linkedin.com/in/francesco-farfan](https://www.linkedin.com/in/francesco-farfan-88857b232/)

All feedback is read and considered for future development!

---

[← Back to Main README](./README.md) | [Technical Documentation →](./TECHNICAL.md)
