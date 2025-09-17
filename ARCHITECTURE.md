# Architecture Design: GitHub-based Service Access with Harness IDP

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               HARNESS IDP PLATFORM                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────────┐  │
│  │   Web Portal    │    │   Workflows     │    │     Dynamic Picker         │  │
│  │                 │    │                 │    │                             │  │
│  │ • User Interface│    │ • JSONata Logic │    │ • Real-time API Calls      │  │
│  │ • Form Context  │    │ • Step Execution│    │ • Form State Management    │  │
│  │ • Multi-step    │    │ • Error Handling│    │ • Dependency Resolution    │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────────┘  │
│           │                       │                          │                  │
│           └───────────────────────┼──────────────────────────┘                  │
│                                   │                                             │
└───────────────────────────────────┼─────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │              │               │
                    ▼              ▼               ▼
    ┌─────────────────────┐ ┌─────────────┐ ┌──────────────────┐
    │    GITHUB API       │ │   GITHUB    │ │   ACCESS LOGS    │
    │                     │ │ REPOSITORY  │ │                  │
    │ • REST API v3       │ │             │ │ • Audit Trail    │
    │ • Raw Content API   │ │ Permissions │ │ • Token Records  │
    │ • GraphQL API       │ │ Database    │ │ • Request History│
    │ • Webhook Events    │ │             │ │ • Compliance     │
    └─────────────────────┘ └─────────────┘ └──────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼
        ┌─────────────────┐ ┌─────────────┐ ┌─────────────────┐
        │  user-groups    │ │  services   │ │   access-logs   │
        │     .json       │ │ -registry   │ │   directory     │
        │                 │ │   .json     │ │                 │
        │ • UG Definitions│ │ • Services  │ │ • Token Files   │
        │ • Permissions   │ │ • Endpoints │ │ • Request Logs  │
        │ • Metadata      │ │ • Required  │ │ • Audit Records │
        │                 │ │   Perms     │ │                 │
        └─────────────────┘ └─────────────┘ └─────────────────┘
```

## 🔄 Component Interaction Flow

### 1. User Journey Flow
```
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    User     │    │  Harness IDP    │    │   GitHub API    │    │  Target Service │
│             │    │   Workflow      │    │                 │    │                 │
└──────┬──────┘    └─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
       │                     │                      │                      │
       │ 1. Access Request   │                      │                      │
       ├────────────────────►│                      │                      │
       │                     │ 2. Fetch UG Data    │                      │
       │                     ├─────────────────────►│                      │
       │                     │ 3. UG List Response │                      │
       │                     │◄─────────────────────┤                      │
       │ 4. UG Selection     │                      │                      │
       │◄────────────────────┤                      │                      │
       │ 5. UG Choice        │                      │                      │
       ├────────────────────►│                      │                      │
       │                     │ 6. Fetch Services   │                      │
       │                     ├─────────────────────►│                      │
       │                     │ 7. Services Data    │                      │
       │                     │◄─────────────────────┤                      │
       │                     │ 8. Filter Services  │                      │
       │                     │    (JSONata Logic)  │                      │
       │ 9. Available Svcs   │                      │                      │
       │◄────────────────────┤                      │                      │
       │ 10. Service Select  │                      │                      │
       ├────────────────────►│                      │                      │
       │                     │ 11. Generate Token  │                      │
       │                     │ 12. Log to GitHub   │                      │
       │                     ├─────────────────────►│                      │
       │ 13. Access Token    │                      │                      │
       │◄────────────────────┤                      │                      │
       │ 14. Service Call with Token                │                      │
       ├─────────────────────────────────────────────────────────────────►│
       │ 15. Service Response                        │                      │
       │◄─────────────────────────────────────────────────────────────────┤
```

### 2. Dynamic Picker Flow
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         DYNAMIC PICKER ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐    │
│  │   Form State    │         │  API Resolver   │         │   Context Mgr   │    │
│  │                 │         │                 │         │                 │    │
│  │ • User Inputs   │◄───────►│ • GitHub Calls  │◄───────►│ • Form Context  │    │
│  │ • Field Values  │         │ • JSONata Eval  │         │ • Dependencies  │    │
│  │ • Validation    │         │ • Data Transform│         │ • State Updates │    │
│  └─────────────────┘         └─────────────────┘         └─────────────────┘    │
│           │                           │                           │              │
│           └───────────────────────────┼───────────────────────────┘              │
│                                       │                                          │
│                   ┌───────────────────┼───────────────────┐                      │
│                   │                   │                   │                      │
│                   ▼                   ▼                   ▼                      │
│       ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐    │
│       │   UG Picker     │ │ Service Picker  │ │     Real-time Filter        │    │
│       │                 │ │                 │ │                             │    │
│       │ • Load from API │ │ • Depends on UG │ │ • Permission Matching       │    │
│       │ • Display Name  │ │ • Filter Logic  │ │ • JSONata Expressions      │    │
│       │ • Description   │ │ • Multi-select  │ │ • Dynamic Availability     │    │
│       └─────────────────┘ └─────────────────┘ └─────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 📊 Data Flow Architecture

### 1. Data Storage Pattern
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            GITHUB REPOSITORY                                   │
│                         (permissions-database)                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  permissions-data/                    access-logs/                             │
│  ├── user-groups.json                 ├── ug1-token123-timestamp.json          │
│  │   {                               │   {                                     │
│  │     "userGroups": {               │     "userGroup": "ug1",                 │
│  │       "ug1": {                    │     "services": ["api1", "api2"],      │
│  │         "name": "Admins",         │     "token": "harness-access-...",      │
│  │         "permissions": [...]      │     "timestamp": "2024-...",            │
│  │       }                           │     "status": "approved"                │
│  │     }                             │   }                                     │
│  │   }                               │                                         │
│  └── services-registry.json          ├── ug2-token456-timestamp.json          │
│      {                               ├── ug3-token789-timestamp.json          │
│        "services": {                 └── ...                                   │
│          "api1": {                                                             │
│            "name": "User API",                                                 │
│            "requiredPermissions": [...],                                       │
│            "endpoints": [...]                                                  │
│          }                                                                     │
│        }                                                                       │
│      }                                                                         │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2. Data Access Pattern
```
API Call Pattern:
GET /repos/org/repo/contents/permissions-data/user-groups.json
├── Response: Base64 encoded JSON
├── Decode: Raw JSON data
└── Process: JSONata filtering

Query Examples:
• List UGs:     .userGroups | keys
• Get UG:       .userGroups.ug1
• Get Perms:    .userGroups.ug1.permissions
• Filter Svcs:  services[requiredPermissions ⊆ userPermissions]
```

### 3. Dynamic Filtering Logic
```javascript
// JSONata Expression for Service Filtering
$userPermissions := userGroups[selectedUserGroup].permissions;
$filter($keys(services), function($serviceId) {
  $service := services[$serviceId];
  $all($service.requiredPermissions, function($perm) {
    $perm in $userPermissions
  })
})

// Permission Validation
$validateAccess := function($userPerms, $requiredPerms) {
  $all($requiredPerms, function($perm) { $perm in $userPerms })
};

// Service Access Determination
$accessibleServices := services[
  $validateAccess($userPermissions, requiredPermissions)
];
```

## 🔒 Security Architecture

### 1. Authentication & Authorization Flow
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SECURITY LAYERS                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────────┐  │
│  │     User        │    │   Harness IDP   │    │       GitHub                │  │
│  │                 │    │                 │    │                             │  │
│  │ • SSO Login     │───►│ • RBAC Check    │───►│ • Token Validation          │  │
│  │ • MFA Required  │    │ • Workflow Auth │    │ • Repo Permissions          │  │
│  │ • Session Mgmt  │    │ • Audit Logging │    │ • Rate Limiting             │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────────┘  │
│           │                       │                          │                  │
│           └───────────────────────┼──────────────────────────┘                  │
│                                   │                                             │
│                   ┌───────────────┼───────────────┐                             │
│                   │               │               │                             │
│                   ▼               ▼               ▼                             │
│       ┌─────────────────┐ ┌─────────────┐ ┌─────────────────────────────┐       │
│       │   Token Gen     │ │   Audit     │ │        Compliance           │       │
│       │                 │ │   Trail     │ │                             │       │
│       │ • JWT Based     │ │ • All Reqs  │ │ • SOX Compliance            │       │
│       │ • Time Limited  │ │ • GitHub    │ │ • PCI DSS                   │       │
│       │ • Scope Limited │ │   Commits   │ │ • ISO 27001                 │       │
│       └─────────────────┘ └─────────────┘ └─────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2. Access Control Matrix
```
Role-Based Access Control:
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│  Actor          │  Permissions    │  GitHub Access  │  Harness Access │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│  End Users      │  Read UG Data   │  No Direct      │  Workflow Only  │
│                 │  Request Access │                 │                 │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│  Admins         │  Manage UGs     │  Repo Write     │  Full Access    │
│                 │  View Logs      │                 │                 │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│  Security Team  │  Audit Access   │  Repo Read      │  View Only      │
│                 │  Compliance     │                 │                 │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│  Service Tokens │  API Access     │  Token Auth     │  Webhook Only   │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

## 📈 Scalability Architecture

### 1. Horizontal Scaling Patterns
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SCALING DIMENSIONS                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  User Groups Scale           Service Scale              Request Scale           │
│  ┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐     │
│  │   1 → 1000s     │        │   10 → 10000s   │        │  10/min → 1000s │     │
│  │                 │        │                 │        │                 │     │
│  │ • JSON-based    │        │ • Registry      │        │ • GitHub API    │     │
│  │ • No Code Change│        │   Pattern       │        │   Rate Limits   │     │
│  │ • Auto Filter   │        │ • Category      │        │ • Harness       │     │
│  │ • O(1) Lookup   │        │   Grouping      │        │   Concurrency   │     │
│  └─────────────────┘        └─────────────────┘        └─────────────────┘     │
│           │                           │                           │             │
│           └───────────────────────────┼───────────────────────────┘             │
│                                       │                                         │
│                   ┌───────────────────┼───────────────────┐                     │
│                   │                   │                   │                     │
│                   ▼                   ▼                   ▼                     │
│       ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐   │
│       │   Data Sharding │ │   Caching       │ │      Load Distribution      │   │
│       │                 │ │                 │ │                             │   │
│       │ • Multi Repos   │ │ • GitHub CDN    │ │ • Multiple Workflows        │   │
│       │ • Org Structure │ │ • Browser Cache │ │ • Regional Deployment       │   │
│       │ • Team Isolation│ │ • Harness Cache │ │ • Auto-scaling              │   │
│       └─────────────────┘ └─────────────────┘ └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2. Performance Optimization
```
Optimization Strategy:
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│  Component      │  Current        │  Optimized      │  Improvement    │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│  UG Load Time   │  2-3 seconds    │  <1 second      │  3x faster      │
│  Service Filter │  O(n×m)         │  O(n)           │  Linear scale   │
│  API Calls      │  2 per request  │  1 per request  │  50% reduction  │
│  Memory Usage   │  Linear         │  Constant       │  O(1) space     │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘

Caching Strategy:
• GitHub API responses cached for 15 minutes
• Browser-side caching for static data
• Harness workflow result caching
• Progressive loading for large datasets
```

## 🚀 Deployment Architecture

### 1. Environment Strategy
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          DEPLOYMENT ENVIRONMENTS                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  Development                Production                Enterprise                 │
│  ┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐     │
│  │   Single Repo   │        │  Multi-Repo     │        │  Multi-Org      │     │
│  │                 │        │                 │        │                 │     │
│  │ • Basic Setup   │        │ • Team Repos    │        │ • Global Setup  │     │
│  │ • Test Data     │        │ • Prod Data     │        │ • Fed/Compliance│     │
│  │ • Full Access   │        │ • RBAC          │        │ • Air-gapped    │     │
│  │ • Local Testing │        │ • Monitoring    │        │ • Custom Auth   │     │
│  └─────────────────┘        └─────────────────┘        └─────────────────┘     │
│           │                           │                           │             │
│           └───────────────────────────┼───────────────────────────┘             │
│                                       │                                         │
│                   ┌───────────────────┼───────────────────┐                     │
│                   │                   │                   │                     │
│                   ▼                   ▼                   ▼                     │
│       ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐   │
│       │   Config Mgmt   │ │   Monitoring    │ │        Disaster Recovery    │   │
│       │                 │ │                 │ │                             │   │
│       │ • GitOps        │ │ • Metrics       │ │ • Backup Strategy           │   │
│       │ • Environments  │ │ • Alerts        │ │ • Multi-region              │   │
│       │ • Secrets       │ │ • Dashboards    │ │ • Failover Plans            │   │
│       └─────────────────┘ └─────────────────┘ └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2. Infrastructure Components
```
GitHub Infrastructure:
├── Primary Repository (permissions-database)
│   ├── permissions-data/ (Master data)
│   ├── access-logs/ (Audit trail)
│   └── .harness/ (Workflow definitions)
├── Backup Repository (permissions-database-backup)
│   └── Automated daily sync
└── Archive Repository (permissions-database-archive)
    └── Historical data retention

Harness Infrastructure:
├── IDP Platform
│   ├── Workflow Engine
│   ├── Dynamic Picker Service
│   └── Form Context Manager
├── Proxy Service
│   ├── GitHub API Proxy
│   ├── Rate Limiting
│   └── Caching Layer
└── Monitoring
    ├── Workflow Metrics
    ├── Usage Analytics
    └── Error Tracking
```

## 🔄 Integration Patterns

### 1. External System Integration
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         INTEGRATION ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  Upstream Systems           Core Platform              Downstream Systems      │
│  ┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐     │
│  │      LDAP       │        │   GitHub +      │        │   Target APIs   │     │
│  │      Azure AD   │───────►│   Harness IDP   │───────►│   Microservices │     │
│  │      SAML SSO   │        │                 │        │   Databases     │     │
│  └─────────────────┘        └─────────────────┘        └─────────────────┘     │
│           │                           │                           │             │
│           │                           │                           │             │
│  ┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐     │
│  │   HR Systems    │        │   Audit &       │        │   Monitoring    │     │
│  │   Identity Mgmt │───────►│   Compliance    │───────►│   SIEM          │     │
│  │   Org Structure │        │                 │        │   Alerting      │     │
│  └─────────────────┘        └─────────────────┘        └─────────────────┘     │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2. API Integration Patterns
```
Integration Methods:
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│  Method         │  Use Case       │  Implementation │  Benefits       │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│  REST API       │  Standard Ops   │  GitHub API     │  Simple, Fast   │
│  GraphQL        │  Complex Query  │  GitHub GraphQL │  Efficient      │
│  Webhooks       │  Real-time      │  GitHub Events  │  Instant Update │
│  Scheduled Sync │  Batch Updates  │  Harness Cron   │  Reliable       │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

This architecture provides a comprehensive, scalable, and secure foundation for GitHub-based service access management using Harness IDP, designed to handle enterprise-scale deployments while maintaining simplicity and performance.