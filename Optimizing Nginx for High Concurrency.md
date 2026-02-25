
Most default Nginx configurations fail under heavy load because the `worker_connections` limit is too low for modern traffic spikes. Here is how I tuned it for the SecOps CTF.

## 1. The Bottleneck

During our initial load testing, the server began dropping packets at 500 concurrent users. A quick check of the error logs revealed the problem:

`[alert] 768#0: *1024 worker_connections are not enough`

Nginx handles connections using worker processes. The formula for max clients is:
`max_clients = worker_processes * worker_connections`

## 2. The Solution

We need to edit the main configuration file to increase these limits and adjust the file descriptor limits at the OS level.

### Step A: OS Level Tuning
First, increase the maximum number of open files allowed by the system:

```bash
ulimit -n 65535

```

### Step B: Nginx Configuration

Open `/etc/nginx/nginx.conf` and apply the following parameters:

```nginx
events {
    worker_connections 4096;
    multi_accept on;
    use epoll; # Optimized for Linux
}

```

## 3. Results

After reloading the daemon (`sudo systemctl reload nginx`), our infrastructure successfully handled **3,000+ simulated concurrent connections** with zero dropped packets and a latency variance of < 12ms.

*Note: Always remember to update your Docker `ulimits` if deploying this inside a Swarm container.*



### Why this is the ultimate workflow:
1.  **Zero Build Config:** You just hit "Submit new issue" on GitHub.
2.  **Native Image Hosting:** If you want to add a screenshot or architecture diagram, you just drag and drop it into the GitHub text box. GitHub hosts the image on their global CDN for free and inserts the Markdown link automatically. Your frontend doesn't get bloated with heavy `.png` files.
3.  **Instant Edits:** Spot a typo? Edit the issue on your phone, and your website updates.

If you create an issue with this exact format, your new `src/pages/wiki/[slug].astro` will parse it perfectly into your technical aesthetic.

Are we ready to wrap up the architecture with the **404 Terminal Crash Page**?

