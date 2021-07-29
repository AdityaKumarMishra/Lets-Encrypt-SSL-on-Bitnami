Generate And Install A Let's Encrypt SSL Certificate For A Bitnami Application

### Introduction

[Let’s Encrypt](https://letsencrypt.org/) is a free Certificate Authority (CA) that issues SSL certificates. You can use these SSL certificates to secure traffic to and from your Bitnami application host.

This guide walks you through the process of generating a Let’s Encrypt SSL certificate for your domain and installing and configuring it to work with your Bitnami application stack.

IMPORTANT: The steps described in this guide are applicable to all Bitnami applications, with the following exceptions:

·     Bitnami GitLab: Use the [Bitnami GitLab guide](https://docs.bitnami.com/general/apps/gitlab/administration/generate-configure-certificate-letsencrypt/) instead

·     Bitnami Mattermost: Use the [Bitnami Mattermost guide](https://docs.bitnami.com/general/apps/mattermost/administration/generate-configure-certificate-letsencrypt/) instead

 

### Assumptions And Prerequisites

This guide assumes that:

·     You have deployed a Bitnami application and the application is available at a public IP address so that the Let’s Encrypt process can verify your domain.

·     You have the necessary credentials to log in to the Bitnami application instance.

·     You own one or more domain names.

·     You have configured the domain name’s DNS record to point to the public IP address of your Bitnami application instance.

### Use The Bitnami HTTPS Configuration Tool

IMPORTANT: The Bitnami HTTPS Configuration Tool does not support IPv6 addresses, load balancers/CDNs or NGINX web servers yet. If you use IPv6 addresses, please disable them before proceeding. If you use NGINX or a load balancer or CDN, please refer to the [alternative approach](https://docs.bitnami.com/general/how-to/generate-install-lets-encrypt-ssl/#alternative-approach) section.

The Bitnami HTTPS Configuration Tool is a command line tool for configuring mainly HTTPS certificates on Bitnami stacks, but also common features such as automatic renewals, redirections (e.g. HTTP to HTTPS), etc. This tool is located in the installation directory of the stack at */opt/bitnami*.

NOTE: Before using the Bitnami HTTPS configuration tool, ensure that your domain’s DNS configuration correctly reflects the host’s IP address and that you are not using IPv6 addresses. You can update your domain’s DNS configuration through your DNS provider.

To launch the Bitnami HTTPS Configuration Tool, execute the following command and follow the prompts:

sudo /opt/bitnami/bncert-tool

Refer to our [guide](https://docs.bitnami.com/general/how-to/understand-bncert/) for more information on this, or if you can’t find the tool in your Bitnami stack.

NOTE: The Bitnami HTTPS Configuration Tool will automatically create a cron job to renew your certificate(s). By default, if the *bitnami* user account exists on the system, the cron jobs will be added under that user account. To view and modify the cron job, use the command *sudo crontab -u bitnami -l*.

If you prefer to manually generate and install Let’s Encrypt certificates, follow this [alternative approach](https://docs.bitnami.com/general/how-to/generate-install-lets-encrypt-ssl/#alternative-approach).

#### Troubleshooting

In case the certificate generation process fails and/or you wish to reset the certificates for any reson, follow the steps below:

·     Remove the cron jobs in the *root* and *bitnami* user’s cron table. Run the following commands and remove any lines/commands related to certificate renewal:

```
·         sudo crontab -e
·         sudo crontab -e -u bitnami
```

·     Modify the Web server configuration file to use the original *server.crt* and *server.key* certificates (these are not renamed or moved by the Bitnami HTTPS Configuration Tool). Alternatively, restore the original Web server configuration file, which is backed up by the tool as *bitnami.conf.back.DATE* in the same directory.

·     Restart all Bitnami services:

```
·         sudo /opt/bitnami/ctlscript.sh start
```

IMPORTANT: Users will see SSL certificate warnings when accessing the website while the dummy certificates are in use. These warnings will disappear after valid SSL certificates are installed for the website.

### Alternative Approach

NOTE: We are in the process of modifying the file structure and configuration for many Bitnami stacks. On account of these changes, the file paths stated in this guide may change depending on whether your Bitnami stack uses native Linux system packages (Approach A), or if it is a self-contained installation (Approach B). To identify your Bitnami installation type and what approach to follow, run the command below:

```
test ! -f "/opt/bitnami/common/bin/openssl" && echo "Approach A: Using system packages." || echo "Approach B: Self-contained installation."
```

The output of the command indicates which approach (A or B) is used by the installation, and will allow you to identify the paths, configuration and commands to use in this guide. [Refer to the FAQ for more information on these changes](https://docs.bitnami.com/general/faq/get-started/understand-upcoming-changes/).

If your Bitnami image does not include the auto-configuration script or the */opt/bitnami/letsencrypt/* directory, you can manually install the Lego client and generate and install the Let’s Encrypt certificates. Follow the steps below.

#### Step 1: Install The Lego Client

The [Lego](https://github.com/xenolf/lego) client simplifies the process of Let’s Encrypt certificate generation. To use it, follow these steps:

·     Log in to the server console as the *bitnami* user.

·     Run the following commands to install the Lego client. Note that you will need to replace the X.Y.Z placeholder with the actual version number of the downloaded archive:

```
·         cd /tmp
·         curl -Ls https://api.github.com/repos/xenolf/lego/releases/latest | grep browser_download_url | grep linux_amd64 | cut -d '"' -f 4 | wget -i -
·         tar xf lego_vX.Y.Z_linux_amd64.tar.gz
·         sudo mkdir -p /opt/bitnami/letsencrypt
·         sudo mv lego /opt/bitnami/letsencrypt/lego
```

These steps will download, extract and copy the Lego client to a directory in your path.

#### Step 2: Generate A Let’s Encrypt Certificate For Your Domain

NOTE: Before proceeding with this step, ensure that your domain name points to the public IP address of the Bitnami application host. If the Bitnami application host is behind a load balancer or CDN, the commands below require additional parameters, which can be provided by [the Bitnami support team](https://community.bitnami.com/) on request.

The next step is to generate a Let’s Encrypt certificate for your domain.

·     Turn off all Bitnami services:

```
·         sudo /opt/bitnami/ctlscript.sh stop
```

·     Request a new certificate for your domain as below, both with and without the *www* prefix.

IMPORTANT: Replace the DOMAIN placeholder with your actual domain name, and the EMAIL-ADDRESS placeholder with your email address.

```
sudo /opt/bitnami/letsencrypt/lego --tls --email="EMAIL-ADDRESS" --domains="DOMAIN" --domains="www.DOMAIN" --path="/opt/bitnami/letsencrypt" run
```

NOTE: You can use more than one domain (for example, *DOMAIN* and *www.DOMAIN*) by specifying the *--domains* option as many times as the number of domains you want to specify. When supplying multiple domains, Lego creates a SAN (Subject Alternate Names) certificate which results in only one certificate valid for all domains you entered. The first domain in your list will be added as the “CommonName” of the certificate and the rest, will be added as “DNSNames” to the SAN extension within the certificate.

·     Agree to the terms of service.

A set of certificates will now be generated in the */opt/bitnami/letsencrypt/certificates* directory. This set includes the server certificate file *DOMAIN.crt* and the server certificate key file *DOMAIN.key*.

IMPORTANT: For security reasons, never post or disclose your server’s SSL private key file in a public forum.

An output message will provide some information, including the expiry date of the certificate. Note this expiry date carefully as you will need to renew your certificate before that date in order for it to remain valid.

An example certificate is shown below:

[![Let’s Encrypt CA certificate](E:\AA\clip_image002.jpg)](https://docs.bitnami.com/images/img/how_to_guides/generate-install-lets-encrypt-ssl/lets-encrypt-1.png)

NOTE: The steps described above will generate certificates for one or more explicitly-named domains. To generate a certificate for a wildcard domain, you will need to use DNS-01 validation when running the *lego* tool, as explained in the [official Let’s Encrypt documentation](https://letsencrypt.org/docs/challenge-types/).

#### Step 3: Configure The Web Server To Use The Let’s Encrypt Certificate

Next, tell the Web server about the new certificate, as follows:

·     Link the new SSL certificate and certificate key file to the correct locations, depending on which Web server you’re using. Update the file permissions to make them readable by the root user only.

IMPORTANT: Remember to replace the DOMAIN placeholder with your actual domain name.

For Apache under Approach A (Bitnami installations using system packages):

```
sudo mv /opt/bitnami/apache2/conf/bitnami/certs/server.crt /opt/bitnami/apache2/conf/bitnami/certs/server.crt.old
sudo mv /opt/bitnami/apache2/conf/bitnami/certs/server.key /opt/bitnami/apache2/conf/bitnami/certs/server.key.old
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.key /opt/bitnami/apache2/conf/bitnami/certs/server.key
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.crt /opt/bitnami/apache2/conf/bitnami/certs/server.crt
sudo chown root:root /opt/bitnami/apache2/conf/bitnami/certs/server*
sudo chmod 600 /opt/bitnami/apache2/conf/bitnami/certs/server*
```

For Apache under Approach B (Self-contained Bitnami installations):

```
sudo mv /opt/bitnami/apache2/conf/server.crt /opt/bitnami/apache2/conf/server.crt.old
sudo mv /opt/bitnami/apache2/conf/server.key /opt/bitnami/apache2/conf/server.key.old
sudo mv /opt/bitnami/apache2/conf/server.csr /opt/bitnami/apache2/conf/server.csr.old
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.key /opt/bitnami/apache2/conf/server.key
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.crt /opt/bitnami/apache2/conf/server.crt
sudo chown root:root /opt/bitnami/apache2/conf/server*
sudo chmod 600 /opt/bitnami/apache2/conf/server*
```

For NGINX under Approach A (Bitnami installations using system packages):

```
sudo mv /opt/bitnami/nginx/conf/bitnami/certs/server.crt /opt/bitnami/nginx/conf/bitnami/certs/server.crt.old
sudo mv /opt/bitnami/nginx/conf/bitnami/certs/server.key /opt/bitnami/nginx/conf/bitnami/certs/server.key.old
sudo mv /opt/bitnami/nginx/conf/bitnami/certs/server.csr /opt/bitnami/nginx/conf/bitnami/certs/server.csr.old
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.key /opt/bitnami/nginx/conf/bitnami/certs/server.key
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.crt /opt/bitnami/nginx/conf/bitnami/certs/server.crt
sudo chown root:root /opt/bitnami/nginx/conf/bitnami/certs/server*
sudo chmod 600 /opt/bitnami/nginx/conf/bitnami/certs/server*
```

For NGINX under Approach B (Self-contained Bitnami installations):

```
sudo mv /opt/bitnami/nginx/conf/server.crt /opt/bitnami/nginx/conf/server.crt.old
sudo mv /opt/bitnami/nginx/conf/server.key /opt/bitnami/nginx/conf/server.key.old
sudo mv /opt/bitnami/nginx/conf/server.csr /opt/bitnami/nginx/conf/server.csr.old
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.key /opt/bitnami/nginx/conf/server.key
sudo ln -sf /opt/bitnami/letsencrypt/certificates/DOMAIN.crt /opt/bitnami/nginx/conf/server.crt
sudo chown root:root /opt/bitnami/nginx/conf/server*
sudo chmod 600 /opt/bitnami/nginx/conf/server*
```

TIP: To find out if your Bitnami stack uses Apache or NGINX, check the output of the command *sudo /opt/bitnami/ctlscript.sh status*.

·     Restart all Bitnami services:

```
·         sudo /opt/bitnami/ctlscript.sh start
```

To add one or more domains to an existing certificate, simply repeat Steps 2 and 3 again, ensuring the same order of domain names is maintained in the *lego* command and adding the new domain name(s) to the end with additional *–domains* arguments.

#### Step 4: Test The Configuration

After reconfirming that your domain name points to the public IP address of the Bitnami application instance, you can test it by browsing to *https://DOMAIN* (replace the DOMAIN placeholder with the correct domain name).

This should display the secure welcome page of the Bitnami application. Clicking the padlock icon in the browser address bar should display the details of the domain and SSL certificate.

[![Let’s Encrypt certificate in action](E:\AA\clip_image004.jpg)](https://docs.bitnami.com/images/img/how_to_guides/generate-install-lets-encrypt-ssl/lets-encrypt-2.png)

#### Step 5: Renew The Let’s Encrypt Certificate

Let’s Encrypt certificates are only valid for 90 days. To renew the certificate before it expires, run the following commands from the server console as the *bitnami* user. Remember to replace the DOMAIN placeholder with your actual domain name, and the EMAIL-ADDRESS placeholder with your email address.

```
sudo /opt/bitnami/ctlscript.sh stop
sudo /opt/bitnami/letsencrypt/lego --tls --email="EMAIL-ADDRESS" --domains="DOMAIN" --path="/opt/bitnami/letsencrypt" renew --days 90
sudo /opt/bitnami/ctlscript.sh start
```

To automatically renew your certificates before they expire, write a script to perform the above tasks and schedule a *cron* job to run the script periodically. To do this:

·     Create a script at */opt/bitnami/letsencrypt/scripts/renew-certificate.sh*

```
·         sudo mkdir -p /opt/bitnami/letsencrypt/scripts
·         sudo nano /opt/bitnami/letsencrypt/scripts/renew-certificate.sh
```

·     Enter the following content into the script and save it. Remember to replace the DOMAIN placeholder with your actual domain name, and the EMAIL-ADDRESS placeholder with your email address.

For Apache:

```
  #!/bin/bash
 
  sudo /opt/bitnami/ctlscript.sh stop apache
  sudo /opt/bitnami/letsencrypt/lego --tls --email="EMAIL-ADDRESS" --domains="DOMAIN" --path="/opt/bitnami/letsencrypt" renew --days 90
  sudo /opt/bitnami/ctlscript.sh start apache
```

For NGINX:

```
  #!/bin/bash
 
  sudo /opt/bitnami/ctlscript.sh stop nginx
  sudo /opt/bitnami/letsencrypt/lego --tls --email="EMAIL-ADDRESS" --domains="DOMAIN" --path="/opt/bitnami/letsencrypt" renew --days 90
  sudo /opt/bitnami/ctlscript.sh start nginx
```

·     Make the script executable:

```
·         sudo chmod +x /opt/bitnami/letsencrypt/scripts/renew-certificate.sh
```

·     Execute the following command to open the crontab editor:

```
·         sudo crontab -e
```

·     Add the following lines to the crontab file and save it:

```
·           0 0 1 * * /opt/bitnami/letsencrypt/scripts/renew-certificate.sh 2> /dev/null
```

NOTE: If renewing multiple domains, remember to update the */opt/bitnami/letsencrypt/renew-certificate.sh* script to include the additional domain name(s) in the *lego* command.

#### Troubleshooting

In case the certificate generation process fails or you wish to start again for any reason, run the commands below to delete the generated output, replace the previous certificates and restart services. You can then go back to [Step 1](https://docs.bitnami.com/general/how-to/generate-install-lets-encrypt-ssl/#step-1-install-the-lego-client). It is important to note that doing this will delete any previously-generated certificates and keys.

```
rm -rf /opt/bitnami/letsencrypt
```

For Apache:

```
sudo mv /opt/bitnami/apache2/conf/server.crt.old /opt/bitnami/apache2/conf/server.crt
sudo mv /opt/bitnami/apache2/conf/server.key.old /opt/bitnami/apache2/conf/server.key
sudo mv /opt/bitnami/apache2/conf/server.csr.old /opt/bitnami/apache2/conf/server.csr
sudo /opt/bitnami/ctlscript.sh restart
```

For NGINX:

```
sudo mv  /opt/bitnami/nginx/conf/server.crt.old /opt/bitnami/nginx/conf/server.crt
sudo mv /opt/bitnami/nginx/conf/server.key.old /opt/bitnami/nginx/conf/server.key
sudo mv /opt/bitnami/nginx/conf/server.csr.old /opt/bitnami/nginx/conf/server.csr
sudo /opt/bitnami/ctlscript.sh restart
```

If you created a cron job for certificate renewal, remove it by opening the crontab editor using the command below and removing the line added for the certificate renewal script:

```
sudo crontab -e
```

 

 