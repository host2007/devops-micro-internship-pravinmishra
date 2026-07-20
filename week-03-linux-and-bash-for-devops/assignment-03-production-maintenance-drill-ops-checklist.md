# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

<![Image1](screenshots/Assignment3_task1.png)>

---

#### Screenshot 2 — Output of `ip a`

<![Image2](screenshots/Assignment3_task2.png)>

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

<![Image3](screenshots/Assignment3_task3.png)>

---

#### Screenshot 4 — Output of `sudo ufw status`

<![Image4](screenshots/Assignment3_task4.png)>

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

When i ran the the command sudo ss -tulpen, the output display 0.0.0.0:80 showing that Nginx is actively listening for HTTP traffice on port 80 accros all network interface as seen below "tcp      LISTEN    0         511                     0.0.0.0:80  ". Additionally, when i copied my Ip address into the browser using http//0.0.0.0.:80 displays my-React-App which is a testament that Nginx runs on port 80

---

**2. What proves SSH is active on port 22?**

The same command also show that port 22 is open and that i can connect to the server from my device through port 22 " tcp      LISTEN    0         4096                    0.0.0.0:22 ... sshd
confirming the SSH deamon running in the background is actively listening on port 22 across all interface. This is what allow my remote login to the server.
---

**3. Did you find any unexpected open ports? Explain briefly.**

No unexpected ports were found opened aside from port Nginx port 80 and SSH port 22. The only other listening port is shown is "tcp      LISTEN    0         4096                 127.0.0.53  127.0.0.54 These ports are not reachable from outside the server.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

<![Image5](screenshots/Assignment3_task5.png)>

---

#### Screenshot 2 — Output of `sudo nginx -t`

<![Image6](screenshots/Assignment3_task6.png)>

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

<![Image7](screenshots/Assignment3_task7.png)>

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart in production, it means that nginx is not active, enabled or running as a managed system service when started the webserver stops serving traffic and my application will be unreachable and cannot be displayed. Any user using the site will get a connection error or timeout

---

**2. What's your basic rollback plan?**

The rollback plan will be to run sudo nginx -t first to validate the config syntax- this catches most errors before they reach a restart stage. 2. If a restart is is attempted and fails, then it will be necessary to check systemctl status nginx --pager and sudo journalctl -u nginx  --pager -n 50 to see the exert error. Additionally, if the failure is due to bad configuration, change the fix to revert the config file back to its last known good version. i.e from a back up or version control and re-run sudo nginx -t followed by sudo systemctl restart nginx again.

It is also important to note that keeping a backup copy of the working configuration file snapshot before making changes is the best safeguard because it allows an immediate rollback without needing to debug under pressure. 

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

<![Image8](screenshots/Assignment3_task8.png)>

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

<![Image9](screenshots/Assignment3_task9.png)>

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

<![Image10](screenshots/Assignment3_task10.png)>

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No Error

---

**2. If there were no errors, what does that indicate about the system?**

It shows that nothing is broken. It means nginx error log is empty or the site has no entry in recent history.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes. After running curl localhost on the server, the request appeared in /var/log/nginx/access.log as 
127.0.0.1 - - [20/Jul/2026:01:42:46 +0000] "GET / HTTP/1.1" 200 644 "-" "curl/8.18.0"
This shows that the response cycle work end to end  and he source (127.0.0.1) confirms local traffic flow.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

<![Image11](screenshots/Assignment3_task11.png)>

---

#### Screenshot 2 — Output of `free -h`

<![Image12](screenshots/Assignment3_task12.png)>

---

#### Screenshot 3 — Output of `df -h`

<![Image](screenshots/Assignment3_task13.png)>

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

<![Image 14](screenshots/Assignment3_task14.png)>

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

The most critical resource is the Memory. This is because the total RAM for our free tier t2.micro is only 908Mi (~1GB)  presently have a space of 585Mi is available, though healthy for now, may be used up quickly in the future.

---

**2. What happens if disk becomes 100% full in a production server?**

if disk become 100% full, the disk can block logging, break deployments, or crash  Nginx itself.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

<![Image15](screenshots/Assignment3_task15.png)>

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

<![Image16](screenshots/Assignment3_task16.png)>

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

<![Image17](screenshots/Assignment3_task17.png)>

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

To confirm that the correct version of the application is deployed i will verify my name on the app and the date i deployed the application. This is a confirmation of my version of the application.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

<![Image18](screenshots/Assignment3_task18.png)>

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

<![Image19](screenshots/Assignment3_task19.png)>

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

<![Image20](screenshots/Assignment3_task20.png)>

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

The configuration failure was due to the emptyness of the /var/www/html file because the content was removed.

---

**2. How did you fix the issue?**

To fix the issue, i moved the content of the backup file to my /var/www/html file and restart nginx and the server came back up.

---

**3. How can you avoid this kind of issue in real production systems?**

To avoid this issue in real life production, prevention needs to go beyond just testing config syntax but ensuring that the directory content contains the require file and that the file content are secure and intact.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

<![Image21](screenshots/Assignment3_task21.png)>

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

<![Image22](screenshots/Assignment3_task22.png)>

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The application could not connect because Nginx was inactive .

---

**2. How did you fix the issue and restore the application?**

I restarted Nginx and once that was done Nginx came back on and was active.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

Ensure that the system health check is automated and that check done prior to production going live 

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH key-based authentication is more secured because the .pem is complex is not easy to guess unlike the usual password that is vulnerable to bruce-force attack.

---

**2. Why should only required ports be open on a production server?**

The required port should be opened on production to minimize possible surface attack and to limit damage should the system get compromised. Also it reduces noise and makes real threats easier to track or spot.

---

**3. Why is it important for Nginx to be enabled on boot?**

It is important for Nginx to be enabled on boot to ensures that the application recovers automatically after a reboot. Additionally, it reduces downtime and reliance on manual reboot of the server.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

 The risks of sharing secrets, keys, or credentials publicly is that it exposes your key to the public allowing unauthorized access to your server. This can cause data breach.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Cloud resources should be stopped or terminated when they are no longer needed to avoid incuring unnecessary cost.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

---

#### Screenshot — Published LinkedIn post

Add your screenshot here.

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [✅ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [✅] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [✅ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [✅ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [✅ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [✅ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [✅ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [✅ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [✅ ] Full Name visible in all required screenshots
- [✅ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*