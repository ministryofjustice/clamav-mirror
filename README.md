# ClamAV Mirror

This repository sets up a **private ClamAV mirror**.

We require a private mirror within the Ministry of Justice (MOJ) to prevent every node from going outbound to the central ClamAV signature database.  
ClamAV signatures update regularly, and standard nodes typically perform at least 24 checks per day.  
If all nodes in our environments pull directly from the central mirror, the traffic from a small set of IP addresses would likely cause throttling issues over time.

---

## What This Service Does

This service uses **lighttpd** and **cron** to:
- Regularly **poll** the central ClamAV CVD mirror for updated virus signatures.
- **Test** the updated signatures for integrity and effectiveness.
- **Publish** the verified signatures for use by other internal ClamAV instances.

---

## How It Works

The main entry point is the `cvdmirror.crontab` file, which defines two scheduled tasks:

```bash
# Keep definitions up to date
*/10 * * * * cvd update -V

# Test and publish signatures
*/50 * * * * ./test.sh
```

- **Every 10 minutes:**  
  Runs a `cvd update` to fetch the latest signature files from the central mirror.
  
- **Every 50 minutes:**  
  Runs `test.sh`, which verifies the updated signatures using known test files.  
  If the signatures pass testing, they are published and made available for download.

---

## Testing Strategy

- Two test files (`safe` and `eicar`) are used to verify the updated signatures.
- **Two instances** of `lighttpd` are used:
  - **Mirror instance:** Hosts the updated ClamAV signatures.
  - **Test client instance:** Simulates pulling the signatures like a real client, to validate end-to-end functionality.

This dual-instance setup ensures the mirror is both **serving requests** and **delivering valid, working signatures**.

---

## Monitoring & Alerts

If the service stops running:
- An alert monitors for successful test runs every hour.
- A successful test verifies:
  - The service is reachable.
  - The published signatures are working correctly.

---

## Environment

| Environment | Host URL |
|:------------|:---------|
| Production  | [https://laa-clamav-mirror-production.apps.live.cloud-platform.service.justice.gov.uk](https://laa-clamav-mirror-production.apps.live.cloud-platform.service.justice.gov.uk) |

---

## Version Information

- **Current ClamAV Version:** `1.2.2-r0`
> **Note:** Updating the ClamAV version requires a manual update in the Dockerfile.

---

## Related Docker Image

- **Repository:** [ministryofjustice/clamav-docker](https://github.com/ministryofjustice/clamav-docker)
- **Docker Pull Command:**
  ```bash
  docker pull ghcr.io/ministryofjustice/clamav-docker/laa-clamav:latest
  ```
