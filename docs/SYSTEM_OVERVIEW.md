# System Overview: GitHub-based Service Access Architecture

## 🎯 Executive Summary

This system provides a **scalable, secure, and simple** solution for managing service access using GitHub as a database and Harness IDP for dynamic workflows. The architecture supports unlimited user groups and services with zero hardcoded mappings.

## 🏗️ Architecture Highlights

### **Core Design Principles**
1. **GitHub-Native**: Leverages GitHub as the source of truth
2. **Dynamic Scaling**: No hardcoded user group mappings  
3. **Zero Dependencies**: Only GitHub + Harness IDP required
4. **Audit-First**: Complete trail of all access requests
5. **Permission-Based**: Services filtered by user permissions

### **Key Components**

#### 1. **Data Layer (GitHub Repository)**
```
permissions-database/
├── permissions-data/
│   ├── user-groups.json      # User groups and their permissions
│   └── services-registry.json # Available services and requirements
└── access-logs/               # Audit trail of all requests
    └── ug1-token123-timestamp.json
```

#### 2. **Workflow Engine (Harness IDP)**
- **Dynamic Picker**: Real-time API calls to GitHub
- **JSONata Logic**: Permission-based service filtering  
- **Form Context**: Multi-step user experience
- **Token Generation**: Secure access credentials

#### 3. **Security Layer**
- **Authentication**: Harness RBAC + GitHub tokens
- **Authorization**: Permission validation per service
- **Audit Trail**: Every request logged to GitHub
- **Compliance**: Enterprise security standards

## 🔄 Request Flow Architecture

```
User Request → UG Selection → Service Filtering → Token Generation → Audit Logging
     ↓              ↓              ↓                ↓                ↓
  Harness IDP → GitHub API → JSONata Logic → Harness Token → GitHub Commit
```

### **Detailed Flow**
1. **User Interface**: Harness IDP presents dynamic form
2. **User Group Loading**: API call fetches available groups from GitHub
3. **Service Filtering**: JSONata expressions filter services by permissions
4. **Permission Validation**: Ensures user can access requested services
5. **Token Generation**: Creates time-limited access token
6. **Audit Logging**: Records request details in GitHub repository

## 📊 Data Architecture

### **User Groups Structure**
```json
{
  "userGroups": {
    "developers": {
      "name": "Development Team",
      "description": "Application developers",
      "permissions": ["read", "deploy-staging", "logs"],
      "metadata": {
        "created": "2024-01-15",
        "owner": "platform-team"
      }
    }
  }
}
```

### **Services Registry Structure**
```json
{
  "services": {
    "user-api": {
      "name": "User Management API",
      "description": "Manage user accounts",
      "category": "api",
      "endpoints": ["/api/users", "/api/profiles"],
      "requiredPermissions": ["read", "write"]
    }
  }
}
```

### **Dynamic Filtering Logic**
```javascript
// JSONata expression for service filtering
$userPermissions := userGroups[selectedUserGroup].permissions;
$filter($keys(services), function($serviceId) {
  $service := services[$serviceId];
  $all($service.requiredPermissions, function($perm) {
    $perm in $userPermissions
  })
})
```

## 🔒 Security Architecture

### **Multi-Layer Security**
1. **Identity Layer**: Harness IDP authentication + RBAC
2. **Access Layer**: GitHub token-based API authentication  
3. **Permission Layer**: JSONata-based service authorization
4. **Audit Layer**: Complete GitHub-based logging

### **Security Controls**
- **Token Management**: Time-limited, scope-limited access tokens
- **API Security**: GitHub rate limiting and authentication
- **Audit Trail**: Immutable Git history for compliance
- **Access Control**: Role-based permissions in Harness

## 📈 Scalability Architecture

### **Horizontal Scaling Capabilities**
- **User Groups**: Scales from 10 to 10,000+ without code changes
- **Services**: Dynamic registry supports unlimited services
- **Requests**: GitHub API can handle enterprise-scale traffic
- **Geographic**: Multi-region deployment support

### **Performance Optimizations**
- **Caching**: GitHub CDN + browser caching
- **Filtering**: O(n) complexity for service filtering
- **API Calls**: Minimized to 2 calls per request
- **Lazy Loading**: Progressive form field population

### **Scaling Patterns**
```
Small Team:     Single repo → Basic workflow → Simple RBAC
Medium Org:     Multi-repo → Enhanced workflow → Team-based RBAC  
Enterprise:     Multi-org → Custom workflow → Advanced compliance
```

## 🚀 Deployment Architecture

### **Environment Strategy**
- **Development**: Single repository, full access, test data
- **Staging**: Production-like setup, limited access, real data subset
- **Production**: Multi-repository, strict RBAC, full audit

### **Infrastructure Components**
- **GitHub**: Repository hosting, API services, webhook delivery
- **Harness**: Workflow engine, form rendering, token generation
- **Monitoring**: Request metrics, error tracking, usage analytics

## 🔧 Integration Patterns

### **Upstream Integrations**
- **Identity Providers**: LDAP, Azure AD, SAML SSO
- **HR Systems**: Organizational structure, team mappings
- **Compliance Tools**: SOX, PCI DSS, ISO 27001 requirements

### **Downstream Integrations**
- **Target Services**: APIs, microservices, databases
- **Monitoring**: SIEM, alerting, dashboard systems
- **Audit Systems**: Compliance reporting, risk management

## 💫 Key Differentiators

### **vs. Traditional IAM**
- ✅ **Simpler**: No complex IAM infrastructure
- ✅ **Cheaper**: Leverages existing GitHub/Harness licenses
- ✅ **Faster**: Minutes to deploy vs. months for traditional IAM
- ✅ **More Transparent**: Git-based audit trail

### **vs. Custom Solutions**
- ✅ **No Maintenance**: GitHub + Harness handle infrastructure
- ✅ **Enterprise Ready**: Built-in security, compliance, scaling
- ✅ **Dynamic**: Real-time permission evaluation
- ✅ **Auditable**: Complete history in version control

## 🎯 Business Value

### **Technical Benefits**
- **Reduced Complexity**: Single workflow handles all scenarios
- **Improved Security**: Permission-based access with audit trail
- **Enhanced Productivity**: Self-service access with instant provisioning
- **Lower Costs**: No additional infrastructure or licenses required

### **Operational Benefits**  
- **Faster Onboarding**: New users get access in minutes
- **Easier Compliance**: Built-in audit trail and controls
- **Better Visibility**: Complete access history in GitHub
- **Simplified Management**: JSON-based configuration vs. complex UIs

## 🔮 Future Enhancements

### **Planned Features**
- **GraphQL Support**: More efficient API queries
- **Advanced Caching**: Redis-based caching layer
- **ML-based Access**: Predictive access recommendations
- **Federation**: Multi-organization support

### **Integration Roadmap**
- **Mobile App**: React Native access request app
- **CLI Tool**: Command-line access management
- **Slack Bot**: Slack-based access requests
- **Terraform Provider**: Infrastructure-as-code integration

This architecture provides a solid foundation for enterprise-scale service access management while maintaining the simplicity needed for rapid deployment and adoption.