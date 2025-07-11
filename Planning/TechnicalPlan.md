# Mealio Technical Plan

## Project Overview
Mealio is a mobile-first meal planning application that helps users manage their weekly food needs through recipe management, meal planning, grocery shopping, and cooking workflows. The project uses a shared-core architecture to support both mobile (React Native) and web (Next.js) platforms.

## Architecture Overview

### Monorepo Structure
```
mealio/
   apps/
      mobile/          # React Native (Expo)
      web/             # Next.js PWA
   packages/
      core/            # Business logic & state management
      ui/              # Cross-platform design system
      adapters/        # Device capability wrappers
      api/             # Backend API
   services/
      recipe-scraper/  # Recipe ingestion service
      notifications/   # Push notification service
   infra/               # Infrastructure as code
```

### Tech Stack
- **Frontend**: React Native (Expo), Next.js, TypeScript
- **Backend**: Node.js, Express/Fastify, PostgreSQL
- **Authentication**: Supabase Auth
- **Database**: PostgreSQL via Supabase
- **Deployment**: Vercel (frontend), Railway (backend)
- **State Management**: Zustand
- **Monorepo**: Turborepo + pnpm workspaces
- **End clients**: iOS, Android, Web

## Implementation Phases

### Phase 1: Core MVP (P0 Features)
**Goal**: Deliver essential meal planning workflow

#### 1.1 Project Setup & Infrastructure
- Initialize monorepo with Turborepo
- Set up workspace structure
- Configure shared packages (@mealio/core, @mealio/ui, @mealio/adapters)
- Set up development environment and CI/CD
- Deploy basic infrastructure (database, auth)

#### 1.2 Backend Core
- Design and implement database schema
- Set up authentication system
- Create REST API endpoints for:
  - User management
  - Recipe CRUD operations
  - Meal planning operations
  - Shopping list generation

#### 1.3 Recipe Ingestion System
- Implement recipe URL scraping using metadata parsing
- Integrate sharp-recipe-parser for ingredient extraction
- Create manual recipe entry system
- Build recipe storage and retrieval

#### 1.4 Core Mobile App
- Set up React Native project with Expo
- Implement navigation structure
- Create core screens:
  - Recipe library
  - Recipe detail view
  - Meal planning queue
  - Shopping list
  - Cooking queue

#### 1.5 Core Web App
- Set up Next.js project with PWA capabilities
- Implement responsive design
- Create web versions of core screens
- Ensure feature parity with mobile

#### 1.6 Shared Business Logic
- Implement state management with Zustand
- Create shared hooks and utilities
- Build recipe usage tracking
- Develop grocery list generation logic

#### 1.7 Testing & Polish
- Comprehensive testing (unit, integration, e2e)
- Bug fixes and performance optimization
- User experience polish
- Deployment preparation

### Phase 2: Automations & Integrations (P1 Features)
**Goal**: Add smart features and external integrations

#### 2.1 Calendar Integration
- Implement calendar API integration
- Build availability insight feature
- Create weekly planning nudges

#### 2.2 Recipe Recommendations
- Develop recommendation algorithm
- Create swipe-based recipe selection UI
- Implement user preference tracking

#### 2.3 Shopping Integrations
- Build Instacart export functionality
- Implement grocery list categorization
- Add third-party grocery API integration

#### 2.4 Enhanced Notifications
- Implement push notification system
- Create planning reminder workflows
- Add cooking timer notifications

### Phase 3: AI & Advanced Features (P2 Features)
**Goal**: Add intelligent features and advanced capabilities

#### 3.1 AI Recipe Generation
- Integrate AI service for recipe creation
- Build preference-based recipe generation
- Implement pantry gap analysis

#### 3.2 Advanced Recommendations
- Implement ML-based recommendation engine
- Add dietary restriction support
- Create seasonal recipe suggestions

#### 3.3 Advanced Features
- Add recipe tagging and filtering
- Implement pantry inventory tracking
- Create meal plan templates

## Detailed Implementation Steps

### Phase 1 Detailed Steps

#### 1.1 Project Setup
```bash
# Initialize monorepo
npx create-turbo@latest mealio
cd mealio

# Configure package.json
# Set up pnpm workspaces
# Initialize packages
pnpm create @mealio/core
pnpm create @mealio/ui
pnpm create @mealio/adapters

# Set up apps
npx create-expo-app@latest apps/mobile
npx create-next-app@latest apps/web

# Configure TypeScript
# Set up ESLint and Prettier
# Configure Turborepo pipeline
```

#### 1.2 Database Schema Design
```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Recipes table
CREATE TABLE recipes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  instructions TEXT[],
  prep_time INTEGER,
  cook_time INTEGER,
  servings INTEGER,
  source_url VARCHAR(500),
  image_url VARCHAR(500),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Ingredients table
CREATE TABLE ingredients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) UNIQUE NOT NULL,
  category VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Recipe ingredients junction table
CREATE TABLE recipe_ingredients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipe_id UUID REFERENCES recipes(id) ON DELETE CASCADE,
  ingredient_id UUID REFERENCES ingredients(id),
  quantity DECIMAL(10,2),
  unit VARCHAR(50),
  notes TEXT
);

-- Meal plans table
CREATE TABLE meal_plans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  week_start DATE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Planned meals table
CREATE TABLE planned_meals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  meal_plan_id UUID REFERENCES meal_plans(id) ON DELETE CASCADE,
  recipe_id UUID REFERENCES recipes(id),
  planned_date DATE,
  meal_type VARCHAR(50), -- breakfast, lunch, dinner, snack
  completed BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Shopping lists table
CREATE TABLE shopping_lists (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  meal_plan_id UUID REFERENCES meal_plans(id),
  user_id UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Shopping list items table
CREATE TABLE shopping_list_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  shopping_list_id UUID REFERENCES shopping_lists(id) ON DELETE CASCADE,
  ingredient_id UUID REFERENCES ingredients(id),
  quantity DECIMAL(10,2),
  unit VARCHAR(50),
  checked BOOLEAN DEFAULT FALSE,
  category VARCHAR(100)
);

-- Recipe usage tracking
CREATE TABLE recipe_usage (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  recipe_id UUID REFERENCES recipes(id),
  used_at TIMESTAMP DEFAULT NOW(),
  rating INTEGER CHECK (rating >= 1 AND rating <= 5)
);
```

#### 1.3 API Endpoints Design
```typescript
// Authentication
POST /auth/login
POST /auth/register
POST /auth/logout
GET /auth/me

// Recipes
GET /api/recipes
POST /api/recipes
GET /api/recipes/:id
PUT /api/recipes/:id
DELETE /api/recipes/:id
POST /api/recipes/scrape

// Meal Planning
GET /api/meal-plans
POST /api/meal-plans
GET /api/meal-plans/:id
PUT /api/meal-plans/:id
DELETE /api/meal-plans/:id

// Shopping Lists
GET /api/shopping-lists
POST /api/shopping-lists/generate
GET /api/shopping-lists/:id
PUT /api/shopping-lists/:id
DELETE /api/shopping-lists/:id

// Recipe Usage
POST /api/recipes/:id/usage
GET /api/recipes/usage-history
```

#### 1.4 Core Components Architecture
```typescript
// @mealio/core package structure
src/
 stores/
    auth.ts
    recipes.ts
    mealPlans.ts
    shoppingLists.ts
 hooks/
    useAuth.ts
    useRecipes.ts
    useMealPlanning.ts
 services/
    api.ts
    recipeParser.ts     # Custom recipe parsing implementation
    storage.ts
 types/
    recipe.ts
    mealPlan.ts
    user.ts
 utils/
     dateHelpers.ts
     recipeHelpers.ts

### Recipe Parsing Implementation

#### Frontend Processing
- **Library**: sharp-recipe-parser (v0.3.1)
- **Purpose**: Client-side parsing of recipe text
- **Key Features**:
  - Ingredient quantity and unit extraction
  - Instruction parsing and normalization
  - Support for various measurement formats
  - Custom parsing rules for edge cases

#### Backend Processing
- **Library**: recipe-scrapers
- **Purpose**: Server-side web scraping of recipe websites
- **Key Features**:
  - Extracts structured recipe data from various sources
  - Handles different website layouts and formats
  - Normalizes data from multiple recipe schemas
  - Includes fallback mechanisms for unsupported sites

#### Data Flow
1. User submits a recipe URL or text
2. For URLs, backend uses recipe-scrapers to fetch and parse the recipe
3. For text input, frontend uses sharp-recipe-parser for immediate feedback
4. All parsed recipes are normalized to Mealio's standard format
5. Data is enriched with additional metadata and stored
```

#### 1.5 Mobile App Structure
```typescript
// apps/mobile structure
src/
 screens/
    auth/
    recipes/
    mealPlanning/
    shopping/
    cooking/
 components/
    common/
    recipe/
    mealPlan/
 navigation/
    AppNavigator.tsx
    AuthNavigator.tsx
    TabNavigator.tsx
 services/
     notifications.ts
     camera.ts
   screens/
      auth/
      recipes/
      mealPlanning/
      shopping/
      cooking/
   components/
      common/
      recipe/
      mealPlan/
   navigation/
      AppNavigator.tsx
      AuthNavigator.tsx
      TabNavigator.tsx
   services/
       notifications.ts
       camera.ts
```

#### 1.6 Testing Strategy
```typescript
// Testing approach
- Unit tests: Jest + React Testing Library
- Integration tests: API endpoints
- E2E tests: Detox (mobile), Playwright (web)
- Manual testing: TestFlight (iOS), Play Console (Android)

// Test coverage targets
- Core business logic: 90%+
- API endpoints: 85%+
- UI components: 70%+
```

## Risk Mitigation

### Technical Risks
1. **Recipe Scraping Reliability**: Implement fallback to manual entry, multiple parsing strategies
2. **Cross-platform Consistency**: Extensive testing, shared component library
3. **Performance**: Lazy loading, efficient state management, caching strategies
4. **Data Synchronization**: Offline-first approach, conflict resolution

### Product Risks
1. **User Adoption**: Comprehensive onboarding, MVP validation
2. **Feature Complexity**: Phased rollout, user feedback integration
3. **Third-party Dependencies**: Fallback options, vendor evaluation

## Success Metrics

### Technical Metrics
- App load time < 2 seconds
- API response time < 500ms
- Crash rate < 1%
- Test coverage > 80%

### Product Metrics
- Recipe scraping success rate > 95%
- Shopping list generation accuracy > 90%
- User retention (7 days) > 70%
- Average recipes per user > 10

## Deployment Strategy

### Development Environment
- Feature branches with PR reviews
- Automated testing on PR
- Staging environment for integration testing

### Production Deployment
- Blue-green deployment for backend
- App store releases for mobile
- CDN deployment for web app
- Database migrations with rollback capability

## Monitoring & Observability

### Application Monitoring
- Error tracking: Sentry
- Performance monitoring: DataDog/New Relic
- User analytics: Mixpanel/Amplitude
- Uptime monitoring: Pingdom

### Infrastructure Monitoring
- Server metrics: CPU, memory, disk
- Database performance: Query time, connections
- API metrics: Response times, error rates
- User experience: Core Web Vitals

## Future Considerations

### Scalability
- Database sharding strategy
- CDN for recipe images
- Microservices architecture
- Caching layer (Redis)

### International Expansion
- Localization framework
- Currency conversion
- Regional grocery chains
- Dietary preferences by region

### Advanced Features
- Voice commands
- Meal plan sharing
- Social features
- Nutrition tracking
- Meal prep optimization

This technical plan provides a comprehensive roadmap for building Mealio from MVP to full-featured application, with clear phases, detailed implementation steps, and risk mitigation strategies.