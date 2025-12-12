# Frontend Architecture Options

## Overview

This document analyzes frontend technology options for the SaaS platform, comparing frameworks, architectures, and implementation approaches. The goal is to build a **flexible, nimble frontend** that delivers excellent user experience while remaining maintainable and scalable.

## Key Requirements

### Functional Requirements
- ✅ **Multi-tenant UI** - Tenant-specific branding and configuration
- ✅ **Responsive design** - Desktop, tablet, mobile support
- ✅ **Real-time updates** - WebSocket support for live data
- ✅ **Rich interactions** - Drag-and-drop, charts, dashboards
- ✅ **Accessibility** - WCAG 2.1 AA compliance
- ✅ **Internationalization** - Multi-language support

### Non-Functional Requirements
- ⚡ **Performance** - < 2s initial load, < 100ms interactions
- 🔒 **Security** - XSS protection, CSP, secure authentication
- 📱 **Mobile-first** - Progressive Web App (PWA) capabilities
- ♿ **Accessibility** - Screen reader support, keyboard navigation
- 🎨 **Customization** - White-label branding per tenant

---

## Framework Comparison

### Option 1: React + TypeScript (Recommended)

**Architecture:** Single Page Application (SPA) with React 18+

```
saas-platform-frontend/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── common/          # Buttons, inputs, modals
│   │   ├── skills/          # Skills-specific components
│   │   ├── assessments/     # Assessment components
│   │   └── layout/          # Layout components
│   ├── pages/               # Page-level components
│   │   ├── Dashboard/
│   │   ├── Profile/
│   │   ├── Assessments/
│   │   └── Admin/
│   ├── hooks/               # Custom React hooks
│   ├── services/            # API clients
│   ├── store/               # State management (Redux/Zustand)
│   ├── utils/               # Utility functions
│   ├── types/               # TypeScript types
│   └── styles/              # Global styles
├── public/
└── package.json
```

**Tech Stack:**
- **Framework:** React 18 with Concurrent Features
- **Language:** TypeScript 5+
- **State Management:** Zustand (lightweight) or Redux Toolkit
- **Routing:** React Router v6
- **UI Library:** Material-UI (MUI) or Ant Design
- **Forms:** React Hook Form + Zod validation
- **Data Fetching:** TanStack Query (React Query)
- **Charts:** Recharts or Apache ECharts
- **Build Tool:** Vite
- **Testing:** Vitest + React Testing Library

**Pros:**
- ✅ **Huge ecosystem** - Extensive libraries and community
- ✅ **TypeScript support** - Excellent type safety
- ✅ **Performance** - Virtual DOM, concurrent rendering
- ✅ **Developer experience** - Hot reload, great tooling
- ✅ **Hiring** - Large talent pool
- ✅ **Component reusability** - Easy to build design system

**Cons:**
- ❌ **Learning curve** - Hooks, state management complexity
- ❌ **Bundle size** - Can be large without optimization
- ❌ **SEO** - Requires SSR for public pages

**Code Example:**
```typescript
// src/pages/Dashboard/SkillsOverview.tsx
import { useQuery } from '@tanstack/react-query';
import { Card, Grid, Typography } from '@mui/material';
import { SkillsChart } from '@/components/skills/SkillsChart';
import { useTenant } from '@/hooks/useTenant';
import { skillsService } from '@/services/skills';

export const SkillsOverview: React.FC = () => {
  const { tenantId } = useTenant();
  
  const { data: skills, isLoading } = useQuery({
    queryKey: ['skills', tenantId],
    queryFn: () => skillsService.getSkills(tenantId),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
  
  if (isLoading) return <LoadingSpinner />;
  
  return (
    <Grid container spacing={3}>
      <Grid item xs={12}>
        <Typography variant="h4">Skills Overview</Typography>
      </Grid>
      <Grid item xs={12} md={6}>
        <Card>
          <SkillsChart data={skills} />
        </Card>
      </Grid>
    </Grid>
  );
};
```

**Recommended For:** ✅ **Best overall choice** - Balanced flexibility, performance, and ecosystem

---

### Option 2: Next.js (React with SSR)

**Architecture:** Hybrid SSR/SSG/CSR with Next.js 14+ (App Router)

```
saas-platform-frontend/
├── app/
│   ├── (auth)/              # Auth layout group
│   │   ├── login/
│   │   └── signup/
│   ├── (dashboard)/         # Dashboard layout group
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── profile/
│   │   ├── assessments/
│   │   └── admin/
│   ├── api/                 # API routes (BFF pattern)
│   │   ├── skills/
│   │   └── assessments/
│   └── layout.tsx
├── components/
├── lib/
└── public/
```

**Tech Stack:**
- **Framework:** Next.js 14+ (App Router)
- **Language:** TypeScript
- **State Management:** React Context + Server Components
- **Styling:** Tailwind CSS + CSS Modules
- **Data Fetching:** Server Components + SWR
- **Auth:** NextAuth.js
- **Deployment:** Vercel or AWS Amplify

**Pros:**
- ✅ **SEO-friendly** - Server-side rendering
- ✅ **Performance** - Automatic code splitting, image optimization
- ✅ **Developer experience** - File-based routing, API routes
- ✅ **Server components** - Reduced client bundle size
- ✅ **Deployment** - Optimized for Vercel

**Cons:**
- ❌ **Complexity** - SSR/CSR mental model
- ❌ **Vendor lock-in** - Best on Vercel
- ❌ **Learning curve** - App Router is new paradigm

**Code Example:**
```typescript
// app/(dashboard)/skills/page.tsx
import { SkillsTable } from '@/components/skills/SkillsTable';
import { getSkills } from '@/lib/skills';

export default async function SkillsPage() {
  // Server component - data fetched on server
  const skills = await getSkills();
  
  return (
    <div>
      <h1>Skills Directory</h1>
      <SkillsTable skills={skills} />
    </div>
  );
}

// app/api/skills/route.ts (BFF pattern)
import { NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';

export async function GET(request: Request) {
  const session = await getServerSession();
  const tenantId = session?.user?.tenantId;
  
  const response = await fetch(
    `${process.env.API_URL}/api/v1/skills`,
    {
      headers: {
        'X-Tenant-ID': tenantId,
        'Authorization': `Bearer ${session?.accessToken}`
      }
    }
  );
  
  const skills = await response.json();
  return NextResponse.json(skills);
}
```

**Recommended For:** ✅ **Public-facing pages** or **SEO-critical** applications

---

### Option 3: Vue.js 3 (Composition API)

**Architecture:** SPA with Vue 3 Composition API

**Tech Stack:**
- **Framework:** Vue 3 with Composition API
- **Language:** TypeScript
- **State Management:** Pinia
- **Routing:** Vue Router 4
- **UI Library:** Vuetify or Element Plus
- **Build Tool:** Vite

**Pros:**
- ✅ **Gentle learning curve** - Easier than React
- ✅ **Performance** - Smaller bundle size
- ✅ **Reactivity** - Built-in reactive system
- ✅ **Single File Components** - HTML/CSS/JS in one file

**Cons:**
- ❌ **Smaller ecosystem** - Fewer libraries than React
- ❌ **Hiring** - Smaller talent pool
- ❌ **Enterprise adoption** - Less common in enterprise

**Code Example:**
```vue
<!-- src/pages/Dashboard.vue -->
<template>
  <div class="dashboard">
    <h1>{{ greeting }}</h1>
    <SkillsChart :data="skills" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useSkillsStore } from '@/stores/skills';
import SkillsChart from '@/components/SkillsChart.vue';

const skillsStore = useSkillsStore();
const skills = computed(() => skillsStore.skills);
const greeting = computed(() => `Welcome, ${skillsStore.userName}`);

onMounted(async () => {
  await skillsStore.fetchSkills();
});
</script>

<style scoped>
.dashboard {
  padding: 2rem;
}
</style>
```

**Recommended For:** ⚠️ **Alternative** if team has Vue expertise

---

### Option 4: Angular

**Architecture:** Full-featured framework with TypeScript

**Tech Stack:**
- **Framework:** Angular 17+
- **Language:** TypeScript (required)
- **State Management:** NgRx or Akita
- **UI Library:** Angular Material
- **Forms:** Reactive Forms

**Pros:**
- ✅ **Enterprise-ready** - Opinionated, batteries-included
- ✅ **TypeScript native** - First-class TypeScript support
- ✅ **Dependency injection** - Built-in DI system
- ✅ **RxJS** - Powerful reactive programming

**Cons:**
- ❌ **Steep learning curve** - Complex framework
- ❌ **Verbose** - More boilerplate code
- ❌ **Bundle size** - Larger than React/Vue
- ❌ **Declining popularity** - Losing market share

**Recommended For:** ⚠️ **Only if** team has strong Angular background

---

## Recommended Architecture: React + TypeScript

### Technology Stack

| Layer | Technology | Justification |
|-------|-----------|---------------|
| **Framework** | React 18 | Industry standard, huge ecosystem |
| **Language** | TypeScript 5+ | Type safety, better DX |
| **Build Tool** | Vite | Fast builds, HMR |
| **State Management** | Zustand | Lightweight, simple API |
| **Routing** | React Router v6 | Standard routing solution |
| **UI Components** | Material-UI (MUI) | Comprehensive, customizable |
| **Forms** | React Hook Form | Performance, DX |
| **Validation** | Zod | TypeScript-first validation |
| **Data Fetching** | TanStack Query | Caching, optimistic updates |
| **Charts** | Recharts | React-native, composable |
| **Tables** | TanStack Table | Headless, flexible |
| **Date/Time** | date-fns | Lightweight, tree-shakeable |
| **HTTP Client** | Axios | Interceptors, cancellation |
| **WebSocket** | Socket.io-client | Real-time updates |
| **Testing** | Vitest + RTL | Fast, modern testing |
| **E2E Testing** | Playwright | Cross-browser testing |
| **Linting** | ESLint + Prettier | Code quality |
| **CSS** | CSS Modules + MUI | Scoped styles, theme system |

### Project Structure

```
saas-platform-frontend/
├── public/
│   ├── favicon.ico
│   └── assets/
├── src/
│   ├── app/
│   │   ├── App.tsx
│   │   ├── Router.tsx
│   │   └── theme.ts
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   ├── Modal/
│   │   │   └── Table/
│   │   ├── layout/
│   │   │   ├── Header/
│   │   │   ├── Sidebar/
│   │   │   └── Footer/
│   │   ├── skills/
│   │   │   ├── SkillCard/
│   │   │   ├── SkillsChart/
│   │   │   └── SkillsTable/
│   │   └── assessments/
│   │       ├── AssessmentForm/
│   │       └── AssessmentResults/
│   ├── pages/
│   │   ├── Dashboard/
│   │   ├── Profile/
│   │   ├── Skills/
│   │   ├── Assessments/
│   │   ├── Teams/
│   │   └── Admin/
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useTenant.ts
│   │   └── useWebSocket.ts
│   ├── services/
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   ├── skills.ts
│   │   │   ├── assessments.ts
│   │   │   └── profiles.ts
│   │   └── websocket/
│   │       └── socket.ts
│   ├── store/
│   │   ├── authStore.ts
│   │   ├── tenantStore.ts
│   │   └── uiStore.ts
│   ├── types/
│   │   ├── api.ts
│   │   ├── models.ts
│   │   └── enums.ts
│   ├── utils/
│   │   ├── formatters.ts
│   │   ├── validators.ts
│   │   └── constants.ts
│   ├── styles/
│   │   ├── global.css
│   │   └── variables.css
│   └── main.tsx
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── .env.example
├── vite.config.ts
├── tsconfig.json
├── package.json
└── README.md
```

---

## Key Implementation Patterns

### 1. Multi-Tenancy

**Tenant Context Provider:**
```typescript
// src/contexts/TenantContext.tsx
import { createContext, useContext, ReactNode } from 'react';

interface TenantContextType {
  tenantId: string;
  tenantName: string;
  tenantConfig: TenantConfig;
  branding: BrandingConfig;
}

const TenantContext = createContext<TenantContextType | null>(null);

export const TenantProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [tenant, setTenant] = useState<TenantContextType | null>(null);
  
  useEffect(() => {
    // Extract tenant from subdomain or JWT
    const subdomain = window.location.hostname.split('.')[0];
    fetchTenantConfig(subdomain).then(setTenant);
  }, []);
  
  if (!tenant) return <LoadingScreen />;
  
  return (
    <TenantContext.Provider value={tenant}>
      {children}
    </TenantContext.Provider>
  );
};

export const useTenant = () => {
  const context = useContext(TenantContext);
  if (!context) throw new Error('useTenant must be used within TenantProvider');
  return context;
};
```

**Dynamic Theming:**
```typescript
// src/app/theme.ts
import { createTheme } from '@mui/material/styles';
import { useTenant } from '@/contexts/TenantContext';

export const useCustomTheme = () => {
  const { branding } = useTenant();
  
  return createTheme({
    palette: {
      primary: {
        main: branding.primaryColor || '#1976d2',
      },
      secondary: {
        main: branding.secondaryColor || '#dc004e',
      },
    },
    typography: {
      fontFamily: branding.fontFamily || 'Roboto, sans-serif',
    },
    components: {
      MuiButton: {
        styleOverrides: {
          root: {
            borderRadius: branding.borderRadius || 4,
          },
        },
      },
    },
  });
};
```

### 2. Authentication

**Auth Hook:**
```typescript
// src/hooks/useAuth.ts
import { useAuthStore } from '@/store/authStore';
import { authService } from '@/services/api/auth';

export const useAuth = () => {
  const { user, token, setAuth, clearAuth } = useAuthStore();
  
  const login = async (email: string, password: string) => {
    const response = await authService.login(email, password);
    setAuth(response.user, response.token);
    return response;
  };
  
  const logout = () => {
    authService.logout();
    clearAuth();
  };
  
  const isAuthenticated = !!token;
  
  return { user, token, login, logout, isAuthenticated };
};
```

**Protected Route:**
```typescript
// src/components/ProtectedRoute.tsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';

export const ProtectedRoute: React.FC<{ children: ReactNode }> = ({ children }) => {
  const { isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return <>{children}</>;
};
```

### 3. API Integration

**API Client:**
```typescript
// src/services/api/client.ts
import axios from 'axios';
import { useAuthStore } from '@/store/authStore';
import { useTenantStore } from '@/store/tenantStore';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
});

// Request interceptor
apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token;
  const tenantId = useTenantStore.getState().tenantId;
  
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  
  if (tenantId) {
    config.headers['X-Tenant-ID'] = tenantId;
  }
  
  return config;
});

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      useAuthStore.getState().clearAuth();
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

**React Query Integration:**
```typescript
// src/services/api/skills.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import apiClient from './client';
import { Skill } from '@/types/models';

export const useSkills = () => {
  return useQuery({
    queryKey: ['skills'],
    queryFn: async () => {
      const { data } = await apiClient.get<Skill[]>('/api/v1/skills');
      return data;
    },
  });
};

export const useCreateSkill = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (skill: Partial<Skill>) => {
      const { data } = await apiClient.post('/api/v1/skills', skill);
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['skills'] });
    },
  });
};
```

### 4. Real-Time Updates

**WebSocket Hook:**
```typescript
// src/hooks/useWebSocket.ts
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';
import { useAuth } from './useAuth';

export const useWebSocket = () => {
  const [socket, setSocket] = useState<Socket | null>(null);
  const { token } = useAuth();
  
  useEffect(() => {
    if (!token) return;
    
    const newSocket = io(import.meta.env.VITE_WS_URL, {
      auth: { token },
    });
    
    setSocket(newSocket);
    
    return () => {
      newSocket.close();
    };
  }, [token]);
  
  const subscribe = (event: string, callback: (data: any) => void) => {
    socket?.on(event, callback);
    return () => socket?.off(event, callback);
  };
  
  const emit = (event: string, data: any) => {
    socket?.emit(event, data);
  };
  
  return { subscribe, emit, isConnected: socket?.connected };
};
```

---

## Performance Optimization

### 1. Code Splitting

```typescript
// Lazy load pages
const Dashboard = lazy(() => import('@/pages/Dashboard'));
const Profile = lazy(() => import('@/pages/Profile'));
const Assessments = lazy(() => import('@/pages/Assessments'));

// Router with Suspense
<Suspense fallback={<LoadingSpinner />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/profile" element={<Profile />} />
    <Route path="/assessments" element={<Assessments />} />
  </Routes>
</Suspense>
```

### 2. Memoization

```typescript
const SkillsTable = memo(({ skills }: { skills: Skill[] }) => {
  const sortedSkills = useMemo(
    () => skills.sort((a, b) => a.name.localeCompare(b.name)),
    [skills]
  );
  
  return <Table data={sortedSkills} />;
});
```

### 3. Virtual Scrolling

```typescript
import { useVirtualizer } from '@tanstack/react-virtual';

const VirtualSkillsList = ({ skills }: { skills: Skill[] }) => {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: skills.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });
  
  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px` }}>
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div key={virtualRow.index} style={{ height: '50px' }}>
            {skills[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
};
```

---

## Deployment

### Build Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom', 'react-router-dom'],
          'vendor-mui': ['@mui/material', '@mui/icons-material'],
          'vendor-query': ['@tanstack/react-query'],
        },
      },
    },
  },
});
```

### AWS CloudFront + S3 Deployment

```bash
# Build for production
npm run build

# Deploy to S3
aws s3 sync dist/ s3://saas-platform-frontend --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

---

## Summary

**Recommended Stack:** React + TypeScript + Vite + MUI

**Key Benefits:**
- ✅ Flexible and nimble architecture
- ✅ Excellent developer experience
- ✅ Strong type safety with TypeScript
- ✅ Rich ecosystem and community
- ✅ Easy to hire developers
- ✅ Performance optimizations built-in

**Next Steps:**
1. Review [BACKEND_OPTIONS.md](file:///Users/rekhasunil/Documents/Sunil/poc-antigravity/saas_platform/docs/BACKEND_OPTIONS.md) for backend choices
2. See [TECHNICAL_DESIGN.md](file:///Users/rekhasunil/Documents/Sunil/poc-antigravity/saas_platform/docs/TECHNICAL_DESIGN.md) for implementation details
3. Check [DEPLOYMENT.md](file:///Users/rekhasunil/Documents/Sunil/poc-antigravity/saas_platform/docs/DEPLOYMENT.md) for deployment guide
