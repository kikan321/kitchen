# 🚀 Event-Driven Pipeline Chaining with GitHub Actions: From CI to Automated Helm Registry

An enterprise-grade, event-driven DevOps ecosystem designed around a robust multi-tier pipeline architecture powered by **GitHub Actions**. 

The primary **GitHub Actions** workflow orchestrates the entire Continuous Integration (CI) lifecycle—governing source code validation, static syntax analysis, automated container compilation, and rigorous security vulnerability scanning. Upon a successful run, this initial stage dynamically broadcasts a completion event that triggers a secondary, chained **GitHub Actions** workflow. This independent pipeline programmatically builds, tags, packages, and indexes a localized Helm Chart, embedding the fresh, immutable Docker image artifact into Kubernetes manifests for cloud-native distribution.

---

## 🏗️ Architecture & Workflow Diagram

Every time code changes in this repository, a series of automated triggers orchestrate a multi-stage workflow across independent platforms:

```text
[ Developer Push ] ──> Triggers ──> ( MAIN BRANCH )
                                           │
 ┌─────────────────────────────────────────┘
 │
 ├──> 1. FASE DE CI (ci.yaml)
 │    ├── 🛠️ Static Code Linting (Hadolint Dockerfile audit)
 │    ├── 🔓 Registry Authentication (Secure Docker Hub handshake)
 │    ├── 📦 Container Compilation (Docker Build local to Runner)
 │    └── 🛡️ DevSecOps Compliance (Trivy Vulnerability Scan)
 │                                         │
 │   [ IF SUCCESSFUL: Push Image to Docker Hub with Git Commit SHA ]
 │                                         │
 └──> 2. FASE DE HELM DISTRIBUTION (helm-publish.yml) [Triggered via workflow_run]
      ├── 💾 Git Checkout & Identity Initialization
      ├── 🔧 Version Ingestion (Injects Git Commit SHA as Chart Version & Image Tag)
      ├── 🗜️ Chart Compilation (Helm Package creates .tgz binary)
      └── 🚀 Distribution & Overwrite (Chart Releaser splits outputs)
                                           │
          ┌────────────────────────────────┴────────────────────────────────┐
          ▼                                                                 ▼
   ( GITHUB RELEASES )                                             ( GH-PAGES BRANCH )
   Stores compiled binaries                                        Stores index.yaml catalog
   [ my-app-chart-1.0.0-sha.tgz ]                                           │
                                                                            ▼
                                                              [ GitHub Pages Infrastructure ]
                                                              Triggers native 'pages-build-deployment'
                                                                            │
                                                                            ▼
                                                                 LIVE HELM REPOSITORY URL:
                                                      https://github.io
```

---

## 🔍 Detailed Pipeline Breakdown

### 1. First Pipeline: The Complete CI Lifecycle (`ci.yml`)
* **Trigger:** Automated via a code `push` or manual `workflow_dispatch` to the `main` branch.
* **Execution Workflow:**
  1. **Checkout:** Fetches raw source assets from Git via `actions/checkout@v4`.
  2. **Linting:** Runs static code testing using **Hadolint** to audit Dockerfile architecture [🔒].
  3. **Authentication:** Handles secure cryptographic handshakes to authenticate against **Docker Hub** [🔒].
  4. **Compilation:** Compiles raw application code and static HTML into a local Docker layer container [🔒].
  5. **Security Scanning:** Runs deep security compliance and CVE vulnerability scanning via **Trivy (Aqua Security)** [🔒].
  6. **Registry Push:** Publishes the verified, immutable container to the registry using the unique **Git Commit SHA** as its tag [🔒].

### 2. Second Pipeline: Dynamic Helm Package & Publish (`helm-publish.yml`)
* **Trigger:** Automatically intercepting the completion signature of the first workflow via `on: workflow_run` [🔒].
* **Execution Workflow:**
  1. **Validation:** Verifies that the upstream CI lifecycle finished with a `success` conclusion status [🔒].
  2. **Environment Setup:** Provisions runtime infrastructure with the **Helm CLI** inside the runner virtual machine [🔒].
  3. **Dependency Injection:** Programmatically modifies `Chart.yaml` and `values.yaml` in memory, replacing generic placeholders with the current **Git Commit SHA** [🔒].
  4. **Packaging:** Compiles the targeted configuration charts into binary asset archives (`.tgz` packages) [🔒].
  5. **Distribution:** Provisions remote public access by deploying the compressed tarballs to **GitHub Releases** and committing the updated deployment ledger (`index.yaml`) directly to the serverless **`gh-pages`** hosting branch [🔒].

---

## 🌿 Branching Strategy & Component Roles

To maintain absolute stability and clean engineering habits, this repository uses isolated git branches and release layers with specific responsibilities:

### 1. `main` Branch (Source of Truth)
* **Purpose:** Houses the clean source code templates and continuous delivery pipelines.
* **Content:** Core logic (`index.html`, `Dockerfile`), infrastructure charts (`helm-charts/my-app-chart/`), and automation triggers (`.github/workflows/`).
* **State:** Immutable placeholders (e.g., `tag: ""` in `values.yaml`). It acts as a generic environment template. No runtime configurations are committed here automatically to prevent pipeline loops.

### 2. `gh-pages` Branch (Public Catalog Server)
* **Purpose:** Acts as a specialized server layer to host the Helm Chart repository database.
* **Content:** Contains only the auto-generated **`index.yaml`** file.
* **Behavior:** Every time a new version is validated, the `chart-releaser-action` automatically purges development scrap files from this branch and updates the index file. GitHub Pages automatically serves this branch to the internet.

### 3. GitHub Releases Layer (Binary Repository Storage)
* **Purpose:** Acts as an immutable storage factory for the distribution packages.
* **Content:** Hosts the final, compressed archive binaries (**`.tgz`** extensions).
* **Traceability:** Unlike the dynamic branches, if you download a `.tgz` file from this section, its internal `values.yaml` will explicitly have the unique **Git Commit SHA long hash** injected into the image tag field, providing 100% historical traceability.

---

## 🛡️ DevSecOps & Best Practices Implemented

* **Shift-Left Testing:** Docker syntax is audited instantly using **Hadolint** to catch misconfigurations before building infrastructure.
* **Vulnerability Scanning:** **Trivy (Aqua Security)** performs real-time CVE detection on compiled layers. If critical security threats are discovered, the deployment automatically self-halts.
* **Immutable Architecture:** Avoids mutable tags like `latest` or static values. Versions are tightly bound to the cryptographic **Git Commit SHA**, ensuring reproducible deployments.
* **Least Privilege Principle:** Utilizes specialized `GITHUB_TOKEN` credentials with scoped scope access (`contents: write`) to securely provision remote assets without exposing root account credentials.

---

## ⚙️ How to Consume this Helm Repository

Once the multi-tier automation finishes executing successfully, the application can be installed into any Kubernetes cluster worldwide using the following universal commands:

```bash
# 1. Add this custom, automated Helm repository to your cluster
helm repo add kitchen-repo https://kikan321.github.io/kitchen/

# 2. Update your local registry cache indexes
helm repo update

# 3. Pull the package from GitHub Releases and launch the container application
helm install my-web-service kitchen-repo/my-app-chart
```
