# 📧 Secure Mail Server with Postfix & Dovecot

This guide provides a step-by-step configuration for setting up a secure mail server using **Postfix** and **Dovecot** with **SSL/TLS** and **Maildir** on a Debian/Ubuntu-based system.

---

## 🔧 Prerequisites

- A Debian/Ubuntu-based server
- Root or sudo access
- A domain name (e.g., `post.local`)

---

## 📦 Install Postfix

```bash
apt-get install postfix -y
```

- Choose Internet Site

- Enter your domain name when prompted



## 🔐 Generate SSL/TLS Certificates
```bash
openssl genrsa -des3 -out post.key 2048      
# Generate a 2048-bit RSA private key, encrypted with a passphrase (Triple DES)

openssl req -new -key post.key -out post.csr 
# Create a Certificate Signing Request (CSR) using the private key

openssl x509 -req -days 365 -in post.csr -signkey post.key -out post.crt
# Self-sign the CSR to generate an SSL certificate valid for 365 days

openssl rsa -in post.key -out post.key.nopass
# Remove the passphrase from the private key for automated service use

mv post.key.nopass post.key

openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 365

chmod 600 post.key
chmod 600 cakey.pem

cp post.key /etc/ssl/private/
cp post.csr /etc/ssl/certs/
cp cakey.pem /etc/ssl/private/
cp cacert.pem /etc/ssl/certs/
```

## ⚙️ Configure Postfix
- vim /etc/postfix/main.cf:
```bash
myhostname = post.local

mydomain = post.local

myorigin = $mydomain

home_mailbox = Maildir/

mailbox_command =
# Disables the default mailbox delivery command (useful when using Maildir)

smtpd_sasl_type = dovecot
# Specifies that Dovecot will handle SMTP authentication

smtpd_sasl_path = private/auth
# Path to the Dovecot authentication socket (Postfix connects here for SASL auth)

smtpd_sasl_auth_enable = yes
# Enables SMTP authentication on the Postfix server

```

- Apply additional TLS settings in command prompt:
```bash
postconf -e "smtpd_tls_auth_only = no"
# Allow SMTP authentication even if TLS is not used (not recommended for production)

postconf -e "smtpd_use_tls = yes"
# Enable TLS encryption for incoming SMTP connections (SMTP daemon)

postconf -e "smtp_use_tls = yes"
# Enable TLS encryption for outgoing SMTP connections (SMTP client)

postconf -e "smtp_tls_note_starttls_offer = yes"
# Log when remote SMTP servers announce STARTTLS support

postconf -e "smtpd_tls_key_file = /etc/ssl/private/post.key"

postconf -e "smtpd_tls_cert_file = /etc/ssl/certs/post.crt"

postconf -e "smtpd_tls_CAfile = /etc/ssl/certs/cacert.pem"

postconf -e "smtpd_tls_loglevel = 1"
# Set TLS logging verbosity level (1 = basic info)

postconf -e "smtpd_tls_received_header = yes"
# Add TLS information to the Received email headers for debugging

postconf -e "smtpd_tls_session_cache_timeout = 3600s"
# Cache TLS session parameters for 1 hour to speed up subsequent connections

```

- Restart Postfix:
```bash
systemctl restart postfix
systemctl status postfix
```

## 📬 Install Dovecot
```bash
apt-get install dovecot-common dovecot-imapd -y
```

### Configure Dovecot
- vim /etc/dovecot/conf.d/10-ssl.conf
```bash
ssl = required
ssl_cert = </etc/ssl/certs/post.crt
ssl_key = </etc/ssl/private/post.key
```
- vim /etc/dovecot/conf.d/10-auth.conf
```bash
disable_plaintext_auth = yes
```
- vim /etc/dovecot/conf.d/10-master.conf
```bash
unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
}
```
unix_listener /var/spool/postfix/private/auth
Defines a UNIX domain socket that Dovecot will create at this path. This socket is used for communication between Postfix and Dovecot for SMTP authentication (SASL).
- vim /etc/dovecot/conf.d/10-mail.conf
```bash
mail_location = maildir:~/Maildir
mail_privileged_group = mail
```


Restart Dovecot:
```bash
systemctl restart dovecot
systemctl status dovecot
```

## ✅ Completed
Your secure mail server using Postfix and Dovecot with SSL/TLS is now up and running with Maildir support.

## 📎 Notes
Ensure that ports 25, 587, and 993 are open in your firewall.

Use an email client like Thunderbird to test IMAP/SMTP connections.

# 📬 Contact
Feel free to fork, improve, and contribute! 

Author: mehdi khaksari Email: mahdikhaksari36@gmail.com
