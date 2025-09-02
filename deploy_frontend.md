Excellent choice. Hosting your React admin panel on AWS S3 with CloudFront as a CDN and automating deployments with GitHub Actions is the **gold standard for deploying modern static web applications.**

This architecture is incredibly fast, scalable, secure, and surprisingly cost-effective.

Here is a complete, step-by-step guide to set up this entire CI/CD pipeline.

---

### **CI/CD Goal: Push to `staging` branch -> Automatically Build & Deploy React App to S3/CloudFront**

### **Phase 1: AWS Setup (One-Time Setup)**

We need to create the S3 bucket for hosting and a CloudFront distribution to serve the content globally and provide HTTPS.

#### **1. Create the S3 Bucket for Static Hosting**

1.  Navigate to the **S3** service in the AWS Console.
2.  Click **"Create bucket"**.
    *   **Bucket name:** Choose a globally unique name that matches your domain, if you have one (e.g., `admin.yourtamilapp.com`). If not, something like `icedt-tamilapp-admin-staging` is fine.
    *   **AWS Region:** Choose a region (e.g., `us-east-1`).
    *   **Block Public Access:** **Keep "Block *all* public access" CHECKED.** We will not make the bucket itself public. We will use CloudFront to securely access the content. This is a security best practice.
    *   Click **"Create bucket"**.

#### **2. Create a CloudFront Distribution**

CloudFront is AWS's Content Delivery Network (CDN). It will cache your app's files at edge locations around the world, making it load extremely fast for all users. It also provides a free SSL/TLS certificate for HTTPS.

1.  Navigate to the **CloudFront** service in the AWS Console.
2.  Click **"Create a CloudFront distribution"**.
3.  **Origin domain:** Click in the box and select your newly created S3 bucket (e.g., `icedt-tamilapp-admin-staging.s3...`).
4.  **Origin access:**
    *   Select **"Origin access control settings (recommended)"**.
    *   Click **"Create control setting"**. Leave the defaults and click **"Create"**. This creates a secure policy that allows CloudFront, and only CloudFront, to read files from your S3 bucket.
    *   After creating, you will see a yellow banner that says you need to update the S3 bucket policy. **Click the "Copy policy" button**, and then click the **"Go to S3 bucket permissions"** link (or open S3 in a new tab).
        *   In your S3 bucket's "Permissions" tab, edit the "Bucket policy" and **paste the policy you just copied**. Save changes. This is a critical step.
5.  **Viewer -> Viewer protocol policy:** Select **"Redirect HTTP to HTTPS"**.
6.  **Web Application Firewall (WAF):** For a staging site, you can select "Do not enable security protections." For production, you would want to configure this.
7.  **Settings -> Default root object:** Enter **`index.html`**. This tells CloudFront to serve `index.html` when someone visits the root URL.
8.  Click **"Create distribution"**.

The distribution will take 5-10 minutes to deploy. Once it's ready, you will get a **Distribution domain name** (e.g., `d1234abcd.cloudfront.net`). This will be the public URL for your admin panel.

#### **3. Update the IAM User Policy**

We need to add permissions to our existing `github-actions-deployer` IAM user so it can upload files to the new S3 bucket and invalidate the CloudFront cache (which is crucial for updates to appear immediately).

1.  Go to the **IAM** service -> Users -> `github-actions-deployer`.
2.  Click on the permissions policy (`GitHubActions-Deploy-Policy`).
3.  Click **"Edit"** and switch to the **JSON** tab.
4.  **Add the following two `Statement` blocks** to the existing policy.

    **IMPORTANT:**
    *   Replace `YOUR-ADMIN-BUCKET-NAME` with the name of the S3 bucket you created in Step 1 (e.g., `icedt-tamilapp-admin-staging`).
    *   Replace `YOUR-CLOUDFRONT-DISTRIBUTION-ID` with the ID of the CloudFront distribution you just created (e.g., `E1234567890ABC`).

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            // ... (Your existing S3 and CodeDeploy statements for the backend)
            
            // --- ADD THIS STATEMENT FOR ADMIN APP S3 BUCKET ---
            {
                "Effect": "Allow",
                "Action": [
                    "s3:PutObject",
                    "s3:ListBucket",
                    "s3:DeleteObject"
                ],
                "Resource": [
                    "arn:aws:s3:::YOUR-ADMIN-BUCKET-NAME",
                    "arn:aws:s3:::YOUR-ADMIN-BUCKET-NAME/*"
                ]
            },
            // --- ADD THIS STATEMENT FOR CLOUDFRONT INVALIDATION ---
            {
                "Effect": "Allow",
                "Action": "cloudfront:CreateInvalidation",
                "Resource": "arn:aws:cloudfront::YOUR-AWS-ACCOUNT-ID:distribution/YOUR-CLOUDFRONT-DISTRIBUTION-ID"
            }
        ]
    }
    ```
5.  Click **"Review policy"** and **"Save changes"**.

---

### **Phase 2: GitHub Actions Workflow**

Now we'll create the CI/CD workflow file in your React admin app repository.

1.  **Add AWS Credentials as Secrets:**
    *   Go to your `ICEDT_Admin_App` repository on GitHub.
    *   Go to **Settings -> Secrets and variables -> Actions**.
    *   Add your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` here, just like you did for the backend repo.
    *   Add a new secret:
        *   Name: `CLOUDFRONT_DISTRIBUTION_ID` -> Value: The ID of your CloudFront distribution (e.g., `E1234567890ABC`).

2.  **Create the Workflow File:**
    *   In your `ICEDT_Admin_App` repository, create the folder path `.github/workflows/`.
    *   Inside, create a new file named `deploy-staging.yml`.

    ```yaml
    name: Deploy Admin App to S3 and CloudFront

    on:
      push:
        branches:
          - staging # Trigger on push to the 'staging' branch

    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout code
            uses: actions/checkout@v3

          - name: Setup Node.js
            uses: actions/setup-node@v3
            with:
              node-version: '18' # Use a current LTS version
              cache: 'npm'

          - name: Install dependencies
            run: npm ci

          - name: Build React App
            # This creates the optimized production build in the 'build' folder
            run: npm run build
            env:
              # Set the production API URL for the build
              REACT_APP_API_BASE_URL: http://13.51.194.17/api

          - name: Configure AWS Credentials
            uses: aws-actions/configure-aws-credentials@v2
            with:
              aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
              aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
              aws-region: us-east-1 # Your AWS region

          - name: Deploy to S3
            # The sync command is powerful: it uploads new/changed files and deletes old ones.
            run: aws s3 sync ./build s3://icedt-tamilapp-admin-staging --delete
            # Replace with your bucket name

          - name: Invalidate CloudFront Cache
            # This step is CRUCIAL. It tells CloudFront to fetch the new files from S3 immediately.
            run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
    ```

### **Final Workflow**

1.  Make your changes to the React admin app.
2.  Commit and push your changes to the `staging` branch.
3.  GitHub Actions will automatically:
    *   Install all dependencies.
    *   Build your React app for production, creating a `build` folder.
    *   Sync the contents of the `build` folder to your S3 bucket.
    *   Create a CloudFront invalidation, which forces the CDN to clear its cache and serve the new version of your app immediately.
4.  You can then visit your CloudFront domain name (e.g., `d1234abcd.cloudfront.net`) and see your updated admin panel live.

This setup is fully automated, highly performant, secure, and the standard way to deploy modern frontend applications.
