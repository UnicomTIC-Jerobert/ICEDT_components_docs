Excellent. This is a very common and important scenario. Getting control of an existing domain name is the final piece of the puzzle to get HTTPS working and make your application truly production-ready.

Here is a complete guide covering **what you need to ask your client** and **what steps you need to take** in AWS to attach `icedt.education` to your servers.

---

### **Part 1: What You Need to Ask Your Client**

To manage their domain's DNS, you need access to their **Domain Registrar**. The registrar is the company where they originally bought the domain `icedt.education` (e.g., GoDaddy, Namecheap, Google Domains, etc.).

**You have two options here. Present both to your client:**

#### **Option A: The Recommended, Secure Method (Delegate DNS to AWS)**

This is the best long-term solution. It will give you full control over the domain's DNS records from within your own AWS account, making future changes (like adding email records) much easier.

*   **What to ask the client:**
    > "Could you please provide me with login access to your domain registrar where `icedt.education` is registered? I need to update the **Name Servers (NS records)**. This will delegate the DNS management for the domain to our AWS account, allowing me to securely point it to our new servers. This is a one-time change, and afterwards, you won't need to give me access to the registrar again."

*   **Why it's better:**
    *   **Full Control:** You manage all records (`A`, `CNAME`, `MX`, etc.) from one place (AWS Route 53).
    *   **Security:** You don't need persistent access to their registrar account.
    *   **Integration:** It integrates seamlessly with other AWS services like Certificate Manager.

#### **Option B: The Simpler, Less Control Method (Client Manages DNS)**

If the client is hesitant to give you access to their registrar or wants to keep managing the DNS themselves.

*   **What to ask the client:**
    > "Could you please log in to your domain registrar for `icedt.education` and give me the ability to **add or edit DNS records**? Specifically, I will need to create two 'A' records, one for our staging server and one for production, to point subdomains to the new server IP addresses."

*   **If they can't give you access, you'll have to provide them with the records to create:**
    > "Please create the following two 'A' records in your DNS management panel for `icedt.education`:"
    >
    > *   **Record 1 (Staging):**
    >     *   **Type:** `A`
    >     *   **Host/Name:** `staging-api` (or `api-staging`)
    >     *   **Value/Points to:** `13.51.194.17` (Your Staging EC2 IP)
    >
    > *   **Record 2 (Production):**
    >     *   **Type:** `A`
    >     *   **Host/Name:** `api`
    >     *   **Value/Points to:** `[Your_Future_Production_EC2_IP]`

---

### **Part 2: Steps to Take in AWS (Assuming You Chose Option A)**

This is the full process for taking over DNS management with AWS Route 53.

#### **Step 1: Create a Public Hosted Zone in AWS Route 53**

This tells AWS that you are now the authoritative source for the `icedt.education` domain.

1.  Navigate to the **Route 53** service in the AWS Console.
2.  In the left sidebar, click on **"Hosted zones"**.
3.  Click the **"Create hosted zone"** button.
4.  **Domain name:** Enter **`icedt.education`** (without `www` or anything else).
5.  **Type:** Select **"Public hosted zone"**.
6.  Click **"Create hosted zone"**.

#### **Step 2: Get the AWS Name Servers**

1.  After the hosted zone is created, you will be on its details page.
2.  You will see a record of type **`NS` (Name Server)**. This record will have **four values** that look like this:
    *   `ns-1234.awsdns-56.co.uk.`
    *   `ns-567.awsdns-01.net.`
    *   `ns-890.awsdns-12.org.`
    *   `ns-12.awsdns-03.com.`
3.  **Copy these four server names.** These are the addresses of AWS's DNS servers.

    

#### **Step 3: Update the Name Servers at the Client's Domain Registrar**

This is the **only step** where you need access to the client's registrar account (e.g., GoDaddy).

1.  Log in to the client's domain registrar.
2.  Find the DNS management or Name Server settings for `icedt.education`.
3.  There will be an option to use "custom name servers."
4.  **Delete any existing name servers** and **enter the four AWS name server addresses** you copied in the previous step.
5.  Save the changes.

**What happens now?** It can take anywhere from a few minutes to 48 hours for this change to propagate across the internet (this is called DNS propagation). Once it's complete, all DNS queries for `icedt.education` will be directed to your AWS Route 53 hosted zone.

#### **Step 4: Create the DNS 'A' Records in Route 53**

Now that AWS is in control, you can create the records to point subdomains to your servers.

1.  Go back to your **Route 53 Hosted Zone** for `icedt.education`.
2.  Click **"Create record"**.
3.  **Create the Staging Record:**
    *   **Record name:** `staging-api` (This will create the full domain `staging-api.icedt.education`)
    *   **Record type:** `A`
    *   **Value:** `13.51.194.17` (Your staging EC2 IP)
    *   **TTL (Time to Live):** Leave as default (e.g., 300 seconds).
    *   Click **"Create records"**.

4.  **Create the Production Record (when ready):**
    *   Click **"Create record"** again.
    *   **Record name:** `api` (This will create `api.icedt.education`)
    *   **Record type:** `A`
    *   **Value:** `[Your_Future_Production_EC2_IP]`
    *   Click **"Create records"**.

#### **Step 5: Provision an SSL Certificate with AWS Certificate Manager (ACM)**

Now you can get a free public SSL certificate for your new subdomains.

1.  Navigate to the **AWS Certificate Manager (ACM)** service. **IMPORTANT: Make sure you are in the `us-east-1` (N. Virginia) region.** Certificates used with CloudFront *must* be in this region.
2.  Click **"Request a certificate"**.
3.  Select **"Request a public certificate"**.
4.  **Fully qualified domain name:** Add both of your subdomains:
    *   `api.icedt.education`
    *   `staging-api.icedt.education`
5.  **Validation method:** Select **"DNS validation"**.
6.  Click **"Request"**.
7.  On the next screen, you will see your new certificate request is "Pending validation". Expand it.
8.  You will see a blue button that says **"Create records in Route 53"**. Click it.
9.  A confirmation screen will appear. Click **"Create records"**.

Because you are using Route 53 for DNS, ACM can automatically create the special CNAME records needed to prove you own the domain. The certificate status will change from "Pending" to "Issued" within a few minutes to an hour.

You now have a valid SSL certificate ready to be used by your CloudFront distributions (for the admin app) and your Nginx servers (for the API). This completely solves the HTTPS issue.
