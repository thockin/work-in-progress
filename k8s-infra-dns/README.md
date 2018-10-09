# Managing DNS for Kubernetes domains

## Zones

Zones we manage:
  - kubernetes.io
  - k8s.io

## How to become an admin

Admin access is granted via
[googlegroups](https://groups.google.com/forum/#!forum/k8s-infra-dns-admins).
To volunteer for this effort, contal the main
[k8s-infra-team](https://groups.google.com/forum/#!forum/k8s-infra-team).

## Where is it hosted?

We host it in Google Cloud DNS.
  * GCP org = kubernetes.io
  * GCP project = kubernetes-public
  * https://console.cloud.google.com/net-services/dns/zones?project=kubernetes-public&organizationId=758905017065

## Status

WIP.  The zones are created and have been synced once for testing, but none of
the rest of this is finalized.  Top-level NS records have not been flipped.

## How to update

We use OctoDNS (https://github.com/github/octodns) to manage the live config.
Eventually we want this automated and triggered by git commits.  Until them,
manual runs are OK.

### Docker image

From this repo:

```sh
./build.sh
```

### Running as yourself

If you want to run it as yourself, using yor own Google Cloud credentials:

```sh
./sync-manual.sh
```

### Running automated

To run it automated (with GCP service account creds), get the JSON for the
service account, edit config.yaml and un-comment the `credentials_file`.


```
docker run -ti \
    -v `pwd`/config:/octodns/config \
    -v `pwd`/octodns-dns-admin-creds.json:/octodns/creds/gcp.json \
    -v `pwd`/config.yaml:/octodns/config.yaml \
    myname/octodns \
    octodns-sync \
        --config-file=/octodns/config.yaml \
        --log-stream-stdout \
        --debug \
        --doit # leave this off if you want to do a dry-run
```

## TODO

  * Move to a subdir of k8s.io repo
  * Get owner names and comments on all records
  * Rationalize and document all deltas between zones
  * Push an official image to GCR
    * When to rebuild?
  * Document how to update from github
    * YAML via URL (raw.github.com)?  Need support in octodns.
    * wget from raw.github before sync
    * git fetch
  * Create a canary zone
  * Script updates
    * dry run to canary, abort on error
    * --doit to canary
    * run a test, abort on error
    * dry run to prod, abort on error
    * --doit to prod
    * run a test, abort on error
  * Document how to handle "too many" updates (--force)
    * Always --force?
  * Billing report
  * Usage report
  * Monitoring / alerts / on-call
  * Figure out who to contact to flip top-level NS records.


How to automate:
  * Need a k8s cluster to run it
  * PR to github, cronjob in a cluster syncs
  * PR to github, manually apply a configmap, inotify/poll in a deployment
  * PR to github, webhook triggers run in a cluster
  * How to handle "too many updates"?
    * manual intervention?
    * always --force?

DNS content fixes:
  * Can we just have a * -> redirect.k8s.io as a catchall?
    * PRO: less rules overall, less churn
    * CON: any rando URL will now land somewhere (we could make them 404, maybe?)
  * Fix dl.k8s.io -> gcsweb or something useful
