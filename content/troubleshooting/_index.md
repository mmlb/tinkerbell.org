+++
title = "Troubleshooting"
date = 2019-01-04T16:16:15+05:30
draft = false
weight = 50
toc = true
+++

The following are possible errors you might run into and their possible solutions.

```
PowerEdge R6415 - BIOS 1.8.7
A system restart is required. The system detected an exception during the UEFI
pre-boot environment.
```
- Set the worker to always PXE

```
Login did not succeed, error: Error response from daemon: Get https://192.168.1.1/v2/: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "Autogenerated CA")
```
- Rebuild the registry container (after rebuilding the certs container)

```
Waiting for docker to respond...
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Error response from daemon: Get https://192.168.1.1/v2/: x509: certificate signed by unknown authority (possibly 
because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "Autogenerated CA")
```
- Copy the ca.pem file over to the /etc/tinkerbell/nginx/workflow directory (after rebuilding the certs container)

```
See 'docker run --help'.
 * start-stop-daemon: failed to start `/sbin/workflow-helper'
 * Failed to start Packet Workflow
 [ !! ]
 * ERROR: workflow-helper failed to start
```
- Log into console with username root and run the following command:

```
docker run -it --privileged=true \
-e "DOCKER_API_VERSION=v1.35" \
-e "WORKER_ID=fde7c87c-d154-447e-9fce-7eb7bdec90c0" \
-e "DOCKER_REGISTRY=192.168.1.1" \
-e "TINKERBELL_GRPC_AUTHORITY=192.168.1.1:42113" \
-e "TINKERBELL_CERT_URL=http://192.168.1.1:42114/cert" \
-e "REGISTRY_USERNAME=admin" \
-e "REGISTRY_PASSWORD=admin123" \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /worker:/worker \
-v /dev:/dev \
-v /etc/docker/cert.d/192.168.1.1:/192.168.1.1 \
192.168.1.1/tink-worker
```
```
ERRO[0000] Worker Finished with error rpc error: code = Unavailable desc = connection error: desc = "transport: authentication handshake failed: x509: certificate is valid for 127.0.0.1, not 192.168.1.1"
 [ ok ]
```
- Tear down the stack with docker-compose down -v
- Rerun the setup script with just the following functions:
  + build_and_setup_certs
  + build_registry_and_update_worker_image
  + start_docker_stack
- Re-push the tink-worker and action images
```
docker push 192.168.1.1/tink-worker
docker push 192.168.1.1/ubuntu:base
docker push 192.168.1.1/disk-wipe:v3
docker push 192.168.1.1/disk-partition:v3
docker push 192.168.1.1/install-root-fs:v3
docker push 192.168.1.1/install-grub:v3
```
- Recreate the workflow