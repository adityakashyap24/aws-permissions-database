# Complete Harness IDP Setup Guide

## 🚀 Step-by-Step Implementation

### **Prerequisites**
- Harness account with IDP module enabled
- GitHub repository access
- Admin access to Harness platform

---

## **Step 1: GitHub Repository Setup**

### 1.1 Create GitHub Repository
```bash
# Create a new repository
gh repo create aws-permissions-database --public
cd aws-permissions-database

# Initialize with our files
git init
git add .
git commit -m "Initial AWS permissions database setup"
git push -u origin main
```

### 1.2 Generate GitHub Personal Access Token
1. Go to GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Select scopes:
   - `repo` (Full control of private repositories)
   - `read:org` (Read org membership)
4. Copy the token - you'll need it in Harness

---

## **Step 2: Harness Platform Configuration**

### 2.1 Create GitHub Connector
1. **Navigate**: Harness Platform → Connectors → New Connector
2. **Select**: Code Repositories → GitHub
3. **Configure**:
   ```yaml
   Name: github-aws-permissions
   Description: GitHub connector for AWS permissions database
   URL Type: Repository
   Connection Type: HTTP
   GitHub Repository URL: https://github.com/your-org/aws-permissions-database
   Username: your-github-username
   Personal Access Token: [paste your token]
   ```
4. **Test Connection** and **Save**

### 2.2 Create Harness Secrets
1. **Navigate**: Account Settings → Account Resources → Secrets
2. **Create Secret**:
   ```yaml
   Secret Type: Text
   Secret Name: github_token
   Secret Value: [your GitHub token]
   ```

### 2.3 Enable Required Harness Modules
Ensure these modules are enabled:
- **Internal Developer Portal (IDP)**
- **Continuous Integration (CI)** - for workflow execution
- **Policy as Code** - for governance (optional)

---

## **Step 3: IDP Configuration**

### 3.1 Configure IDP Settings
1. **Navigate**: Harness Platform → Internal Developer Portal
2. **Go to**: Admin → Configuration
3. **Add GitHub Integration**:
   ```yaml
   GitHub Integration:
     Host: github.com
     Token: github_token (use secret reference)
     Organization: your-org
   ```

### 3.2 Setup Proxy Configuration
1. **Navigate**: IDP → Admin → Settings → Proxy
2. **Add Proxy Endpoint**:
   ```yaml
   Endpoint: /github-api
   Target: https://api.github.com
   Headers:
     Authorization: token <+secrets.getValue("github_token")>
     Accept: application/vnd.github.v3+json
     User-Agent: Harness-IDP-AWS-Access
   Change Origin: true
   Path Rewrite:
     "^/proxy/github-api": ""
   ```

---

## **Step 4: Import Workflow Templates**

### 4.1 Upload Workflow to GitHub
1. **Place the workflow file** in your repository:
   ```
   .harness/
   └── workflows/
       └── aws-service-access-workflow.yaml
   ```

### 4.2 Register Workflow in Harness IDP
1. **Navigate**: IDP → Workflows → Import
2. **Select**: Import from Git Repository
3. **Configure**:
   ```yaml
   Repository: github-aws-permissions (your connector)
   Branch: main
   File Path: .harness/workflows/aws-service-access-workflow.yaml
   ```
4. **Click Import**

### 4.3 Verify Workflow Import
1. **Check**: IDP → Workflows
2. **You should see**: "AWS Service Access Request" workflow
3. **Status should be**: Active

---

## **Step 5: Configure Catalog Integration**

### 5.1 Setup Auto-Discovery
1. **Navigate**: IDP → Admin → Integrations → GitHub
2. **Add Location**:
   ```yaml
   Type: GitHub
   Target: https://github.com/your-org/aws-permissions-database/blob/main/.harness/catalog-info.yaml
   ```

### 5.2 Verify Catalog Registration
1. **Navigate**: IDP → Catalog
2. **Check**: Your components should appear
3. **Verify**: Links to GitHub repository work

---

## **Step 6: Test the Complete Flow**

### 6.1 Execute Workflow
1. **Navigate**: IDP → Workflows
2. **Click**: "AWS Service Access Request"
3. **Fill Form**:
   - **GitHub Org**: your-org
   - **GitHub Repo**: aws-permissions-database
   - **User Group**: Select from dropdown (should load dynamically)
   - **AWS Services**: Should populate based on user group
   - **Environment**: dev/staging/prod
   - **Duration**: 8h
   - **Justification**: "Testing the workflow"

### 6.2 Verify Dynamic Loading
1. **User Group Dropdown**: Should load from GitHub API
2. **Service Selection**: Should change based on user group selection
3. **Form Validation**: Should prevent invalid combinations

### 6.3 Check Results
1. **Workflow Execution**: Should complete successfully
2. **GitHub Audit Log**: Check access-logs/ directory for new file
3. **Output**: Should show AWS role ARN and session token

---

## **Step 7: Troubleshooting Common Issues**

### 7.1 GitHub API Issues
**Problem**: Dynamic picker not loading user groups
**Solution**:
```bash
# Test GitHub API access
curl -H "Authorization: token YOUR_TOKEN" \
     "https://api.github.com/repos/your-org/aws-permissions-database/contents/permissions-data/user-groups.json"

# Check proxy configuration in Harness IDP
```

### 7.2 JSONata Expression Errors
**Problem**: Service filtering not working
**Solution**:
```javascript
// Test JSONata expression separately
$userGroupData := userGroups["ug1"];
$userAWSServices := $userGroupData.awsServices;
$filter($keys(awsServices), function($serviceId) {
  $serviceId in $userAWSServices
})
```

### 7.3 Workflow Execution Failures
**Problem**: Workflow steps failing
**Solutions**:
1. **Check Secrets**: Ensure `github_token` is accessible
2. **Verify Connector**: Test GitHub connector connectivity
3. **Review Logs**: Check workflow execution logs in Harness
4. **Validate JSON**: Ensure GitHub files have valid JSON syntax

### 7.4 Dynamic Picker Not Working
**Problem**: SelectFieldFromApi not populating or JSON parsing errors
**Solutions**:
1. **Check Proxy Configuration**: Ensure paths use correct Harness IDP format:
   ```yaml
   # Correct proxy path format for Harness IDP
   path: "api/proxy/github-raw/org/repo/main/file.json"
   ```
2. **Verify arraySelector**: Must match JSON structure:
   ```yaml
   # For aws-services.json
   arraySelector: "awsServices"
   valueSelector: "$key"
   labelSelector: "name"
   ```
3. **Test API Access**: Manually test proxy endpoints:
   ```bash
   # Test the proxy endpoint (replace ACCOUNT_ID with your actual account ID)
   curl "https://idp.harness.io/Npsd6WrETY-Baq6iHeOHGw/idp/api/proxy/github-raw/org/repo/main/permissions-data/aws-services.json"
   ```
4. **Check Authentication**: Ensure GITHUB_TOKEN is accessible
5. **Validate JSON**: Ensure GitHub files have valid JSON syntax

---

## **Step 8: Production Deployment**

### 8.1 Environment-Specific Configuration
```yaml
Development:
  GitHub Repo: aws-permissions-database-dev
  Branch: develop
  Auto-approval: enabled

Staging:
  GitHub Repo: aws-permissions-database-staging  
  Branch: staging
  Manual approval: required

Production:
  GitHub Repo: aws-permissions-database
  Branch: main
  Multi-level approval: required
  Audit logging: enhanced
```

### 8.2 RBAC Configuration
1. **Navigate**: Account Settings → Access Control
2. **Create Roles**:
   ```yaml
   AWS-Service-Users:
     - Execute workflows
     - View audit logs
   
   AWS-Service-Admins:
     - Manage workflows
     - Edit user groups
     - View all audit logs
   ```

### 8.3 Monitoring Setup
1. **Navigate**: IDP → Admin → Monitoring
2. **Configure Alerts**:
   - Workflow execution failures
   - GitHub API rate limits
   - Unauthorized access attempts

---

## **Step 9: Advanced Configuration**

### 9.1 Custom Domain (Optional)
```yaml
IDP Custom Domain: aws-access.company.com
SSL Certificate: company-cert
DNS Configuration: CNAME → your-harness-idp-url
```

### 9.2 SSO Integration
```yaml
Authentication Provider: Okta/Azure AD/Google
SAML Configuration: Import metadata
User Group Mapping: 
  Okta Groups → Harness User Groups → AWS Services
```

### 9.3 Compliance Features
```yaml
Audit Requirements:
  - SOX compliance logging
  - PCI DSS access controls
  - ISO 27001 documentation

Policy Enforcement:
  - Max access duration limits
  - Environment restrictions
  - Service approval workflows
```

---

## **Step 10: Validation Checklist**

### ✅ **Pre-Production Checklist**
- [ ] GitHub connector tests successfully
- [ ] Dynamic user group loading works
- [ ] Service filtering based on user group works
- [ ] Environment selection enforced
- [ ] Audit logs created in GitHub
- [ ] AWS role ARN generation works
- [ ] Session token validation passes
- [ ] Error handling works for denied services
- [ ] All required secrets configured
- [ ] RBAC permissions tested

### ✅ **Production Readiness**
- [ ] Load testing completed
- [ ] Security review passed
- [ ] Backup strategy implemented
- [ ] Monitoring alerts configured
- [ ] Documentation updated
- [ ] Team training completed
- [ ] Rollback plan prepared
- [ ] Compliance requirements met

---

## **🔧 Configuration Files Reference**

All the configuration files are provided in the repository:
- `.harness/workflows/aws-service-access-workflow.yaml` - Main workflow
- `.harness/app-config.yaml` - Proxy and integration config
- `.harness/catalog-info.yaml` - Service catalog registration
- `permissions-data/user-groups.json` - User group definitions
- `permissions-data/aws-services.json` - AWS services catalog

**Next Steps**: Follow this guide step-by-step, and you'll have a fully functional AWS service access system using GitHub as the database and Harness IDP for the dynamic workflow interface!