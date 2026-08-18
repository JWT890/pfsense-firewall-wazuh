# Wazuh Dashboard Backend Timeout Recovery

## Summary

The Wazuh Dashboard login page was reachable, but pages such as **Server management → Status** returned:

```text
Something went wrong.
timeout of 20000ms exceeded
```

The issue was resolved by restoring the Wazuh Dashboard-to-Indexer authentication configuration after initializing the Wazuh Indexer security configuration.

## Environment

- Wazuh all-in-one deployment
- Dashboard URL: `https://192.168.56.119`
- Wazuh Dashboard: HTTPS/TCP 443
- Wazuh Indexer: HTTPS/TCP 9200 on `127.0.0.1`
- Wazuh Manager API: HTTPS/TCP 55000

## Symptoms

- The dashboard login page loaded successfully.
- Dashboard pages that required backend data timed out after 20 seconds.
- Wazuh Dashboard journal contained:

```text
[TimeoutError]: Request timed out
ERR_SSL_SSLV3_ALERT_CERTIFICATE_UNKNOWN
[ResponseError]: Response Error
Unable to retrieve version information from OpenSearch nodes.
```

- Wazuh Indexer logs initially showed:

```text
Not yet initialized (you may need to run securityadmin)
Failure no such index [.opendistro_security]
Authentication finally failed for admin
```

## Root cause

The Wazuh Indexer security configuration was not initialized. Therefore, the Indexer security index (`.opendistro_security`) was absent and the Dashboard could not authenticate or reliably query OpenSearch.

After Indexer security initialization, the Dashboard-to-Indexer TLS relationship was verified as healthy. The remaining requirement was a valid `kibanaserver` password stored in the Wazuh Dashboard keystore as `opensearch.password`.

## Resolution

### 1. Initialize Wazuh Indexer security

```bash
sudo systemctl reset-failed wazuh-indexer
sudo systemctl start wazuh-indexer
sleep 30

sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
```

This loads the Indexer security configuration and creates the required security index.

### 2. Verify the Dashboard can reach the Indexer

```bash
sudo -u wazuh-dashboard curl -skv \
  --cacert /etc/wazuh-dashboard/certs/root-ca.pem \
  --cert /etc/wazuh-dashboard/certs/wazuh-dashboard.pem \
  --key /etc/wazuh-dashboard/certs/wazuh-dashboard-key.pem \
  https://127.0.0.1:9200/
```

An `HTTP/1.1 401 Unauthorized` response is useful here: it confirms the TLS handshake and Dashboard client certificate were accepted. The remaining issue is HTTP authentication.

### 3. Retrieve generated credentials

The installation archive was stored under root's installation directory:

```bash
sudo tar -xOf /root/wazuh-install/wazuh-install-files.tar \
  wazuh-install-files/wazuh-passwords.txt | less
```

Locate the password for `kibanaserver`. Do not commit this file or any password to source control.

### 4. Store the Indexer password in the Dashboard keystore

```bash
sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-keystore \
  --allow-root add -f opensearch.password
```

Paste the generated `kibanaserver` password at the prompt.

Confirm the setting exists:

```bash
sudo /usr/share/wazuh-dashboard/bin/opensearch-dashboards-keystore \
  --allow-root list | grep '^opensearch.password$'
```

### 5. Test Indexer authentication

Replace the placeholder locally with the `kibanaserver` password:

```bash
sudo -u wazuh-dashboard curl -sS \
  --cacert /etc/wazuh-dashboard/certs/root-ca.pem \
  --cert /etc/wazuh-dashboard/certs/wazuh-dashboard.pem \
  --key /etc/wazuh-dashboard/certs/wazuh-dashboard-key.pem \
  -u 'kibanaserver:PASTE_PASSWORD_HERE' \
  https://127.0.0.1:9200/
```

Expected result: JSON containing Indexer/OpenSearch node information. A `401 Unauthorized` result means the supplied password does not match the Indexer security configuration.

### 6. Restart the Dashboard

```bash
sudo systemctl restart wazuh-dashboard
sleep 30
```

Then open the dashboard in a private browser window:

```text
https://192.168.56.119
```

## Validation

Confirm all Wazuh central services are healthy:

```bash
sudo systemctl is-active wazuh-indexer wazuh-manager wazuh-dashboard filebeat
```

Expected output:

```text
active
active
active
active
```

Confirm that **Server management → Status** loads without a 20-second timeout.

## Notes

- The deprecated `agentkeepalive` messages in the Dashboard journal are warnings and were not the cause of the outage.
- Matching root CA fingerprints and a successful mutual-TLS handshake prove the certificate chain is valid; do not disable TLS verification to work around this problem.
- Keep generated credentials and `wazuh-install-files.tar` out of a Git repository. Store secrets in a password manager or a protected secret store.
- Use a static IP address or DHCP reservation before enrolling agents or configuring firewall/syslog forwarding.
