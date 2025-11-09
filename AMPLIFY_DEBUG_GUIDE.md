# Amplify Deployment Debugging Guide

## Issues Identified

### 1. **CRITICAL: amplify.yml Configuration Issues**

The current `amplify.yml` has incorrect artifact paths for Next.js deployment.

**Current Issues:**
- `baseDirectory: client` is correct, but artifacts need to include `.next` folder
- Build command uses `--turbopack` which may not work in Amplify production builds
- Missing proper Next.js output configuration

### 2. **CRITICAL: Missing Environment Variables**

Your application requires these environment variables in AWS Amplify:

**Required Environment Variables:**
```
NEXT_PUBLIC_API_URL=<your-backend-api-url>
NEXT_PUBLIC_S3_BASE_URL=https://falsenine-image-storage.s3.ap-south-1.amazonaws.com
NEXT_PUBLIC_RAZORPAY_TEST_KEY_ID=<your-razorpay-key>
NEXT_PUBLIC_IMAGE_HERO_SECTION=hero-section-image.png
NEXT_PUBLIC_IMAGE_BANNER=banner-image.png
NEXT_PUBLIC_IMAGE_ABOUT_SECTION=about-section-image.png
NEXT_PUBLIC_IMAGE_CONTACT_SECTION=contact-section-image.png
NEXT_PUBLIC_IMAGE_LEFT_SIDE=left-side-image.png
NEXT_PUBLIC_IMAGE_RIGHT_SIDE=right-side-image.png
```

**Most Critical:** `NEXT_PUBLIC_API_URL` - Without this, your app will try to connect to `http://localhost:4000` which doesn't exist in production!

### 3. **Hardcoded Localhost URLs**

Found hardcoded `localhost:4000` URLs in:
- `client/src/components/razorpaybutton.tsx` (lines 13, 26)

These will cause API calls to fail in production.

---

## Step-by-Step Debugging & Fix Steps

### Step 1: Fix amplify.yml Configuration

Update your `amplify.yml` file with the correct configuration:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - cd client
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: client/.next
    files:
      - "**/*"
  cache:
    paths:
      - client/node_modules/**/*
      - client/.next/cache/**/*
```

**OR** if the above doesn't work, try this alternative:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - cd client
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: client
    files:
      - ".next/**/*"
      - "public/**/*"
      - "package.json"
      - "package-lock.json"
      - "node_modules/**/*"
      - ".next/standalone/**/*"
      - ".next/static/**/*"
  cache:
    paths:
      - client/node_modules/**/*
      - client/.next/cache/**/*
```

### Step 2: Remove --turbopack from Build Command

**Issue:** The `package.json` has `"build": "next build --turbopack"` which may not work in Amplify.

**Fix:** Temporarily change the build command in `package.json` to:
```json
"build": "next build"
```

Or update `amplify.yml` to override:
```yaml
build:
  commands:
    - npm run build -- --no-turbopack
```

### Step 3: Verify Environment Variables in Amplify Console

1. Go to AWS Amplify Console
2. Select your app
3. Go to **App settings** → **Environment variables**
4. Verify ALL these variables are set:
   - `NEXT_PUBLIC_API_URL` ⚠️ **MOST CRITICAL**
   - `NEXT_PUBLIC_S3_BASE_URL`
   - `NEXT_PUBLIC_RAZORPAY_TEST_KEY_ID`
   - `NEXT_PUBLIC_IMAGE_HERO_SECTION`
   - `NEXT_PUBLIC_IMAGE_BANNER`
   - `NEXT_PUBLIC_IMAGE_ABOUT_SECTION`
   - `NEXT_PUBLIC_IMAGE_CONTACT_SECTION`
   - `NEXT_PUBLIC_IMAGE_LEFT_SIDE`
   - `NEXT_PUBLIC_IMAGE_RIGHT_SIDE`

5. **Important:** After adding/updating environment variables, you MUST redeploy!

### Step 4: Check Build Logs for Errors

In Amplify Console:
1. Go to your app → **Deployments**
2. Click on the latest deployment
3. Check the build logs for:
   - Environment variable warnings
   - Build errors
   - Missing dependencies
   - Next.js build errors

Look for these specific errors:
- `NEXT_PUBLIC_API_URL` undefined → Will default to localhost
- Build failures related to `.next` folder
- Missing environment variables warnings

### Step 5: Verify Next.js Build Output

After build completes, check if these folders exist:
- `client/.next/`
- `client/.next/static/`
- `client/.next/server/`

If these don't exist, the build failed or artifacts are misconfigured.

### Step 6: Test API Endpoint

Your app needs a backend API. Verify:
1. Your backend is deployed and accessible
2. `NEXT_PUBLIC_API_URL` points to the correct backend URL
3. CORS is configured on your backend to allow requests from your Amplify domain

### Step 7: Check Browser Console

After deployment:
1. Open your Amplify domain
2. Open browser DevTools (F12)
3. Check Console for errors:
   - Network errors (404, CORS, etc.)
   - Environment variable warnings
   - API connection failures

### Step 8: Verify Routes

Check if these routes work:
- `/` (home page)
- `/drip-room`
- `/locker`
- `/player-card`

If all return 404, the routing is broken (likely artifact configuration issue).

---

## Common 404 Causes in Amplify

1. **Wrong baseDirectory in amplify.yml** → Fix: Point to `.next` folder
2. **Missing .next folder in artifacts** → Fix: Include `.next/**/*` in files
3. **Next.js not building correctly** → Fix: Remove `--turbopack` flag
4. **Environment variables not set** → Fix: Add all required env vars
5. **Build failing silently** → Fix: Check build logs

---

## Quick Fix Checklist

- [ ] Update `amplify.yml` with correct artifact paths
- [ ] Remove `--turbopack` from build command (or override in amplify.yml)
- [ ] Add `NEXT_PUBLIC_API_URL` environment variable in Amplify Console
- [ ] Add all other required environment variables
- [ ] Redeploy after environment variable changes
- [ ] Check build logs for errors
- [ ] Verify `.next` folder exists after build
- [ ] Test the deployed domain

---

## Expected Build Output Structure

After a successful build, you should see:
```
client/
  .next/
    static/
    server/
    BUILD_ID
    package.json
  public/
  package.json
  node_modules/
```

---

## If Still Getting 404

1. **Check Amplify Rewrites/Routes:**
   - Go to App settings → Rewrites and redirects
   - Add rewrite rule: `/*` → `/index.html` (if using static export)
   - OR ensure Next.js server-side routing is enabled

2. **Check Next.js Output Mode:**
   - If using static export, add to `next.config.ts`:
     ```typescript
     output: 'export'
     ```
   - If using SSR, ensure Amplify supports Node.js runtime

3. **Verify Domain Configuration:**
   - Check if custom domain is properly configured
   - Verify SSL certificate is active

---

## Notes

- Environment variables prefixed with `NEXT_PUBLIC_` are embedded at build time
- You MUST rebuild after changing environment variables
- Next.js 15 with App Router requires proper server configuration for SSR
- Amplify supports Next.js, but configuration must be correct

