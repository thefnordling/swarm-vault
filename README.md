# homelab-vault
This is the HashiCorp Vault setup I use in my home lab environment.  I use the PKI plugin to create a Root Certificate for the home lab (home.arpa), as well as a leaf/intermediate certificate (generated/signed by the root CA) which will be used to create the hosting certificates that the various services I run in my home lab will use for TLS.

Secrets that applications and services will use will be stored in the Key-Value secret plugin (v2).

I use the approle plugin to limit access to PKI and KV secrets via policy based access control.

Unlock-Keys, root tokens and other such secrets will be saved outside of vault in a password manager.

Persistent Storage:

I will be running Vault as a stateless docker service.  We will be storing all vault data on an NFS share so all of our approles, secrets and PKI data will be not be lost after a restart or host crashing.  All of our docker hosts will have the same NFS volume mounted at the same location (via /etc/fstab on host) to store the encrypted Vault Data.

We will be taking a daily backup of vault, and retaining that backup for 30 days.  Backups will be stored on a separate NFS share, which is set up similarly on each host.  At some point in the near future will be switching to S3 (minio or aws) to store the backups (so if my nas dies i can recover from a separate store).  I will likely write a simple [C# Consoel App](https://github.com/thefnordling/dotnet-s3-example) for that when i have some time.

Setup:

After provisioning storage, setting up NFS mounts and installing docker - I fire up the [initial service](./docker/initial/docker-compose.yml).  This is essentially a development instance of Vault.  Once the service starts up - i attach to the container and initialize the vault by running the command `vault operator init -key-shares=1 -key-threshhold=1`.  If you have more time and enjoy a more complex/secure treatment - you should generate more keys and have a higher number of keys required to unlock (but for a home lab POC this was fine for me).  This command will produce the unseal keys as well as a root token - save those someplace safe and secure you will need them for future maintenance.