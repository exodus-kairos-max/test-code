# 🚨 CRITICAL: AWS Amplify Setup for Next.js SSR

## ⚠️ MOST IMPORTANT: Platform Configuration

Your Next.js app uses **Server-Side Rendering (SSR)**, which requires Amplify to be set to **`WEB_COMPUTE`** platform, NOT the default `WEB` platform.

### How to Check/Change Platform:

1. **Via AWS Console:**
   - Go to AWS Amplify Console
   - Select your app
   - Go to **App settings** → **General**
   - Check the **Platform** setting
   - If it shows `WEB`, you need to change it to `WEB_COMPUTE`

2. **Via AWS CLI:**
   ```bash
   aws amplify update-app \
     --app-id <YOUR_APP_ID> \
     --platform WEB_COMPUTE \
     --region ap-south-1
   ```

3. **If you can't change it via Console:**
   - You may need to recreate the app with `WEB_COMPUTE` platform
   - OR use AWS CLI/API to update it

**⚠️ Without `WEB_COMPUTE` platform, your Next.js SSR app will NOT work correctly and will show 404 errors!**

---

## ✅ Environment Variables Setup

### Required Variables in Amplify Console:

Go to: **App settings** → **Environment variables**

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

**After adding variables, you MUST redeploy!**

---

## 📋 Complete Deployment Checklist

### Step 1: Verify Platform (CRITICAL)
- [ ] Check if platform is `WEB_COMPUTE`
- [ ] If not, change to `WEB_COMPUTE` (may require CLI or app recreation)

### Step 2: Environment Variables
- [ ] Add `NEXT_PUBLIC_API_URL` with your Lambda URL
- [ ] Add all other required environment variables
- [ ] Verify no typos in variable names

### Step 3: Fix Hardcoded URLs (REQUIRED)
- [ ] Update `razorpaybutton.tsx` - replace localhost URLs
- [ ] Update `locker/page.tsx` - replace localhost URLs
- [ ] Update `test-order/page.tsx` - replace localhost URLs

### Step 4: Build Configuration
- [ ] `amplify.yml` is updated (already done)
- [ ] If build fails with turbopack, remove `--turbopack` flag

### Step 5: Redeploy
- [ ] Commit and push changes
- [ ] Trigger new deployment in Amplify
- [ ] Monitor build logs

### Step 6: Verify
- [ ] Check build logs for errors
- [ ] Visit deployed domain
- [ ] Open browser console - check for API errors
- [ ] Test main functionality

---

## 🔍 How to Verify Platform Setting

### Option 1: AWS Console
1. Amplify Console → Your App
2. App settings → General
3. Look for "Platform" field
4. Should show: **WEB_COMPUTE**

### Option 2: AWS CLI
```bash
aws amplify get-app --app-id <YOUR_APP_ID> --region ap-south-1
```
Look for `"platform": "WEB_COMPUTE"` in the output.

---

## 🐛 If Platform Can't Be Changed

If you cannot change the platform via Console:

1. **Option A: Use AWS CLI**
   ```bash
   aws amplify update-app \
     --app-id <YOUR_APP_ID> \
     --platform WEB_COMPUTE \
     --region ap-south-1
   ```

2. **Option B: Recreate App with Correct Platform**
   - Create new Amplify app
   - Select `WEB_COMPUTE` platform during creation
   - Connect to same repository
   - Configure environment variables
   - Deploy

3. **Option C: Use Static Export (Not Recommended)**
   - Change `next.config.ts` to use `output: 'export'`
   - This disables SSR but may work with `WEB` platform
   - You'll lose SSR benefits

---

## 📝 Summary

**The #1 cause of 404 errors with Next.js on Amplify:**
- ❌ Using `WEB` platform instead of `WEB_COMPUTE` for SSR apps

**Other critical issues:**
- Missing `NEXT_PUBLIC_API_URL` environment variable
- Hardcoded localhost URLs in code
- Incorrect `amplify.yml` artifacts path (already fixed)

**Action Items:**
1. ✅ Verify/Change platform to `WEB_COMPUTE`
2. ✅ Add all environment variables
3. ✅ Fix hardcoded localhost URLs
4. ✅ Redeploy and test

---

## 🔗 References

- [AWS Amplify Next.js SSR Documentation](https://docs.aws.amazon.com/amplify/latest/userguide/deploy-nextjs-app.html)
- Your API: https://keqnorvuhe.execute-api.ap-south-1.amazonaws.com

