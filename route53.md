# Route 53 for Subdomain
To route traffic from your Google Cloud subdomains to your AWS Route 53 hosted zone, you'll need to update the DNS records in Google Cloud DNS to point to the corresponding AWS Route 53 name servers.

Here's a general guide on how to do this:

### 1. Get AWS Route 53 Name Servers:

In the AWS Management Console:

- Go to the Route 53 console.
- Select the hosted zone for your domain (e.g., `skyticket.com`).
- Note down the values of the four name servers listed in the "NS" (Name Server) records.

### 2. Update Google Cloud DNS:

In the Google Cloud Console:

- Go to the "Networking" section.
- Select "Cloud DNS."
- Click on the DNS zone for your domain (e.g., `skyticket.com`).
- Update the DNS records for each subdomain:

  - For `abctest.skyticket.com`:
    - Create an "A" or "CNAME" record pointing to the IP address or hostname in AWS where the service is hosted.

  - For `abcstage.skyticket.com`:
    - Create an "A" or "CNAME" record pointing to the corresponding AWS resource.

  - For `abcprod.skyticket.com`:
    - Create an "A" or "CNAME" record pointing to the corresponding AWS resource.

  - For `abcde**.skyticket.com` (wildcard subdomain):
    - Create a wildcard "A" or "CNAME" record pointing to the corresponding AWS resource.

### 3. Wait for DNS Propagation:

Changes to DNS records may take some time to propagate throughout the internet. This process is known as DNS propagation, and it usually takes anywhere from a few minutes to 48 hours.

### 4. Test:

After DNS propagation is complete, you can test if the subdomains are correctly resolving to the AWS resources. You can use tools like `dig` or `nslookup` to check the DNS resolution.

Keep in mind that AWS Route 53 is just a DNS service, and you need to ensure that the actual services or resources in AWS are configured to handle requests for these subdomains.

Remember to adapt these instructions based on the specific configurations and services you have set up in both Google Cloud and AWS.

# Whild card subdomain
When you're using wildcard DNS records in AWS Route 53, the DNS itself only handles the resolution of subdomains to IP addresses or endpoints. The routing to different services based on the subdomain is typically done at the service level within AWS.

Here's a general guide on how you might route different wildcard subdomains to different services:

### Example: Routing Different Wildcard Subdomains to Different Services

#### 1. **Set Up Different Services:**

For each distinct service or resource you want to associate with a wildcard subdomain:

- **Example Services:**
  - S3 bucket for static content (`abcde-static-content`).
  - EC2 instance for a dynamic web application (`abcde-dynamic-app`).
  - Elastic Load Balancer (`abcde-load-balancer`).
  - API Gateway (`abcde-api-gateway`).

#### 2. **Configure Each Service:**

- Ensure that each service is configured to handle requests for the corresponding wildcard subdomain.

  - **Example Configurations:**
    - For S3, configure the bucket and enable static website hosting.
    - For EC2, configure the web server to handle requests for the specific subdomain.
    - For Load Balancer, configure listeners and routes.
    - For API Gateway, configure the API to handle requests for the subdomain.

#### 3. **Update Route 53 Wildcard Records:**

- Go to the Route 53 console.
- Select the hosted zone for your domain (`skyticket.com`).
- Create a new record set for the wildcard subdomain (`abcde*.skyticket.com`).
- For each record set, set the value to the corresponding service's endpoint.

  - **Example Records:**
    - `abcde01.sktyticket.com` points to the S3 bucket.
    - `abcde-stage.sktyticket.com` points to the EC2 instance.
    - `abcde-xyz.sktyticket.com` points to the Load Balancer.
    - And so on.

### Important Notes:

- Each service or resource needs to be appropriately configured to handle the requests for the specific subdomain.

- DNS changes may take some time to propagate, so be patient and test after a while.

- Adjust the instructions based on the specific AWS service you are using for each wildcard subdomain.

By following these steps, you can route different wildcard subdomains to different services within AWS based on your specific requirements.
