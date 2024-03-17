---
layout: default
title: Installing eScriptorium on a Cloud Server
parent: Software Documentation
nav_order: 1
---

# Installing eScriptorium on a Cloud Server (Debian) with a Fully Qualified Domain Name

eScriptorium is an open source application for handwritten text recognition (HTR). Since its release in 2018, this application has become popular with library professionals and researchers who work with medieval manuscripts. To learn more about the project, see the official website.

> In February 2024, I attempted to install a eScriptorium site on a cloud server for my wife, who is a librarian. I found a few sets of installation instructions, but none of them were aimed at the goal I had in mind: setting up a publicly accessible eScriptorium site, with a Fully Qualified Domain Name, running on a small, cheap cloud server.
> 
> I made a start with the UB Mannheim installation instructions. Then, I had an email exchange with Stefan Weil, who is a member of the eScriptorium project, to figure out how to add the missing pieces to reach my goal. Thanks to his very helpful guidance, I managed to get an eScriptorium site online. 
> 
> Here, I present instructions for installing eScriptorium on a small Debian-based cloud server with a Fully Qualified Domain Name (FQDN). &mdash;*Ravi Murugesan (RM)*

## System Requirements

To follow this guide, you should first create a cloud server with...

- At least 2 CPU cores (a shared CPU plan should be fine to get started)
- At least 4 GB of RAM
- At least 20 GB of disk space
- An IPv4 address
- A fresh installation of Debian (Debian 12 as of March 2024)

> [!NOTE]
> The below instructions have been tested on Hetzner's Shared vCPU server with an x86 processor with 2 vCPUs, 4 GB RAM, and 40 GB SSD. However, a shared CPU server may not be ideal for high or sustained workloads. (The author of this article is not affiliated with Hetzner in any way.)

## Domain Requirements

In this guide, `escriptorium.example.com` is used as the FQDN for the eScriptorium site. As you go through this guide, replace this domain name with your own domain name.

1. Access the DNS records of your domain name.
2. Add an A record for `escriptorium` that points to the IPv4 address of the server where you plan to install eScriptorium.

> [!TIP]
> If you don't have access to the DNS records of your domain name, ask your domain administrator to create a subdomain record to point to the IPv4 address of your server.

## Before You Begin

You should

- have access to a terminal application that supports SSH connections
- know how to use SSH to sign in to a remote server
- have basic Debian command line skills
- know how to use a text editor in Debian (such as vi or nano)
- have root access to the Debian server where you want to install eScriptorium (or login credentials for a non-root user with sudo permissions)
- set aside at least 1 to 2 hours for this procedure

## Step 1: Secure the Server

1. Open the terminal application of your choice on your computer.

1. Locate the access credentials of the Debian server you have created (see System Requirements above).

1. From your terminal application, sign into the server as the root user, using the password or SSH key generated at the time of creating the server.

    > If you can't sign in to the server using SSH, it could mean that SSH connections are blocked. In that case, sign into your account with your hosting provider and access the server using the hosting provider's browser-based terminal. You can use your terminal application later on after enabling SSH.

1. Update the server.

    ```
    apt update
    apt upgrade
    ```

1. Install the firewall application (ufw).

    ```
    apt install ufw
    ```

1. Ensure SSH is allowed.

    ```
    ufw allow ssh
    ```

1. Enable the firewall after allowing SSH.

    ```
    ufw enable
    ```

1. Create a non-root user. Replace `htrfan` with the username of your choice. And in the steps that follow, replace `htrfan` with the username you have created.

    ```
    adduser htrfan
    ```

    > Generate a strong password when prompted by the `adduser` command, and save this password.

1. Give `sudo` permissions to the newly created non-root user.

    ```
    usermod -aG sudo htrfan
    ```

1. From a new window or tab of your terminal application, access the server with an SSH connection using the sudo user account. Replace `htrfan` with your the username you have created, and `77.77.77.77` with the IP address of your server.

    For password authentication:

    ```
    ssh htrfan@77.77.77.77
    ```

    For key-based authentication (replace `./private-key` with the location and filename of your private key):

    ```
    ssh -i ./private-key htrfan@77.77.77.77
    ```

> [!TIP]
> For more rigorous hardening and securing, such as setting up key-based authentication for the non-root user and blocking malicious login attempts, see this guide on [the Linux Handbook website](https://linuxhandbook.com/things-to-do-after-installing-linux-server/). 

## Step 2: Install Apache and Certbot



## Step 3: Install the Programs Required for eScriptorium

Note: These instructions are an edited version of the instructions from the UB Mannheim site.

1. First, install all the programs you need to use or install eScriptorium. These include Git, NPM, Postgresql installations, the Redis server, third-party tools, and Python and the virtual environment:

    ```
    sudo apt install git postgresql postgresql-contrib libpq-dev redis-server gettext netcat-traditional jpegoptim pngcrush libvips build-essential python3 python3-dev python3-venv npm
    ```

    > [!IMPORTANT]
    > If error messages appear during the Postgresql installation, enter the following commands.
        
        sudo apt update
        sudo apt update --fix-missing
        sudo apt install postgresql

1. Start the servers for Postgresql and Redis

    ```
    sudo service postgresql start
    sudo service redis-server start
    ```

    Note
    The current status of Postgresql can be queried at any time with the following command:
    
    ```
    sudo service postgresql status
    ```

1. Create username for eScriptorium

    Decide if you want to use your Linux username as the eScriptorium username.

    If yes, enter this command:
    
    ```    
    (cd /tmp && sudo -u postgres createuser --superuser $(whoami))
    ```

    If no, enter this command, replacing 'yourusername' with the name you choose.

    ```
    sudo -u postgres createuser --superuser yourusername
    ```

1. Create a database for eScriptorium

    ```
    createdb escriptorium
    ```

1. Clone the eScriptorium repository from GitLab

    ```
    git clone https://gitlab.com/scripta/escriptorium.git ~/escriptorium
    ```

1. Create and activate a virtual environment

    Go to the newly created escriptorium directory.

    cd ~/escriptorium

    Enter the following commands.

    ```
    python3 -m venv env
    ```

    ```
    source env/bin/activate
    ```
    
    ```
    pip install -U pip setuptools wheel
    ```

    Install all necessary Python packages:

    ```
    pip install -r app/requirements.txt -r app/requirements-dev.txt
    ```

1.  Create local settings

    There is a sample local settings file that is simply copied here.

cp app/escriptorium/local_settings.py.example app/escriptorium/local_settings.py
OPTIONAL: Use the following command to access the settings. Changes can be made here. However, you don't have to change anything to get started, so you can skip this step.

vi app/escriptorium/local_settings.py
9. Install NPM
Then change to the directory frontand install with NPM:

cd front
--Aktualisiere npm (vorinstallierte Version ist meist veraltet).
sudo npm install -g n
sudo n latest
--Vergiss das alte npm, damit das neue npm verwendet wird.
hash -r
--Installiere die Pakete für eScriptorium.
npm install
OPTIONAL: If error messages appear that there are vulnerabilities (especially in the red area), then you should run the following command:

npm audit fix
Often not all weak points can be corrected. This would be a security risk for installation on a server, but in a local installation without external access it is usually not critical.

10. Productive Use
To use eScriptorium productively (developers are better off choosing other options), run the following commands:

npm run production
11. Check installation
To do this, run the following command:

cd ../app
python manage.py check --settings escriptorium.local_settings
12. SQL tables
Now create the required SQL tables in the database:

python manage.py migrate --settings escriptorium.local_settings
13. Update translations
The translations of the user interface may still need to be updated (in addition to the English, there are largely translated German texts):

python manage.py makemessages --all --settings escriptorium.local_settings
python manage.py compilemessages --settings escriptorium.local_settings
14. OPTIONAL
Now you can create a superuser

python manage.py createsuperuser --settings escriptorium.local_settings
15. Celery Worker
Open a new terminal and run a simple Celery Worker:

DJANGO_SETTINGS_MODULE=escriptorium.local_settings celery -A escriptorium worker -l INFO &
Confirm with Enter. Using this command, Celery will run directly in the background and you can continue.

16. Starting the server
python manage.py runserver --settings escriptorium.local_settings
17. Use of eScriptorium
Now you can use eScriptorium, simply enter the following link into your browser: http://localhost:8000/ (Note: If you have installed eScriptorium in a virtual machine, use the browser in it.)

Attention: eScriptorium automatically creates a user with the name “admin” and the password “admin”. You should know this because this is how unauthorized people can break into your system, especially if you later make your eScriptorium installation available as a web service. In this case, change the password or remove this user.

18. Optional settings
~/escriptorium/app/local_settings.pyLocal settings can optionally be made in the file .

With DEBUG = Falseyou switch off the display of the debug tools (top right in the web interface).

The export formats OpenITI Markdown and TEI XML can be activated with EXPORT_OPENITI_MARKDOWN_ENABLED = Trueor EXPORT_TEI_XML_ENABLED = True.

With you TEXT_ALIGNMENT_ENABLED = Truecan activate text alignment as an additional functionality.

DISABLE_ELASTICSEARCH = Falseswitches on the full-text search. This requires additional settings and the installation of Elasticsearch.

19. Reuse
Once you have restarted your PC or virtual machine, you must activate eScriptorium again via the terminal.

cd ~/escriptorium
source env/bin/activate
cd app
sudo service postgresql start
sudo service redis-server start
DJANGO_SETTINGS_MODULE=escriptorium.local_settings celery -A escriptorium worker -l INFO &
python manage.py runserver --settings escriptorium.local_settings
Browser: http://localhost:8000/

## Step 4: Enable Reverse Proxy



## Step 5: Make eScriptorium Run as a Service



## Verify Installation


## Post Installation


## Troubleshooting


## Next Steps

