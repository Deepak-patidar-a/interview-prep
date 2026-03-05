# Deployment Pipeline

## Question
Walk me through your complete deployment process — from a developer raising a PR to code being live on a customer's server. Where does each tool fit — Git, Jenkins, WinSCP, PuTTY, PM2?

---

## My Answer
Developer raises a PR, we do code review and merge to the main branch. For deployment we create a release branch and trigger a Jenkins job configured with build parameters — branch, environment, port, and .env configuration. Once the TAR file of both client and server code is ready, we move it to the remote server using WinSCP. From there we unzip using PuTTY commands, give read/write permissions, point to the deployed service, and start the server. PM2 manages running Node.js processes, handles logs, restarts, and shows which instances are running.

## What I Got Right
- Correct Jenkins build with parameters setup
- TAR file bundling client + server — correct artifact concept
- WinSCP for file transfer — correct usage
- PuTTY for remote commands — correct
- PM2 role — logs, restart, instance management — correct

## What I Got Wrong / Could Add
- The pipeline has a manual file transfer step via WinSCP — this makes it semi-automated, not fully automated
- Did not mention PM2 cluster mode and its connection to load balancers
- Did not explain what WinSCP and PuTTY being Windows tools tells us about the infrastructure

---

## Model Answer
When a PR is merged to the release branch, we trigger a Jenkins job configured with build parameters — branch name, target environment, port, and environment variables. Jenkins bundles the frontend and backend into a single TAR file. We transfer that TAR to the remote server using WinSCP, unzip it via PuTTY commands, set correct file permissions, and point the service to the new build. PM2 manages the running Node.js processes in cluster mode to utilize all CPU cores, handles automatic restarts on crashes, and provides centralized logging. A load balancer sits in front distributing traffic across PM2 instances.

---

## Key Things to Remember

**Each tool's role:**
- Git — source control, branching, PR process
- Jenkins — CI/CD, build with parameters, creates TAR artifact
- WinSCP — file transfer from build machine to server (Windows-based tool)
- PuTTY — remote terminal to run commands on server (Windows-based tool)
- PM2 — Node.js process manager on the server

**What WinSCP and PuTTY reveal:**
- Both are Windows tools — means developers are on Windows machines connecting to servers
- This is important infrastructure context worth mentioning in interviews

**PM2 cluster mode:**
- Spins up one Node.js instance per CPU core
- 4 core server = 4 PM2 instances running in parallel
- Load balancer distributes incoming requests across those 4 instances using round-robin
- If one instance crashes, PM2 restarts it automatically while others keep serving traffic
- This is horizontal scaling on a single server without needing Docker or Kubernetes

**Load balancer role:**
- Sits in front of all PM2 instances
- Distributes traffic so no single instance gets overwhelmed
- Handles failover — if one instance crashes, traffic goes to others automatically

**Current limitation — semi-automated pipeline:**
- Manual WinSCP file transfer step means a human is involved in deployment
- Fully automated would have Jenkins SSH directly into the server and transfer files
- This is technical debt worth acknowledging in interviews

---

## Things to Improve
- Learn how to make Jenkins SSH directly into server to remove manual WinSCP step
- Learn PM2 cluster mode commands: `pm2 start app.js -i max`
- Understand the difference between vertical scaling (bigger server) and horizontal scaling (more instances)
