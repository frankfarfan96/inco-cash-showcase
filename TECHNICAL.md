# 📖 Inco.cash - Technical Documentation

> Deep dive into architecture, implementation details, and engineering decisions

[← Back to Main README](./README.md)

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Tech Stack Deep Dive](#tech-stack-deep-dive)
3. [Database Design](#database-design)
4. [Backend Architecture](#backend-architecture)
5. [Frontend Architecture](#frontend-architecture)
6. [Redux Store Architecture](#redux-store-architecture)
7. [Multi-Currency System](#multi-currency-system)
8. [Internationalization System](#internationalization-system)
9. [Security & Encryption](#security--encryption)
10. [Scheduled Tasks](#scheduled-tasks)
11. [Code Structure](#code-structure)
12. [Development Challenges](#development-challenges)
13. [Performance & Optimization](#performance--optimization)
14. [Production Metrics](#production-metrics)

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────┐
│      React Frontend (Vanilla JS)    │
│  ┌────────────────────────────────┐ │
│  │  Redux Store                   │ │
│  │  - Multi-layer caching         │ │
│  │  - Month-based history         │ │
│  │  - Intelligent data reuse      │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │  Custom i18n System (5000+)    │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │  Nivo Charts                   │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │  Currency Conversion           │ │
│  │  (Client-side)                 │ │
│  └────────────────────────────────┘ │
└──────────────┬──────────────────────┘
               │ Axios HTTP
               │ JWT Bearer Token
               ▼
┌──────────────────────────────────────┐
│     Spring Boot REST API             │
│  ┌────────────────────────────────┐  │
│  │  JWT Authentication            │  │
│  │  - Access Token (7 days)       │  │
│  │  - Refresh Token (30 days)     │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  Exchange Rate Service         │  │
│  │  - Serves rates to frontend    │  │
│  │  - Daily sync (14:20 UTC)      │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  Data Encryption               │  │
│  │  - AES encryption              │  │
│  │  - Names & Prices              │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  Email Service                 │  │
│  │  - Password Reset              │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  Scheduled Tasks               │  │
│  │  - Rate Updates (14:20 UTC)    │  │
│  │  - Token Cleanup (2 AM)        │  │
│  └────────────────────────────────┘  │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│           MySQL Database             │
│  ┌────────────────────────────────┐  │
│  │  Users & Authentication        │  │
│  │  Assets & Liabilities          │  │
│  │    - Encrypted names           │  │
│  │    - Encrypted prices          │  │
│  │  Categories & Subcategories    │  │
│  │  Exchange Rates (JSON)         │  │
│  │  Refresh Tokens                │  │
│  │  Password Reset Tokens         │  │
│  │  User Preferences              │  │
│  └────────────────────────────────┘  │
│  Flyway Migrations (4 versions)      │
└──────────────────────────────────────┘
               ▲
               │
      ┌────────┴────────┐
      │  Contabo VPS    │
      │  - SSL/HTTPS    │
      │  - Schedulers   │
      │  - Monitoring   │
      └─────────────────┘

      External Services:
      ┌────────────────┐
      │ Currency API   │
      │ Updates 14:00  │
      │ We sync 14:20  │
      └────────────────┘
      ┌────────────────┐
      │ Email Service  │
      │ (SMTP)         │
      └────────────────┘
      ┌────────────────┐
      │ Google         │
      │ Analytics      │
      └────────────────┘
```

---

## Tech Stack Deep Dive

### Frontend Technologies

#### **React (Vanilla JavaScript)**

- Built with vanilla JavaScript for rapid development and simplicity
- Focus on functionality over type safety
- TypeScript migration planned for future scalability
- Component Architecture: Functional components with hooks
- Performance: Optimized re-renders using React.memo and useMemo

#### **Redux**

```javascript
Store Structure:
├── auth (user, token, isAuthenticated)
├── financial (assets, liabilities, balance)
├── ui (theme, language, loading states)
├── categories (all categories and subcategories)
├── currency (selected currency, rates)
└── monthHistory (month-based cache tracking)
```

**Why Redux over Context API?**

- **Context API** = Built-in React state management (simpler, but less scalable)
- **Redux** = External library with more structure

**Benefits of Redux:**

- Predictable state updates
- Better performance for frequent updates
- Time-travel debugging
- Middleware support for async actions
- More scalable for large applications
- Essential for complex caching strategy

#### **Nivo Charts**

- **Chosen for:** Customization flexibility, React-native, responsive by default
- **Chart types used:** Pie (category distribution), Bar (trends), Custom (balance diagrams)
- **Performance:** Animated transitions, interactive tooltips

#### **CSS Modules**

**Advantages:**

- Scoped styles per component
- No naming conflicts
- Smaller production bundles (unused styles removed)
- CSS files organized alongside components

**Example structure:**

```
Dashboard/
├── Dashboard.jsx
└── Dashboard.module.css  ← Same folder as component
```

#### **Custom i18n System**

**Why not react-i18next?**

- Needed full control over translation loading
- 5,000+ translations would bloat bundle with external library
- Custom optimization for our specific use case
- No external dependencies

**Current Architecture:**

```javascript
Translator.js (5000+ lines)
├── translations object (all 7 languages)
├── getTranslation(key, language)
├── formatCurrency(amount, currency, language)
└── formatDate(date, language)
```

**Future: Modular structure (Q1 2026)**

```
languages/
├── en.js
├── es.js
├── it.js
├── fr.js
├── pl.js
├── ru.js
├── pt.js
└── de.js (NEW - German)
```

---

### Backend Technologies

#### **Java 17 + Spring Boot 3.x**

**Why Java?**

- Strong typing for financial calculations (critical for accuracy)
- Mature ecosystem (Spring, Flyway, Hibernate)
- Reliable and well-documented
- Widely used in production financial applications

**Spring Boot Advantages:**

- Convention over configuration
- Built-in security features
- Easy REST API creation
- Excellent documentation and community support

#### **MySQL 8.0**

**Why MySQL?**

- Familiarity and existing expertise
- Excellent performance for our use case
- Strong community support
- JSON column support (used for exchange rates)

#### **Flyway Migrations**

**Why Flyway?**

- Version control for database schema
- Safe production deployments
- Easy rollback if needed
- Prevents schema conflicts

---

## Database Design

### Flyway Migration History

#### **V1: Refresh Tokens Table**

```sql
CREATE TABLE refresh_tokens (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    token VARCHAR(255) NOT NULL UNIQUE,
    user_entity_id BIGINT NOT NULL,
    expiry_date TIMESTAMP NOT NULL,
    session_expiry_date TIMESTAMP NOT NULL,
    revoked BOOLEAN DEFAULT FALSE,
    used BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (user_entity_id) REFERENCES user_entity(id)
);

CREATE INDEX idx_refresh_token ON refresh_tokens(token);
-- Accelerates: refreshTokenService.findByToken(requestToken)

CREATE INDEX idx_refresh_user ON refresh_tokens(user_entity_id);
-- Accelerates: refreshTokenRepository.deleteByUser(user)
```

**Purpose:**

- Stores long-lived refresh tokens (30 days)
- Enables extended session continuity
- Supports token revocation

---

#### **V2: Terms & Conditions Acceptance**

```sql
ALTER TABLE user_entity
ADD COLUMN accepted_terms_and_conditions BOOLEAN NOT NULL DEFAULT FALSE;

ALTER TABLE user_entity
ADD COLUMN terms_accepted_at TIMESTAMP NULL;

-- For existing users, mark as accepted
UPDATE user_entity
SET accepted_terms_and_conditions = TRUE,
    terms_accepted_at = CURRENT_TIMESTAMP
WHERE accepted_terms_and_conditions = FALSE;
```

**Purpose:**

- Track user consent for terms and conditions
- Legal compliance (GDPR, user agreements)
- Audit trail for when users accepted

---

#### **V3: Password Reset Tokens**

```sql
CREATE TABLE password_reset_tokens (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    token VARCHAR(500) NOT NULL UNIQUE,
    user_id BIGINT NOT NULL,
    expiry_date TIMESTAMP NOT NULL,
    used BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_password_reset_token_user
        FOREIGN KEY (user_id)
        REFERENCES user_entity(id)
        ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE INDEX idx_password_reset_tokens_token ON password_reset_tokens(token);
CREATE INDEX idx_password_reset_tokens_user_id ON password_reset_tokens(user_id);
CREATE INDEX idx_password_reset_tokens_expiry_date ON password_reset_tokens(expiry_date);
CREATE INDEX idx_password_reset_tokens_used ON password_reset_tokens(used);
```

**Purpose:**

- Secure password reset via email
- Token expiration (24 hours)
- Prevents token reuse

**Foreign Key with CASCADE:**

```sql
FOREIGN KEY (user_id) REFERENCES user_entity(id) ON DELETE CASCADE
```

**What this does:**

- If user is deleted → their password reset tokens are automatically deleted
- Maintains database integrity
- Prevents leftover data (orphaned records)

---

#### **V4: Exchange Rates with JSON Storage**

```sql
CREATE TABLE exchange_rates (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    base_currency VARCHAR(3) NOT NULL UNIQUE,
    conversion_rates_json JSON NOT NULL,
    last_updated TIMESTAMP NOT NULL,
    INDEX idx_exchange_rates_base_currency (base_currency),
    INDEX idx_exchange_rates_last_updated (last_updated)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Purpose:**

- Stores exchange rates for 6 currencies
- JSON column holds ALL conversion rates (flexible)
- Daily updates via scheduler

---

### Key Design Decisions

#### **1. JSON Column for Exchange Rates**

**Entity structure:**

```java
@Entity
@Table(name = "exchange_rates")
public class ExchangeRateEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "base_currency", nullable = false, unique = true, length = 3)
    private String baseCurrency;  // EUR, USD, COP, PEN, PLN, RUB

    @JdbcTypeCode(SqlTypes.JSON)
    @Column(name = "conversion_rates_json", nullable = false, columnDefinition = "json")
    private Map<String, Double> conversionRates;
    // {"USD": 1.09, "COP": 4320.5, "GBP": 0.85, "JPY": 160.5, ...}

    @Column(name = "last_updated", nullable = false)
    private LocalDateTime lastUpdated;
}
```

**Why JSON instead of normalized tables?**

- ✅ Stores complete API response (100+ currency pairs)
- ✅ No schema changes needed when API adds new currencies
- ✅ Eliminates complex join tables
- ✅ MySQL 8.0 JSON functions enable efficient querying
- ✅ One row per base currency (6 rows total)

**Storage example:**

```json
{
  "id": 1,
  "base_currency": "EUR",
  "conversion_rates_json": {
    "USD": 1.09,
    "COP": 4320.5,
    "PEN": 4.02,
    "PLN": 4.25,
    "RUB": 98.5,
    "GBP": 0.85
    // ... 100+ more currencies
  },
  "last_updated": "2026-01-14T14:20:00"
}
```

---

#### **2. Comprehensive Indexing Strategy**

**Authentication Indexes:**

```sql
CREATE INDEX idx_refresh_token ON refresh_tokens(token);
CREATE INDEX idx_password_reset_tokens_token ON password_reset_tokens(token);
```

- Critical path: token lookups happen on every authenticated request
- Milliseconds matter for user experience

**User-based Indexes:**

```sql
CREATE INDEX idx_refresh_user ON refresh_tokens(user_entity_id);
```

- Fast user-based queries (e.g., logout all sessions)

**Cleanup Indexes:**

```sql
CREATE INDEX idx_password_reset_tokens_expiry_date ON password_reset_tokens(expiry_date);
```

- Optimizes scheduled cleanup tasks
- Find expired tokens quickly

**Currency Indexes:**

```sql
CREATE INDEX idx_exchange_rates_base_currency ON exchange_rates(base_currency);
```

- Fast lookups when fetching rates for specific currency

---

#### **3. Data Encryption at Application Layer**

**Current entity structure:**

```java
@Entity(name = "passive")
public class PassiveEntity extends BaseEntity {

    @Column(nullable = false, length = 255)
    @Convert(converter = AttributeEncryptor.class)
    private String name;  // Transaction name (encrypted)

    @Column(nullable = false)
    private LocalDate date;

    @Column(nullable = false)
    @ValidPrice
    @Convert(converter = AttributeEncryptor.class)
    private String price;  // Amount in wallet currency (encrypted)

    @Column
    private Boolean proudForIt;

    @Column
    private Boolean reallyNecessary;

    @Column(nullable = false)
    private PassiveCategory category;

    @ManyToOne
    @JoinColumn(name = "sub_category_id", nullable = false)
    private SubCategoriesEntity subCategory;

    @ManyToOne
    @JoinColumn(name = "portfoglio_id")
    private PortfoglioEntity portfoglio;

    @ManyToOne
    @JoinColumn(name = "bilance_id")
    private BilanceEntity bilance;
}
```

**What gets encrypted:**

- ✅ Transaction names (descriptions)
- ✅ Transaction amounts (prices)

**Why encrypt at application layer?**

- Database compromise doesn't expose sensitive data
- Privacy for users (first users were fellow programmers)
- Encryption keys stored separately from database
- Decrypt only when needed for display

**Same encryption applied to AssetEntity** (income transactions)

---

## Backend Architecture

### Service Layer Organization

**Package Structure:**

```
com.inco.backend/
├── controller/
│   ├── AuthController.java
│   ├── AssetController.java
│   ├── LiabilityController.java
│   ├── CategoryController.java
│   └── CurrencyController.java
│
├── service/
│   ├── interface/
│   │   ├── AssetService.java
│   │   ├── LiabilityService.java
│   │   ├── AuthService.java
│   │   ├── CurrencyService.java
│   │   ├── EmailService.java
│   │   └── UserService.java
│   └── implementation/
│       ├── AssetServiceImpl.java
│       ├── LiabilityServiceImpl.java
│       ├── AuthServiceImpl.java
│       ├── CurrencyServiceImpl.java
│       ├── EmailServiceImpl.java
│       └── UserServiceImpl.java
│
├── database/
│   ├── entities/
│   │   ├── PassiveEntity.java
│   │   ├── AssetEntity.java
│   │   ├── ExchangeRateEntity.java
│   │   ├── RefreshToken.java
│   │   ├── UserEntity.java
│   │   └── SubCategoriesEntity.java
│   └── repositories/
│       ├── AssetRepository.java
│       ├── LiabilityRepository.java
│       ├── ExchangeRateRepository.java
│       ├── RefreshTokenRepository.java
│       └── UserRepository.java
│
├── security/
│   ├── JwtTokenProvider.java
│   ├── AttributeEncryptor.java
│   └── SecurityConfig.java
│
└── scheduler/
    └── ExchangeRateScheduler.java
```

**Design Pattern: Interface-Implementation Separation**

**Why this pattern?**

1. **Clear Contracts:** Interfaces define WHAT services do
2. **Loose Coupling:** Controllers depend on interfaces, not implementations
3. **Easy Testing:** Mock interfaces in unit tests
4. **Flexibility:** Can swap implementations without changing consumers
5. **SOLID Principles:** Interface Segregation + Dependency Inversion
6. **Spring AOP:** Enables proxying for transactions, security, caching

**Example:**

```java
// Interface - defines the contract
public interface AssetService {
    Asset createAsset(AssetDTO dto, User user);
    List<Asset> getAssetsByUser(Long userId);
    Asset updateAsset(Long assetId, AssetDTO dto, Long userId);
    void deleteAsset(Long assetId, Long userId);
}

// Implementation - contains business logic
@Service
public class AssetServiceImpl implements AssetService {

    @Autowired
    private AssetRepository assetRepository;

    @Autowired
    private AttributeEncryptor encryptor;

    @Override
    public Asset createAsset(AssetDTO dto, User user) {
        // Business logic
        // Validation
        // Encryption
        // Save to database
        return assetRepository.save(asset);
    }

    // ... other methods
}
```

**Controllers depend on interface:**

```java
@RestController
@RequestMapping("/api/assets")
public class AssetController {

    @Autowired
    private AssetService assetService;  // Interface, not implementation

    @PostMapping
    public ResponseEntity<Asset> createAsset(@RequestBody AssetDTO dto) {
        Asset asset = assetService.createAsset(dto, getCurrentUser());
        return ResponseEntity.ok(asset);
    }
}
```

---

## Frontend Architecture

### Redux State Management

**Store Structure:**

```javascript
{
  auth: {
    user: {
      id,
      email,
      name,
      walletCurrency,
      language,
      theme
    },
    accessToken: "jwt...",
    isAuthenticated: true
  },

  financial: {
    assets: [...],
    liabilities: [...],
    balance: {
      income: 5000,
      expenses: 3000,
      net: 2000
    },
    loading: false
  },

  ui: {
    theme: "dark",
    language: "it",
    sidebarOpen: true
  },

  categories: {
    assetCategories: [...],
    liabilityCategories: [...],
    subcategories: [...]
  },

  currency: {
    selectedCurrency: "EUR",
    rates: { /* from backend */ }
  }
}
```

**Why Redux for this project:**

- State shared across many components
- Frequent updates to financial data
- Need predictable state management
- Better debugging with Redux DevTools
- Essential for complex caching strategy

---

### Utils Organization

```
utils/
├── currenciesUtils.js      # Currency operations, validation, conversion
├── validators.js           # Form validation (email, password)
├── constants.js            # API endpoints, app constants
└── apiInterceptors.js      # Axios request/response handling
```

---

**Currency Management (currenciesUtils.js):**

**1. API Integration:**

```javascript
export const fetchCurrencies = async (walletCurrencyName) => {
  // Fetches from backend: /api/currency/for-user
  // Returns array of currencies with rates
  // Backend response includes 162 currencies
  // Transforms to frontend-friendly format
};
```

**Backend Response Structure:**

```json
{
  "walletCurrency": "EUR",
  "walletSymbol": "€",
  "currencies": [
    {
      "code": "USD",
      "symbol": "$",
      "name": "United States Dollar",
      "rate": 1.1547,
      "isWalletCurrency": false
    },
    {
      "code": "EUR",
      "symbol": "€",
      "name": "Euro",
      "rate": 1.0,
      "isWalletCurrency": true
    }
    // ... 160+ more currencies
  ],
  "totalCurrencies": 162
}
```

**2. Price Validation:**

```javascript
export const validatePrice = (priceInput, setPriceError) => {
  if (!priceInput || parseFloat(priceInput) <= 0) {
    setPriceError(true);
    setTimeout(() => setPriceError(false), 3000);
    return false;
  }
  setPriceError(false);
  return true;
};
```

**3. Currency Conversion:**

```javascript
export const convertCurrencyAndProceed = (
  priceInput,
  currencies,
  selectedCurrency,
  userInfo,
  setConvertedPrice,
  setCurrentStep
) => {
  // FALLBACK: If currencies didn't load, use price directly
  if (!Array.isArray(currencies) || currencies.length === 0) {
    setConvertedPrice(parseFloat(priceInput).toFixed(2));
    return;
  }

  // NORMAL FLOW: Convert using rates
  const isWalletCurrency = currencyToUse.name === userInfo.currencyName;
  const priceToSave = isWalletCurrency
    ? parseFloat(priceInput)
    : parseFloat(priceInput) / currencyToUse.rate;

  setConvertedPrice(priceToSave.toFixed(2));
};
```

**4. Custom Hooks:**

```javascript
// Automatic currency loading
export const useCurrencyLoader = (userInfo, shouldReload, dispatch) => {
  useEffect(() => {
    // Loads currencies on mount or when wallet currency changes
    // Caches results in Redux
    // Handles errors gracefully
  }, [userInfo?.currencyName, shouldReload]);
};

// Default currency selection
export const useDefaultCurrency = (currencies, setSelectedCurrency) => {
  useEffect(() => {
    // Automatically selects wallet currency as default
    // Runs when currencies load
  }, [currencies]);
};
```

**Usage Example:**

```javascript
// In Passives/Assets components
import {
  validatePrice,
  convertCurrencyAndProceed,
  useCurrencyLoader,
  useDefaultCurrency,
} from "../utils/currenciesUtils";

// Auto-load currencies
useCurrencyLoader(userInfo, shouldReload, dispatch, lastFetched);

// Set default currency
useDefaultCurrency(currencies, selectedCurrency, setSelectedCurrency);

// Validate before submission
if (!validatePrice(priceInput, setPriceError)) return;

// Convert and proceed
convertCurrencyAndProceed(
  priceInput,
  currencies,
  selectedCurrency,
  userInfo,
  setConvertedPrice,
  setCurrentStep
);
```

---

### Date Formatting & Selection Strategies

**Three different use cases with different implementations:**

---

#### **1. Simple Date Parsing (Internal Use)**

```javascript
// Helper function for DD/MM/YYYY format conversion
const formatDateToDDMMYYYY = (isoDateString) => {
  const date = new Date(isoDateString);
  const day = date.getDate().toString().padStart(2, "0");
  const month = (date.getMonth() + 1).toString().padStart(2, "0");
  const year = date.getFullYear();
  return `${day}/${month}/${year}`;
};

// Used for: Internal date conversions and API communication
// Why: Fast, no dependencies, consistent format for backend
```

---

#### **2. Localized Month Selection (All Pages)**

**Component:** `TimeRangeDisplay.jsx`  
**Used in:** Passives, Assets, Balance

```javascript
import { format } from "date-fns";
import { it, enUS, fr, pl, pt, ru, es } from "date-fns/locale";

const localeMap = {
  Spanish: es,
  Italian: it,
  English: enUS,
  French: fr,
  Polish: pl,
  Portuguese: pt,
  Russian: ru,
};

// Display current month in user's language
const selectedLocale = localeMap[language] || enUS;
const formattedDate = format(selectedMonth, "LLLL yyyy", {
  locale: selectedLocale,
});

// Capitalize first letter
const capitalizedMonth = capitalizeFirstLetter(formattedDate);

// Examples:
// Italian:     "Gennaio 2026"
// English:     "January 2026"
// Spanish:     "Enero 2026"
```

**Features:**

- Month navigation with arrow buttons (previous/next)
- Jump to current month button
- Localized month names for all 7 languages
- Shared component across all pages
- Smart caching (checks if month data already loaded)

**User Flow:**

```
User clicks left arrow
↓
Load January 2025
↓
Check: Is 2025-01 in cache?
↓
If YES → Filter from Redux (instant)
If NO → Fetch from API, add to cache
```

---

#### **3. Localized Date Range Selection (Passives/Assets Only)**

**Component:** `CalendarPicker.jsx`  
**Used in:** Passives, Assets (NOT Balance)

```javascript
import { DateRange } from "react-date-range";
import { format, isWithinInterval, parse } from "date-fns";
import { it, enUS, fr, pl, pt, ru, es } from "date-fns/locale";

const localeMap = {
  Spanish: es,
  Italian: it,
  English: enUS,
  French: fr,
  Polish: pl,
  Portuguese: pt,
  Russian: ru,
};

const selectedLocale = localeMap[language] || enUS;

// Display selected date range
const displayRange = `${format(date[0].startDate, "dd/MM/yyyy", {
  locale: selectedLocale,
})} - ${format(date[0].endDate, "dd/MM/yyyy", {
  locale: selectedLocale,
})}`;

// Use react-date-range with localized calendar
<DateRange
  editableDateInputs={true}
  moveRangeOnFirstSelection={false}
  onChange={(item) => setDate([item.selection])}
  ranges={date}
  maxDate={new Date()}
  minDate={new Date(2012, 0, 1)}
  locale={selectedLocale}
/>;
```

**Features:**

- Custom date range selection (e.g., "30 Dec 2024 → 4 May 2025")
- Localized calendar interface
- Intelligent caching (checks if data already loaded for selected months)
- Prevents API calls for already-cached date ranges
- Multi-case optimization strategy

**Intelligent Caching Strategy:**

**Case 1: Dates within already-cached months**

```javascript
// Selected: 15/01/2025 - 20/01/2025
// Cache has: 2025-01

// Action: Filter from cache, instant result
const filteredItems = allItems.filter((item) => {
  const itemDate = parse(item.date, "dd/MM/yyyy", new Date());
  return isWithinInterval(itemDate, {
    start: date[0].startDate,
    end: date[0].endDate,
  });
});
```

**Case 2: Dates within previously-fetched range**

```javascript
// Previously fetched: 01/12/2024 - 31/01/2025
// Now selecting: 10/12/2024 - 15/01/2025

// Action: Filter from existing filtered data (even faster!)
const filteredItems = allLastFilteredItems.filter((item) => {
  const itemDate = parse(item.date, "dd/MM/yyyy", new Date());
  return isWithinInterval(itemDate, { start, end });
});
```

**Case 3: Extended range (partial overlap)**

```javascript
// Previously fetched: 01/12/2024 - 31/01/2025
// Cache has months: 2025-02, 2025-03
// Now selecting: 15/12/2024 - 15/03/2025

// Action: Combine cached data, no API call needed
const filteredFromHistory = allLastFilteredItems.filter(/* 12/2024-01/2025 */);
const filteredFromMonths = allItems.filter(/* 02/2025-03/2025 */);
const combined = [...filteredFromHistory, ...filteredFromMonths];
```

**Case 4: New range (no cached data)**

```javascript
// Selected: 01/06/2024 - 30/06/2024
// Cache doesn't have: 2024-06

// Action: Fetch from API
const items = await getByPeriod(startDate, endDate, userId);
```

**Benefits:**

- Minimizes API calls (only fetch when necessary)
- Fast response time (filter from memory)
- Supports complex user workflows (jumping between date ranges)
- Intelligent cache reuse

---

#### **4. Period Selection (Balance Only)**

**Component:** `CalendarPeriodByMonthPicker.jsx`  
**Used in:** Balance page only

```javascript
import { format } from "date-fns";
import { it, enUS, fr, pl, pt, ru, es } from "date-fns/locale";

const localeMap = {
  Spanish: es,
  Italian: it,
  English: enUS,
  French: fr,
  Polish: pl,
  Portuguese: pt,
  Russian: ru,
};

// Get localized month name
const getMonthName = (monthIndex, year) => {
  const date = new Date(year, monthIndex);
  const monthName = format(date, "LLLL", { locale: selectedLocale });
  return capitalizeFirstLetter(monthName);
};

// Generate month range display
const startMonthName = getMonthName(selectedMonth - 1, selectedYear);
const endMonthName = getMonthName(endDate.getMonth(), endDate.getFullYear());
const monthYearRange = `${startMonthName} ${selectedYear} - ${endMonthName} ${endDate.getFullYear()}`;

// Examples:
// Italian (Quarterly):     "Gennaio 2026 - Marzo 2026"
// English (Semi-annual):   "January 2026 - June 2026"
// Spanish (Annual):        "Enero 2026 - Diciembre 2026"
```

**Four period options:**

1. **Quarterly (Trimestrale):** 3 months
2. **Four-monthly (Quadrimestrale):** 4 months
3. **Semi-annual (Mezzo Anno):** 6 months
4. **Annual (Annuale):** 12 months

**Why period selection only on Balance?**

- Balance shows aggregated financial health over time
- Quarters and annual views are standard for financial reporting
- Users want to see trends: "How did I do this quarter?"
- Passives/Assets are more granular (day-to-day tracking)

---

### Date Selection Summary

| Page         | Monthly Selection   | Date Range Selection | Period Selection               |
| ------------ | ------------------- | -------------------- | ------------------------------ |
| **Passives** | ✅ TimeRangeDisplay | ✅ CalendarPicker    | ❌                             |
| **Assets**   | ✅ TimeRangeDisplay | ✅ CalendarPicker    | ❌                             |
| **Balance**  | ✅ TimeRangeDisplay | ❌                   | ✅ CalendarPeriodByMonthPicker |

**Why these differences?**

**Passives/Assets (Day-to-Day Tracking):**

- Need flexible date ranges for analysis
- Example: "Show me all expenses during my vacation (Dec 30 - Jan 10)"
- Example: "Compare spending Nov-Dec vs Jan-Feb"
- Use case: Granular transaction analysis

**Balance (Financial Overview):**

- Need standard reporting periods
- Example: "How was my Q4 2025?"
- Example: "Show me full year 2025"
- Use case: High-level financial health assessment

---

### date-fns Usage Justification

**Why date-fns instead of native Date?**

1. **Localization:** Native Date doesn't handle month names in different languages
2. **Interval checking:** `isWithinInterval` handles edge cases correctly
3. **Date arithmetic:** `addMonths`, `subMonths`, `startOfMonth` are timezone-safe
4. **Formatting:** Consistent formatting across all date operations
5. **Calendar integration:** react-date-range requires date-fns locales

**Trade-offs:**

- ✅ Robust, battle-tested library
- ✅ Handles edge cases (leap years, DST, month boundaries)
- ✅ Consistent API across all date operations
- ❌ Adds ~15KB to bundle (minified + gzipped)
- ❌ External dependency

**For 7 languages + complex date operations:** Worth the bundle size.

---

**Currency Formatting (Inline):**

```javascript
// Uses native Intl.NumberFormat API inline in components

new Intl.NumberFormat(language, {
  style: "currency",
  currency: currency,
  minimumFractionDigits: 2,
  maximumFractionDigits: 2,
}).format(amount);

// Example outputs:
// "1.234,56 €" (Italian)
// "$1,234.56" (English)
// "50.000 COP" (Spanish)
```

**Why inline instead of utility function?**

- Used sparingly (only display layer)
- Intl API is concise enough (2-3 lines)
- Different formatting options needed per context

---

**Design Decisions:**

**Why custom hooks?**

- Encapsulates complex currency loading logic
- Reusable across Passives, Assets, Budget components
- Clean separation of concerns
- Redux integration handled internally

**Why client-side conversion?**

- Backend provides rates, frontend calculates
- Reduces backend load
- Immediate user feedback
- Flexible for different UX flows

---

## Redux Store Architecture

**Problem:** Balance, Passives, and Assets pages all need financial data. Multiple filter combinations, date ranges, and period selections create complex data management challenges.

**Solution:** Multi-layered caching strategy with intelligent data reuse.

---

### Store Structure

**Three-layer architecture:**

```javascript
{
  // Layer 1: Main cache (never filtered, only grows)
  allPassives: {
    lastAllPassives: [...],     // All loaded passives
  },
  allActives: {
    lastAllActives: [...],      // All loaded actives
  },

  // Layer 2: Page-specific filtering
  allPassives: {
    filteredPassives: [...],    // Currently visible on Passives page
  },
  allActives: {
    filteredActives: [...],     // Currently visible on Assets page
  },

  // Layer 3: Balance-specific cache
  activesAndPassivesForBalance: {
    storeAllActivesAndPassivesForBalance: {
      actives: [...],           // All balance data loaded
      passives: [...]
    },
    activesAndPassivesForBalanceRendering: {
      actives: [...],           // Currently visible on Balance page
      passives: [...]
    }
  },

  // Layer 4: History tracking (month granularity)
  monthHistory: {
    bilanceHistory: ["2025-01", "2025-02"],     // Months with complete balance data
    passiveHistory: ["2025-01", "2025-02"],     // Months with passives loaded
    activeHistory: ["2025-01", "2025-02"]       // Months with actives loaded
  }
}
```

---

### Month-Based History Tracking

**Why track by month?**

- Users typically view data by month/quarter/year
- Granular enough to avoid over-fetching
- Simple to check: `allMonthsHistory.includes("2025-03")`
- Easy to generate ranges: `["2025-01", "2025-02", "2025-03"]`

**History types:**

1. **bilanceHistory:** Complete balance data (actives + passives together)
2. **passiveHistory:** Passives loaded separately
3. **activeHistory:** Actives loaded separately

---

### Cache Decision Logic (Balance Page)

**When user selects a period (quarterly, semi-annual, etc):**

```javascript
const handleAccept = async () => {
  // 1. Generate month range
  const monthsInRange = generateMonthsRange(firstMonth, secondMonth);
  // Example: ["2025-01", "2025-02", "2025-03"]

  // 2. Check what data we have
  const allMonthsExistInBalance = monthsInRange.every((month) =>
    allMonthsBilance.includes(month)
  );

  const allMonthsHavePassives = monthsInRange.every((month) =>
    allMonthsPassives.includes(month)
  );

  const allMonthsHaveActives = monthsInRange.every((month) =>
    allMonthsActives.includes(month)
  );

  // 3. Decide: Fetch or use cache?
  if (
    !allMonthsExistInBalance &&
    !(allMonthsHavePassives && allMonthsHaveActives)
  ) {
    // 🚨 FETCH: Missing data
    console.log("💾 Fetching complete balance data");
    bilance = await updateBilanceDatesForMonths(
      userId,
      firstMonth,
      secondMonth
    );

    // Update history
    monthsInRange.forEach((month) => {
      if (!allMonthsBilance.includes(month)) {
        dispatch(addBilanceHistory(month));
      }
    });

    dispatch(setStoreAllActivesAndPassivesForBalance(bilance));
  } else {
    // ✅ CACHE HIT: Use existing data

    if (allMonthsExistInBalance) {
      // Case 1: Complete balance data exists
      console.log("📊 Using complete balance data from cache");

      const passivesForBilance =
        storeActivesAndPassivesForBalance.passives.filter((passive) =>
          isWithinInterval(parse(passive.date), { start, end })
        );

      const activesForBilance =
        storeActivesAndPassivesForBalance.actives.filter((active) =>
          isWithinInterval(parse(active.date), { start, end })
        );

      bilance = { actives: activesForBilance, passives: passivesForBilance };
    } else if (allMonthsHavePassives && allMonthsHaveActives) {
      // Case 2: Have BOTH actives AND passives separately
      console.log("🔄 Combining separate actives and passives");

      const filteredPassives = allPassives.filter((passive) =>
        isWithinInterval(parse(passive.date), { start, end })
      );

      const filteredActives = allActives.filter((active) =>
        isWithinInterval(parse(active.date), { start, end })
      );

      bilance = { actives: filteredActives, passives: filteredPassives };

      // Update balance store for future queries
      dispatch(setStoreAllActivesAndPassivesForBalance(bilance));

      // Add to balance history
      monthsInRange.forEach((month) => {
        if (!allMonthsBilance.includes(month)) {
          dispatch(addBilanceHistory(month));
        }
      });
    }
  }

  // Update rendering data
  dispatch(setActivesAndPassivesForBalanceRendering(bilance));
};
```

---

### Date Range Filtering

**Using date-fns for precise date handling:**

```javascript
import {
  parse,
  isWithinInterval,
  startOfDay,
  endOfDay,
  lastDayOfMonth,
} from "date-fns";

// Create interval with inclusive boundaries
const adjustedStart = startOfDay(
  parse(firstMonth + "-01", "yyyy-MM-dd", new Date())
);

const adjustedEnd = endOfDay(
  lastDayOfMonth(parse(secondMonth + "-01", "yyyy-MM-dd", new Date()))
);

// Filter transactions within range
const filteredPassives = allPassives.filter((passive) => {
  const passiveDate = parse(passive.date, "dd/MM/yyyy", new Date());
  return isWithinInterval(passiveDate, {
    start: adjustedStart,
    end: adjustedEnd,
  });
});
```

**Why date-fns here?**

- Handles edge cases (last day of month, leap years)
- Timezone-safe operations
- Inclusive interval checking
- Used only for complex date operations (not simple formatting)

---

### Duplicate Prevention

**Problem:** Multiple sources can try to add same transaction

**Solution:** Comprehensive duplicate checking

```javascript
setStoreAllActivesAndPassivesForBalance: (state, action) => {
  const { actives, passives } = action.payload;

  // Helper for duplicate detection
  const isDuplicate = (existing, newItem) => {
    return (
      existing.id === newItem.id ||
      (existing.nome === newItem.nome &&
        existing.date === newItem.date &&
        existing.price === newItem.price &&
        existing.type === newItem.type)
    );
  };

  // Filter out duplicates
  const newUniqueActives = actives.filter(
    (newActive) =>
      !state.storeAllActivesAndPassivesForBalance.actives.some(
        (existingActive) => isDuplicate(existingActive, newActive)
      )
  );

  // Append only unique items
  state.storeAllActivesAndPassivesForBalance.actives = [
    ...state.storeAllActivesAndPassivesForBalance.actives,
    ...newUniqueActives,
  ];
};
```

**Duplicate checking:**

- Primary: ID match
- Fallback: Name + Date + Price + Type match

---

### Synchronized Updates

**Problem:** When user edits/deletes transaction, multiple store locations need update

**Solution:** Centralized update actions

```javascript
// Update in all locations at once
updateActiveEverywhere: (state, action) => {
  const { id, updatedData } = action.payload;

  // Update main cache
  state.lastAllActives = state.lastAllActives.map((active) =>
    active.id === id ? { ...active, ...updatedData } : active
  );

  // Update filtered view
  state.filteredActives = state.filteredActives.map((active) =>
    active.id === id ? { ...active, ...updatedData } : active
  );
};

// Update with conditional rendering
syncActiveAcrossLists: (state, action) => {
  const { active, shouldBeInFiltered } = action.payload;

  // Always update main cache
  const allIndex = state.lastAllActives.findIndex((a) => a.id === active.id);
  if (allIndex !== -1) {
    state.lastAllActives[allIndex] = active;
  } else {
    state.lastAllActives.push(active);
  }

  // Conditionally update filtered list
  const filteredIndex = state.filteredActives.findIndex(
    (a) => a.id === active.id
  );

  if (shouldBeInFiltered) {
    if (filteredIndex !== -1) {
      state.filteredActives[filteredIndex] = active;
    } else {
      state.filteredActives.push(active);
    }
  } else {
    // Remove from filtered if conditions no longer met
    if (filteredIndex !== -1) {
      state.filteredActives.splice(filteredIndex, 1);
    }
  }
};
```

**Use case:**

```javascript
// User edits transaction date to move it outside current filter
dispatch(
  syncActiveAcrossLists({
    active: updatedActive,
    shouldBeInFiltered: isWithinCurrentDateRange(updatedActive.date),
  })
);

// Result:
// - Main cache updated ✅
// - Filtered view updated if date still matches ✅
// - Filtered view removed if date no longer matches ✅
```

---

### Performance Benefits

**Without this architecture:**

- API call every page navigation: ~500ms
- Balance view change: ~500ms
- Date filter change: ~500ms

**With this architecture:**

- First load: ~500ms (API call)
- Subsequent views: ~50ms (Redux read + filter)
- **10x faster** for repeat operations

**Cache hit scenarios:**

- User navigates from Passives → Assets: **Instant** (data in Redux)
- User changes Balance period within loaded months: **Instant** (filter existing data)
- User applies category filter: **Instant** (filter from cache)

---

### Month Range Generation

**Generate array of months between two dates:**

```javascript
const generateMonthsRange = (start, end) => {
  const currentDate = new Date();
  const currentYear = currentDate.getFullYear();
  const currentMonth = currentDate.getMonth() + 1;

  const months = [];
  let [startYear, startMonth] = start.split("-").map(Number);
  let [endYear, endMonth] = end.split("-").map(Number);

  while (
    startYear < endYear ||
    (startYear === endYear && startMonth <= endMonth)
  ) {
    // Exclude future months
    if (
      startYear < currentYear ||
      (startYear === currentYear && startMonth <= currentMonth)
    ) {
      months.push(`${startYear}-${startMonth.toString().padStart(2, "0")}`);
    }

    startMonth++;
    if (startMonth > 12) {
      startMonth = 1;
      startYear++;
    }
  }

  return months;
};

// Example:
generateMonthsRange("2025-01", "2025-04");
// Returns: ["2025-01", "2025-02", "2025-03", "2025-04"]
```

**Validation:**

- Excludes future months (can't load data that doesn't exist)
- Returns empty array if invalid range
- UI shows error if empty array returned

---

### Date Selection Options

**Passives & Assets Pages:**

- **Default:** Current month
- **Selector:** Previous months (dropdown)
- **Custom range:** User selects start and end date
- **Example:** "30 Dec 2024 → 4 May 2025"

**Balance Page:**

- **Periods:** Quarterly, Four-monthly, Semi-annual, Annual
- **Selector:** Starting month + year
- **Automatic calculation:** End month based on period
- **Example:** "Jan 2025 (Quarterly)" = Jan-Mar 2025

**Common logic:**

- Both use month-based history tracking
- Both benefit from cache reuse
- Different UX, same underlying architecture

---

### Why This Architecture Works

**Key principles:**

1. **Separate cache from view** - lastAll* vs filtered*
2. **Track what's loaded** - monthHistory arrays
3. **Check before fetching** - Avoid redundant API calls
4. **Filter client-side** - Fast and flexible
5. **Centralized updates** - One source of truth
6. **Duplicate prevention** - Data integrity

**Trade-offs:**

- ✅ Faster user experience (10x)
- ✅ Reduced backend load
- ✅ Flexible filtering without re-fetching
- ✅ Supports complex date ranges
- ❌ More Redux complexity
- ❌ Larger memory footprint
- ❌ More state management code

**For 200+ users:** The trade-offs are worth it. User experience and backend efficiency are priorities.

---

## Multi-Currency System

### Current Implementation

**Architecture Flow:**

```
┌─────────────────────────────────────────────────────────┐
│                    BACKEND                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Scheduled Task (14:20 UTC daily)                │   │
│  │  1. Calls external currency API                  │   │
│  │  2. For each of 6 currencies (EUR,USD,COP,etc)   │   │
│  │  3. Gets complete conversion map                 │   │
│  │  4. Stores in exchange_rates table (JSON)        │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  REST API Endpoint                                │   │
│  │  GET /api/exchange-rates/{baseCurrency}          │   │
│  │  Returns: { "USD": 1.09, "COP": 4320, ... }      │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                           ▲
                           │ HTTP Request
                           │
┌─────────────────────────────────────────────────────────┐
│                    FRONTEND                              │
│  ┌──────────────────────────────────────────────────┐   │
│  │  User Action: Add Expense                         │   │
│  │  - User wallet: EUR                               │   │
│  │  - User enters: 50,000 COP                        │   │
│  └──────────────────────────────────────────────────┘   │
│                           │                              │
│                           ▼                              │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Currency Conversion (Client-side)                │   │
│  │  1. Fetch rates: GET /api/exchange-rates/EUR     │   │
│  │  2. Extract COP rate: 4500                        │   │
│  │  3. Calculate: 50,000 ÷ 4500 = 11.11 EUR         │   │
│  └──────────────────────────────────────────────────┘   │
│                           │                              │
│                           ▼                              │
│  ┌──────────────────────────────────────────────────┐   │
│  │  POST /api/liabilities                            │   │
│  │  Body: {                                          │   │
│  │    name: "Dinner in Bogotá",                      │   │
│  │    price: "11.11",   // Already converted to EUR  │   │
│  │    date: "2026-01-14",                            │   │
│  │    category: "DOLCE_VITA"                         │   │
│  │  }                                                │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    BACKEND                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Receives already-converted amount: "11.11"       │   │
│  │  1. Validates data                                │   │
│  │  2. Encrypts name: "Dinner in Bogotá"            │   │
│  │  3. Encrypts price: "11.11"                       │   │
│  │  4. Saves to database                             │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

### Database Storage (Current)

**PassiveEntity / AssetEntity:**

```java
@Entity
public class PassiveEntity extends BaseEntity {

    @Column(nullable = false)
    @Convert(converter = AttributeEncryptor.class)
    private String name;   // "Dinner in Bogotá" (encrypted)

    @Column(nullable = false)
    @Convert(converter = AttributeEncryptor.class)
    private String price;  // "11.11" (encrypted, in EUR)

    @Column(nullable = false)
    private LocalDate date;

    @Column(nullable = false)
    private PassiveCategory category;

    @ManyToOne
    private SubCategoriesEntity subCategory;

    // NO original currency stored
    // NO original amount stored
    // NO exchange rate stored
}
```

**What we store:**

- ✅ Transaction name (encrypted)
- ✅ Final amount in wallet currency (encrypted)
- ✅ Date, category, subcategory
- ❌ Original amount entered (50,000)
- ❌ Original currency (COP)
- ❌ Exchange rate used (0.00022)

---

### Benefits of Current Approach

**Advantages:**

- ✅ **Simple backend:** No conversion logic needed
- ✅ **Smaller database:** Fewer fields per transaction
- ✅ **Client-side conversion:** Reduces server load
- ✅ **Encrypted storage:** Protects sensitive data
- ✅ **Works well:** Meets current user needs

**Limitations:**

- ❌ **No dual display:** Cannot show "50,000 COP (11.11 EUR)"
- ❌ **No multi-currency reports:** All data in wallet currency only
- ❌ **No historical accuracy:** If user changes wallet, old data doesn't recalculate
- ❌ **No exchange rate history:** Cannot see rate at transaction time

---

### Planned Enhancement: Dual Storage (Q1 2026)

**Goal:** Preserve original transaction data for enhanced user experience

**New fields to add:**

```java
@Column
private String originalCurrency;  // "COP", "USD", "PEN"

@Column
@Convert(converter = AttributeEncryptor.class)
private String originalPrice;  // "50000" (encrypted, as entered)

@Column
private Double exchangeRateUsed;  // 0.00022 (rate at transaction time)
```

**Updated flow:**

```
Frontend sends:
{
  name: "Dinner in Bogotá",
  price: "11.11",              // Converted (EUR)
  originalPrice: "50000",      // As entered (COP)
  originalCurrency: "COP",
  exchangeRateUsed: 0.00022,   // Rate at that moment
  date: "2026-01-14",
  category: "DOLCE_VITA"
}

Backend stores ALL fields (with encryption)
```

**What this enables:**

- ✅ Display: "50,000 COP (11.11 EUR)"
- ✅ Historical accuracy (exchange rate preserved)
- ✅ Multi-currency reporting
- ✅ Recalculation if wallet currency changes
- ✅ Better audit trail for users

**Implementation plan:**

1. **V5 Flyway migration** - Add new columns
2. **Update frontend** - Capture original currency data
3. **Update backend entities** - Store new fields with encryption
4. **Data migration** - Existing records use wallet currency as "original"
5. **UI enhancement** - Toggle "Show in original / Show in wallet"
6. **Gradual rollout** - Test with small user group first

**Why not implemented yet:**

- Current system meets immediate needs
- Focus on core features (Budget, Stock, Mission)
- Dual storage adds complexity - implement when users request it
- Backward compatibility must be ensured

---

## Scheduled Tasks

### 1. Exchange Rate Synchronization

**Schedule:** `@Scheduled(cron = "0 20 14 * * *")`  
**Time:** 14:20 UTC daily

**Timezone Coverage:**

```
UTC:                14:20
Italy (CET/CEST):   15:20 / 16:20
Poland:             15:20 / 16:20
Colombia/Peru:      09:20
Mexico:             08:20
```

**Why 14:20 UTC?**

- External currency API updates at **14:00 UTC**
- We wait **20 minutes** to ensure API has fully updated data
- Strategic timing covers European and Latin American users

**Implementation:**

```java
@Scheduled(cron = "0 20 14 * * *")
public void updateExchangeRates() {
    System.out.println("🔄 Iniciando actualización de tasas de cambio...");

    int totalUpdated = 0;
    int totalErrors = 0;

    for (Currency baseCurrency : ACTIVE_CURRENCIES) {
        // EUR, USD, COP, PEN, PLN, RUB
        try {
            // Call external API
            String url = String.format("%s/%s/latest/%s",
                apiBaseUrl, apiKey, baseCurrency.name());

            String response = restTemplate.getForObject(url, String.class);
            JsonNode rootNode = objectMapper.readTree(response);

            // Extract conversion rates
            JsonNode ratesNode = rootNode.get("conversion_rates");
            Map<String, Double> conversionRates = new HashMap<>();
            ratesNode.fields().forEachRemaining(entry -> {
                conversionRates.put(entry.getKey(), entry.getValue().asDouble());
            });

            // Update or insert
            ExchangeRateEntity existingRate = exchangeRateRepository
                .findByBaseCurrency(baseCurrency.name())
                .orElse(null);

            if (existingRate != null) {
                existingRate.setConversionRates(conversionRates);
                existingRate.setLastUpdated(LocalDateTime.now());
                exchangeRateRepository.save(existingRate);
            } else {
                ExchangeRateEntity newRate = new ExchangeRateEntity();
                newRate.setBaseCurrency(baseCurrency.name());
                newRate.setConversionRates(conversionRates);
                newRate.setLastUpdated(LocalDateTime.now());
                exchangeRateRepository.save(newRate);
            }

            totalUpdated++;
            Thread.sleep(1000); // 1 second pause between calls

        } catch (Exception e) {
            System.err.println("❌ Error: " + e.getMessage());
            totalErrors++;
        }
    }

    System.out.println("✅ Actualización completada");
    System.out.println("   - Monedas actualizadas: " + totalUpdated);
    System.out.println("   - Errores: " + totalErrors);
}
```

**What gets stored:**

```json
{
  "base_currency": "EUR",
  "conversion_rates_json": {
    "USD": 1.09,
    "COP": 4320.5,
    "PEN": 4.02,
    "PLN": 4.25,
    "RUB": 98.5,
    "GBP": 0.85,
    "JPY": 160.5
    // ... 100+ more currencies from API
  },
  "last_updated": "2026-01-14T14:20:00"
}
```

**Benefits:**

- One API call per currency (6 total)
- Stores ALL available rates (not just our 6)
- Future-proof (new currencies automatically included)
- 1-second pause between calls (API courtesy)

---

### 2. Token Cleanup

**Schedule:** `@Scheduled(cron = "0 0 2 * * ?")`  
**Time:** 02:00 AM (server time)

**Implementation:**

```java
@Scheduled(cron = "0 0 2 * * ?")
@Transactional
public void cleanExpiredTokens() {
    Instant now = Instant.now();

    int deletedCount = refreshTokenRepository
        .deleteExpiredAndRevokedTokens(now);

    System.out.println("Eliminati " + deletedCount +
                       " refresh token scaduti/revocati");
}
```

**What gets cleaned:**

- Refresh tokens with `expiry_date < now` (> 30 days old)
- Refresh tokens with `revoked = TRUE`
- Password reset tokens with `used = TRUE` (> 24 hours old)

**Why 2 AM?**

- Low traffic period (European night, Latin American evening)
- After midnight (fresh day start)
- Before European morning traffic

**Benefits:**

- Keeps database lean and fast
- Removes security risk of old tokens
- Automated (zero manual intervention)
- Runs daily without fail

---

## Security & Encryption

### Data Encryption Implementation

**What gets encrypted:**

- Transaction names (PassiveEntity.name, AssetEntity.name)
- Transaction amounts (PassiveEntity.price, AssetEntity.price)

**Implementation:**

```java
@Column(nullable = false)
@Convert(converter = AttributeEncryptor.class)
private String name;

@Column(nullable = false)
@Convert(converter = AttributeEncryptor.class)
private String price;
```

**AttributeEncryptor:**

- Uses AES encryption
- Encryption keys stored separately from database
- Transparent encryption/decryption at application layer
- Database stores encrypted values only

**Why encrypt at application layer?**

1. **Database compromise protection:** Even if database is accessed, data is encrypted
2. **Privacy-first design:** Originally built for fellow programmers
3. **User trust:** 200+ users' financial data protected
4. **Compliance ready:** GDPR-compatible approach

**What's NOT encrypted:**

- User email (needed for login)
- Dates (needed for queries and filtering)
- Categories (needed for aggregation)
- Boolean flags (proudForIt, reallyNecessary)

---

### JWT Authentication Flow

```
1. User Login
   ↓
2. Backend validates email + password (BCrypt)
   ↓
3. Generate:
   - Access Token (7 days)
   - Refresh Token (30 days)
   ↓
4. Store Refresh Token in database
   (refresh_tokens table with revoked/used flags)
   ↓
5. Return both tokens to frontend
   ↓
6. Frontend stores:
   - Access Token: Memory
   - Refresh Token: localStorage
   ↓
7. All API calls:
   Authorization: Bearer {accessToken}
   ↓
8. After 7 days → Access Token expires
   ↓
9. Axios intercepts 401 error
   ↓
10. Auto call /auth/refresh with Refresh Token
    ↓
11. Get new Access Token (new 7-day token)
    ↓
12. Retry original request
    ↓
13. Seamless to user ✅
    ↓
14. After 30 days → User must login again
    ↓
15. Daily cleanup (2 AM) removes expired tokens
```

**Token Configuration:**

```properties
# Access Token: 7 days (604,800,000 ms)
auth.token.expirationInMils=604800000

# Refresh Token: 30 days (2,592,000,000 ms)
auth.token.refreshExpirationDateInMs=2592000000
```

**Security layers:**

- Access tokens: 7 days (balance between security and UX)
- Refresh tokens: 30 days (extended session)
- Refresh tokens in database → Can revoke server-side
- Automated cleanup → Removes old tokens daily (> 30 days)
- BCrypt password hashing → Strong password protection

**Why 7-day access tokens?**

- Balance between security and user experience
- Users don't need to refresh frequently during active use
- Still shorter than refresh token for security hierarchy
- Can be revoked via logout

**Why 30-day refresh tokens?**

- Extended session for better UX
- Users stay logged in for a month
- Still secure (revocable, tracked in database)
- Auto-refresh keeps access token current

---

## Internationalization System

### Current Architecture (Monolithic)

**Translator.js - 5,000+ lines:**

```javascript
class Translator {
  constructor() {
    this.translations = {
      en: {
        "nav.home": "Home",
        "nav.liabilities": "Liabilities",
        "category.forme": "For Me",
        // ... 5000+ more entries
      },
      es: {
        /* same keys, Spanish */
      },
      it: {
        /* same keys, Italian */
      },
      fr: {
        /* same keys, French */
      },
      pl: {
        /* same keys, Polish */
      },
      ru: {
        /* same keys, Russian */
      },
      pt: {
        /* same keys, Portuguese */
      },
    };
  }

  translate(key, language) {
    return this.translations[language][key] || key;
  }

  formatCurrency(amount, currency, language) {
    return new Intl.NumberFormat(language, {
      style: "currency",
      currency: currency,
    }).format(amount);
  }
}

export default new Translator();
```

**Why custom built:**

- Full control over implementation
- No external dependencies
- Optimized for our use case
- Started simple, works well

**Trade-offs:**

- ✅ Zero dependencies
- ✅ Complete control
- ✅ Fast for our needs
- ❌ 5,000-line file hard to maintain
- ❌ Adding German means editing huge file
- ❌ Higher maintenance burden

---

### Future Architecture (Modular - Q1 2026)

**Planned structure:**

```
i18n/
├── Translator.js              # Core logic only
└── languages/
    ├── en.js                  # English translations
    ├── es.js                  # Spanish translations
    ├── it.js                  # Italian translations
    ├── fr.js                  # French translations
    ├── pl.js                  # Polish translations
    ├── ru.js                  # Russian translations
    ├── pt.js                  # Portuguese translations
    └── de.js                  # German (NEW)
```

**Benefits:**

- Easier to maintain (one file per language)
- Easier for translators (separate files)
- Smaller bundle size (lazy load only needed language)
- Simpler to add German
- Better code organization

**Migration plan:**

- Extract translations from monolithic file
- Create separate language files
- Update Translator.js to import dynamically
- Test thoroughly
- Deploy gradually

---

## Code Structure

### Complete Directory Structure

```
inco-cash/
│
├── backend/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/inco/
│   │   │   │   ├── controller/
│   │   │   │   │   ├── AuthController.java
│   │   │   │   │   ├── AssetController.java
│   │   │   │   │   ├── LiabilityController.java
│   │   │   │   │   ├── CategoryController.java
│   │   │   │   │   └── CurrencyController.java
│   │   │   │   │
│   │   │   │   ├── service/
│   │   │   │   │   ├── interface/
│   │   │   │   │   │   ├── AssetService.java
│   │   │   │   │   │   ├── LiabilityService.java
│   │   │   │   │   │   ├── AuthService.java
│   │   │   │   │   │   ├── CurrencyService.java
│   │   │   │   │   │   ├── EmailService.java
│   │   │   │   │   │   └── UserService.java
│   │   │   │   │   └── implementation/
│   │   │   │   │       ├── AssetServiceImpl.java
│   │   │   │   │       ├── LiabilityServiceImpl.java
│   │   │   │   │       ├── AuthServiceImpl.java
│   │   │   │   │       ├── CurrencyServiceImpl.java
│   │   │   │   │       ├── EmailServiceImpl.java
│   │   │   │   │       └── UserServiceImpl.java
│   │   │   │   │
│   │   │   │   ├── database/
│   │   │   │   │   ├── entities/
│   │   │   │   │   │   ├── PassiveEntity.java
│   │   │   │   │   │   ├── AssetEntity.java
│   │   │   │   │   │   ├── ExchangeRateEntity.java
│   │   │   │   │   │   ├── RefreshToken.java
│   │   │   │   │   │   ├── UserEntity.java
│   │   │   │   │   │   └── SubCategoriesEntity.java
│   │   │   │   │   └── repositories/
│   │   │   │   │       ├── AssetRepository.java
│   │   │   │   │       ├── LiabilityRepository.java
│   │   │   │   │       ├── ExchangeRateRepository.java
│   │   │   │   │       ├── RefreshTokenRepository.java
│   │   │   │   │       └── UserRepository.java
│   │   │   │   │
│   │   │   │   ├── security/
│   │   │   │   │   ├── JwtTokenProvider.java
│   │   │   │   │   ├── AttributeEncryptor.java
│   │   │   │   │   └── SecurityConfig.java
│   │   │   │   │
│   │   │   │   └── scheduler/
│   │   │   │       └── ExchangeRateScheduler.java
│   │   │   │
│   │   │   └── resources/
│   │   │       ├── application.properties
│   │   │       └── db/migration/
│   │   │           ├── V1__create_refresh_tokens.sql
│   │   │           ├── V2__add_terms_acceptance.sql
│   │   │           ├── V3__create_password_reset.sql
│   │   │           └── V4__create_exchange_rates.sql
│   │   │
│   │   └── test/
│   │
│   └── pom.xml
│
└── frontend/
    └── src/
        ├── components/
        │   ├── common/
        │   │   ├── TimeRangeDisplay.jsx
        │   │   ├── CalendarPicker.jsx
        │   │   └── CalendarPeriodByMonthPicker.jsx
        │   ├── charts/
        │   ├── forms/
        │   └── layout/
        │
        ├── pages/
        │   ├── Home/
        │   ├── Liabilities/
        │   ├── Assets/
        │   ├── Balance/
        │   └── Profile/
        │
        ├── redux/
        │   ├── store.js
        │   ├── activesAndPassivesForBalanceSlice.js
        │   ├── listActualActivesSlice.js
        │   ├── listActualPassivesSlice.js
        │   ├── monthHistorySlice.js
        │   ├── currenciesSlice.js
        │   └── selectedDatesSlice.js
        │
        ├── utils/
        │   ├── currenciesUtils.js
        │   ├── validators.js
        │   └── apiInterceptors.js
        │
        ├── i18n/
        │   └── Translator.js
        │
        └── package.json
```

---

## Performance & Optimization

### Database Optimization

**Indexes created:**

```sql
-- Authentication (critical path)
CREATE INDEX idx_refresh_token ON refresh_tokens(token);
CREATE INDEX idx_password_reset_tokens_token ON password_reset_tokens(token);

-- User-based queries
CREATE INDEX idx_refresh_user ON refresh_tokens(user_entity_id);
CREATE INDEX idx_password_reset_tokens_user_id ON password_reset_tokens(user_id);

-- Scheduled cleanup
CREATE INDEX idx_password_reset_tokens_expiry_date ON password_reset_tokens(expiry_date);
CREATE INDEX idx_password_reset_tokens_used ON password_reset_tokens(used);

-- Currency queries
CREATE INDEX idx_exchange_rates_base_currency ON exchange_rates(base_currency);
CREATE INDEX idx_exchange_rates_last_updated ON exchange_rates(last_updated);
```

**Query optimization strategies:**

- Filter at database level, not in application code
- Use pagination for large result sets
- Leverage indexes for common queries

---

### Frontend Performance

**Bundle Optimization:**

- CSS Modules for scoped styling and smaller bundles
- Production build removes unused code
- Minification and compression

**Memoization:**

```javascript
// Prevent unnecessary recalculations
const chartData = useMemo(
  () => processLiabilitiesForChart(liabilities),
  [liabilities]
);

// Prevent unnecessary re-renders
const LiabilityRow = React.memo(({ liability }) => {
  return <tr>{/* ... */}</tr>;
});
```

**Redux Performance:**

- Multi-layer caching (10x faster for repeat operations)
- Month-based history tracking
- Intelligent cache reuse
- Duplicate prevention

**Benefits:**

- Charts only recalculate when data changes
- Components only re-render when props change
- Better user experience (smoother UI)
- Drastically reduced API calls

---

## Development Challenges

### Challenge 1: Custom Internationalization

**Problem:** Need 7 languages, 5,000+ translations, no external library bloat

**Solution:** Built custom system

- Created Translator.js with all translations
- Works well for current needs
- Refactor to modular planned for Q1 2026

**Learning:** Sometimes simple monolithic approach is right to start. Refactor when it becomes a problem, not before.

---

### Challenge 2: Multi-Currency Without Dual Storage

**Problem:** Users in 6+ countries spending in different currencies

**Current Solution:** Client-side conversion

- Frontend fetches rates
- Frontend converts before sending to backend
- Backend stores only final amount

**Works for now, but limitations exist:**

- Cannot show original amounts
- Cannot generate multi-currency reports

**Future Solution:** Dual storage (Q1 2026)

- Store both original and converted
- Preserve exchange rate
- Enable better reporting

**Learning:** Ship what users need today. Add complexity when users ask for it, not before.

---

### Challenge 3: Data Encryption for Privacy

**Problem:** First users were fellow programmers - needed trust

**Solution:** Encrypt sensitive fields at application layer

- Transaction names encrypted
- Transaction amounts encrypted
- Transparent to application logic

**Result:**

- Users trust the system
- Privacy protected even if database compromised
- Became selling point for all 200+ users

**Learning:** Security and privacy build trust. Worth the extra complexity.

---

### Challenge 4: Redux Store Complexity

**Problem:** Multiple pages need same data, different filters, different time ranges

**Solution:** Multi-layered caching architecture

- Main cache (lastAll\*)
- Filtered views (filtered\*)
- Balance-specific cache
- Month-based history tracking

**Result:**

- 10x faster for repeat operations
- Flexible filtering without re-fetching
- Complex but maintainable

**Learning:** Complex problems sometimes require complex solutions. Document well, test thoroughly, reap performance benefits.

---

### Challenge 5: Token Lifetime Balance

**Problem:** Balance security with user experience

**Initial thought:** Short tokens = more secure but annoying UX

**Solution:** Two-tier token strategy

- 7-day access tokens (weekly refresh acceptable)
- 30-day refresh tokens (monthly login acceptable)
- Automatic refresh before expiry
- Database tracking and cleanup

**Result:**

- Users stay logged in for 30 days
- Seamless token refresh
- Can revoke tokens server-side
- Daily cleanup maintains security

**Learning:** Security doesn't have to sacrifice UX. Two-tier strategy provides both.

---

### Challenge 6: Date Selection Complexity

**Problem:** Three different pages with different date selection needs

**Solution:** Three specialized components

- TimeRangeDisplay: Monthly navigation (all pages)
- CalendarPicker: Date range selection (Passives/Assets)
- CalendarPeriodByMonthPicker: Period selection (Balance)

**Result:**

- Each page gets appropriate UI for its use case
- Intelligent caching prevents redundant API calls
- Shared month tracking across components
- Localized in all 7 languages

**Learning:** One size doesn't fit all. Tailor solutions to specific use cases while maintaining shared infrastructure.

---

## Production Metrics

### System Performance

**Uptime:** 99.5% since launch  
**API Response Time:** < 200ms average  
**Database Queries:** < 50ms average  
**Scheduled Tasks Success Rate:** 100%

**Infrastructure:**

- Server: Contabo VPS (4 CPU, 8GB RAM)
- Database: MySQL 8.0 (50GB used, 500GB capacity)
- Can handle: 1,000 concurrent users

---

### User Metrics

**Active Users:** 200+  
**Countries:** 6+ (Italy, Poland, Colombia, Peru, USA, Russia)  
**Languages Used:** All 7 actively used  
**Average Transactions per User per Month:** ~45  
**Most Active Times:** Evenings (7-10 PM local time)

**Feature Usage:**

- Multi-currency: 40% of users
- Custom subcategories: Average 8 per user
- Theme preference: 60% use dark theme

---

### User Tracking

**Google Analytics Integration:**

- User behavior insights
- Page view tracking
- Feature usage patterns
- Performance monitoring
- Helps prioritize development

**Privacy-respecting:**

- No personal data sent to GA
- Anonymized user IDs
- Opt-out available

---

## Deployment & DevOps

### Contabo VPS Setup

**Server Specifications:**

- 4 CPU cores
- 8GB RAM
- 200GB SSD
- Ubuntu 22.04 LTS
- Location: Germany (low latency for European users)

**Services Running:**

```
├── Nginx (Reverse Proxy + SSL)
│   ├── Port 443 (HTTPS)
│   ├── Proxy to Spring Boot (port 8080)
│   └── Serves static files
│
├── Spring Boot Application
│   ├── Port 8080 (internal)
│   ├── Runs as systemd service
│   ├── Auto-restart on failure
│   └── Logs to /var/log/inco/
│
├── MySQL 8.0
│   ├── Port 3306 (internal only)
│   └── Binary logging enabled
│
└── Certbot (SSL/TLS)
    └── Auto-renewal every 90 days
```

**Deployment Process:**

```bash
# Frontend
cd frontend
npm run build  # Creates optimized production build

# Backend
cd backend
mvn clean package -DskipTests  # Creates executable JAR

# Deploy
scp target/inco-1.0.jar server:/opt/inco/
ssh server "systemctl restart inco"

# Flyway migrations run automatically on startup
```

---

## Lessons Learned

### What Worked Well

1. **Interface/Implementation Pattern** - Architecture decision that paid off
2. **Flyway Migrations** - Saved from schema disasters multiple times
3. **JSON Column for Rates** - Flexible and fast, no regrets
4. **Data Encryption** - Built trust with users from day one
5. **Custom i18n** - Full control, zero dependencies
6. **Client-side conversion** - Simple and works well
7. **Redux caching strategy** - 10x performance improvement
8. **7/30 day token strategy** - Perfect balance between security and UX
9. **Date selection specialization** - Each page gets the right tool for its job

### What Could Be Improved

1. **Monolithic Translator.js** - Refactor to modular (Q1 2026)
2. **No dual storage yet** - Users starting to request it
3. **Limited test coverage** - Should have written tests earlier
4. **Manual deployments** - Need CI/CD pipeline
5. **Basic monitoring** - Need better observability tools

### Technical Debt

**High Priority:**

- Modular i18n structure
- Dual storage for multi-currency
- Automated testing

**Medium Priority:**

- CI/CD pipeline
- Better monitoring/alerting
- Code documentation

**Low Priority:**

- TypeScript migration
- Microservices architecture (if needed at scale)

---

## Conclusion

Inco.cash demonstrates full-stack engineering from concept to production, serving 200+ real users across 6 countries.

**Technical Achievements:**

- ✅ 7 languages (custom-built, 5,000+ translations)
- ✅ 6 currencies (daily automated sync, client-side conversion)
- ✅ Data encryption (names and amounts protected)
- ✅ JWT with refresh tokens (7-day access, 30-day refresh, automated cleanup)
- ✅ Flyway migrations (4 versions, safe schema evolution)
- ✅ Service layer architecture (Interface/Implementation pattern)
- ✅ Redux caching (multi-layer, intelligent reuse, 10x faster)
- ✅ Date selection systems (3 specialized components, localized, cached)
- ✅ Google Analytics (user insights)
- ✅ 99.5% uptime

**Technologies Mastered:**

- **Frontend:** React, Redux, Nivo, CSS Modules, Custom i18n, date-fns, react-date-range
- **Backend:** Java 17, Spring Boot 3.x, MySQL 8.0, Flyway
- **Security:** JWT, BCrypt, AES encryption, HTTPS
- **DevOps:** Contabo VPS, Nginx, systemd, Certbot
- **Architecture:** REST API, scheduled tasks, JSON storage, encryption at rest, multi-layer caching

This project represents the full software engineering lifecycle: design, architecture, implementation, security, optimization, deployment, and ongoing maintenance of a production system with real users and real financial data.

---

[← Back to Main README](./README.md) | [View Detailed Roadmap →](./ROADMAP.md)
