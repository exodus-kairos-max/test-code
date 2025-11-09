# 🚨 Critical Deployment Issues & Fix Steps

## ✅ FIXED: amplify.yml Configuration
I've updated your `amplify.yml` to point to the correct `.next` folder. This was likely causing the 404 errors.

**Changed:**
- `baseDirectory: client` → `baseDirectory: client/.next`

---

## ⚠️ CRITICAL: Environment Variable Setup

### Step 1: Add Environment Variables in AWS Amplify Console

Go to **AWS Amplify Console** → Your App → **App settings** → **Environment variables**

Add these variables:

```bash
NEXT_PUBLIC_API_URL=https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com
NEXT_PUBLIC_S3_BASE_URL=https://falsenine-image-storage.s3.ap-south-1.amazonaws.com
NEXT_PUBLIC_RAZORPAY_TEST_KEY_ID=<your-razorpay-key>
NEXT_PUBLIC_IMAGE_HERO_SECTION=hero-section-image.png
NEXT_PUBLIC_IMAGE_BANNER=banner-image.png
NEXT_PUBLIC_IMAGE_ABOUT_SECTION=about-section-image.png
NEXT_PUBLIC_IMAGE_CONTACT_SECTION=contact-section-image.png
NEXT_PUBLIC_IMAGE_LEFT_SIDE=left-side-image.png
NEXT_PUBLIC_IMAGE_RIGHT_SIDE=right-side-image.png
```

**⚠️ IMPORTANT:** 
- Replace `<your-razorpay-key>` with your actual Razorpay key
- After adding variables, you MUST redeploy (trigger a new build)

---

## 🚨 CRITICAL: Hardcoded Localhost URLs Found

**These files have hardcoded `localhost:4000` URLs that WILL FAIL in production:**

1. **`client/src/components/razorpaybutton.tsx`**
   - Line 13: `"http://localhost:4000/api/v2/razorpay/create-transaction"`
   - Line 26: `"http://localhost:4000/api/v2/razorpay/verify-payment"`

2. **`client/src/app/locker/page.tsx`**
   - Line 185: `"http://localhost:4000/api/v2/razorpay/create-transaction"`
   - Line 233: `"http://localhost:4000/api/orders/test-save"`

3. **`client/src/app/test-order/page.tsx`**
   - Line 72: `"http://localhost:4000/api/orders/test-save"`

**These need to be replaced with:**
- Use `API_BASE_URL` from `@/api/config` (which already uses `NEXT_PUBLIC_API_URL`)
- OR use `process.env.NEXT_PUBLIC_API_URL` directly

**Example fix for razorpaybutton.tsx:**
```typescript
// Instead of:
"http://localhost:4000/api/v2/razorpay/create-transaction"

// Use:
`${process.env.NEXT_PUBLIC_API_URL}/api/v2/razorpay/create-transaction`
// OR import and use:
import { API_BASE_URL } from "@/api/config";
`${API_BASE_URL}/api/v2/razorpay/create-transaction`
```

---

## ⚠️ Potential Issue: Turbopack in Production Build

Your `package.json` has:
```json
"build": "next build --turbopack"
```

**If the build fails**, override it in `amplify.yml`:
```yaml
build:
  commands:
    - npm run build -- --no-turbopack
```

Or temporarily change the build script to:
```json
"build": "next build"
```

---

## 📋 Step-by-Step Deployment Checklist

### Immediate Actions:

1. ✅ **amplify.yml fixed** - Already updated to use `client/.next`

2. ⚠️ **Add Environment Variables in Amplify Console:**
   - Go to App settings → Environment variables
   - Add all variables listed above
   - **Most critical:** `NEXT_PUBLIC_API_URL=https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com`

3. ⚠️ **Fix Hardcoded URLs** (Required for production):
   - Update `razorpaybutton.tsx` to use environment variable
   - Update `locker/page.tsx` to use environment variable  
   - Update `test-order/page.tsx` to use environment variable

4. 🔄 **Redeploy:**
   - After adding environment variables, trigger a new deployment
   - After fixing hardcoded URLs, commit and push to trigger deployment

5. ✅ **Verify Build:**
   - Check build logs in Amplify Console
   - Ensure `.next` folder is created
   - Check for any build errors

6. ✅ **Test Deployment:**
   - Visit your Amplify domain
   - Open browser DevTools (F12) → Console
   - Check for API errors
   - Test payment flow (if applicable)

---

## 🔍 How to Verify Environment Variables Are Working

After deployment, check the browser console:

1. Open your deployed site
2. Open DevTools (F12) → Console
3. Type: `console.log(process.env.NEXT_PUBLIC_API_URL)`
4. Should show: `https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com`

If it shows `undefined` or `http://localhost:4000`, the environment variable wasn't set correctly.

---

## 🐛 Debugging 404 Errors

If you still get 404 after these fixes:

1. **Check Build Logs:**
   - Amplify Console → Deployments → Latest build
   - Look for errors about `.next` folder
   - Check if build completed successfully

2. **Verify Artifacts:**
   - After build, check if `client/.next` folder exists
   - Should contain: `static/`, `server/`, `BUILD_ID`

3. **Check Amplify Rewrites:**
   - App settings → Rewrites and redirects
   - May need to add: `/*` → `/index.html` (for static export)
   - OR ensure Next.js SSR is properly configured

4. **Test API Endpoint:**
   - Verify your Lambda URL works: https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com
   - Should return: `{"message":"server chal raha hai"}`
   - Check CORS settings on your Lambda function

---

## 📝 Summary

**What I Fixed:**
- ✅ Updated `amplify.yml` to use correct `.next` directory

**What You Need to Do:**
1. ⚠️ Add `NEXT_PUBLIC_API_URL` and other env vars in Amplify Console
2. ⚠️ Fix hardcoded localhost URLs in 3 files (critical for production)
3. 🔄 Redeploy after making changes
4. ✅ Test the deployed site

**Most Likely Cause of 404:**
- The `amplify.yml` artifact path (now fixed)
- Missing environment variables (you need to add these)
- Hardcoded localhost URLs (will cause API failures)

---

## 🔗 Your API Endpoint

Your Lambda API is working: https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com

Make sure this URL is set as `NEXT_PUBLIC_API_URL` in Amplify environment variables!

