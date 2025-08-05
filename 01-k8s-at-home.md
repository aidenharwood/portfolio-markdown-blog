---
title: Running a Website on Kubernetes at Home
date: 08-05-2025
slug: running-website-kubernetes-home
excerpt: How I began hosting a modern web stack on Kubernetes using an old laptop, with CI/CD and GitOps workflows.
---

## Running This Website on Kubernetes: A DIY Cloud Journey

Setting up this website has been a rewarding hands-on project, blending modern cloud-native practices with a bit of home-lab ingenuity. Here’s how I’ve architected the stack to run on [Kubernetes](https://kubernetes.io/), hosted on an old laptop.

### The Hardware

Instead of relying on a commercial cloud provider, I repurposed an old laptop as my Kubernetes node by installing [k0s](https://k0sproject.io/). This approach keeps costs low and gives me full control over the environment, making it perfect for experimentation and learning.

### The Application Stack

- **Frontend:** The website is built with [Vue](https://vuejs.org/) and [Vite](https://vitejs.dev/), providing a fast, modern user experience.
- **Backend API:** The API runs alongside the frontend, enabling seamless communication.
- **Nginx Proxy:** Both the frontend and API are served under the same domain, with [Nginx](https://nginx.org/) acting as a reverse proxy. This setup simplifies CORS management and provides a unified entry point for all traffic.

### Blog System

This blog system is designed for flexibility and ease of content management:

- The **frontend** calls a backend API to fetch blog posts.
- The **API** retrieves markdown files from a mounted volume. This volume is kept in sync with a Git repository using [git-sync](https://github.com/kubernetes/git-sync), ensuring the latest content is always available.
- When a blog post is requested, the API reads the corresponding markdown file and returns its contents to the frontend, which then renders it directly in the browser.

This approach allows for seamless updates to blog content via simple git commits, with no need to rebuild or redeploy the application.

### CI/CD with GitHub Actions and ArgoCD

To streamline deployments, I’ve set up a CI/CD pipeline using [GitHub Actions](https://github.com/features/actions):

1. **Build & Push:** On every commit, Actions build Docker images for the frontend and API, then push them to a container registry.
2. **Helm Chart Updates:** The pipeline automatically updates [Helm](https://helm.sh/) chart values with the new image tags.
3. **ArgoCD Sync:** [ArgoCD](https://argo-cd.readthedocs.io/) monitors the Helm charts and triggers rolling updates in the Kubernetes cluster whenever changes are detected.

This workflow ensures that code changes are quickly and reliably deployed, with minimal manual intervention.

### Why This Setup?

- **Learning:** Running Kubernetes at home is a great way to deepen my understanding of container orchestration, networking, and GitOps workflows.
- **Flexibility:** Hosting on my own hardware means I can experiment freely without worrying about cloud costs.
- **Modern Practices:** Using tools like Helm, ArgoCD, and GitHub Actions brings real-world DevOps practices into my personal projects.

---

If you’re interested in setting up something similar, I recommend starting small - Kubernetes can run on surprisingly modest hardware, and the open-source ecosystem makes it easy to automate and manage deployments. Happy tinkering!
