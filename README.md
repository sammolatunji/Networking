**Networking: Domain + EC2 + DNS**

**Deploying Nginx on an EC2 Instance & Connecting It to a Cloudflare
Domain**

This project walks through the full process of purchasing a domain on
Cloudflare, launching an EC2 instance running **Nginx**, assigning
Public **IP**, and linking everything together using DNS A-record.

You'll also learn how to customize the Nginx frontend and how to enable
HTTPS on your domain.

**Step 1: Purchase a Domain on Cloudflare**

- Navigate to Cloudflare

- Select Register a domain

- Search for an available domain and purchase

**Step 2: Launch an EC2 Instance:**

- Go to EC2 and click **Launch Instance**

- Choose **Ubuntu**

![](media/image1.png){width="6.298611111111111in" height="2.375in"}

- Select **t3.micro** instance type

![](media/image2.png){width="6.298611111111111in" height="2.175in"}

- Create a new key pair (.pem RSA)

- Configure security group:

  a.  *Allow SSH -\> My IP*

  b.  *Allow HTTPs*

  c.  *Allow HTTP*

![](media/image3.png){width="6.298611111111111in"
height="2.623611111111111in"}

Create a new Key pair (.pem, RSA)

![](media/image4.png){width="5.771129702537183in"
height="4.423838582677165in"}

**Launch the instance.**

- **Step 3: SSH into the EC2 Instance:**

- Select your instance and click **Connect**

- Open your terminal and navigate to the directory with your **.pem**
  file

- Secure the key:

  chmod 400 \<name_of_pem_file\>.pem

- Connect via SSH:

  ssh -i \<name_of_pem_file\>.pem
  <ubuntu@ec2-16-171-152-217.eu-north-1.compute.amazonaws.com>

![](media/image5.png){width="6.298611111111111in"
height="3.7506944444444446in"}

**Step 5: Link Your Domain (Cloudflare -\> EC2)**

> **Add the A record in Cloudflare**

1.  Log in to the Cloudflare dashboard and click your domain.

2.  Go to **DNS → Records** and click **Add record**.

3.  Fill in:

    - **Type:** A

    - **Name:** nginx.your domain

    - **IPv4 address:** your Elastic IP from AWS Instance

    - **SSL:** Flexible

    - **TTL:** Flexible

![](media/image6.png){width="6.298611111111111in"
height="3.546527777777778in"}

Configure SSL/TLS as Flexible

![](media/image7.png){width="6.298611111111111in" height="2.83125in"}

> **What is an A Record?**
>
> It maps a **domain name** to an **IPv4 address**.\
> This allows users to enter *yourdomain.com* instead of an IP.
>
> DNS workflow (simplified):

- Browser asks DNS resolver

- If not cached -\> resolver asks Root Server

- Root Server -\> sends TLD server address

- TLD server -\> sends authoritative nameserver

- Nameserver -\> returns the correct IP

- Resolver -\> sends IP back to browser

  **Step 6: Install & Run Nginx**

<!-- -->

- Update packages and install Nginx:

- sudo apt update

- sudo apt upgrade

- sudo apt install nginx

- Start Nginx:

- Check status: sudo systemctl status nginx

  ![](media/image8.png){width="6.298611111111111in"
  height="2.5055555555555555in"}

  **Checked the config and tested locally**

- cd /etc/nginx/sites-available

- cd /etc/nginx/sites-available

- sudo vi default

- sudo nginx -t

- sudo ss -tlnp

![](media/image9.png){width="6.298611111111111in"
height="2.472916666666667in"}

> **Now visit your domain or IP, the default Nginx page should appear.**

**Final Result :** Tested in a browser

> A fully deployed, fully customised Nginx server running on an EC2
> instance, connected to your own Cloudflare domain, with a stable
> Elastic IP and with HTTPS.
>
> ![](media/image10.png){width="6.298611111111111in" height="3.5125in"}
>
> **Challenges and how I solved them**
>
> **1. Permission denied (publickey) on SSH**
>
> **Problem:** SSH successfully connected to the server, but the server
> rejected the public key presented during authentication**.**
>
> **Cause:** my SSH client was configured with an incorrect key path. As
> a result, SSH sent a public key that did not match any authorized key
> on the server, causing the server to deny access.
>
> **Fix:** I found the right path to my private key pair on the terminal
> and SSH with a correct key pair
>
> **2. Getting the systemctl syntax right**
>
> **Problem:** systemctl nginx and nginx start didn\'t do what I
> expected.
>
> **Fix:** I read man systemctl. The syntax is systemctl \<action\>
> \<service\>, for example sudo systemctl status nginx.
>
> **3. Cloudflare Error 521: Web server is down**
>
> **Problem:** The domain showed a Cloudflare 521 error, but NGINX was
> running.
>
> **Cause:** A 521 means Cloudflare reached my server but the connection
> was refused. Cloudflare was trying to connect to my server over HTTPS
> on port 443, and NGINX was only listening on port 80.
>
> **Fix:** I changed Cloudflare\'s SSL/TLS encryption mode to
> **Flexible**, so Cloudflare connects to my server over HTTP on port
> 80.
>
> **Trade-off:** In Flexible mode, the traffic between Cloudflare and
> EC2 isn\'t encrypted. The proper fix is **Full (strict)**: put a
> certificate on NGINX (a Cloudflare Origin Certificate or Let\'s
> Encrypt) so it serves HTTPS on 443.
>
> **What I learnt**

- **CIDR and subnets:** /26 means 26 network bits and 6 host bits, so 64
  addresses (62 usable). /32 means exactly one IP, which is why I used
  it to restrict SSH.

- **A records:** they map a hostname to an IPv4 address.

- **Cloudflare proxying:** with the orange cloud on, DNS lookups return
  Cloudflare\'s IPs, not my server\'s. Cloudflare sits in the middle and
  hides the origin IP.

- **Security groups:** an AWS firewall at the instance level. Inbound
  traffic is denied unless a rule allows it.

- **Cloudflare error codes:** 521 means the connection was refused
  (nothing listening), and 522 means it timed out (usually a firewall or
  security group).

- **Debugging one layer at a time:** I worked out what failed using
  nslookup by checking each layer in turn: DNS → network → server →
  service.

- **Good habits:** back up a config before editing it, run nginx -t
  before reloading, and apt update on a new server.
