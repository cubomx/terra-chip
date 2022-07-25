# TFE Daily

## TFE Monitoring
- Health check: endpoint `/_health_check`. 200 is OK.
- Metrics & Telemetry: I/O, RAM, CPU, Disk

### Internal Monitoring
Using a statsd collector. To disable:
- Access installer dashbaord (port 8800)
- Advanced Configuration on the config page (under Terra Build Worker Image)
- Uncheck `Enable metrics collection`
- Restart the app from it

## Logging
- App: info about the services that comprise TFE
    - TFE run in a se ot Docker containers, so any tooling that interact with Docker logs can read the logs (`docker logs` or logspout). Supports logging
    drivers (auto forward logs to other services)
- Audit: info about resources managed by TFE is changed

Log forwarding using **Fluent Bit**. Destination:
- Splunk
- AWS CloudWatch

**Audit logs** emitted by the `ptfe_atlas` container. Read reqs will be logged for resources
deemed sensitive (Auth tokens, config versions, policy versions, SSH Keys, State versions, 
Variables). 
- Structure:
    - The actor (users (IP addresses), VCS users, SA, TFE)
    - The action (CRUD, add actions (`/actions/`), Webhooks API calls)
    - Target, time, where

## Upgrading
- **Online**: from the dashboard, click `Check Now`, `View Update`, `Install Update`.
- **Offline**: 
    - Put the **update path** for searching for new `.airgap`. 
    - Download it and put the `.airgap` in the path.
    - The same as online in the dashboard.

## Backup & Restore
Separate API from TFE app level APIs. Separate authorization token (Found in the dashboard at the bottom of `https://<TFE_HOSTNAME>:8800/settings`). Only way to migrate between 
operational modes.
- Backup all data store (blob storage & PostgreSQL DB)
- *Not back up the installation config* (Replicated Snapshot).

To consider:
- Cannot change Terra version.
- TFE installation must be clean without app data
- After restore, app needs to be restarted.
- Vault encryption are not preserved. 
    - Data is decrypted by Vault and encrypted by a pass you provided. During restore you provide 
    this pass (**expected as a JSON object within the request payload**)

Initiate a backup:
```sh
curl \
 --header "Authorization: Bearer $TOKEN" \
 --request POST \
 --data @payload.json \
 --output backup.blob \
 https://<TFE_HOSTNAME>/_backup/api/v1/backup
 ```

 Consider:
 - Specify an output file for the encrypted backup blob
 - Be prepared to download & store GBs of data.
 - For better performance, send this request from a server colocated with the TFE installation.
 - Treat it as sensitive data.
 - Remember the password.

 Initiate a restore when TFE app is up & running:
 ```sh
 curl \
 --header "Authorization: Bearer $TOKEN" \
 --request POST \
 --form config=@payload.json \ // password json
 --form snapshot=@backup.blob \
 https://<TFE_HOSTNAME>/_backup/api/v1/restore
 ```

 Restart the app at the end.