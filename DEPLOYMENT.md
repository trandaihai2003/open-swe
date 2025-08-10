# Open SWE Deployment Guide

This guide walks you through deploying Open SWE to the internet using LangGraph Platform and Vercel.

## Overview

Open SWE consists of two main components that need to be deployed:
1. **Backend**: LangGraph agent deployed to LangGraph Platform
2. **Frontend**: Next.js web app deployed to Vercel

## Prerequisites

- GitHub repository with Open SWE code
- LangGraph Platform account (Cloud SaaS, Self-Hosted, or BYOC)
- Vercel account
- GitHub App for OAuth authentication

## Step 1: Deploy LangGraph Backend

### 1.1 Set up LangGraph Platform

1. Create an account on [LangGraph Platform](https://smith.langchain.com/)
2. Create a new deployment and obtain the following values:
   - `CONTROL_PLANE_HOST`: LangGraph Platform control plane URL
   - `LANGSMITH_API_KEY`: Your LangSmith API key
   - `INTEGRATION_ID`: LangGraph Platform integration identifier
   - `DEPLOYMENT_ID`: Your specific deployment identifier

### 1.2 Configure GitHub Secrets

Add the following secrets to your GitHub repository (Settings > Secrets and variables > Actions):

```
CONTROL_PLANE_HOST=your_control_plane_host
LANGSMITH_API_KEY=your_langsmith_api_key
INTEGRATION_ID=your_integration_id
DEPLOYMENT_ID=your_deployment_id
```

### 1.3 Deploy Backend

The backend will automatically deploy when you:
- Push changes to the `main` branch affecting `apps/open-swe/**` or `packages/shared/**`
- Or manually trigger the "Deploy LangGraph" workflow in GitHub Actions

Your backend will be available at: `https://your-deployment-id.us.langsmith.com`

## Step 2: Create GitHub App

### 2.1 Create GitHub App

1. Go to GitHub Settings > Developer settings > GitHub Apps
2. Click "New GitHub App"
3. Fill in the required fields:
   - **App name**: `open-swe-production` (or your preferred name)
   - **Homepage URL**: `https://your-vercel-app.vercel.app` (you'll get this after Vercel deployment)
   - **Callback URL**: `https://your-vercel-app.vercel.app/api/auth/github/callback`
   - **Webhook URL**: `https://your-vercel-app.vercel.app/api/webhooks/github` (optional)

### 2.2 Configure Permissions

Set the following permissions:
- **Repository permissions**:
  - Contents: Read & Write
  - Issues: Read & Write
  - Pull requests: Read & Write
  - Metadata: Read
- **Account permissions**:
  - Email addresses: Read

### 2.3 Generate Private Key

1. After creating the app, scroll down to "Private keys"
2. Click "Generate a private key"
3. Download the `.pem` file - you'll need its contents for environment variables

## Step 3: Deploy Frontend to Vercel

### 3.1 Connect Repository to Vercel

1. Go to [Vercel Dashboard](https://vercel.com/dashboard)
2. Click "New Project"
3. Import your GitHub repository
4. Select the repository root (not a subdirectory)

### 3.2 Configure Vercel Project Settings

1. **Framework Preset**: Next.js
2. **Root Directory**: `apps/web`
3. **Build Command**: `cd ../.. && yarn build --filter=@open-swe/web`
4. **Output Directory**: `apps/web/.next`
5. **Install Command**: `cd ../.. && yarn install --immutable`

### 3.3 Set Environment Variables in Vercel

Add the following environment variables in Vercel Project Settings > Environment Variables:

#### Production URLs
```
NEXT_PUBLIC_API_URL=https://your-vercel-app.vercel.app/api
LANGGRAPH_API_URL=https://your-deployment-id.us.langsmith.com
```

#### GitHub App Configuration
```
NEXT_PUBLIC_GITHUB_APP_CLIENT_ID=your_github_app_client_id
GITHUB_APP_CLIENT_SECRET=your_github_app_client_secret
GITHUB_APP_REDIRECT_URI=https://your-vercel-app.vercel.app/api/auth/github/callback
GITHUB_APP_NAME=open-swe-production
GITHUB_APP_ID=your_github_app_id
GITHUB_APP_PRIVATE_KEY=-----BEGIN RSA PRIVATE KEY-----
...your private key content...
-----END RSA PRIVATE KEY-----
GITHUB_APP_WEBHOOK_SECRET=your_webhook_secret
```

#### Security & Configuration
```
SECRETS_ENCRYPTION_KEY=your_32_byte_hex_encryption_key
NEXT_PUBLIC_ASSISTANT_ID=your_assistant_id
NEXT_PUBLIC_RESTRICT_TO_LANGCHAIN_AUTH=false
NODE_ENV=production
```

### 3.4 Generate Encryption Key

Generate a secure encryption key:
```bash
openssl rand -hex 32
```

### 3.5 Deploy

1. Click "Deploy" in Vercel
2. Wait for the deployment to complete
3. Note your Vercel app URL (e.g., `https://your-app-name.vercel.app`)

## Step 4: Update GitHub App Settings

After getting your Vercel URL, update your GitHub App:

1. Go to your GitHub App settings
2. Update the **Homepage URL** to your Vercel app URL
3. Update the **Callback URL** to `https://your-vercel-app.vercel.app/api/auth/github/callback`
4. Save changes

## Step 5: Test Deployment

1. Visit your Vercel app URL
2. Try logging in with GitHub
3. Create a test task to verify the connection to the LangGraph backend
4. Check that all functionality works as expected

## Environment Variables Reference

### Frontend (Vercel)
| Variable | Description | Example |
|----------|-------------|---------|
| `NEXT_PUBLIC_API_URL` | Frontend API endpoint | `https://your-app.vercel.app/api` |
| `LANGGRAPH_API_URL` | Backend LangGraph Platform URL | `https://deployment-id.us.langsmith.com` |
| `SECRETS_ENCRYPTION_KEY` | 32-byte hex encryption key | Generated with `openssl rand -hex 32` |
| `NEXT_PUBLIC_GITHUB_APP_CLIENT_ID` | GitHub App Client ID | From GitHub App settings |
| `GITHUB_APP_CLIENT_SECRET` | GitHub App Client Secret | From GitHub App settings |
| `GITHUB_APP_PRIVATE_KEY` | GitHub App Private Key | Contents of downloaded .pem file |

### Backend (GitHub Secrets)
| Variable | Description | Source |
|----------|-------------|--------|
| `CONTROL_PLANE_HOST` | LangGraph Platform control plane | LangGraph Platform dashboard |
| `LANGSMITH_API_KEY` | LangSmith API key | LangSmith settings |
| `INTEGRATION_ID` | LangGraph Platform integration ID | LangGraph Platform dashboard |
| `DEPLOYMENT_ID` | LangGraph Platform deployment ID | LangGraph Platform dashboard |

## Troubleshooting

### Common Issues

1. **Build fails on Vercel**: Ensure the build command includes the monorepo context
2. **Environment variables not working**: Check that all variables are set in Vercel dashboard
3. **GitHub OAuth fails**: Verify callback URL matches exactly in GitHub App settings
4. **Backend connection fails**: Confirm `LANGGRAPH_API_URL` points to correct deployment

### Logs and Debugging

- **Vercel logs**: Check Function logs in Vercel dashboard
- **GitHub Actions logs**: Check workflow runs for backend deployment issues
- **LangGraph Platform logs**: Check deployment logs in LangGraph Platform dashboard

## Security Considerations

1. **Environment Variables**: Never commit actual values to version control
2. **GitHub App**: Use minimal required permissions
3. **Encryption Key**: Generate a strong, unique key for production
4. **HTTPS**: Ensure all URLs use HTTPS in production
5. **Secrets Rotation**: Regularly rotate API keys and secrets

## Maintenance

- **Updates**: Both deployments will automatically update when you push to main branch
- **Monitoring**: Monitor both Vercel and LangGraph Platform dashboards for issues
- **Scaling**: Adjust Vercel plan and LangGraph Platform resources as needed
