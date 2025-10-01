# 🔑 Personal Access Token Setup Guide

This guide shows you how to create and configure a Personal Access Token (PAT) for your GitHub workflows to merge branches with enhanced permissions.

## 📋 **Step 1: Create a Personal Access Token**

### **Classic Token (Recommended for most use cases)**

1. **Go to GitHub Settings**
   - Click your profile picture → **Settings**
   - Or visit: https://github.com/settings/tokens

2. **Navigate to Developer Settings**
   - Scroll down → **Developer settings** (left sidebar)
   - Click **Personal access tokens** → **Tokens (classic)**

3. **Generate New Token**
   - Click **Generate new token** → **Generate new token (classic)**
   - Give it a descriptive name: `GitHub Actions Branch Merge`

4. **Set Expiration**
   - Choose appropriate expiration (90 days, 1 year, or no expiration)
   - ⚠️ **Security Tip**: Use shorter expiration for better security

5. **Select Scopes (Permissions)**
   ```
   ✅ repo (Full control of private repositories)
      ├── ✅ repo:status (Access commit status)
      ├── ✅ repo_deployment (Access deployment status)
      ├── ✅ public_repo (Access public repositories)
      └── ✅ repo:invite (Access repository invitations)
   
   ✅ admin:org (Full control of orgs and teams, read and write org projects)
      └── ✅ write:org (Write org and team membership, read and write org projects)
   
   ✅ admin:repo_hook (Full control of repository hooks)
      └── ✅ write:repo_hook (Write repository hooks)
   
   ✅ admin:org_hook (Full control of organization hooks)
   
   ✅ delete_repo (Delete repositories)
   
   ✅ notifications (Access notifications)
   
   ✅ user (Update ALL user data)
   
   ✅ delete_user (Delete user account)
   
   ✅ gist (Create gists)
   
   ✅ read:org (Read org and team membership, read org projects)
   
   ✅ workflow (Update GitHub Action workflows)
   ```

6. **Generate Token**
   - Click **Generate token**
   - ⚠️ **IMPORTANT**: Copy the token immediately! You won't be able to see it again.

---

## 🏢 **Fine-Grained Tokens (Alternative - More Secure)**

### **For Organizations or Advanced Users**

1. **Go to Personal Access Tokens**
   - Settings → Developer settings → Personal access tokens
   - Click **Fine-grained tokens** → **Generate new token**

2. **Configure Token**
   - **Token name**: `GitHub Actions Merge`
   - **Expiration**: Choose appropriate duration
   - **Resource owner**: Select your account or organization

3. **Repository Access**
   - **Selected repositories**: Choose specific repos or **All repositories**
   - **Repository permissions**:
     ```
     ✅ Contents: Read and write
     ✅ Metadata: Read
     ✅ Pull requests: Write
     ✅ Actions: Read (optional, for workflow triggers)
     ```

4. **Account Permissions**
   ```
   ✅ Commit statuses: Read and write
   ✅ Deployments: Read and write
   ✅ Issues: Read and write (optional)
   ✅ Members: Read (if working with org repos)
   ```

---

## 🔧 **Step 2: Add Token to Repository Secrets**

### **Method 1: GitHub Web Interface**

1. **Go to Repository Settings**
   - Navigate to your repository
   - Click **Settings** tab
   - Click **Secrets and variables** → **Actions**

2. **Add New Secret**
   - Click **New repository secret**
   - **Name**: `PERSONAL_ACCESS_TOKEN`
   - **Secret**: Paste your token
   - Click **Add secret**

### **Method 2: GitHub CLI (Command Line)**

```bash
# Install GitHub CLI if you haven't already
# https://cli.github.com/

# Authenticate with GitHub
gh auth login

# Add the secret to your repository
gh secret set PERSONAL_ACCESS_TOKEN --body "your_token_here"
```

### **Method 3: Organization Secrets (For Multiple Repos)**

If you want to use the same token across multiple repositories:

1. **Organization Settings**
   - Go to your organization settings
   - **Secrets and variables** → **Actions**
   - Click **New organization secret**

2. **Configure Access**
   - **Name**: `PERSONAL_ACCESS_TOKEN`
   - **Secret**: Your token
   - **Repository access**: Select repositories that can use this secret

---

## 🔒 **Step 3: Security Best Practices**

### **Token Security**
- ✅ **Never commit tokens to code**
- ✅ **Use repository secrets only**
- ✅ **Set appropriate expiration dates**
- ✅ **Use fine-grained tokens when possible**
- ✅ **Regularly rotate tokens**
- ✅ **Monitor token usage in GitHub audit logs**

### **Minimal Permissions**
```yaml
# For basic branch merging, you only need:
repo:status          # Read commit status
repo                 # Read/write repository contents
```

### **Advanced Permissions**
```yaml
# For bypassing branch protection:
admin:repo_hook      # Full control of repository hooks
admin:org            # Organization admin (if needed)
```

---

## 🧪 **Step 4: Test Your Setup**

### **Test Token Permissions**

Create a simple test workflow to verify your token works:

```yaml
name: Test Personal Access Token

on:
  workflow_dispatch:

jobs:
  test-token:
    runs-on: ubuntu-latest
    steps:
      - name: Test Token Permissions
        env:
          GITHUB_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
        run: |
          # Test basic API access
          curl -s -H "Authorization: Bearer $GITHUB_TOKEN" \
            "https://api.github.com/user" | jq '.login'
          
          # Test repository access
          curl -s -H "Authorization: Bearer $GITHUB_TOKEN" \
            "https://api.github.com/repos/${{ github.repository }}" | jq '.name'
          
          # Test branch access
          curl -s -H "Authorization: Bearer $GITHUB_TOKEN" \
            "https://api.github.com/repos/${{ github.repository }}/branches" | jq '.[].name'
```

### **Verify Branch Protection Bypass**

If you need to bypass branch protection:

```bash
# Test with curl
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  "https://api.github.com/repos/OWNER/REPO/merges" \
  -d '{
    "base": "main",
    "head": "feature-branch",
    "commit_message": "Test merge"
  }'
```

---

## 🚨 **Troubleshooting**

### **Common Issues**

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Unauthorized` | Invalid token | Check token is correct and not expired |
| `403 Forbidden` | Insufficient permissions | Add required scopes to token |
| `422 Unprocessable Entity` | Branch protection | Use token with admin permissions or create PR |
| `409 Conflict` | Merge conflicts | Handle conflicts manually or create PR |

### **Debug Token Permissions**

```bash
# Check what your token can access
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.github.com/user" | jq '.permissions'
```

### **Check Repository Access**

```bash
# Verify repository permissions
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  "https://api.github.com/repos/OWNER/REPO" | jq '.permissions'
```

---

## 📚 **Additional Resources**

- [GitHub Personal Access Tokens Documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Fine-grained Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-fine-grained-personal-access-token)
- [GitHub API Documentation](https://docs.github.com/en/rest)
- [Repository Secrets Documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

## ✅ **Quick Checklist**

- [ ] Created Personal Access Token with appropriate scopes
- [ ] Added token to repository secrets as `PERSONAL_ACCESS_TOKEN`
- [ ] Updated workflows to use `${{ secrets.PERSONAL_ACCESS_TOKEN }}`
- [ ] Tested token permissions
- [ ] Verified branch protection bypass (if needed)
- [ ] Set up token rotation schedule
- [ ] Documented token usage for team members

Your workflows are now ready to use your personal access token for enhanced branch merging capabilities! 🎉
