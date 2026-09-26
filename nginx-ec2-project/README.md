# 🚀 NGINX on AWS EC2 with a Custom Domain

A step-by-step record of how I launched an NGINX web server on Amazon EC2 and mapped it to a custom domain purchased through Cloudflare — part of my hands-on Networking & DevOps learning journey.

By the end, the site is live at my own domain, powered by AWS EC2 and NGINX.

---

## 🌐 Step 1 — Buy a Domain on Cloudflare

1. Went to the [Cloudflare Registrar](https://www.cloudflare.com/products/registrar/)
2. Searched for a domain and purchased **mustafawebsite.co.uk** for **2 years at £8.03**
3. Cloudflare automatically manages DNS for the domain once purchased — no extra setup needed at this stage

> ✅ AWS Route 53 works just as well if you'd rather stay entirely inside AWS.

---

## 🖥 Step 2 — Launch an EC2 Instance

Opened the **AWS EC2 console** → searched "EC2" in the AWS search bar → **Launch Instance**.

**Basic settings used:**
| Setting | Value |
|---|---|
| Name | `nginx-server` |
| AMI | Amazon Linux 2023 (Free Tier eligible) |
| Instance type | `t3.micro` (Free Tier eligible) |
| Key pair | Created new — `nginx-ec2-key.pem` (downloaded and kept safe) |

**Security group — inbound rules:**
| Type | Port | Source |
|---|---|---|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

Clicked **Launch Instance** and waited for the state to show **Running**.

Then went to the instance → **Instance Summary** → copied the **Public IPv4 address** for later use.

---

## 🔑 Step 3 — Connect to the EC2 Instance via SSH

Since I was working from **WSL**, the `.pem` file downloaded to my Windows filesystem first, so I had to move it into Linux before use:

```bash
mkdir -p ~/.ssh
cp /mnt/c/Users/<MyWindowsUsername>/Downloads/nginx-ec2-key.pem ~/.ssh/
chmod 400 ~/.ssh/nginx-ec2-key.pem
```

Then connected:

```bash
ssh -i ~/.ssh/nginx-ec2-key.pem ec2-user@13.42.76.65
```

---

## ⚙️ Step 4 — Install & Start NGINX on EC2

Ran the following once connected:

```bash
sudo yum update -y
sudo yum install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx   # confirmed "active (running)"
```

**Test:** opened `http://13.42.76.65` in a browser — the default NGINX welcome page loaded successfully.

---

## 🌍 Step 5 — Point the Domain to the EC2 IP (Cloudflare)

1. Cloudflare dashboard → my domain → **DNS** → **Add record**
2. Configured:

| Field | Value |
|---|---|
| Type | A |
| Name | `@` (root domain) |
| IPv4 address | `13.42.76.65` |
| TTL | Auto |
| Proxy status | DNS only (grey cloud) — for initial testing |

3. Saved the record.
<img width="1777" height="721" alt="image" src="https://github.com/user-attachments/assets/4a62d664-f611-4ecc-a701-f6262b4382a0" />

---

## ✅ Step 6 — Confirm DNS Is Live

From my local machine:

```bash
nslookup mustafawebsite.co.uk
# or
dig +short mustafawebsite.co.uk
```

Both returned the EC2 public IP as expected.

Visited `http://mustafawebsite.co.uk` in the browser — the NGINX welcome page loaded correctly, confirming the domain was correctly routed to the instance.

---

## 🏁 Milestone Reached

NGINX is live on EC2 and successfully mapped to my custom domain.

---

## 🛠 Extra — Replaced the Default Page with a Custom One

**Goal:** swap NGINX's default landing page for a personal one.

```bash
cd /usr/share/nginx/html
sudo cp index.html index.html.bak    # backup the original
sudo nano index.html                 # deleted default content, pasted in custom HTML
```

Saved with `Ctrl+O`, `Enter`, exited with `Ctrl+X`, then reloaded NGINX:

```bash
sudo systemctl reload nginx
```

Visiting my domain now shows a custom landing page introducing myself and the tech stack used (AWS EC2, NGINX, Cloudflare DNS) instead of the NGINX default.

---

## 📦 Stack Used

- **AWS EC2** — Amazon Linux 2023, t3.micro
- **NGINX** — web server
- **Cloudflare** — domain registration + DNS
- **WSL (Ubuntu on Windows)** — local dev environment

---


