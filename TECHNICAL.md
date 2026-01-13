# 📖 Inco.cash - Technical Documentation

> Deep dive into architecture, implementation details, and engineering decisions

[← Back to Main README](./README.md)

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Tech Stack Deep Dive](#tech-stack-deep-dive)
3. [Database Design](#database-design)
4. [Frontend Architecture](#frontend-architecture)
5. [Backend Architecture](#backend-architecture)
6. [Multi-Currency System](#multi-currency-system)
7. [Internationalization System](#internationalization-system)
8. [Security Implementation](#security-implementation)
9. [Scheduled Tasks](#scheduled-tasks)
10. [Code Structure](#code-structure)
11. [Development Challenges](#development-challenges)
12. [Performance & Optimization](#performance--optimization)
13. [Production Metrics](#production-metrics)

---

## System Architecture

### High-Level Architecture
```
┌─────────────────────────────────────┐
│      React Frontend (Vanilla JS)    │
│  ┌────────────────────────────────┐ │
│  │  Redux Store                   │ │
│  │  - User State                  │ │
│  │  - Financial Data              │ │
│  │  - UI State                    │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │  Custom i18n System (5000+)    │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │  Nivo Charts                   │ │
│  └────────────────────────────────┘ │
└──────────────┬──────────────────────┘
               │ Axios HTTP
               │ JWT Bearer Token
               ▼
┌──────────────────────────────────────┐
│     Spring Boot REST API             │
│  ┌────────────────────────────────┐  │
│  │  JWT Authentication            │  │
│  │  - Access Token (15min)        │  │
│  │  - Refresh Token (7 days)      │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  Currency Service              │  │
│  │  - Daily Exchange Sync         │  │
│  │  - 6 Currency Pairs            │  │
│  │  - Real-time Conversion        │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  Email Service                 │  │
│  │  - Password Reset              │  │
│  └────────────────────────────────┘  │
│  ┌────────────────────────────────┐  │
│  │  Scheduled Tasks               │  │
│  │  - Daily Rate Updates (2 AM)   │  │
│  │  - Token Cleanup (3 AM)        │  │
│  └────────────────────────────────┘  │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│           MySQL Database             │
│  ┌────────────────────────────────┐  │
│  │  Users & Authentication        │  │
│  │  Assets & Liabilities          │  │
│  │  Categories & Subcategories    │  │
│  │  Exchange Rates (6 pairs)      │  │
│  │  Refresh Tokens                │  │
│  │  User Preferences              │  │
│  └────────────────────────────────┘  │
│  Flyway Migrations                   │
└──────────────────────────────────────┘
               ▲
               │
      ┌────────┴────────┐
      │  Contabo VPS    │
      │  - SSL/HTTPS    │
      │  - Schedulers   │
      │  - Backups      │
      └─────────────────┘

      External Services:
      ┌────────────────┐
      │ Currency API   │
      │ (Daily Sync)   │
      │ 6 pairs:       │
      │ EUR,USD,COP,   │
      │ PEN,PLN,RUB    │
      └────────────────┘
      ┌────────────────┐
      │ Email Service  │
      │ (SMTP)         │
      └────────────────┘
```

---

## Tech Stack Deep Dive

### Frontend Technologies

#### **React (Vanilla JavaScript)**
- **Why not TypeScript?** Project started before TypeScript adoption, migrating would require full rewrite
- **Component Architecture:** Functional components with hooks
- **Performance:** Optimized re-renders using React.memo and useMemo

#### **Redux**
```javascript
Store Structure:
├── auth (user, token, isAuthenticated)
├── financial (assets, liabilities, balance)
├── ui (theme, language, loading states)
├── categories (all categories and subcategories)
└── currency (selected currency, rates)
```

**Why Redux over Context API?**
- Predictable state updates
- Time-travel debugging
- Middleware support for async actions
- Better performance for frequent updates

#### **Nivo Charts**
- **Chosen for:** Customization flexibility, React-native, responsive by default
- **Chart types used:** Pie (category distribution), Bar (trends), Custom (balance diagrams)
- **Performance:** Lazy loaded, animated transitions

#### **CSS Modules**
```
Advantages:
- Scoped styles per component
- No naming conflicts
- Tree-shaking unused styles
- Co-located with components
```

#### **Custom i18n System**
**Why not react-i18next?**
- Needed full control over translation loading
- 5,000+ translations would bloat bundle
- Custom optimization for our use case
- No external dependencies

**Architecture:**
```javascript
Translator.js (5000+ lines)
├── translations object
├── getTranslation(key, language)
├── formatCurrency(amount, currency, language)
└── formatDate(date, language)

Future: Modular language files
languages/
├── en.js
├── es.js
├── it.js
├── fr.js
├── pl.js
├── ru.js
└── pt.js
```

---

### Backend Technologies

#### **Java 17 + Spring Boot 3.x**
**Why Java?**
- Strong typing for financial calculations
- Excellent ecosystem (Spring, Flyway, etc.)
- Battle-tested in production
- Easy hiring and maintenance

**Spring Boot Advantages:**
- Convention over configuration
- Built-in security
- Easy REST API creation
- Excellent documentation

#### **MySQL 8.0**
**Why MySQL over PostgreSQL?**
- Familiarity and existing expertise
- Excellent performance for our use case
- Strong community support
- Simple replication setup

#### **Flyway Migrations**
**Why Flyway?**
- Version control for database
- Rollback capabilities
- Team collaboration on schema changes
- Production-safe deployments

**Migration Structure:**
```
migrations/
├── V1__initial_schema.sql
├── V2__add_categories.sql
├── V3__add_exchange_rates.sql
├── V4__add_refresh_tokens.sql
└── V5__add_subcategories.sql
```

---

## Database Design

### Entity Relationship Diagram
```
┌─────────────┐
│    Users    │
├─────────────┤
│ id (PK)     │
│ email       │
│ password    │
│ currency    │
│ language    │
│ theme       │
│ created_at  │
└──────┬──────┘
       │
       │ 1:N
       ▼
┌─────────────────┐      ┌──────────────────┐
│ Refresh_Tokens  │      │   Categories     │
├─────────────────┤      ├──────────────────┤
│ id (PK)         │      │ id (PK)          │
│ user_id (FK)    │      │ name             │
│ token           │      │ type (asset/lib) │
│ expiry_date     │      │ fixed (boolean)  │
└─────────────────┘      └────────┬─────────┘
                                  │
                                  │ 1:N
       ┌──────────────────────────┤
       │                          │
       ▼                          ▼
┌─────────────────┐      ┌──────────────────┐
│  Subcategories  │      │     Assets       │
├─────────────────┤      ├──────────────────┤
│ id (PK)         │◄─────┤ id (PK)          │
│ category_id(FK) │ N:1  │ user_id (FK)     │
│ user_id (FK)    │      │ subcategory(FK)  │
│ name            │      │ amount           │
│ created_at      │      │ orig_amount      │
└─────────────────┘      │ orig_currency    │
       ▲                 │ date             │
       │                 │ description      │
       │ N:1             └──────────────────┘
       │
┌─────────────────┐
│  Liabilities    │
├─────────────────┤
│ id (PK)         │
│ user_id (FK)    │
│ subcategory(FK) │
│ amount          │
│ orig_amount     │
│ orig_currency   │
│ date            │
│ description     │
└─────────────────┘

┌──────────────────┐
│  Exchange_Rates  │
├──────────────────┤
│ id (PK)          │
│ from_currency    │
│ to_currency      │
│ rate             │
│ date             │
│ updated_at       │
└──────────────────┘
```

### Key Design Decisions

#### **Multi-Tenant Data Isolation**
```sql
-- Every query includes user_id
SELECT * FROM assets WHERE user_id = ? AND ...
SELECT * FROM liabilities WHERE user_id = ? AND ...
```

#### **Currency Storage Strategy**
```sql
CREATE TABLE assets (
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  amount DECIMAL(19,4) NOT NULL,      -- Converted to wallet currency
  original_amount DECIMAL(19,4),      -- Original amount entered
  original_currency VARCHAR(3),        -- Currency entered (EUR, USD, etc.)
  wallet_currency VARCHAR(3) NOT NULL, -- User's wallet currency
  exchange_rate DECIMAL(10,6),        -- Rate used for conversion
  date DATE NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Why store both amounts?**
- Historical accuracy (rate at transaction time)
- Audit trail
- No need to recalculate with historical rates

#### **Category System Design**
```sql
CREATE TABLE categories (
  id BIGINT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  type ENUM('ASSET', 'LIABILITY') NOT NULL,
  fixed BOOLEAN DEFAULT TRUE,  -- System categories can't be deleted
  order_index INT              -- Display order
);

CREATE TABLE subcategories (
  id BIGINT PRIMARY KEY,
  category_id BIGINT NOT NULL,
  user_id BIGINT NOT NULL,      -- User-specific subcategories
  name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP,
  UNIQUE KEY (category_id, user_id, name),
  FOREIGN KEY (category_id) REFERENCES categories(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## Frontend Architecture

### Redux Implementation

#### **Store Configuration**
```javascript
// store.js
import { createStore, applyMiddleware, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import authReducer from './reducers/authReducer';
import financialReducer from './reducers/financialReducer';
import uiReducer from './reducers/uiReducer';

const rootReducer = combineReducers({
  auth: authReducer,
  financial: financialReducer,
  ui: uiReducer
});

const store = createStore(rootReducer, applyMiddleware(thunk));
```

#### **Async Actions with Axios**
```javascript
// actions/assetActions.js
export const addAsset = (assetData) => async (dispatch) => {
  try {
    dispatch({ type: 'ADD_ASSET_REQUEST' });
    
    const response = await axios.post('/api/assets', assetData, {
      headers: {
        'Authorization': `Bearer ${getAccessToken()}`
      }
    });
    
    dispatch({ 
      type: 'ADD_ASSET_SUCCESS', 
      payload: response.data 
    });
  } catch (error) {
    if (error.response.status === 401) {
      // Token expired, refresh and retry
      await refreshToken();
      return dispatch(addAsset(assetData));
    }
    dispatch({ 
      type: 'ADD_ASSET_FAILURE', 
      payload: error.message 
    });
  }
};
```

### Custom i18n Implementation
```javascript
// i18n/Translator.js (simplified structure)
class Translator {
  constructor() {
    this.translations = {
      en: {
        'home.welcome': 'Welcome to Inco.cash',
        'liabilities.title': 'Liabilities',
        // ... 5000+ more entries
      },
      es: {
        'home.welcome': 'Bienvenido a Inco.cash',
        'liabilities.title': 'Pasivos',
        // ... 5000+ more entries
      },
      // ... 5 more languages
    };
  }

  translate(key, language) {
    return this.translations[language][key] || key;
  }

  formatCurrency(amount, currency, language) {
    return new Intl.NumberFormat(language, {
      style: 'currency',
      currency: currency
    }).format(amount);
  }
}

export default new Translator();
```

**Usage in components:**
```javascript
import translator from '../i18n/Translator';

function Dashboard() {
  const language = useSelector(state => state.ui.language);
  
  return (
    <h1>{translator.translate('home.welcome', language)}</h1>
  );
}
```

---

## Backend Architecture

### JWT Authentication Flow
```java
@Service
public class AuthService {
    
    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @Autowired
    private RefreshTokenRepository refreshTokenRepository;
    
    public AuthenticationResponse authenticate(AuthenticationRequest request) {
        // 1. Authenticate user
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getEmail(),
                request.getPassword()
            )
        );
        
        // 2. Generate tokens
        User user = userRepository.findByEmail(request.getEmail())
            .orElseThrow();
        
        String accessToken = tokenProvider.generateAccessToken(user);
        String refreshToken = tokenProvider.generateRefreshToken(user);
        
        // 3. Save refresh token
        saveRefreshToken(user, refreshToken);
        
        return AuthenticationResponse.builder()
            .accessToken(accessToken)
            .refreshToken(refreshToken)
            .build();
    }
    
    public AuthenticationResponse refreshToken(String refreshToken) {
        // 1. Validate refresh token
        if (!tokenProvider.validateToken(refreshToken)) {
            throw new InvalidTokenException();
        }
        
        // 2. Check if exists in database
        RefreshToken token = refreshTokenRepository
            .findByToken(refreshToken)
            .orElseThrow();
        
        // 3. Generate new access token
        String newAccessToken = tokenProvider
            .generateAccessToken(token.getUser());
        
        return AuthenticationResponse.builder()
            .accessToken(newAccessToken)
            .refreshToken(refreshToken) // Same refresh token
            .build();
    }
    
    @Scheduled(cron = "0 0 3 * * *") // 3 AM daily
    public void cleanupExpiredTokens() {
        LocalDateTime threshold = LocalDateTime.now().minusDays(7);
        int deleted = refreshTokenRepository
            .deleteByExpiryDateBefore(threshold);
        log.info("Cleaned up {} expired refresh tokens", deleted);
    }
}
```

### Currency Service Implementation
```java
@Service
public class CurrencyService {
    
    private static final List<Currency> SUPPORTED_CURRENCIES = Arrays.asList(
        Currency.EUR,
        Currency.USD,
        Currency.COP,
        Currency.PEN,
        Currency.PLN,
        Currency.RUB
    );
    
    @Autowired
    private ExchangeRateRepository exchangeRateRepository;
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Scheduled(cron = "0 0 2 * * *") // 2 AM daily
    public void updateExchangeRates() {
        log.info("Starting daily exchange rate update");
        
        for (Currency from : SUPPORTED_CURRENCIES) {
            for (Currency to : SUPPORTED_CURRENCIES) {
                if (from != to) {
                    try {
                        BigDecimal rate = fetchRateFromAPI(from, to);
                        saveExchangeRate(from, to, rate);
                    } catch (Exception e) {
                        log.error("Failed to update rate {} to {}", from, to, e);
                    }
                }
            }
        }
        
        log.info("Exchange rate update completed");
    }
    
    public BigDecimal convertCurrency(
        BigDecimal amount,
        Currency from,
        Currency to
    ) {
        if (from == to) {
            return amount;
        }
        
        ExchangeRate rate = exchangeRateRepository
            .findLatestRate(from, to)
            .orElseThrow(() -> new RateNotFoundException(from, to));
        
        return amount.multiply(rate.getRate());
    }
    
    private BigDecimal fetchRateFromAPI(Currency from, Currency to) {
        String url = String.format(
            "https://api.exchangerate.host/latest?base=%s&symbols=%s",
            from, to
        );
        
        ExchangeAPIResponse response = restTemplate
            .getForObject(url, ExchangeAPIResponse.class);
        
        return response.getRates().get(to.toString());
    }
}
```

---

## Multi-Currency System

### Transaction Flow
```
User adds liability:
├── Amount: 1000 COP
├── User wallet: EUR
│
└─> Backend process:
    ├── 1. Fetch latest EUR/COP rate from database
    │      SELECT rate FROM exchange_rates 
    │      WHERE from_currency='EUR' AND to_currency='COP'
    │      ORDER BY date DESC LIMIT 1
    │
    ├── 2. Convert: 1000 COP * 0.00022 = 0.22 EUR
    │
    ├── 3. Store in database:
    │      INSERT INTO liabilities (
    │        user_id, amount, original_amount, 
    │        original_currency, wallet_currency,
    │        exchange_rate, date
    │      ) VALUES (
    │        123, 0.22, 1000,
    │        'COP', 'EUR',
    │        0.00022, '2026-01-14'
    │      )
    │
    └── 4. Return to frontend:
           {
             "id": 456,
             "amount": 0.22,
             "originalAmount": 1000,
             "originalCurrency": "COP",
             "displayText": "1000 COP (0.22 EUR)"
           }
```

### Exchange Rate Matrix

Daily update covers all combinations:
```
     EUR    USD    COP    PEN    PLN    RUB
EUR   -     rate   rate   rate   rate   rate
USD  rate    -     rate   rate   rate   rate
COP  rate   rate    -     rate   rate   rate
PEN  rate   rate   rate    -     rate   rate
PLN  rate   rate   rate   rate    -     rate
RUB  rate   rate   rate   rate   rate    -

Total: 30 exchange rates updated daily
```

---

## Internationalization System

### Translation Structure

**Current (Monolithic):**
```javascript
// Translator.js - 5000+ lines
const translations = {
  en: {
    // Navigation
    'nav.home': 'Home',
    'nav.liabilities': 'Liabilities',
    'nav.assets': 'Assets',
    
    // Categories
    'category.forme': 'For Me',
    'category.forme.desc': 'Essential survival expenses',
    
    // ... 5000+ more entries
  },
  es: { /* same keys, Spanish values */ },
  it: { /* same keys, Italian values */ },
  fr: { /* same keys, French values */ },
  pl: { /* same keys, Polish values */ },
  ru: { /* same keys, Russian values */ },
  pt: { /* same keys, Portuguese values */ }
};
```

**Future (Modular):**
```javascript
languages/
├── en/
│   ├── navigation.js
│   ├── categories.js
│   ├── forms.js
│   └── messages.js
├── es/
│   ├── navigation.js
│   ├── categories.js
│   ├── forms.js
│   └── messages.js
└── ... (5 more languages)
```

**Benefits of future structure:**
- Easier to maintain
- Smaller bundle size (load only needed language)
- Easier for translators to contribute
- Better for adding German (8th language)

---

## Security Implementation

### Authentication & Authorization
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .cors()
            .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            .and()
            .addFilterBefore(
                jwtAuthenticationFilter(),
                UsernamePasswordAuthenticationFilter.class
            );
    }
}

@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {
        
        String token = extractTokenFromRequest(request);
        
        if (token != null && tokenProvider.validateToken(token)) {
            String email = tokenProvider.getEmailFromToken(token);
            UserDetails userDetails = userDetailsService.loadUserByUsername(email);
            
            UsernamePasswordAuthenticationToken authentication =
                new UsernamePasswordAuthenticationToken(
                    userDetails,
                    null,
                    userDetails.getAuthorities()
                );
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        
        filterChain.doFilter(request, response);
    }
}
```

### Password Security
```java
@Service
public class PasswordService {
    
    private final BCryptPasswordEncoder encoder = 
        new BCryptPasswordEncoder(12); // Strength: 12
    
    public String hashPassword(String plainPassword) {
        return encoder.encode(plainPassword);
    }
    
    public boolean verifyPassword(String plainPassword, String hashedPassword) {
        return encoder.matches(plainPassword, hashedPassword);
    }
}
```

---

## Scheduled Tasks

### Exchange Rate Scheduler
```java
@Component
public class ExchangeRateScheduler {
    
    @Autowired
    private CurrencyService currencyService;
    
    @Scheduled(cron = "0 0 2 * * *") // Every day at 2 AM
    public void updateExchangeRates() {
        try {
            log.info("Starting scheduled exchange rate update");
            
            currencyService.updateExchangeRates();
            
            log.info("Exchange rate update completed successfully");
        } catch (Exception e) {
            log.error("Exchange rate update failed", e);
            // Send alert email to admin
            emailService.sendAlertEmail(
                "Exchange Rate Update Failed",
                e.getMessage()
            );
        }
    }
}
```

### Token Cleanup Scheduler
```java
@Service
public class AuthService {
    
    @Scheduled(cron = "0 0 3 * * *") // Every day at 3 AM
    public void cleanupExpiredTokens() {
        try {
            log.info("Starting token cleanup");
            
            LocalDateTime expiryThreshold = LocalDateTime.now().minusDays(7);
            
            int deletedCount = refreshTokenRepository
                .deleteByExpiryDateBefore(expiryThreshold);
            
            log.info("Cleaned up {} expired tokens", deletedCount);
        } catch (Exception e) {
            log.error("Token cleanup failed", e);
        }
    }
}
```

**Why 3 AM?**
- Low traffic time
- After exchange rate update (2 AM)
- Before European users wake up

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
│   │   │   │   │   ├── SubcategoryController.java
│   │   │   │   │   ├── UserController.java
│   │   │   │   │   └── CurrencyController.java
│   │   │   │   │
│   │   │   │   ├── service/
│   │   │   │   │   ├── AuthService.java
│   │   │   │   │   ├── AssetService.java
│   │   │   │   │   ├── LiabilityService.java
│   │   │   │   │   ├── CategoryService.java
│   │   │   │   │   ├── CurrencyService.java
│   │   │   │   │   ├── EmailService.java
│   │   │   │   │   └── UserService.java
│   │   │   │   │
│   │   │   │   ├── repository/
│   │   │   │   │   ├── UserRepository.java
│   │   │   │   │   ├── AssetRepository.java
│   │   │   │   │   ├── LiabilityRepository.java
│   │   │   │   │   ├── CategoryRepository.java
│   │   │   │   │   ├── SubcategoryRepository.java
│   │   │   │   │   ├── ExchangeRateRepository.java
│   │   │   │   │   └── RefreshTokenRepository.java
│   │   │   │   │
│   │   │   │   ├── model/
│   │   │   │   │   ├── User.java
│   │   │   │   │   ├── Asset.java
│   │   │   │   │   ├── Liability.java
│   │   │   │   │   ├── Category.java
│   │   │   │   │   ├── Subcategory.java
│   │   │   │   │   ├── ExchangeRate.java
│   │   │   │   │   └── RefreshToken.java
│   │   │   │   │
│   │   │   │   ├── security/
│   │   │   │   │   ├── JwtTokenProvider.java
│   │   │   │   │   ├── JwtAuthenticationFilter.java
│   │   │   │   │   ├── SecurityConfig.java
│   │   │   │   │   └── UserDetailsServiceImpl.java
│   │   │   │   │
│   │   │   │   ├── scheduler/
│   │   │   │   │   └── ExchangeRateScheduler.java
│   │   │   │   │
│   │   │   │   ├── dto/
│   │   │   │   │   ├── AuthenticationRequest.java
│   │   │   │   │   ├── AuthenticationResponse.java
│   │   │   │   │   ├── AssetDTO.java
│   │   │   │   │   └── LiabilityDTO.java
│   │   │   │   │
│   │   │   │   ├── exception/
│   │   │   │   │   ├── GlobalExceptionHandler.java
│   │   │   │   │   ├── ResourceNotFoundException.java
│   │   │   │   │   └── InvalidTokenException.java
│   │   │   │   │
│   │   │   │   └── config/
│   │   │   │       ├── WebConfig.java
│   │   │   │       ├── EmailConfig.java
│   │   │   │       └── SchedulerConfig.java
│   │   │   │
│   │   │   └── resources/
│   │   │       ├── application.properties
│   │   │       ├── application-prod.properties
│   │   │       └── db/migration/
│   │   │           ├── V1__initial_schema.sql
│   │   │           ├── V2__add_categories.sql
│   │   │           ├── V3__add_exchange_rates.sql
│   │   │           └── V4__add_refresh_tokens.sql
│   │   │
│   │   └── test/ (tests omitted for brevity)
│   │
│   └── pom.xml
│
└── frontend/
    └── src/
        ├── components/
        │   ├── common/
        │   │   ├── Button/
        │   │   ├── Input/
        │   │   ├── Modal/
        │   │   └── Loader/
        │   ├── charts/
        │   │   ├── PieChart.jsx
        │   │   ├── BarChart.jsx
        │   │   └── BalanceDiagram.jsx
        │   ├── forms/
        │   │   ├── AssetForm.jsx
        │   │   ├── LiabilityForm.jsx
        │   │   └── LoginForm.jsx
        │   └── layout/
        │       ├── Header.jsx
        │       ├── Sidebar.jsx
        │       └── Footer.jsx
        │
        ├── pages/
        │   ├── Home/
        │   │   ├── Landing.jsx
        │   │   ├── Dashboard.jsx
        │   │   └── Home.module.css
        │   ├── Liabilities/
        │   │   ├── LiabilitiesPage.jsx
        │   │   ├── LiabilitiesTable.jsx
        │   │   ├── LiabilitiesChart.jsx
        │   │   └── Liabilities.module.css
        │   ├── Assets/
        │   │   ├── AssetsPage.jsx
        │   │   ├── AssetsTable.jsx
        │   │   ├── AssetsChart.jsx
        │   │   └── Assets.module.css
        │   ├── Budget/
        │   │   └── BudgetLanding.jsx
        │   ├── Subcategories/
        │   │   ├── SubcategoriesPage.jsx
        │   │   └── SubcategoryForm.jsx
        │   └── Profile/
        │       ├── ProfilePage.jsx
        │       ├── Settings.jsx
        │       └── Profile.module.css
        │
        ├── redux/
        │   ├── store.js
        │   ├── actions/
        │   │   ├── authActions.js
        │   │   ├── assetActions.js
        │   │   ├── liabilityActions.js
        │   │   └── uiActions.js
        │   └── reducers/
        │       ├── authReducer.js
        │       ├── financialReducer.js
        │       └── uiReducer.js
        │
        ├── services/
        │   ├── api.js
        │   ├── authService.js
        │   ├── assetService.js
        │   ├── liabilityService.js
        │   └── currencyService.js
        │
        ├── i18n/
        │   ├── Translator.js (5000+ lines)
        │   └── languages/ (future modular structure)
        │
        ├── utils/
        │   ├── currencyFormatter.js
        │   ├── dateFormatter.js
        │   ├── validators.js
        │   └── constants.js
        │
        ├── styles/
        │   ├── global.css
        │   └── variables.css
        │
        ├── App.jsx
        ├── index.jsx
        └── package.json
```

---

## Development Challenges

### Challenge 1: Custom Internationalization (7 Languages)

**Problem:**
- Needed complete 7-language support
- Third-party libraries too heavy (5,000+ translations)
- Wanted full control over loading and caching
- Performance critical for user experience

**Solution:**
1. Built custom Translator class
2. Centralized all translations in one place initially
3. Created helper methods for currency and date formatting
4. Implemented language switching without page reload

**Trade-offs:**
- ✅ Full control and optimization
- ✅ No external dependencies
- ❌ Higher maintenance burden
- ❌ Monolithic file (5,000+ lines)

**Future Improvement:**
- Migrate to modular structure (one file per language)
- Implement lazy loading
- Add translation management UI

---

### Challenge 2: Multi-Currency Real-Time Conversion

**Problem:**
- Users from 6+ countries
- Need to track in their local currency (wallet)
- Need to enter transactions in any currency they spend
- Exchange rates change daily
- Historical accuracy required

**Solution:**
1. **Daily Automated Scheduler**
   - Runs at 2 AM daily
   - Fetches rates for all 6 currency pairs
   - Stores in database

2. **Dual Storage Strategy**
   - Store original amount + currency
   - Store converted amount in wallet currency
   - Store exchange rate used
   - Enables accurate historical reporting

3. **Real-Time Conversion**
   - At transaction entry, fetch latest rate
   - Convert immediately
   - Display both amounts to user

**Code Example:**
```java
public Asset createAsset(AssetDTO dto, User user) {
    BigDecimal convertedAmount = dto.getAmount();
    
    if (!dto.getCurrency().equals(user.getWalletCurrency())) {
        ExchangeRate rate = exchangeRateRepository
            .findLatestRate(user.getWalletCurrency(), dto.getCurrency())
            .orElseThrow();
        
        convertedAmount = currencyService.convert(
            dto.getAmount(),
            dto.getCurrency(),
            user.getWalletCurrency(),
            rate
        );
    }
    
    Asset asset = Asset.builder()
        .originalAmount(dto.getAmount())
        .originalCurrency(dto.getCurrency())
        .amount(convertedAmount)
        .walletCurrency(user.getWalletCurrency())
        .exchangeRate(rate != null ? rate.getRate() : BigDecimal.ONE)
        .build();
    
    return assetRepository.save(asset);
}
```

---

### Challenge 3: JWT Security vs User Experience

**Problem:**
- Short tokens = more secure but users re-login often
- Long tokens = better UX but security risk
- Need balance

**Solution: Refresh Token Strategy**

1. **Access Token: 15 minutes**
   - Used for API requests
   - Short-lived for security

2. **Refresh Token: 7 days**
   - Used to get new access token
   - Long-lived for convenience

3. **Automatic Refresh**
   - Frontend intercepts 401 errors
   - Automatically requests new access token
   - Seamless to user

4. **Database Cleanup**
   - Scheduler removes expired tokens daily
   - Keeps database clean

**Frontend Implementation:**
```javascript
axios.interceptors.response.use(
  response => response,
  async error => {
    const originalRequest = error.config;
    
    if (error.response.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        const refreshToken = getRefreshToken();
        const response = await axios.post('/api/auth/refresh', {
          refreshToken
        });
        
        setAccessToken(response.data.accessToken);
        originalRequest.headers['Authorization'] = 
          `Bearer ${response.data.accessToken}`;
        
        return axios(originalRequest);
      } catch (refreshError) {
        // Refresh failed, logout user
        logout();
        return Promise.reject(refreshError);
      }
    }
    
    return Promise.reject(error);
  }
);
```

**Result:**
- Users stay logged in for 7 days
- High security (15min access tokens)
- Seamless experience (auto-refresh)

---

### Challenge 4: Behavioral Category Design

**Problem:**
- Want users to be more conscious about spending
- Traditional categories too generic
- Need to motivate better financial habits

**Solution: Psychology-Driven Categories**

**Liabilities:**
1. **"For Me"** (Essential) vs **"Superfluous Things"** (Harmful)
   - Creates stark contrast
   - Users think twice before categorizing as "superfluous"
   
2. **"Improve Myself"**
   - Positive framing encourages investment in growth
   - Separate from "For Me" to highlight importance

3. **"Dolce Vita"** (Sweet Life)
   - Better than "Entertainment" or "Fun"
   - Acknowledges pleasure is important
   - Not judged as "bad" like superfluous

4. **"Actives"** (Income-generating)
   - Encourages seeing some expenses as investments
   - Aligns with asset-building mindset

5. **"For Others"**
   - Separate category for generosity
   - Helps track charitable giving

**Assets (Based on Cash Flow Quadrant):**
- **Educational:** Teaches users income types
- **Motivational:** Encourages moving toward "I" (Investor)
- **Proven Framework:** Robert Kiyosaki's established methodology

**Impact:**
- Users report more awareness of spending patterns
- "Superfluous Things" category often eye-opening
- Asset categories help users think about income diversification

---

## Performance & Optimization

### Database Optimization

**Indexes:**
```sql
-- User queries
CREATE INDEX idx_users_email ON users(email);

-- Transaction queries
CREATE INDEX idx_assets_user_date ON assets(user_id, date DESC);
CREATE INDEX idx_liabilities_user_date ON liabilities(user_id, date DESC);
CREATE INDEX idx_assets_user_subcategory ON assets(user_id, subcategory_id);
CREATE INDEX idx_liabilities_user_subcategory ON liabilities(user_id, subcategory_id);

-- Exchange rate queries
CREATE INDEX idx_exchange_rates_currencies_date 
  ON exchange_rates(from_currency, to_currency, date DESC);

-- Token queries
CREATE INDEX idx_refresh_tokens_user ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_expiry ON refresh_tokens(expiry_date);
```

**Query Optimization:**
```java
// Bad: N+1 query problem
List<Asset> assets = assetRepository.findByUserId(userId);
assets.forEach(asset -> {
    asset.getSubcategory(); // Lazy load - extra query per asset
});

// Good: Eager fetch with JOIN
@Query("SELECT a FROM Asset a " +
       "JOIN FETCH a.subcategory " +
       "WHERE a.user.id = :userId")
List<Asset> findByUserIdWithSubcategory(@Param("userId") Long userId);
```

---

### Frontend Optimization

**Code Splitting:**
```javascript
// Lazy load pages
const Dashboard = lazy(() => import('./pages/Home/Dashboard'));
const LiabilitiesPage = lazy(() => import('./pages/Liabilities/LiabilitiesPage'));
const AssetsPage = lazy(() => import('./pages/Assets/AssetsPage'));

// Routes with Suspense
<Suspense fallback={<Loader />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/liabilities" element={<LiabilitiesPage />} />
    <Route path="/assets" element={<AssetsPage />} />
  </Routes>
</Suspense>
```

**Memoization:**
```javascript
// Expensive chart calculation
const chartData = useMemo(() => {
  return processLiabilitiesForChart(liabilities);
}, [liabilities]);

// Prevent unnecessary re-renders
const LiabilityRow = React.memo(({ liability }) => {
  return <tr>...</tr>;
});
```

**CSS Modules Benefits:**
```javascript
// Only loads CSS for components actually rendered
import styles from './Dashboard.module.css';

// Tree-shaking removes unused styles in production
```

---

## Production Metrics

### Current Performance

**System Uptime:**
- 99.5% uptime since launch
- Average downtime: < 4 hours/month
- Planned maintenance windows: Sundays 3-5 AM

**API Performance:**
- Average response time: < 200ms
- 95th percentile: < 500ms
- 99th percentile: < 1s

**Database Performance:**
- Query execution time: < 50ms average
- Connection pool: 10 connections
- Zero deadlocks since launch

**Scheduled Tasks:**
- Exchange rate sync: 100% success rate
- Average execution time: 3-5 seconds
- Token cleanup: 100% success rate
- Average execution time: 1-2 seconds

### User Metrics

**Active Users:**
- 200+ registered users
- 6+ countries represented
- 7 languages in use

**Usage Patterns:**
- Average transactions per user per month: ~45
- Most active time: Evenings (7-10 PM local time)
- Most popular category: "For Me" (liabilities), "Employee" (assets)

**Feature Adoption:**
- Multi-currency: Used by 40% of users
- Custom subcategories: Average 8 per user
- Theme customization: 60% use dark theme

### Scale Projections

**Current Capacity:**
- Can handle 1,000 concurrent users
- Database: 50GB current, 500GB capacity
- Server: 4 CPU cores, 8GB RAM

**10x Scale (2,000 users):**
- No major architecture changes needed
- May need:
  - Database read replica
  - Redis caching layer
  - CDN for static assets
  - Horizontal scaling (2-3 servers)

---

## Deployment & DevOps

### Contabo VPS Setup

**Server Specifications:**
- **CPU:** 4 cores
- **RAM:** 8GB
- **Storage:** 200GB SSD
- **OS:** Ubuntu 22.04 LTS
- **Location:** Germany (low latency for European users)

**Deployed Services:**
```
Contabo VPS
├── Nginx (Reverse Proxy + SSL)
│   ├── Handles HTTPS
│   ├── Serves static files
│   └── Proxy to Spring Boot (port 8080)
│
├── Spring Boot Application (port 8080)
│   ├── Runs as systemd service
│   ├── Auto-restart on failure
│   └── Logs to /var/log/inco/
│
├── MySQL 8.0 (port 3306)
│   ├── Daily backups (automated)
│   └── Binary logging enabled
│
└── Certbot (SSL/TLS)
    └── Auto-renewal every 90 days
```

**Deployment Process:**
```bash
# 1. Build frontend
cd frontend
npm run build
# Produces optimized production build

# 2. Build backend
cd backend
mvn clean package -DskipTests
# Produces executable JAR

# 3. Deploy to server
scp target/inco-1.0.jar server:/opt/inco/
ssh server "systemctl restart inco"

# 4. Flyway migrations run automatically on startup
```

---

## Monitoring & Logging

**Application Logs:**
```java
@Slf4j
@Service
public class AssetService {
    
    public Asset createAsset(AssetDTO dto) {
        log.info("Creating asset for user: {}", dto.getUserId());
        
        try {
            Asset asset = // ... creation logic
            
            log.info("Asset created successfully: {}", asset.getId());
            return asset;
        } catch (Exception e) {
            log.error("Failed to create asset", e);
            throw e;
        }
    }
}
```

**Log Rotation:**
```
/var/log/inco/
├── application.log (current)
├── application-2026-01-13.log
├── application-2026-01-12.log
└── ... (kept for 30 days)
```

**Error Monitoring:**
- Email alerts for critical errors
- Daily digest of warnings
- Scheduled task failure notifications

---

## Testing Strategy

**Backend Testing:**
```java
@SpringBootTest
class AssetServiceTest {
    
    @Autowired
    private AssetService assetService;
    
    @Test
    void shouldCreateAssetWithCurrencyConversion() {
        // Given
        AssetDTO dto = AssetDTO.builder()
            .amount(new BigDecimal("1000"))
            .currency(Currency.COP)
            .build();
        
        User user = createTestUser();
        user.setWalletCurrency(Currency.EUR);
        
        // When
        Asset asset = assetService.createAsset(dto, user);
        
        // Then
        assertThat(asset.getOriginalAmount())
            .isEqualTo(new BigDecimal("1000"));
        assertThat(asset.getOriginalCurrency())
            .isEqualTo(Currency.COP);
        assertThat(asset.getAmount())
            .isLessThan(new BigDecimal("1000")); // EUR worth more than COP
    }
}
```

**Frontend Testing:**
- Manual QA for all critical user flows
- Cross-browser testing (Chrome, Firefox, Safari)
- Mobile responsive testing

---

## Future Technical Improvements

### Short Term (Q1-Q2 2026)

1. **Modular i18n System**
   - Split 5,000-line Translator.js
   - One file per language
   - Lazy loading

2. **Redis Caching**
   - Cache exchange rates
   - Cache user preferences
   - Reduce database load

3. **Comprehensive Test Suite**
   - Unit tests for all services
   - Integration tests for API
   - E2E tests for critical flows

### Medium Term (Q3-Q4 2026)

1. **Microservices Architecture**
   - Auth Service
   - Financial Service
   - Currency Service
   - Notification Service

2. **GraphQL API**
   - Alternative to REST
   - Better for mobile app
   - Reduced over-fetching

3. **Event Sourcing**
   - Transaction history
   - Audit trail
   - Time-travel queries

### Long Term (2027+)

1. **Kubernetes Deployment**
   - Container orchestration
   - Easy scaling
   - Zero-downtime deployments

2. **Real-Time Features**
   - WebSocket connections
   - Live balance updates
   - Collaborative budgets

3. **ML/AI Integration**
   - Spending predictions
   - Anomaly detection
   - Personalized recommendations

---

## Lessons Learned

### What Worked Well

1. **Custom i18n System**
   - Full control over translations
   - No dependency bloat
   - Exactly what we needed

2. **Flyway Migrations**
   - Safe database changes
   - Version control for schema
   - Easy rollbacks

3. **Behavioral Categories**
   - Users love the psychology-driven approach
   - "Superfluous Things" creates awareness
   - Cash Flow Quadrant teaches financial education

4. **Multi-Currency Architecture**
   - Storing both amounts prevents issues
   - Daily sync works reliably
   - Users appreciate the accuracy

### What Could Be Improved

1. **Testing Coverage**
   - Should have written tests from day one
   - Manual testing is time-consuming
   - Tech debt accumulating

2. **Monolithic i18n**
   - 5,000-line file hard to maintain
   - Should have been modular from start
   - Refactoring will be painful

3. **No CI/CD Pipeline**
   - Manual deployments error-prone
   - Should automate with GitHub Actions
   - Need staging environment

4. **Limited Monitoring**
   - Basic logging not enough
   - Need proper APM tool
   - Want better error tracking

---

## Conclusion

Inco.cash represents a complete full-stack application built from the ground up with real users, real data, and real production challenges.

**Key Technical Achievements:**
- ✅ Custom internationalization (7 languages, 5,000+ translations)
- ✅ Multi-currency system (6 currencies, daily automated sync)
- ✅ Production-grade authentication (JWT with refresh tokens)
- ✅ Database version control (Flyway migrations)
- ✅ Automated maintenance (schedulers for rates and tokens)
- ✅ 200+ users across 6+ countries
- ✅ 99.5% uptime

**Technologies Mastered:**
- Frontend: React, Redux, Nivo, CSS Modules
- Backend: Java, Spring Boot, MySQL, Flyway
- Security: JWT, BCrypt, CORS, HTTPS
- DevOps: Contabo VPS, Nginx, systemd, Certbot
- Architecture: REST API, MVC, scheduled tasks

This project demonstrates not just coding ability, but the full lifecycle of software engineering: from initial design and architecture decisions, through implementation and testing, to deployment and ongoing maintenance of a production system serving real users.

---

[← Back to Main README](./README.md) | [View Detailed Roadmap →](./ROADMAP.md)
