# --- Step 1: Install Homebrew (if you haven't already) ---
# Homebrew is a package manager for macOS.
# If you get an error that Homebrew is already installed, you can skip this.
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# --- Step 2: Install Docker Desktop, Git, and AdoptOpenJDK (for SQL Developer) ---
# Docker Desktop is essential for running Docker containers on macOS.
# Git is needed to clone the Oracle Docker images repository.
# AdoptOpenJDK is a Java Development Kit required for SQL Developer.
# If you encounter "Error: It seems there is already a Bash Completion" or similar for docker-compose,
# it means a previous attempt left some files. You can try:
# rm /opt/homebrew/etc/bash_completion.d/docker
# Then re-run the brew install command.
brew install --cask docker
brew install git adoptopenjdk/openjdk/adoptopenjdk8

# After installing Docker Desktop, make sure to open it at least once
# from your Applications folder and complete its initial setup.
# You should see the Docker icon in your menu bar indicating it's running.

# --- Step 3: Download and set up Oracle Instant Client and SQL*Plus ---
# Oracle Instant Client allows you to connect to the Oracle database from your macOS.
# SQL*Plus is a command-line tool for interacting with Oracle databases.

# Create a directory for the instant client
mkdir -p ~/oracle/instantclient

# IMPORTANT: You need to manually download these files from Oracle's website.
# Go to: https://www.oracle.com/database/technologies/instant-client/downloads.html
# Choose the "Instant Client for macOS (Intel x86)" or "macOS (ARM64)" depending on your Mac's chip.
# Download the "Basic" package and the "SQL*Plus" package.
# For example, for version 19.3.0.0.0dbru (adjust version numbers if newer):
# instantclient-basic-macos.x64-19.3.0.0.0dbru.zip
# instantclient-sqlplus-macos.x64-19.3.0.0.0dbru.zip

# Place the downloaded zip files into the `~/oracle/instantclient` directory you just created.
# Then, unzip them:
# (Replace the filenames with the actual names of the files you downloaded)
unzip ~/oracle/instantclient/instantclient-basic-macos.x64-*.zip -d ~/oracle/instantclient/
unzip ~/oracle/instantclient/instantclient-sqlplus-macos.x64-*.zip -d ~/oracle/instantclient/

# Find the exact directory name created by unzipping (e.g., instantclient_19_3)
# You can use `ls ~/oracle/instantclient/` to see it.
# Let's assume the unzipped directory is named `instantclient_19_3` for the next steps.
# You might need to adjust `instantclient_19_3` to the actual folder name.

# Add Instant Client to your shell's PATH.
# This makes the `sqlplus` command available directly in your terminal.
# If you are using Zsh (default for modern macOS), edit ~/.zshrc:
echo 'export ORACLE_HOME=~/oracle/instantclient/instantclient_19_3' >> ~/.zshrc
echo 'export PATH=$ORACLE_HOME:$PATH' >> ~/.zshrc
echo 'export LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH' >> ~/.zshrc

# If you are using Bash, edit ~/.bash_profile or ~/.bashrc:
# echo 'export ORACLE_HOME=~/oracle/instantclient/instantclient_19_3' >> ~/.bash_profile
# echo 'export PATH=$ORACLE_HOME:$PATH' >> ~/.bash_profile
# echo 'export LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH' >> ~/.bash_profile

# Apply the changes to your current terminal session:
source ~/.zshrc
# or `source ~/.bash_profile` if you are using Bash

# Verify SQL*Plus installation (should show version number)
sqlplus -V

# --- Step 4: Clone Oracle Docker Images and Build the 18c XE Image ---
# This repository contains scripts to build official Oracle database Docker images.
mkdir -p ~/oracle/docker-images
git clone https://github.com/oracle/docker-images.git ~/oracle/docker-images

# Navigate to the 18c XE Dockerfiles directory
cd ~/oracle/docker-images/OracleDatabase/SingleInstance/dockerfiles/

# Build the Docker image for Oracle 18c XE. This will take some time.
# The `-v 18.4.0` specifies the version, and `-x` accepts the license.
# Ensure you are in the `dockerfiles` directory when running this command.
./buildContainerImage.sh -v 18.4.0 -x

# --- Step 5: Run the Oracle 18c XE Docker Container ---
# This command starts your Oracle database instance within Docker.
# Replace `<your_container_name>` with a name you prefer (e.g., `oracle-xe-18c`).
# Replace `<your_password>` with a strong password for the SYS, SYSTEM, and PDBADMIN users.
# The password must meet Oracle's complexity requirements (e.g., at least 8 characters,
# including uppercase, lowercase, numbers, and special characters).
# Example: `MySecureP@ssw0rd`
docker run --name <your_container_name> \
  -p 1521:1521 \
  -e ORACLE_PWD=<your_password> \
  -e ORACLE_EDITION=XE \
  -d oracle/database:18.4.0-xe

# It might take a few minutes for the database to start up completely inside the container.
# You can check the logs to see the status:
docker logs <your_container_name> --follow

# Look for messages like "DATABASE IS READY TO USE!" in the logs.

# --- Step 6: Connect to the database using SQL*Plus ---
# Once the Docker container is running and the database is ready.
# Use the password you set in the `docker run` command.
sqlplus system/<your_password>@//localhost:1521/XEPDB1

# You should now be connected to your Oracle 18c XE database.
# To exit SQL*Plus, type `exit` and press Enter.

# --- Optional: Stop and Remove the Docker Container ---
# To stop the running database container:
# docker stop <your_container_name>

# To remove the container (this will delete the database data unless you've used volumes):
# docker rm <your_container_name>
