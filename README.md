# Automated Deployment to EC2 Using GitHub Actions

This repository uses GitHub Actions to automatically deploy the entire
project to an Amazon EC2 server whenever new changes are pushed to the
**main** branch.\
The workflow handles permissions, securely uploads files, and restarts
Nginx so updates go live instantly.

This README explains every section of the deployment workflow and how it
works behind the scenes.

------------------------------------------------------------------------

##  1. Workflow Name

**Section:**\
`name: Push-to-EC2`

This defines the workflow's title inside GitHub Actions and helps you
easily recognize this deployment pipeline.

------------------------------------------------------------------------

##  2. Trigger Condition

**Section:**

    on:
      push:
        branches:
          - main

This configuration tells GitHub Actions to:

**"Run this workflow automatically whenever new commits are pushed to
the `main` branch."**

Your production server stays up to date without manual deployment.

------------------------------------------------------------------------

##  3. Job Definition

**Section:**

    jobs:
      deploy:
        name: Deploy Entire Repo to EC2
        runs-on: ubuntu-latest

This defines a job named **Deploy Entire Repo to EC2** and instructs
GitHub Actions to run it in an Ubuntu environment.\
All subsequent deployment steps execute inside this runner.

------------------------------------------------------------------------

##  4. Step 1 --- Checkout Repository

**Section:**\
`uses: actions/checkout@v3`

This step downloads your repository into the GitHub Actions runner.\
It ensures that the workflow has access to the files that need to be
deployed.

------------------------------------------------------------------------

##  5. Step 2 --- Fix File Permissions on EC2

**Section:**\
Executed using `appleboy/ssh-action@master`

Commands performed:

-   `sudo chown -R $USER:$USER /var/www/html`\
    Makes your EC2 user the owner of the web directory.

-   `sudo chmod -R 755 /var/www/html`\
    Ensures correct read/write/execute permissions.

This step prevents permission errors and ensures the directory is ready
for deployment.

------------------------------------------------------------------------

##  6. Step 3 --- Upload the Entire Repository

**Section:**\
Uses `easingthemes/ssh-deploy@main`

This securely copies your entire project from GitHub to the EC2 server.

### Key configuration includes:

-   SSH private key (stored in GitHub Secrets)
-   EC2 public DNS or IP address
-   SSH username (e.g., ubuntu)
-   Source directory: `./`
-   Target directory: `/var/www/html`
-   rsync arguments: `-rlgoDzvc --delete`

### What the rsync flags do:

-   **r** --- recursive (includes all folders and files)\
-   **l** --- copies symlinks\
-   **g/o** --- preserves group and owner\
-   **D** --- copies device files\
-   **z** --- compresses files during transfer\
-   **v** --- verbose mode\
-   **c** --- checksum comparison\
-   **--delete** --- removes files from EC2 that no longer exist in the
    repo

This ensures the EC2 server always mirrors your exact GitHub repository.

------------------------------------------------------------------------

##  7. Step 4 --- Restart Nginx

**Section:**\
Runs using `appleboy/ssh-action@master` again

Command executed:

    sudo systemctl restart nginx

This ensures: - New application files are loaded - Cached assets are
refreshed - Your updated site is served instantly

------------------------------------------------------------------------

##  Required GitHub Secrets

Your workflow requires the following encrypted secrets:

  Secret            Purpose
  ----------------- ----------------------------------------
  **EC2_SSH_KEY**   SSH private key used to connect to EC2
  **HOST_DNS**      Public IP or DNS of the EC2 server
  **USERNAME**      SSH username (e.g., ubuntu)

Configure them under:\
**GitHub → Settings → Secrets → Actions**

------------------------------------------------------------------------

##  Summary of the Deployment Flow

1.  Developer pushes code to `main`\
2.  GitHub Actions checks out the repository\
3.  EC2 directory permissions are fixed\
4.  The entire repo is securely synchronized to EC2\
5.  Nginx restarts to serve the new version\
6.  Deployment completes automatically

This creates a smooth, automated CI/CD pipeline requiring zero manual
server updates.
