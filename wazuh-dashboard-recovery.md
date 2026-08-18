# Wazuh All-in-One Dashboard Recovery

## Summary

The Wazuh Dashboard was reachable over HTTPS but showed errors including:

```text
Something went wrong.
timeout of 20000ms exceeded
```

The root cause was an uninitialized Wazuh Indexer security configuration. The indexer log reported that the `.opendistro_security` index did not exist and that `securityadmin` initialization was required.

## Environment

- Wazuh all-in-one deployment
- Dashboard access: `https://192.168.56.119`
- Wazuh Dashboard listener: TCP 443
- Wazuh Indexer listener: `127.0.0.1:9200`
- Wazuh Manager API listener: TCP 55000

> `10.0.10.10` was not assigned to the Wazuh VM. The VM's usable host-only address was `192.168.56.119`.

## Symptoms

- Browser connection to `https://10.0.10.10` timed out.
- Local dashboard test returned an HTTP `302` redirect to `/app/login`, confirming the dashboard itself was running.
- The dashboard later returned a 20-second timeout and an internal server error related to `wazuh-alerts-*` index patterns.
- `wazuh-indexer.service` failed with a systemd startup timeout.
- The indexer log contained:

```text
Not yet initialized (you may need to run securityadmin)
Failure no such index [.opendistro_security]
Authentication finally failed for admin
```

## Resolution

### 1. Use the actual VM address

Verify the addresses assigned to the Wazuh VM:

```bash
ip a
```

Access the dashboard using the actual host-only IP:

```text
https://192.168.56.119
```

### 2. Confirm dashboard connectivity

On the Wazuh VM:

```bash
curl -kI https://127.0.0.1
sudo ss -lntp | grep -E ':(443|9200|55000)\b'
```

A `302 Found` response to `/app/login` confirms that the dashboard is available locally.

### 3. Check Indexer logs

The relevant Wazuh Indexer log was:

```bash
sudo tail -n 200 /var/log/wazuh-indexer/wazuh-cluster.log
```

The log directory and ownership were valid:

```bash
sudo ls -ld /var/log/wazuh-indexer
sudo ls -la /var/log/wazuh-indexer
```

### 4. Initialize Indexer security

Start the indexer, then initialize its security configuration:

```bash
sudo systemctl reset-failed wazuh-indexer
sudo systemctl start wazuh-indexer
sleep 30
sudo ss -lntp | grep ':9200'

sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
```

This creates and loads the indexer security configuration, including the `.opendistro_security` index.

### 5. Restart dependent services

```bash
sudo systemctl restart wazuh-manager
sudo systemctl restart filebeat
sudo systemctl restart wazuh-dashboard
```

### 6. Verify service health

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

Optionally verify local indexer health using the generated `admin` password:

```bash
sudo tar -O -xvf ~/wazuh-install/wazuh-install-files.tar \
  wazuh-install-files/wazuh-passwords.txt | grep '^admin:'

curl -k -u "admin:YOUR_ADMIN_PASSWORD" \
  https://127.0.0.1:9200/_cluster/health?pretty
```

## Login

Use:

```text
URL: https://192.168.56.119
Username: admin
Password: value generated in wazuh-passwords.txt
```

To retrieve the dashboard administrator password:

```bash
sudo tar -O -xvf ~/wazuh-install/wazuh-install-files.tar \
  wazuh-install-files/wazuh-passwords.txt | grep '^admin:'
```

## Lessons learned

- Verify the VM's actual IP with `ip a`; do not rely on an address entered in an installer configuration file if it is not assigned to a network interface.
- `config.yml` is installation input. Editing it after deployment does not automatically update running Wazuh services or TLS certificates.
- A dashboard login page loading does not prove that the Indexer backend is healthy.
- If the Indexer log reports `.opendistro_security` missing or advises running `securityadmin`, run Wazuh's supported `indexer-security-init.sh` script.
- Use a static IP address or DHCP reservation before onboarding agents or configuring firewall/syslog forwarding.
