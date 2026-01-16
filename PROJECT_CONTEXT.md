# Multi-Tenant Workflow Management System

## 1. Project Overview

Build a configurable, multi-tenant workflow orchestration platform where each tenant can define its own workflows, approval chains, roles, and SLAs, while sharing a common infrastructure.

The system must orchestrate human and system tasks using state-machine-based workflows, enforce IAM rules, and provide full auditability.

## 2. Core Principles

- **Multi-tenant by design** (strict tenant isolation)
- **Configuration-driven workflows** (no hardcoded flows)
- **State-machine-based execution**
- **Role-based and rule-based approvals**
- **Event-driven where applicable**
- **API-first architecture**

## 3. Key Concepts (Use These Consistently)

- **Tenant** – isolated customer environment
- **Workflow Definition** – tenant-specific state machine blueprint
- **Workflow Instance** – running execution of a workflow
- **State** – current step in workflow
- **Transition** – move between states
- **Task** – human or system action
- **Actor** – user, role, or vendor
- **SLA** – time-based constraints and escalation

## 4. System Modules

### 4.1 Tenant Module

- Tenant creation and configuration
- Tenant-scoped data isolation
- Tenant-specific:
  - Workflow definitions
  - Roles
  - SLAs
  - Configuration

### 4.2 IAM (Identity & Access Management)

- Users belong to tenants
- Role-based access control (RBAC)
- Permissions applied to:
  - Workflow transitions
  - Data visibility
- Support internal users and vendor users
- All actions must be auditable

### 4.3 Building / Asset Module

- Buildings contain assets
- Assets have lifecycle workflows
- Asset events can trigger workflows (maintenance, inspection)

### 4.4 Vendor Module

- Vendor onboarding workflow
- Vendor assignment to requests and projects
- Vendors act as restricted external actors
- SLA and performance tracking

### 4.5 Project Module

- Projects are long-running composite workflows
- Includes:
  - Contract approval
  - Scope management
  - Task scheduling
  - Milestone approvals
- Project tasks are workflow steps

### 4.6 Request Module

- Primary workflow entry point
- Examples:
  - Maintenance request
  - Change request
  - Incident request
- Typical request workflow:
  ```
  Draft
  → Submitted
  → Manager Approval
  → Vendor Assignment
  → In Progress
  → Verification
  → Closed
  ```

## 5. Workflow Engine Requirements

The workflow engine must:

- Execute tenant-defined state machines
- Support:
  - Conditional transitions
  - Role-based approvals
  - Rule-based routing
- Track:
  - Current state
  - Transition history
  - Actor and timestamps
- Support:
  - SLA timers
  - Escalation rules
  - Notifications via events

## 6. Workflow Definition Format

- Definitions must be data-driven (JSON or YAML)
- Stored per tenant
- Editable without redeployment
- Each definition must include:
  - States
  - Transitions
  - Conditions
  - Required roles
  - Actions (assign, notify, escalate)

## 7. Multi-Tenancy Constraints

- Strong tenant isolation at all layers
- No cross-tenant access
- Workflow definitions are tenant-scoped
- IAM enforced on every workflow transition

## 8. Non-Functional Requirements

- Scalable to many tenants and workflows
- Full audit and history tracking
- Extensible for future modules
- High availability and reliability

## 9. Deliverables Expected

Provide:

1. High-level architecture diagram (described in text)
2. Database schema / ER design
3. Workflow definition schema (JSON/YAML example)
4. API design (key endpoints)
5. Technology stack recommendation
6. Example end-to-end workflow execution
7. Explanation of tenant isolation strategy

## 10. Constraints & Assumptions

- This is not a BPMN visual designer
- This is a backend orchestration platform
- UI consumes workflow state and permissions dynamically
- Vendors are first-class but restricted actors

## 11. Design Guidance

- Favor configuration over code
- Separate:
  - Workflow engine
  - Business modules
  - IAM
- Treat workflows as first-class entities
- Ensure auditability and traceability everywhere

## 12. Tech Context

### Architecture Overview

**Architecture Style**: Modular monolith with clear service boundaries, optimized for Next.js App Router.

The system follows a **layered architecture** pattern:
- **API Layer**: Next.js Route Handlers (App Router) for REST endpoints
- **Service Layer**: Business logic organized by domain modules
- **Data Layer**: MongoDB with Mongoose ODM for schema validation
- **Background Layer**: Node.js workers for async processing (SLA timers, notifications)

**Deployment Model**: Single Next.js application with server-side API routes and background job workers. Frontend and backend share the same codebase but are logically separated.

**Monorepo Structure** (Recommended):
```
workflow-management/
├── app/                    # Next.js App Router
│   ├── api/               # API route handlers
│   └── (frontend)/        # UI pages (if needed)
├── lib/                   # Shared libraries
│   ├── services/          # Business logic services
│   ├── models/            # Mongoose models
│   ├── workflow-engine/   # Workflow execution engine
│   ├── iam/               # IAM services
│   └── utils/             # Utilities
├── workers/               # Background job workers
├── types/                 # TypeScript types
└── config/                # Configuration files
```

### Backend Stack

**Core Technologies**:
- **Next.js 14+** (App Router) - API backend via Route Handlers
- **Node.js 20+** - Runtime environment
- **TypeScript 5+** - Type safety and developer experience
- **MongoDB 7+** - Primary database
- **Mongoose 8+** - ODM for MongoDB schema management

**Key Libraries**:
- **Zod** - Runtime validation for API inputs and workflow definitions
- **jsonwebtoken** / **jose** - JWT authentication
- **bcrypt** / **argon2** - Password hashing
- **BullMQ** / **Bull** - Job queue for background processing (Redis-backed)
- **ioredis** - Redis client for caching and job queues
- **date-fns** / **luxon** - Date/time manipulation for SLA calculations
- **winston** / **pino** - Structured logging
- **express-rate-limit** - API rate limiting

**Why Next.js for Backend**:
- Route Handlers provide clean API endpoint definitions
- Built-in middleware support for authentication/authorization
- Server Components can be used for admin/internal tools
- Unified deployment model
- Excellent TypeScript support

**API Boundaries**: Each module exposes services through a service layer. Route Handlers are thin wrappers that:
1. Extract and validate request data
2. Call service methods
3. Return standardized responses
4. Handle errors uniformly

**Server-Only Logic**: All workflow engine logic, business rules, and data access must be server-only. Use `"use server"` directives and ensure no client-side exposure of sensitive logic.

### Frontend Stack

**UI Framework**:
- **React Flow** - Workflow visualization and designer interface
- **Tailwind CSS** - Utility-first CSS framework for styling
- **Next.js App Router** - Server and client components
- **TypeScript** - Type safety for frontend code

**Key Frontend Libraries**:
- **reactflow** - Workflow diagram rendering and interaction
- **tailwindcss** - Utility-first CSS framework
- **next-intl** - Internationalization (i18n) for multi-language support
- **zustand** - Lightweight state management
- **next-themes** - Theme management (light/dark mode)
- **swagger-ui-react** - Interactive API documentation UI

**UI Features**:
- **Workflow Designer**: Visual workflow builder using React Flow
- **Theme Switcher**: Light/dark mode with system preference detection
- **Localization**: Multi-language support (English, Spanish, French initially)
- **API Documentation**: Swagger UI for interactive API exploration
- **Storybook**: Component documentation and visual testing
- **Playground**: Interactive testing environment for workflows and APIs
- **Public Pages**: Example workflows and feature showcase

**Why React Flow**:
- Powerful workflow visualization
- Customizable nodes and edges
- Built-in pan/zoom and controls
- Good performance for complex workflows
- Active community and maintenance

**Why Tailwind CSS**:
- Utility-first approach for rapid UI development
- Highly customizable with design tokens
- Excellent performance (purges unused styles)
- Consistent design system through configuration
- Great developer experience with IntelliSense

### Database Design (MongoDB)

**Collection Strategy**: **Shared collections with tenantId** (recommended approach)

**Rationale**:
- Simpler operations and maintenance than database-per-tenant
- Better resource utilization than collection-per-tenant
- Easier cross-tenant analytics (when needed)
- Standard MongoDB scaling patterns apply
- Tenant isolation enforced at application layer

**Core Collections**:

1. **tenants** - Tenant configuration
   ```typescript
   {
     _id: ObjectId,
     name: string,
     slug: string, // unique identifier
     status: 'active' | 'inactive' | 'suspended',
     config: {
       features: object,
       limits: object
     },
     createdAt: Date,
     updatedAt: Date
   }
   ```

2. **users** - User accounts (tenant-scoped)
   ```typescript
   {
     _id: ObjectId,
     tenantId: ObjectId, // REQUIRED - indexed
     email: string,
     passwordHash: string,
     type: 'internal' | 'vendor',
     vendorId?: ObjectId,
     status: 'active' | 'inactive',
     profile: object,
     createdAt: Date
   }
   ```

3. **roles** - RBAC roles (tenant-scoped)
   ```typescript
   {
     _id: ObjectId,
     tenantId: ObjectId, // REQUIRED - indexed
     name: string,
     permissions: string[],
     isSystem: boolean
   }
   ```

4. **workflow_definitions** - Workflow blueprints (tenant-scoped)
   ```typescript
   {
     _id: ObjectId,
     tenantId: ObjectId, // REQUIRED - indexed
     name: string,
     version: number,
     definition: {
       states: State[],
       transitions: Transition[],
       initialState: string
     },
     status: 'active' | 'draft' | 'archived',
     createdAt: Date
   }
   ```

5. **workflow_instances** - Running workflow executions
   ```typescript
   {
     _id: ObjectId,
     tenantId: ObjectId, // REQUIRED - indexed
     definitionId: ObjectId,
     currentState: string,
     status: 'running' | 'completed' | 'cancelled' | 'failed',
     context: object, // workflow-specific data
     createdAt: Date,
     updatedAt: Date
   }
   ```

6. **workflow_transitions** - Transition history (append-only)
   ```typescript
   {
     _id: ObjectId,
     tenantId: ObjectId, // REQUIRED - indexed
     instanceId: ObjectId, // indexed
     fromState: string,
     toState: string,
     actorId: ObjectId,
     timestamp: Date, // indexed
     reason?: string,
     metadata: object
   }
   ```

7. **requests** - Request entities
   ```typescript
   {
     _id: ObjectId,
     tenantId: ObjectId, // REQUIRED - indexed
     type: string,
     workflowInstanceId: ObjectId,
     status: string,
     metadata: object,
     createdBy: ObjectId,
     assignedTo?: ObjectId,
     createdAt: Date
   }
   ```

8. **audit_logs** - Audit trail (append-only, time-series)
   ```typescript
   {
     _id: ObjectId,
     tenantId: ObjectId, // REQUIRED - indexed
     entityType: string,
     entityId: ObjectId,
     action: string,
     actorId: ObjectId,
     changes: object,
     timestamp: Date, // indexed
     ipAddress?: string
   }
   ```

**Indexing Strategy**:

**Critical Indexes** (all collections):
- `{ tenantId: 1 }` - Tenant isolation queries
- `{ tenantId: 1, _id: 1 }` - Compound for tenant-scoped lookups

**Performance Indexes**:
- `workflow_instances`: `{ tenantId: 1, status: 1 }`, `{ tenantId: 1, definitionId: 1 }`
- `workflow_transitions`: `{ tenantId: 1, instanceId: 1, timestamp: -1 }`
- `users`: `{ tenantId: 1, email: 1 }` (unique)
- `requests`: `{ tenantId: 1, type: 1, status: 1 }`
- `audit_logs`: `{ tenantId: 1, timestamp: -1 }`, `{ tenantId: 1, entityType: 1, entityId: 1 }`

**Data Modeling Principles**:
- Always include `tenantId` as first field in compound indexes
- Use embedded documents for one-to-few relationships (e.g., user roles)
- Use references for one-to-many and many-to-many (e.g., workflow instances)
- Denormalize frequently accessed data (e.g., tenant name in audit logs)
- Use TTL indexes for temporary data (e.g., SLA timers)

### Multi-Tenancy Strategy

**Approach**: **Shared Database, Tenant-Id Filtering**

**Implementation**:

1. **Tenant Context Middleware**: Extract tenantId from JWT token or subdomain, inject into request context
   ```typescript
   // Middleware ensures tenantId is always present
   export function requireTenant(req: Request): ObjectId {
     const tenantId = req.headers['x-tenant-id'] || extractFromJWT(req);
     if (!tenantId) throw new UnauthorizedError();
     return tenantId;
   }
   ```

2. **Query Scoping**: All database queries MUST include tenantId filter
   ```typescript
   // Mongoose helper
   WorkflowInstance.find({ tenantId, ...otherFilters })
   ```

3. **Schema-Level Enforcement**: Mongoose schemas include tenantId as required field
   ```typescript
   const workflowInstanceSchema = new Schema({
     tenantId: { type: Schema.Types.ObjectId, required: true, index: true },
     // ... other fields
   });
   ```

4. **Authorization Checks**: Verify user belongs to tenant before any operation
   ```typescript
   async function authorizeTenantAccess(userId: ObjectId, tenantId: ObjectId) {
     const user = await User.findOne({ _id: userId, tenantId });
     if (!user) throw new ForbiddenError();
   }
   ```

**Tenant Isolation Guarantees**:
- Application layer enforces tenantId in all queries
- Database indexes ensure efficient tenant-scoped queries
- API middleware validates tenant membership
- No cross-tenant data leakage possible if middleware is correctly applied

**Trade-offs**:
- ✅ Simpler operations than database-per-tenant
- ✅ Better resource utilization
- ✅ Easier backups and maintenance
- ⚠️ Requires strict application-level enforcement
- ⚠️ Potential for query performance issues if not properly indexed

### Workflow Engine Design

**Implementation**: Custom state machine engine (NOT BPMN)

**Core Components**:

1. **Workflow Definition Parser**: Validates and parses JSON/YAML workflow definitions
   ```typescript
   interface WorkflowDefinition {
     states: State[];
     transitions: Transition[];
     initialState: string;
   }
   
   interface State {
     id: string;
     type: 'manual' | 'automatic' | 'wait';
     sla?: SLADefinition;
     actions?: Action[];
   }
   
   interface Transition {
     from: string;
     to: string;
     conditions?: Condition[];
     requiredRoles?: string[];
     requiredPermissions?: string[];
   }
   ```

2. **State Machine Executor**: Deterministic state transitions
   ```typescript
   class WorkflowEngine {
     async executeTransition(
       instanceId: ObjectId,
       transitionId: string,
       actorId: ObjectId,
       context: object
     ): Promise<void> {
       // 1. Load instance and definition
       // 2. Validate transition conditions
       // 3. Check authorization
       // 4. Execute transition
       // 5. Update state
       // 6. Record audit log
       // 7. Trigger actions (notifications, SLA timers)
     }
   }
   ```

3. **Condition Evaluator**: Evaluates transition conditions using JSON expression language
   ```typescript
   // Simple expression evaluator (or use jsonata/jsonpath)
   evaluateCondition(condition: Condition, context: object): boolean {
     // Supports: eq, ne, gt, lt, contains, in, etc.
   }
   ```

4. **Action Executor**: Executes state actions (assign, notify, escalate)
   ```typescript
   async executeActions(
     actions: Action[],
     instance: WorkflowInstance,
     context: object
   ): Promise<void> {
     for (const action of actions) {
       switch (action.type) {
         case 'assign': await assignTask(action.target, instance);
         case 'notify': await sendNotification(action.recipients, instance);
         case 'escalate': await triggerEscalation(action.rules, instance);
       }
     }
   }
   ```

**Execution Model**:
- **Synchronous**: Immediate state transitions (API-driven)
- **Asynchronous**: Background jobs for SLA timers, automatic transitions
- **Retry Logic**: Exponential backoff for failed system tasks
- **Idempotency**: Transition IDs prevent duplicate executions

**State Storage**:
- Current state stored in `workflow_instances.currentState`
- Full history in `workflow_transitions` (append-only)
- Context data in `workflow_instances.context` (JSON field)

### IAM & Authorization

**Authentication**:
- **JWT-based** authentication
- Tokens include: `userId`, `tenantId`, `roles`, `permissions`
- Token expiration: 15 minutes (access), 7 days (refresh)
- Refresh token rotation for security

**Authorization Model**: **RBAC with Resource-Level Permissions**

**Permission Structure**:
```typescript
// Permission format: resource:action:scope
// Examples:
// - workflow:transition:approve
// - request:create:own
// - project:view:all
// - vendor:assign:restricted
```

**Authorization Checks**:
1. **Role-Based**: User has required role for transition
2. **Permission-Based**: User has specific permission
3. **Resource-Based**: User owns/assigned to resource
4. **Rule-Based**: Custom business rules (e.g., vendor restrictions)

**Implementation**:
```typescript
class AuthorizationService {
  async canTransition(
    userId: ObjectId,
    tenantId: ObjectId,
    transition: Transition,
    resource: object
  ): Promise<boolean> {
    const user = await this.getUserWithPermissions(userId, tenantId);
    
    // Check role requirement
    if (transition.requiredRoles && 
        !transition.requiredRoles.some(r => user.roles.includes(r))) {
      return false;
    }
    
    // Check permission requirement
    if (transition.requiredPermissions) {
      const hasPermission = transition.requiredPermissions.every(
        perm => user.permissions.includes(perm)
      );
      if (!hasPermission) return false;
    }
    
    // Check resource ownership/assignment
    if (transition.requiresOwnership && 
        resource.assignedTo !== userId) {
      return false;
    }
    
    return true;
  }
}
```

**Vendor Restrictions**:
- Vendor users have `type: 'vendor'` flag
- Restricted permissions set (read-only on most resources)
- Can only access assigned requests/projects
- Cannot modify workflow definitions or tenant settings

### Background Jobs & Events

**Job Queue**: **BullMQ** (Redis-backed)

**Why BullMQ**:
- Built for Node.js/TypeScript
- Supports delayed jobs (perfect for SLA timers)
- Job prioritization and retries
- Dashboard for monitoring

**Job Types**:

1. **SLA Timer Jobs**: Scheduled when workflow enters state with SLA
   ```typescript
   // Create delayed job for SLA expiration
   await slaQueue.add('check-sla', {
     instanceId,
     stateId,
     slaDuration,
   }, {
     delay: slaDuration, // milliseconds
   });
   ```

2. **Notification Jobs**: Async email/SMS sending
   ```typescript
   await notificationQueue.add('send-email', {
     recipient,
     template,
     data,
   });
   ```

3. **Escalation Jobs**: Trigger escalation workflows
   ```typescript
   await escalationQueue.add('escalate', {
     instanceId,
     escalationRule,
   });
   ```

4. **Workflow Auto-Transitions**: Automatic state changes
   ```typescript
   await workflowQueue.add('auto-transition', {
     instanceId,
     targetState,
   });
   ```

**Event System**: **In-Memory Event Emitter + Job Queue**

**Event Types**:
- `workflow.transitioned` - State change occurred
- `workflow.completed` - Workflow finished
- `sla.violated` - SLA deadline passed
- `task.assigned` - Task assigned to user
- `request.created` - New request created

**Event Handlers**: Subscribe to events and trigger actions (webhooks, notifications, integrations)

**Worker Process**: Separate Node.js process running BullMQ workers
```typescript
// workers/index.ts
import { Worker } from 'bullmq';

const slaWorker = new Worker('sla', async (job) => {
  // Check SLA violation
  // Trigger escalation
});

const notificationWorker = new Worker('notifications', async (job) => {
  // Send notification
});
```

### SLA & Escalation

**SLA Timer Management**:

1. **Timer Creation**: When workflow enters state with SLA definition
   ```typescript
   interface SLADefinition {
     duration: number; // milliseconds
     escalationRules: EscalationRule[];
   }
   
   async function createSLATimer(
     instanceId: ObjectId,
     stateId: string,
     sla: SLADefinition
   ) {
     // Store SLA timer record
     await SLATimer.create({
       instanceId,
       stateId,
       expiresAt: new Date(Date.now() + sla.duration),
       status: 'active',
     });
     
     // Schedule job for expiration check
     await slaQueue.add('check-sla', { instanceId, stateId }, {
       delay: sla.duration,
     });
   }
   ```

2. **Timer Checking**: Background job checks expired timers
   ```typescript
   slaWorker.process(async (job) => {
     const { instanceId, stateId } = job.data;
     
     const instance = await WorkflowInstance.findById(instanceId);
     if (instance.currentState !== stateId) {
       // State changed, cancel SLA check
       return;
     }
     
     // SLA violated - trigger escalation
     await triggerEscalation(instance, stateId);
   });
   ```

3. **Escalation Execution**: Apply escalation rules
   ```typescript
   async function triggerEscalation(
     instance: WorkflowInstance,
     stateId: string
   ) {
     const definition = await getWorkflowDefinition(instance.definitionId);
     const state = definition.states.find(s => s.id === stateId);
     const escalationRules = state.sla?.escalationRules || [];
     
     for (const rule of escalationRules) {
       // Notify escalation recipients
       // Reassign to escalation actor
       // Create escalation audit log
     }
   }
   ```

**SLA Tracking**: Store SLA violations in `sla_violations` collection for reporting

### Audit & Observability

**Audit Logging**:

**Strategy**: **Structured, append-only audit logs**

**Implementation**:
```typescript
class AuditService {
  async log(
    tenantId: ObjectId,
    entityType: string,
    entityId: ObjectId,
    action: string,
    actorId: ObjectId,
    changes?: object
  ) {
    await AuditLog.create({
      tenantId,
      entityType,
      entityId,
      action,
      actorId,
      changes,
      timestamp: new Date(),
      ipAddress: req.ip,
    });
  }
}
```

**What to Audit**:
- All workflow transitions
- User authentication events
- Permission changes
- Tenant configuration changes
- Data modifications (create, update, delete)

**Logging**:

**Library**: **Winston** or **Pino** for structured logging

**Log Levels**:
- `error` - Exceptions, failures
- `warn` - Warnings, SLA violations
- `info` - Business events, transitions
- `debug` - Detailed execution traces

**Log Format**: JSON structured logs
```typescript
logger.info('Workflow transition', {
  tenantId,
  instanceId,
  fromState,
  toState,
  actorId,
  timestamp,
});
```

**Observability**:

**Metrics**: Track key metrics (workflow execution time, SLA violations, API latency)
- Use **Prometheus** client for metrics collection
- Export metrics endpoint for scraping

**Health Checks**: `/api/health` endpoint
- Database connectivity
- Redis connectivity
- Worker process status

**Distributed Tracing**: Consider **OpenTelemetry** for request tracing across services

### API Design

**Style**: **RESTful API** with consistent patterns

**Base URL**: `/api/v1`

**Endpoint Structure**:
```
GET    /api/v1/tenants                    # List tenants
POST   /api/v1/tenants                    # Create tenant
GET    /api/v1/tenants/:id                # Get tenant
PUT    /api/v1/tenants/:id                # Update tenant
DELETE /api/v1/tenants/:id                # Delete tenant

GET    /api/v1/workflows/definitions      # List definitions
POST   /api/v1/workflows/definitions      # Create definition
GET    /api/v1/workflows/definitions/:id  # Get definition
PUT    /api/v1/workflows/definitions/:id  # Update definition

POST   /api/v1/workflows/instances        # Create instance
GET    /api/v1/workflows/instances/:id    # Get instance
POST   /api/v1/workflows/instances/:id/transitions  # Execute transition

GET    /api/v1/requests                   # List requests
POST   /api/v1/requests                   # Create request
GET    /api/v1/requests/:id               # Get request
```

**Request/Response Format**:

**Request**:
```typescript
// Headers
Authorization: Bearer <jwt-token>
X-Tenant-ID: <tenant-id> // Optional if in JWT
Content-Type: application/json

// Body (for POST/PUT)
{
  "name": "string",
  "data": object
}
```

**Response**:
```typescript
// Success (200)
{
  "success": true,
  "data": object,
  "meta": {
    "timestamp": "ISO8601",
    "requestId": "uuid"
  }
}

// Error (4xx/5xx)
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": object
  },
  "meta": {
    "timestamp": "ISO8601",
    "requestId": "uuid"
  }
}
```

**Error Codes**:
- `UNAUTHORIZED` - Missing/invalid token
- `FORBIDDEN` - Insufficient permissions
- `NOT_FOUND` - Resource not found
- `VALIDATION_ERROR` - Invalid input
- `WORKFLOW_ERROR` - Workflow execution error
- `TENANT_MISMATCH` - Tenant isolation violation

**Pagination**:
```
GET /api/v1/requests?page=1&limit=20&sort=createdAt:desc
```

**Response**:
```typescript
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

### Security

**Authentication Security**:
- Password hashing: **Argon2** (preferred) or **bcrypt**
- JWT signing: **RS256** (asymmetric keys) or **HS256** (symmetric, simpler)
- Token storage: HTTP-only cookies (preferred) or localStorage
- Refresh token rotation on use

**Authorization Security**:
- Always validate tenant membership
- Check permissions on every transition
- Validate resource ownership for sensitive operations
- Rate limit API endpoints per tenant/user

**Data Security**:
- Encrypt sensitive fields at application level (e.g., passwords, tokens)
- Use MongoDB encryption at rest (if available)
- HTTPS/TLS for all API communications
- Input validation and sanitization (Zod schemas)

**Tenant Isolation Security**:
- Middleware enforces tenantId in all queries
- No cross-tenant queries possible
- Database indexes prevent accidental cross-tenant access
- Audit logs track all tenant-scoped operations

**API Security**:
- Rate limiting: **express-rate-limit** (per tenant/user)
- CORS: Restrict to known origins
- Input validation: Zod schemas for all inputs
- SQL/NoSQL injection: Mongoose parameterization prevents injection
- XSS: Sanitize all user inputs

**Vendor Security**:
- Vendor users have restricted permission set
- Can only access assigned resources
- Cannot modify tenant configuration
- All vendor actions are audited

### Scalability & Future Considerations

**Current Architecture Scalability**:

**Horizontal Scaling**:
- Next.js API routes can scale horizontally (stateless)
- MongoDB supports replica sets and sharding
- Redis supports clustering for BullMQ
- Worker processes can scale independently

**Database Scaling**:
- **Read Replicas**: For read-heavy workloads
- **Sharding**: By tenantId if needed (future)
- **Indexing**: Critical for performance at scale
- **Connection Pooling**: Mongoose connection pool tuning

**Caching Strategy**:
- **Redis** for:
  - Workflow definitions (cache per tenant)
  - User permissions (cache per user)
  - Frequently accessed data
- Cache invalidation on updates
- TTL-based expiration

**Future Evolution Path**:

1. **Microservices Migration** (if needed):
   - Extract workflow engine to separate service
   - Extract IAM to separate service
   - Use message queue (RabbitMQ/Kafka) for inter-service communication
   - API Gateway for routing

2. **Event Sourcing** (for audit trail):
   - Store all state changes as events
   - Rebuild state from event log
   - Better auditability and debugging

3. **GraphQL API** (if frontend needs it):
   - Add GraphQL layer on top of REST API
   - Use **GraphQL Yoga** or **Apollo Server**

4. **Multi-Region Deployment**:
   - MongoDB multi-region clusters
   - Redis replication across regions
   - CDN for static assets

5. **Advanced Features**:
   - Workflow versioning and migration
   - Workflow templates marketplace
   - Advanced analytics and reporting
   - Machine learning for workflow optimization

**Performance Optimization**:
- Database query optimization (explain plans)
- Aggregation pipelines for complex queries
- Background job processing for heavy operations
- CDN for static assets (if frontend included)
- Database connection pooling optimization

### Project Structure

**Next.js Folder Structure (App Router)**:

This structure is optimized for:
- Clear domain boundaries
- Workflow engine isolation
- Multi-tenancy & IAM enforcement
- Backend-first architecture

```
/src
├── app
│   ├── api
│   │   ├── auth
│   │   │   └── route.ts
│   │   ├── tenants
│   │   │   └── route.ts
│   │   ├── workflows
│   │   │   ├── definitions
│   │   │   │   └── route.ts
│   │   │   ├── instances
│   │   │   │   └── route.ts
│   │   │   └── transitions
│   │   │       └── route.ts
│   │   ├── requests
│   │   │   └── route.ts
│   │   ├── projects
│   │   │   └── route.ts
│   │   ├── vendors
│   │   │   └── route.ts
│   │   ├── assets
│   │   │   └── route.ts
│   │   └── audit
│   │       └── route.ts
│   ├── api
│   │   └── docs
│   │       └── route.ts          # Swagger API documentation
│   ├── (ui)
│   │   ├── workflow-designer
│   │   │   └── page.tsx          # Workflow designer page
│   │   ├── dashboard
│   │   │   └── page.tsx          # Main dashboard
│   │   ├── playground
│   │   │   ├── page.tsx          # Playground page
│   │   │   └── workflow
│   │   │       └── page.tsx      # Workflow playground
│   │   └── layout.tsx            # UI layout with Tailwind CSS
│   ├── (public)
│   │   ├── page.tsx              # Public landing page
│   │   ├── features
│   │   │   └── page.tsx          # Features showcase
│   │   └── examples
│   │       └── page.tsx          # Example workflows
│   ├── layout.tsx
│   └── page.tsx
│
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
│   ├── playground
│   │   ├── WorkflowPlayground.tsx
│   │   ├── APITester.tsx
│   │   └── CodeEditor.tsx
│   └── ui
│       └── ...                   # Tailwind CSS styled components
│
├── hooks
│   ├── useTheme.ts               # Theme hook
│   ├── useLocale.ts              # Locale hook
│   └── useWorkflowDesigner.ts    # Workflow designer logic
│
├── locales
│   ├── en
│   │   ├── common.json
│   │   ├── errors.json
│   │   └── api.json
│   ├── es
│   │   └── ...
│   └── fr
│       └── ...
│
├── stories
│   ├── workflow
│   │   ├── FlowCanvas.stories.tsx
│   │   ├── StateNode.stories.tsx
│   │   └── TransitionEdge.stories.tsx
│   ├── common
│   │   ├── ThemeSwitcher.stories.tsx
│   │   └── LocaleSwitcher.stories.tsx
│   └── examples
│       └── WorkflowDesigner.stories.tsx
│
├── styles
│   ├── themes
│   │   ├── light.ts              # Light theme tokens
│   │   ├── dark.ts               # Dark theme tokens
│   │   └── index.ts              # Theme configuration
│   └── globals.css               # Global styles
│
├── core
│   ├── workflow
│   │   ├── engine.ts
│   │   ├── state-machine.ts
│   │   ├── transition-evaluator.ts
│   │   ├── executor.ts
│   │   ├── validators.ts
│   │   └── types.ts
│   │
│   ├── iam
│   │   ├── auth.ts
│   │   ├── permissions.ts
│   │   ├── rbac.ts
│   │   └── types.ts
│   │
│   ├── tenant
│   │   ├── resolver.ts
│   │   ├── context.ts
│   │   └── types.ts
│   │
│   ├── sla
│   │   ├── timers.ts
│   │   ├── escalation.ts
│   │   └── types.ts
│   │
│   └── events
│       ├── dispatcher.ts
│       ├── handlers.ts
│       └── types.ts
│
├── modules
│   ├── requests
│   │   ├── service.ts
│   │   ├── repository.ts
│   │   ├── validators.ts
│   │   └── types.ts
│   │
│   ├── projects
│   │   ├── service.ts
│   │   ├── repository.ts
│   │   ├── scheduler.ts
│   │   └── types.ts
│   │
│   ├── vendors
│   │   ├── service.ts
│   │   ├── repository.ts
│   │   └── types.ts
│   │
│   ├── assets
│   │   ├── service.ts
│   │   ├── repository.ts
│   │   └── types.ts
│   │
│   └── tenants
│       ├── service.ts
│       ├── repository.ts
│       └── types.ts
│
├── db
│   ├── mongo.ts
│   ├── collections.ts
│   ├── migrations
│   │   └── index.ts
│   └── indexes.ts
│
├── lib
│   ├── http
│   │   ├── response.ts
│   │   └── errors.ts
│   ├── validation
│   │   └── schema.ts
│   ├── i18n
│   │   ├── server.ts             # Server-side i18n
│   │   ├── client.ts             # Client-side i18n
│   │   └── locale-detector.ts    # Locale detection
│   ├── openapi
│   │   ├── generator.ts          # OpenAPI spec generator
│   │   ├── decorators.ts         # Route decorators for metadata
│   │   └── schemas.ts            # Shared schema definitions
│   ├── logger.ts
│   └── constants.ts
│
├── middleware.ts
│
├── types
│   ├── common.ts
│   └── api.ts
│
├── config
│   ├── env.ts
│   ├── auth.ts
│   ├── workflow.ts
│   ├── theme.ts                  # Theme configuration
│   └── i18n.ts                   # i18n configuration
│
├── .storybook
│   ├── main.ts                   # Storybook configuration
│   ├── preview.tsx               # Global decorators
│   └── theme.ts                  # Storybook theme
│
└── tests
    ├── workflow
    ├── api
    └── modules
```

**Key Design Rationale**:

1. **`app/api/*`** - Pure HTTP boundary (no business logic)
   - Calls services from `/core` and `/modules`
   - Enforces:
     - Auth
     - Tenant resolution
     - Input validation

2. **`core/`** - Platform-Level Logic
   - Contains cross-cutting, reusable infrastructure logic:
     - `workflow/` → workflow engine (state machine, transitions)
     - `iam/` → authentication, RBAC, permission checks
     - `tenant/` → tenant resolution & context propagation
     - `sla/` → timers, escalation logic
     - `events/` → internal event dispatcher
   - No domain-specific logic here

3. **`modules/`** - Business Domains
   - Each module owns:
     - Business rules
     - Repositories
     - Validation
     - Integration with workflow engine
   - Modules never talk to each other directly — communication happens via:
     - Workflow engine
     - Events

4. **`db/`** - MongoDB connection & indexes
   - Shared collection definitions
   - Migrations (manual or scripted)

5. **`middleware.ts`** - Runs on every request
   - Handles:
     - Auth token parsing
     - Tenant resolution
     - Request context injection

6. **`lib/`** - Generic utilities
   - No domain knowledge

**Multi-Tenancy Enforcement (Where It Lives)**:
- `middleware.ts` → resolve tenant
- `core/tenant/context.ts` → inject tenantId
- Repositories enforce `{ tenantId }` in all queries
- Workflow engine validates tenant ownership

**Workflow Execution Flow (Example)**:
```
API Route
→ Auth & Tenant Middleware
→ Module Service
→ Workflow Engine
→ State Transition
→ Event Dispatch
→ SLA Timer (if applicable)
→ Audit Log
```

**Ready for Scale**:
- Works well on Vercel or self-hosted
- Can be split into microservices later
- Keeps workflows as first-class citizens
- Prevents "god services"

