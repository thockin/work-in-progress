Running DNS for Kubernetes

Zones: kubernetes.io and k8s.io

Admin access via googlegroups: https://groups.google.com/forum/#!forum/k8s-infra-dns-admins

Running in GCP org kubernetes.io, project kubernetes-public: https://console.cloud.google.com/net-services/dns/zones?project=kubernetes-public&organizationId=758905017065

Status: WIP.  The zones are created and have been synced, but none of the
rest of this is finalized.  Top-level NS records have not been flipped.

------------------------

HOW TO RUN:

docker build -t thockin/octodns .

# To run as yourself:
docker run -ti \
    -u `id -u` \
    -v ~/.config/gcloud:/.config/gcloud:ro \
    -v `pwd`/config:/octodns/config:ro \
    -v `pwd`/config.yaml:/octodns/config.yaml:ro \
    thockin/octodns \
    octodns-sync \
        --config-file=/octodns/config.yaml \
        --log-stream-stdout \
        --debug \
        --doit


# To run automated (with service account creds), get the JSON for the service
# account, edit config.yaml and un-comment the `credentials_file`.
docker run -ti \
    -v `pwd`/config:/octodns/config \
    -v `pwd`/octodns-dns-admin-creds.json:/octodns/creds/gcp.json \
    -v `pwd`/config.yaml:/octodns/config.yaml \
    thockin/octodns \
    octodns-sync \
        --config-file=/octodns/config.yaml \
        --log-stream-stdout \
        --debug \
        --doit


------------------------

TODO:

Get owner names and comments on all records

New github repo or subdir, with OWNERS

Dockerfile and image push and ownership
- use explicit versions

Can we just have a * -> redirect.k8s.io as a catchall?
- PRO: less rules overall, less churn
- CON: any rando URL will now land somewhere (we could make them 404, maybe?)

Need a k8s cluster to run it

How to trigger?
- PR to github, cronjob in a cluster syncs
- PR to github, manually apply a configmap, inotify/poll in a deployment
- PR to github, webhook triggers run in a cluster

How to pull from github
- YAML via URL (add support)
- wget from raw.github before sync

How to sync:
- dry run, exit on error
- --doit
- apply and test a canary zone first?
  - needs a test written
- how to handle cses with too many updates?
  - manual intervention?
  - always --force?

Figure out who to contact to flip top-level NS records.

Fix dl.k8s.io -> gcsweb or something useful
