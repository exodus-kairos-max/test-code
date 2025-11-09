# 🚀 Final Deployment Checklist

## ✅ Code Changes Completed

All code fixes have been applied:

1. ✅ **Fixed hardcoded localhost URLs** in:
   - `client/src/components/razorpaybutton.tsx`
   - `client/src/app/locker/page.tsx`
   - `client/src/app/test-order/page.tsx`

2. ✅ **Updated build command** - Removed `--turbopack` from production build

3. ✅ **Updated amplify.yml** - Correct artifact paths for Next.js

---

## ⚠️ CRITICAL: Environment Variables Setup

### Step 1: Go to AWS Amplify Console

1. Navigate to: https://console.aws.amazon.com/amplify
2. Select your app
3. Go to: **App settings** → **Environment variables**

### Step 2: Add These Environment Variables

Copy and paste these EXACT variable names and values:

```bash
NEXT_PUBLIC_API_URL=https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com
```

```bash
NEXT_PUBLIC_S3_BASE_URL=https://falsenine-image-storage.s3.ap-south-1.amazonaws.com
```

```bash
NEXT_PUBLIC_RAZORPAY_TEST_KEY_ID=<YOUR_RAZORPAY_KEY_HERE>
```

```bash
NEXT_PUBLIC_IMAGE_HERO_SECTION=hero-section-image.png
```

```bash
NEXT_PUBLIC_IMAGE_BANNER=banner-image.png
```

```bash
NEXT_PUBLIC_IMAGE_ABOUT_SECTION=about-section-image.png
```

```bash
NEXT_PUBLIC_IMAGE_CONTACT_SECTION=contact-section-image.png
```

```bash
NEXT_PUBLIC_IMAGE_LEFT_SIDE=left-side-image.png
```

```bash
NEXT_PUBLIC_IMAGE_RIGHT_SIDE=right-side-image.png
```

**⚠️ IMPORTANT:**
- Replace `<YOUR_RAZORPAY_KEY_HERE>` with your actual Razorpay test key ID
- Make sure there are NO spaces before or after the `=` sign
- Variable names are case-sensitive - use EXACT names shown above

---

## 🚨 MOST CRITICAL: Platform Configuration

### Check Your Amplify Platform Setting

Your Next.js app uses **Server-Side Rendering (SSR)**, which requires:

**Platform must be: `WEB_COMPUTE`** (NOT `WEB`)

### How to Check/Change:

1. **In Amplify Console:**
   - Go to: **App settings** → **General**
   - Look for **Platform** field
   - If it shows `WEB`, you MUST change it to `WEB_COMPUTE`

2. **If you can't change it in Console, use AWS CLI:**
   ```bash
   aws amplify update-app \
     --app-id <YOUR_APP_ID> \
     --platform WEB_COMPUTE \
     --region ap-south-1
   ```

3. **To find your App ID:**
   - In Amplify Console, look at the URL or app details
   - Format: `d2lm32ln0kwos3` (from your logs)

**⚠️ Without `WEB_COMPUTE` platform, your app will show 404 errors!**

---

## 📋 Complete Deployment Steps

### Step 1: Commit and Push Code Changes
```bash
git add .
git commit -m "Fix: Replace hardcoded localhost URLs with environment variables"
git push origin main
```

### Step 2: Set Environment Variables in Amplify
- Follow the environment variables section above
- Add ALL 9 variables
- **Most critical:** `NEXT_PUBLIC_API_URL`

### Step 3: Verify Platform Setting
- Check that platform is `WEB_COMPUTE`
- Change if necessary (may require CLI)

### Step 4: Trigger New Deployment
- After adding environment variables, Amplify will auto-deploy
- OR manually trigger: **Deployments** → **Redeploy this version**

### Step 5: Monitor Build Logs
- Go to: **Deployments** → Latest build
- Check for:
  - ✅ Build completed successfully
  - ✅ No environment variable warnings
  - ✅ `.next` folder created

### Step 6: Test Deployment
1. Visit your Amplify domain
2. Open browser DevTools (F12) → Console
3. Check for errors
4. Test main functionality:
   - Home page loads
   - API calls work (check Network tab)
   - Payment flow (if applicable)

---

## 🔍 Verification Steps

### Verify Environment Variables Are Working

After deployment, open browser console on your site and run:

```javascript
console.log('API URL:', process.env.NEXT_PUBLIC_API_URL);
```

**Expected output:**
```
API URL: https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com
```

**If it shows `undefined` or `http://localhost:4000`:**
- Environment variable wasn't set correctly
- Re-check variable name (case-sensitive)
- Redeploy after fixing

### Verify API Connection

1. Open browser DevTools → Network tab
2. Perform an action that calls your API
3. Check if requests go to: `https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com`
4. Should NOT see requests to `localhost:4000`

---

## 🐛 Troubleshooting

### If you still get 404 errors:

1. **Check Platform:**
   - Must be `WEB_COMPUTE`, not `WEB`
   - This is the #1 cause of 404s

2. **Check Build Logs:**
   - Look for errors about `.next` folder
   - Check if build completed successfully

3. **Check Environment Variables:**
   - Verify all variables are set
   - Check for typos in variable names
   - Ensure `NEXT_PUBLIC_API_URL` is set correctly

4. **Clear Cache and Redeploy:**
   - App settings → Build settings → Clear cache
   - Trigger new deployment

### If API calls fail:

1. **Check CORS on Lambda:**
   - Your Lambda function must allow requests from your Amplify domain
   - Add your Amplify domain to CORS allowed origins

2. **Verify Lambda URL:**
   - Test: https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com
   - Should return: `{"message":"server chal raha hai"}`

3. **Check Network Tab:**
   - See what URL is being called
   - Check for CORS errors
   - Check response status codes

---

## 📝 Summary of Changes Made

### Files Modified:

1. **`client/src/components/razorpaybutton.tsx`**
   - Added: `import { API_BASE_URL } from "@/api/config"`
   - Changed: `"http://localhost:4000/..."` → `${API_BASE_URL}/...`

2. **`client/src/app/locker/page.tsx`**
   - Added: `import { API_BASE_URL } from "@/api/config"`
   - Changed: 2 hardcoded URLs to use `API_BASE_URL`

3. **`client/src/app/test-order/page.tsx`**
   - Added: `import { API_BASE_URL } from "@/api/config"`
   - Changed: hardcoded URL to use `API_BASE_URL`

4. **`client/package.json`**
   - Changed: `"build": "next build --turbopack"` → `"build": "next build"`

5. **`amplify.yml`**
   - Changed: `baseDirectory: client` → `baseDirectory: client/.next`

---

## ✅ Pre-Deployment Checklist

Before deploying, verify:

- [ ] All code changes committed and pushed
- [ ] Environment variables added in Amplify Console
- [ ] `NEXT_PUBLIC_API_URL` set to your Lambda URL
- [ ] Platform is set to `WEB_COMPUTE`
- [ ] Razorpay key ID is set (if using payments)
- [ ] All image environment variables are set

---

## 🎯 Expected Result

After completing all steps:

1. ✅ Build completes successfully
2. ✅ No 404 errors on deployed site
3. ✅ API calls go to Lambda URL (not localhost)
4. ✅ All pages load correctly
5. ✅ Payment/order functionality works

---

## 📞 Quick Reference

- **Your Lambda API:** https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com
- **API Config File:** `client/src/api/config.ts`
- **Environment Variables:** Amplify Console → App settings → Environment variables
- **Platform Setting:** Amplify Console → App settings → General → Platform

---

**Good luck with your deployment! 🚀**

