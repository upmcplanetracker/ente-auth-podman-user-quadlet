Ente Auth: User-Mode Podman Quadlets
====================================

This guide explains how to deploy **Ente Auth** as a set of rootless Podman Quadlets. This setup allows the containers to be managed by systemd under a specific user account without requiring root privileges.

1\. Directory Structure & File Locations
----------------------------------------

For **User Quadlets**, the configuration files must be placed in the following directory. If it doesn't exist, create it:

    mkdir -p ~/.config/containers/systemd/

You will need to place the following files in that directory:

*   `ente.network` (Internal container network)
*   `ente-database.container` (Postgres 18)
*   `ente-museum.container` (Ente Backend)
*   `ente-web.container` (Web Frontend)

2\. Variables to Customize
--------------------------

Before deploying, ensure you update the following variables in your `.container` files:

*   `${USER_HOME}`: Usually `/home/yourusername`.
*   `${DB_PASSWORD}`: A strong, unique password for the database.
*   `${FQDN}`: Your local or public domain (e.g., `ente.example.com`).
*   `${TIMEZONE}`: Your local timezone (e.g., `America/New_York`).

3\. Permissions & Ownership
---------------------------

Rootless containers map the internal container users to a range of sub-UIDs on the host. For standard database and file storage, follow these steps:

1

**Create Data Directories:**

    mkdir -p ~/my-ente/db-data ~/my-ente/data ~/my-ente/ente_logs

2

**Correct Ownership:**

Because Podman uses user namespaces, simply using `chown` to your user isn't always enough for the internal Postgres user (usually UID 999). Use `podman unshare`:

    podman unshare chown -R 999:999 ~/my-ente/db-data

3

**File Permissions:**

    chmod 700 ~/my-ente/db-data
    chmod 755 ~/my-ente/data ~/my-ente/ente_logs

4\. Installation & Activation
-----------------------------

Once your files are in `~/.config/containers/systemd/`, run the following commands to generate the services:

    # Reload the user-level systemd daemon
    systemctl --user daemon-reload
    
    # Start the web service (this will trigger dependencies)
    systemctl --user start ente-web.service
    
    # Check the status
    systemctl --user status ente-web.service

**Critical Step: Linger**  
To ensure your containers start automatically when the computer boots (and keep running when you log out), you **must** enable lingering for your user:

    loginctl enable-linger $USER

5\. Troubleshooting
-------------------

If things aren't starting as expected, check the logs via journalctl:

    # View logs for the Museum backend
    journalctl --user -u ente-museum.service -f
    
    # Check for Quadlet generation errors
    /usr/lib/systemd/system-generators/podman-system-generator --user --dryrun
