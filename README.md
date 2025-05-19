# WordPress Deployment with Ansible

This Ansible project automates the deployment of WordPress with Nginx and MySQL on a target server. This README provides step-by-step instructions on how to use this project.

## Prerequisites

Before using this Ansible project, ensure you have:

1. **Ansible installed** on your local machine (version 2.9 or higher)
2. **SSH access** to your target server
3. A **Ubuntu/Debian-based** target server (Ubuntu 20.04 LTS or newer recommended)
4. **Python 3** installed on the target server

## Project Structure

```
wordpress-ansible/
├── inventory.ini           # Target server information
├── wordpress.yml           # Main playbook
├── group_vars/
│   └── all.yml             # Variables configuration
└── roles/
    ├── common/             # Common packages and configurations
    ├── mysql/              # MySQL database role
    ├── nginx/              # Nginx web server role
    ├── php/                # PHP and extensions role
    └── wordpress/          # WordPress installation and setup
```

## Configuration

### 1. Update Inventory

Edit the `inventory.ini` file to add your target server information:

```ini
[wordpress]
wordpress-server ansible_host=YOUR_SERVER_IP ansible_user=YOUR_SSH_USER ansible_ssh_private_key_file=PATH_TO_YOUR_SSH_KEY
```

Replace:
- `YOUR_SERVER_IP` with your server's IP address
- `YOUR_SSH_USER` with your SSH username (must have sudo privileges)
- `PATH_TO_YOUR_SSH_KEY` with the path to your SSH private key

### 2. Configure Variables

Edit the `group_vars/all.yml` file to customize your deployment:

```yaml
# MySQL Settings
mysql_root_password: "StrongRootPassword123!"  # Change this!
mysql_wordpress_db: "wordpress_db"
mysql_wordpress_user: "wordpress_user"
mysql_wordpress_password: "StrongWPPassword123!"  # Change this!

# WordPress Settings
wordpress_version: "6.4.2"  # Set the desired WordPress version
wordpress_site_title: "My WordPress Site"
wordpress_admin_user: "admin"
wordpress_admin_password: "StrongAdminPassword123!"  # Change this!
wordpress_admin_email: "admin@example.com"
wordpress_site_url: "example.com"  # Change to your domain

# Nginx Settings
nginx_server_name: "example.com www.example.com"  # Change to your domain
```

**Important**: Change all passwords and domain-specific settings to match your requirements.

## Deployment Steps

### 1. Verify Connectivity

Test SSH connectivity to your target server:

```bash
ansible -i inventory.ini wordpress -m ping
```

You should see a success message for your server.

### 2. Run the Playbook

Deploy WordPress, Nginx, and MySQL by running:

```bash
ansible-playbook -i inventory.ini wordpress.yml
```

This process will:
1. Install common packages and tools
2. Install and configure MySQL database
3. Install and configure Nginx web server
4. Install PHP and required extensions
5. Download, install, and configure WordPress

The deployment takes approximately 5-10 minutes depending on your server's performance.

### 3. Access Your WordPress Site

Once the playbook completes successfully, you can access your WordPress site at:

```
http://YOUR_SERVER_IP
```

Or if you've configured DNS for your domain:

```
http://your-domain.com
```

Login to the WordPress admin panel at:

```
http://YOUR_SERVER_IP/wp-admin
```

Use the admin credentials you configured in `group_vars/all.yml`.

## Advanced Configuration

### SSL/HTTPS Setup

To enable HTTPS with Let's Encrypt:

1. Ensure your domain is pointed to your server.
2. Install Certbot:

```bash
ansible -i inventory.ini wordpress -m apt -a "name=certbot python3-certbot-nginx state=present" -b
```

3. Obtain and configure SSL certificate:

```bash
ansible -i inventory.ini wordpress -m shell -a "certbot --nginx -d example.com -d www.example.com --non-interactive --agree-tos --email your-email@example.com" -b
```

Replace `example.com` and `your-email@example.com` with your values.

### Custom WordPress Plugins

To install additional plugins, add the following task to `roles/wordpress/tasks/main.yml`:

```yaml
- name: Install WordPress plugins
  shell: |
    cd /var/www/wordpress
    wp plugin install plugin-name --activate --allow-root
  become_user: www-data
```

Replace `plugin-name` with the desired plugin name or slug.

## Troubleshooting

### Common Issues

1. **SSH Connection Errors**: Verify your SSH key path and permissions.

2. **Permission Denied Errors**: Ensure your SSH user has sudo privileges.

3. **MySQL Connection Issues**: Check MySQL service status and credentials.

4. **Nginx Configuration Errors**: Check Nginx error logs:

```bash
ansible -i inventory.ini wordpress -m shell -a "cat /var/log/nginx/error.log" -b
```

5. **WordPress Installation Failure**: Check WordPress logs:

```bash
ansible -i inventory.ini wordpress -m shell -a "cat /var/www/wordpress/wp-content/debug.log" -b
```

## Maintenance

### Updating WordPress

To update WordPress to the latest version:

```bash
ansible -i inventory.ini wordpress -m shell -a "cd /var/www/wordpress && wp core update --allow-root" -b
```

### Backing Up the Site

To create a backup of your WordPress site:

```bash
ansible -i inventory.ini wordpress -m shell -a "cd /var/www && tar -czf wordpress-backup-$(date +%Y%m%d).tar.gz wordpress" -b
```

### Backing Up the Database

To backup the MySQL database:

```bash
ansible -i inventory.ini wordpress -m shell -a "mysqldump wordpress_db > /tmp/wordpress-db-backup-$(date +%Y%m%d).sql -u root -p{{ mysql_root_password }}" -b
```

## Security Considerations

This setup includes basic security configurations. For production environments, consider:

1. Enabling and configuring a firewall (UFW)
2. Implementing fail2ban for brute force protection
3. Regular security updates for all components
4. Using stronger passwords than the examples
5. Implementing WordPress security plugins

## License

This project is available under the MIT License.

## Support

For issues or questions, please file an issue in the project repository.
