# Docker google-drive-ocamlfuse
Docker image to mount a google drive with google-drive-ocamlfuse shared with the host (Fully Tested on Debian Buster Host, but should work on any Linux system).

### Main Repository page (maintained by [@astrada](https://github.com/astrada)):
(all issues that are not specifically docker related should be posted there) 

https://github.com/astrada/google-drive-ocamlfuse

### Docker Hub page:

https://hub.docker.com/r/maltokyo/docker-google-drive-ocamlfuse

### Environment Variables
* `PUID`: User ID to run google-drive-ocamlfuse
* `PGID`: Group ID to run google-drive-ocamlfuse
* `MOUNT_OPTS`: Additional mount options (user_allow_other is configured in /etc/fuse.conf)
* `CLIENT_ID`: Google oAuth client ID without trailing `.apps.googleusercontent.com`
* `CLIENT_SECRET`: Google oAuth client secret
* `VERIFICATION_CODE`: Google oAuth verification code you will need to obtain manually (and prior to launching the container) from accepting the prompts at the following URL with client_id substituted:
    - `https://accounts.google.com/o/oauth2/auth?client_id=${CLIENT_ID}.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&response_type=code&access_type=offline&approval_prompt=force`
* `TEAM_DRIVE_ID`: (Optional) Team Drive Id to access a team folder instead of your private folder. The id can be found in the URL if you open the team folder in your browser (e.g. https://drive.google.com/drive/u/1/folders/${TEAM_DRIVE_ID})

### Host Configuration (one-time effort)
1. If using systemd to manage the docker daemon process make sure that the service is configured either explicitly with a `shared` mountflag or un-configured and defaults to `shared`.
2. The mount point will also need to have it's propagation explicitly marked as shared.

Without this the fuse mount will not propagate back to the host.

````
# Ensure docker daemon uses shared mount flags (this was not needed on Debian Buster, check your distro accordingly)
sed -i 's/MountFlags=\(private\|slave\)/MountFlags=shared/' /etc/systemd/system/docker.service
systemctl daemon-reload
systemctl restart docker.service

# Specify the mount points propagation as shared (execute as root), change the path according to your preference, this is just an example
mount --bind /mnt/drive /mnt/drive
mount --make-shared /mnt/drive
````

### Usage with Docker or docker-compose

Docker Run:

````
docker run -d \
-e CLIENT_ID='my-client-id' \
-e CLIENT_SECRET='my-client-secret' \
-e VERIFICATION_CODE='my-verification-code' \
--security-opt apparmor:unconfined \
--cap-add mknod \
--cap-add sys_admin \
--device=/dev/fuse \
-v /mnt/drive:/mnt/gdrive:shared \
maltokyo/docker-google-drive-ocamlfuse:0.7.19
````

Docker Compose (recommended):

Use the docker-compose file included at https://github.com/maltokyo/docker-google-drive-ocamlfuse

### Structure
* `/mnt/gdrive`: Google Drive Fuse mount directory inside container
