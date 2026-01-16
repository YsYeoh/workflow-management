# Amendment: UI Framework, Theme Switcher, Localization, API Documentation, and Developer Tools

## Summary of Change

This amendment extends the existing multi-tenant Workflow Management System plan to include:

1. **UI Framework Integration**: React Flow for workflow visualization/designer and Tailwind CSS for styling
2. **Theme Switcher**: Dark/light mode support with system preference detection
3. **Localization (i18n)**: Multi-language support for UI and API responses
4. **Swagger API Documentation**: Auto-generated OpenAPI/Swagger documentation for all API endpoints
5. **Developer Tools**: Storybook for component documentation, playground for testing, and example public page

**Impact Level**: Medium - Additive changes, no breaking modifications to existing backend architecture.

## Affected Components

### New Components
- UI layer (previously backend-first, now full-stack)
- Workflow Designer UI (React Flow integration)
- Theme management system
- Localization infrastructure
- API documentation generation
- Developer tooling (Storybook, playground)

### Modified Components
- `app/` directory structure (adds UI pages and components)
- `lib/` utilities (adds i18n helpers, theme utilities)
- `config/` (adds theme and i18n configuration)
- API routes (adds OpenAPI annotations)

### Unchanged Components
- Backend core (`core/workflow`, `core/iam`, `core/tenant`)
- Database schema (no changes)
- Workflow engine logic
- Multi-tenancy enforcement
- API business logic

## Proposed Additions / Modifications

### Backend

#### 1. API Documentation (Swagger/OpenAPI)

**New Files**:
```
/src
├── app
│   ├── api
│   │   └── docs
│   │       └── route.ts          # Swagger UI endpoint
│   └── openapi.json              # Generated OpenAPI spec
├── lib
│   └── openapi
│       ├── generator.ts          # OpenAPI spec generator
│       ├── decorators.ts         # Route decorators for metadata
│       └── schemas.ts            # Shared schema definitions
```

**Implementation**:
- Use `swagger-jsdoc` or `next-swagger-doc` to generate OpenAPI spec from JSDoc comments
- Add OpenAPI annotations to all API route handlers
- Generate JSON schema from Zod validators
- Serve Swagger UI at `/api/docs`

**Example Route Annotation**:
```typescript
// app/api/workflows/instances/route.ts
/**
 * @swagger
 * /api/v1/workflows/instances:
 *   post:
 *     summary: Create a new workflow instance
 *     tags: [Workflows]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/CreateWorkflowInstance'
 *     responses:
 *       201:
 *         description: Workflow instance created
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/WorkflowInstance'
 */
export async function POST(request: Request) {
  // ... existing implementation
}
```

**New Dependencies**:
- `swagger-jsdoc` - Generate OpenAPI from JSDoc
- `swagger-ui-react` - Swagger UI component
- `@apidevtools/swagger-jsdoc` - Alternative OpenAPI generator

#### 2. Localization Support (Backend)

**New Files**:
```
/src
├── lib
│   └── i18n
│       ├── server.ts             # Server-side i18n
│       ├── messages.ts           # Translation messages
│       └── locale-detector.ts    # Locale detection from headers
├── locales
│   ├── en
│   │   ├── common.json
│   │   ├── errors.json
│   │   └── api.json
│   ├── es
│   │   └── ...
│   └── fr
│       └── ...
```

**Implementation**:
- Detect locale from `Accept-Language` header or query parameter
- Return localized error messages in API responses
- Support tenant-specific locale preferences (stored in tenant config)

**API Response Enhancement**:
```typescript
// lib/http/response.ts
export function successResponse<T>(
  data: T,
  locale: string = 'en',
  meta?: object
) {
  return {
    success: true,
    data,
    meta: {
      ...meta,
      locale,
      timestamp: new Date().toISOString(),
    },
  };
}

export function errorResponse(
  code: string,
  message: string,
  locale: string = 'en',
  details?: object
) {
  const localizedMessage = i18n.t(`errors.${code}`, { locale });
  return {
    success: false,
    error: {
      code,
      message: localizedMessage || message,
      details,
    },
    meta: { locale },
  };
}
```

**New Dependencies**:
- `next-intl` or `i18next` - Internationalization framework
- `accept-language-parser` - Parse Accept-Language header

### Database

**No schema changes required**. However, consider adding:

**Optional Enhancement** (for tenant-specific locale preferences):
```typescript
// tenants collection - add to config
{
  config: {
    // ... existing config
    locale: 'en', // Default locale for tenant
    supportedLocales: ['en', 'es', 'fr'], // Available locales
  }
}
```

### Workflow Engine

**No changes to core workflow engine logic**.

**Enhancement**: Workflow definitions can include localized labels:
```typescript
interface WorkflowDefinition {
  // ... existing fields
  labels?: {
    [locale: string]: {
      name: string;
      description: string;
      states: { [stateId: string]: { label: string } };
    };
  };
}
```

### UI / Workflow Designer

#### 1. React Flow Integration

**New Files**:
```
/src
├── app
│   ├── (ui)
│   │   ├── workflow-designer
│   │   │   └── page.tsx          # Workflow designer page
│   │   ├── dashboard
│   │   │   └── page.tsx          # Main dashboard
│   │   └── layout.tsx            # UI layout with Tailwind CSS
│   └── layout.tsx                # Root layout
├── components
│   ├── workflow
│   │   ├── FlowCanvas.tsx        # React Flow canvas
│   │   ├── StateNode.tsx         # Custom state node component
│   │   ├── TransitionEdge.tsx    # Custom transition edge
│   │   ├── NodePanel.tsx         # Node properties panel
│   │   └── Toolbar.tsx           # Designer toolbar
│   ├── common
│   │   ├── ThemeProvider.tsx     # Theme context provider
│   │   ├── ThemeSwitcher.tsx     # Theme toggle component
│   │   └── LocaleSwitcher.tsx    # Language switcher
│   └── ui
│       └── ...                    # Tailwind CSS styled components
├── hooks
│   ├── useTheme.ts               # Theme hook
│   ├── useLocale.ts              # Locale hook
│   └── useWorkflowDesigner.ts    # Workflow designer logic
├── styles
│   ├── themes
│   │   ├── light.ts              # Light theme tokens
│   │   ├── dark.ts               # Dark theme tokens
│   │   └── index.ts              # Theme configuration
│   └── globals.css               # Global styles
└── lib
    └── i18n
        └── client.ts             # Client-side i18n
```

**React Flow Implementation**:
```typescript
// components/workflow/FlowCanvas.tsx
'use client';

import ReactFlow, {
  Node,
  Edge,
  Background,
  Controls,
  MiniMap,
} from 'reactflow';
import 'reactflow/dist/style.css';
import { StateNode } from './StateNode';
import { TransitionEdge } from './TransitionEdge';

const nodeTypes = {
  state: StateNode,
};

const edgeTypes = {
  transition: TransitionEdge,
};

export function FlowCanvas({
  nodes,
  edges,
  onNodesChange,
  onEdgesChange,
  onConnect,
}: WorkflowDesignerProps) {
  return (
    <ReactFlow
      nodes={nodes}
      edges={edges}
      nodeTypes={nodeTypes}
      edgeTypes={edgeTypes}
      onNodesChange={onNodesChange}
      onEdgesChange={onEdgesChange}
      onConnect={onConnect}
      fitView
    >
      <Background />
      <Controls />
      <MiniMap />
    </ReactFlow>
  );
}
```

**New Dependencies**:
- `reactflow` - React Flow library
- `@reactflow/core` - Core React Flow types
- `tailwindcss` - Utility-first CSS framework
- `@headlessui/react` - Unstyled, accessible UI components (optional)
- `lucide-react` or `heroicons` - Icon library
- `zustand` or `jotai` - State management for designer state

#### 2. Tailwind CSS Integration

**Configuration**:
```typescript
// app/(ui)/layout.tsx
'use client';

import { ThemeProvider } from '@/components/common/ThemeProvider';
import { useTheme } from '@/hooks/useTheme';
import '@/styles/globals.css';

export default function UILayout({ children }: { children: React.ReactNode }) {
  const { resolvedTheme } = useTheme();
  
  return (
    <ThemeProvider>
      <div className={`theme-${resolvedTheme}`}>
        {children}
      </div>
    </ThemeProvider>
  );
}
```

**Tailwind Configuration**:
```javascript
// tailwind.config.js
module.exports = {
  darkMode: ['class', '[data-theme="dark"]'],
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        // Custom theme colors
      },
    },
  },
  plugins: [],
};
```

#### 3. Theme Switcher

**Implementation**:
```typescript
// hooks/useTheme.ts
'use client';

import { create } from 'zustand';
import { persist } from 'zustand/middleware';

type Theme = 'light' | 'dark' | 'system';

interface ThemeStore {
  theme: Theme;
  resolvedTheme: 'light' | 'dark';
  setTheme: (theme: Theme) => void;
}

export const useTheme = create<ThemeStore>()(
  persist(
    (set, get) => ({
      theme: 'system',
      resolvedTheme: 'light',
      setTheme: (theme: Theme) => {
        const resolvedTheme = 
          theme === 'system'
            ? window.matchMedia('(prefers-color-scheme: dark)').matches
              ? 'dark'
              : 'light'
            : theme;
        
        set({ theme, resolvedTheme });
        document.documentElement.setAttribute('data-theme', resolvedTheme);
      },
    }),
    { name: 'theme-storage' }
  )
);
```

**Component**:
```typescript
// components/common/ThemeSwitcher.tsx
'use client';

import { MoonIcon, SunIcon, ComputerDesktopIcon } from '@heroicons/react/24/outline';
import { useTheme } from '@/hooks/useTheme';

export function ThemeSwitcher() {
  const { theme, setTheme } = useTheme();
  
  return (
    <div className="relative inline-block">
      <button
        className="flex items-center gap-2 px-4 py-2 rounded-lg bg-gray-100 dark:bg-gray-800 hover:bg-gray-200 dark:hover:bg-gray-700 transition-colors"
        onClick={() => {
          const themes: Theme[] = ['light', 'dark', 'system'];
          const currentIndex = themes.indexOf(theme);
          const nextTheme = themes[(currentIndex + 1) % themes.length];
          setTheme(nextTheme);
        }}
      >
        {theme === 'light' && <SunIcon className="w-5 h-5" />}
        {theme === 'dark' && <MoonIcon className="w-5 h-5" />}
        {theme === 'system' && <ComputerDesktopIcon className="w-5 h-5" />}
        <span className="capitalize">{theme}</span>
      </button>
    </div>
  );
}
```

**New Dependencies**:
- `zustand` - State management
- `next-themes` (optional) - Alternative theme management

#### 4. Localization (Frontend)

**Implementation**:
```typescript
// lib/i18n/client.ts
'use client';

import { create } from 'next-intl/create';
import { getRequestConfig } from 'next-intl/server';

export const locales = ['en', 'es', 'fr'] as const;
export type Locale = (typeof locales)[number];

export default getRequestConfig(async ({ locale }) => ({
  messages: (await import(`../../locales/${locale}/common.json`)).default,
}));
```

**Usage in Components**:
```typescript
// components/workflow/FlowCanvas.tsx
'use client';

import { useTranslations } from 'next-intl';

export function FlowCanvas() {
  const t = useTranslations('workflow');
  
  return (
    <div>
      <h1>{t('designer.title')}</h1>
      {/* ... */}
    </div>
  );
}
```

**Locale Switcher Component**:
```typescript
// components/common/LocaleSwitcher.tsx
'use client';

import { useLocale, useTranslations } from 'next-intl';
import { useRouter, usePathname } from 'next/navigation';

export function LocaleSwitcher() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();
  const t = useTranslations('common');
  
  const handleChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    const newLocale = e.target.value;
    router.replace(pathname.replace(`/${locale}`, `/${newLocale}`));
  };
  
  return (
    <select
      value={locale}
      onChange={handleChange}
      className="px-4 py-2 rounded-lg border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-800 text-gray-900 dark:text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
    >
      <option value="en">English</option>
      <option value="es">Español</option>
      <option value="fr">Français</option>
    </select>
  );
}
```

**New Dependencies**:
- `next-intl` - Next.js internationalization
- `@formatjs/intl-localematcher` - Locale matching

#### 5. Storybook

**New Files**:
```
/.storybook
├── main.ts                        # Storybook configuration
├── preview.tsx                    # Global decorators
└── theme.ts                       # Storybook theme
/stories
├── workflow
│   ├── FlowCanvas.stories.tsx
│   ├── StateNode.stories.tsx
│   └── TransitionEdge.stories.tsx
├── common
│   ├── ThemeSwitcher.stories.tsx
│   └── LocaleSwitcher.stories.tsx
└── examples
    └── WorkflowDesigner.stories.tsx
```

**Configuration**:
```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: ['../stories/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
  ],
  framework: {
    name: '@storybook/nextjs',
    options: {},
  },
};

export default config;
```

**Example Story**:
```typescript
// stories/workflow/FlowCanvas.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { FlowCanvas } from '@/components/workflow/FlowCanvas';

const meta: Meta<typeof FlowCanvas> = {
  title: 'Workflow/FlowCanvas',
  component: FlowCanvas,
  decorators: [
    (Story) => (
      <div style={{ height: '600px' }}>
        <Story />
      </div>
    ),
  ],
};

export default meta;
type Story = StoryObj<typeof FlowCanvas>;

export const Default: Story = {
  args: {
    nodes: [
      { id: '1', type: 'state', position: { x: 0, y: 0 }, data: { label: 'Draft' } },
      { id: '2', type: 'state', position: { x: 200, y: 0 }, data: { label: 'Submitted' } },
    ],
    edges: [
      { id: 'e1-2', source: '1', target: '2', type: 'transition' },
    ],
  },
};
```

**New Dependencies**:
- `@storybook/nextjs` - Storybook for Next.js
- `@storybook/addon-essentials` - Essential Storybook addons
- `@storybook/addon-interactions` - Interaction testing
- `@storybook/addon-a11y` - Accessibility testing

#### 6. Playground

**New Files**:
```
/src
├── app
│   └── (ui)
│       └── playground
│           ├── page.tsx          # Playground page
│           ├── workflow
│           │   └── page.tsx      # Workflow playground
│           └── api
│               └── page.tsx      # API testing playground
├── components
│   └── playground
│       ├── WorkflowPlayground.tsx
│       ├── APITester.tsx
│       └── CodeEditor.tsx
```

**Playground Features**:
- Interactive workflow designer
- API endpoint tester with request/response viewer
- Code editor for workflow definitions (JSON/YAML)
- Real-time validation
- Export/import workflow definitions

**Implementation**:
```typescript
// app/(ui)/playground/page.tsx
'use client';

import { useState } from 'react';
import { WorkflowPlayground } from '@/components/playground/WorkflowPlayground';
import { APITester } from '@/components/playground/APITester';

export default function PlaygroundPage() {
  const [activeTab, setActiveTab] = useState('workflow');
  
  return (
    <div className="w-full">
      <div className="border-b border-gray-200 dark:border-gray-700">
        <nav className="flex space-x-8">
          <button
            onClick={() => setActiveTab('workflow')}
            className={`py-4 px-1 border-b-2 font-medium text-sm ${
              activeTab === 'workflow'
                ? 'border-blue-500 text-blue-600 dark:text-blue-400'
                : 'border-transparent text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300'
            }`}
          >
            Workflow Designer
          </button>
          <button
            onClick={() => setActiveTab('api')}
            className={`py-4 px-1 border-b-2 font-medium text-sm ${
              activeTab === 'api'
                ? 'border-blue-500 text-blue-600 dark:text-blue-400'
                : 'border-transparent text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300'
            }`}
          >
            API Tester
          </button>
        </nav>
      </div>
      <div className="mt-4">
        {activeTab === 'workflow' && <WorkflowPlayground />}
        {activeTab === 'api' && <APITester />}
      </div>
    </div>
  );
}
```

**New Dependencies**:
- `monaco-editor` or `@monaco-editor/react` - Code editor
- `react-json-view` - JSON viewer

#### 7. Example Public Page

**New Files**:
```
/src
├── app
│   └── (public)
│       ├── layout.tsx            # Public layout (no auth)
│       ├── page.tsx              # Landing page
│       ├── features
│       │   └── page.tsx          # Features showcase
│       └── examples
│           └── page.tsx          # Example workflows
```

**Public Page Features**:
- Landing page with system overview
- Feature showcase
- Example workflow visualizations (read-only)
- API documentation link
- Sign-up/Login CTAs

**Implementation**:
```typescript
// app/(public)/page.tsx
'use client';

import Link from 'next/link';
import { FlowCanvas } from '@/components/workflow/FlowCanvas';

export default function PublicLandingPage() {
  return (
    <div className="min-h-screen bg-white dark:bg-gray-900">
      <section className="container mx-auto px-4 py-16">
        <h1 className="text-4xl font-bold text-gray-900 dark:text-white mb-4">
          Workflow Management System
        </h1>
        <p className="text-xl text-gray-600 dark:text-gray-300 mb-8">
          Multi-tenant workflow orchestration platform
        </p>
        <Link
          href="/api/docs"
          className="inline-block px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
        >
          View API Docs
        </Link>
      </section>
      
      <section className="container mx-auto px-4 py-16">
        <h2 className="text-3xl font-bold text-gray-900 dark:text-white mb-8">
          Example Workflow
        </h2>
        <div className="bg-gray-50 dark:bg-gray-800 rounded-lg p-4">
          <FlowCanvas
            nodes={exampleNodes}
            edges={exampleEdges}
            readOnly
          />
        </div>
      </section>
    </div>
  );
}
```

### IAM (if applicable)

**Enhancement**: Add permission for accessing playground and public pages:
- `playground:access` - Access to playground (dev/testing)
- `public:view` - View public pages (no auth required)

**No changes to core IAM logic** - only permission additions.

## Backward Compatibility

### What Remains Unchanged

1. **Backend API Contracts**: All existing API endpoints remain unchanged. New endpoints are additive.
2. **Database Schema**: No breaking changes. Optional enhancements only.
3. **Workflow Engine**: Core logic unchanged. Only UI representation added.
4. **Multi-Tenancy**: Isolation enforcement remains the same.
5. **Authentication**: JWT-based auth flow unchanged.

### Migration Strategy

**No data migration required**. However:

1. **Theme Preferences**: User theme preferences stored in localStorage (client-side only)
2. **Locale Preferences**: 
   - Default: `en` (English)
   - User preferences stored in user profile (optional enhancement)
   - Tenant default locale in tenant config (optional)

3. **Workflow Definitions**: Existing definitions work without labels. Labels are optional enhancement:
   ```typescript
   // Old format still works
   {
     states: [...],
     transitions: [...]
   }
   
   // New format with labels (optional)
   {
     states: [...],
     transitions: [...],
     labels: {
       en: { name: "Request Workflow" },
       es: { name: "Flujo de Solicitud" }
     }
   }
   ```

4. **API Documentation**: 
   - Existing endpoints continue to work
   - Swagger docs are additive (read-only)
   - No changes to request/response formats

## Risks & Considerations

### Technical Risks

1. **Bundle Size**: 
   - **Risk**: React Flow adds significant bundle size
   - **Mitigation**: 
     - Code splitting for workflow designer (lazy load)
     - Tailwind CSS purges unused styles automatically (production build)
     - Consider dynamic imports for heavy components
     - Use Tailwind's JIT mode for optimal CSS bundle size

2. **Theme Performance**:
   - **Risk**: Theme switching may cause flash of unstyled content
   - **Mitigation**: 
     - Store theme preference in localStorage
     - Apply theme before first render (script in `<head>`)
     - Use CSS variables for theme tokens

3. **Localization Complexity**:
   - **Risk**: Managing translations across many components
   - **Mitigation**: 
     - Use translation keys consistently
     - Extract all strings to translation files
     - Consider translation management tool (e.g., Crowdin, Lokalise)

4. **Storybook Maintenance**:
   - **Risk**: Stories may become outdated as components evolve
   - **Mitigation**: 
     - Include Storybook in CI/CD
     - Use visual regression testing
     - Keep stories close to components

### Operational Considerations

1. **Development Workflow**:
   - Frontend developers need access to Storybook
   - Playground requires development environment setup
   - Consider separate deployment for Storybook (static site)

2. **Localization Workflow**:
   - Need process for adding new languages
   - Translation review process
   - Consider professional translation services for production

3. **API Documentation**:
   - Keep Swagger annotations up-to-date with code changes
   - Consider automated OpenAPI generation from Zod schemas
   - Version API documentation

### Performance Considerations

1. **React Flow Performance**:
   - Large workflows (>100 nodes) may be slow
   - **Mitigation**: Virtualization, pagination, or simplified view mode

2. **Theme Switching**:
   - Minimal performance impact (CSS variable updates)
   - Consider CSS-in-JS performance if using styled-components

3. **Localization**:
   - Bundle size increases with each language
   - **Mitigation**: Lazy load translation files, use dynamic imports

### Security Considerations

1. **Playground Access**:
   - **Risk**: Playground may expose internal APIs or workflows
   - **Mitigation**: 
     - Restrict playground to development/staging
     - Require authentication for playground
     - Sanitize all user inputs in playground

2. **Public Page**:
   - **Risk**: May expose sensitive information
   - **Mitigation**: 
     - Review all content on public pages
     - Use read-only example workflows
     - No sensitive data in examples

3. **Swagger Documentation**:
   - **Risk**: May expose API structure to unauthorized users
   - **Mitigation**: 
     - Require authentication for `/api/docs`
     - Or restrict to internal network
     - Consider separate public API docs (limited endpoints)

## Implementation Priority

### Phase 1: Foundation (Week 1-2)
1. Tailwind CSS integration and theme system
2. Basic UI layout and navigation
3. Theme switcher implementation

### Phase 2: Core UI (Week 3-4)
1. React Flow integration
2. Workflow designer basic functionality
3. Localization setup (English first)

### Phase 3: Documentation & Tools (Week 5-6)
1. Swagger API documentation
2. Storybook setup and initial stories
3. Playground basic features

### Phase 4: Polish (Week 7-8)
1. Additional language support
2. Public page and examples
3. Performance optimization
4. Testing and bug fixes

## Additional Notes

### Folder Structure Updates

The existing structure is extended as follows:

```
/src
├── app
│   ├── api
│   │   └── docs                    # NEW: Swagger docs
│   ├── (ui)                        # MODIFIED: Full UI pages
│   │   ├── workflow-designer       # NEW
│   │   ├── playground              # NEW
│   │   └── dashboard               # EXISTING
│   └── (public)                    # NEW: Public pages
│       ├── page.tsx
│       └── examples
├── components                       # NEW: React components
│   ├── workflow
│   ├── common
│   └── ui
├── hooks                           # NEW: React hooks
├── locales                         # NEW: Translation files
├── stories                         # NEW: Storybook stories
└── styles                          # NEW: Theme styles
```

### Dependencies Summary

**New Production Dependencies**:
- `reactflow` - Workflow visualization
- `tailwindcss` - Utility-first CSS framework
- `@headlessui/react` - Unstyled, accessible UI components (optional)
- `lucide-react` or `@heroicons/react` - Icon library
- `next-intl` - Internationalization
- `zustand` - State management
- `swagger-jsdoc` - API documentation

**New Development Dependencies**:
- `@storybook/nextjs` - Component documentation
- `@monaco-editor/react` - Code editor (playground)
- `swagger-ui-react` - Swagger UI component

### Testing Considerations

1. **Component Testing**: Use React Testing Library for UI components
2. **Storybook**: Use for visual regression testing
3. **Playground**: Manual testing environment, not for automated tests
4. **Localization**: Test with multiple languages, RTL support if needed
5. **Theme**: Test both light and dark themes

