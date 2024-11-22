
# OHIF v3 with Orthanc

## Intro:
> **🩻 Ohif v3:** (OHIF) Viewer is an open source, web-based, medical imaging platform.

> **📀 Orthanc:** Its free and open-source, lightweight DICOM server for medical imaging.

### Create new docker network `pacs`

```bash
docker network create pacs
```

### Clone This Repo

```bash
git clone https://github.com/imodoye94/pacs-deploy.git
cd pacs-deploy
```

### Set Orthanc UserName & Password
> Use you favourite text editor
```bash
nano ./orthanc/config/orthanc.json
```
```json
   "AuthenticationEnabled": true,
  "RegisteredUsers": {
    "client1": "@password#1234-1"
  },
```

### Create generate the base64-encoded string
```bash
echo -n 'client1": "@password#1234-1' | base64
```

### Update Nginx config for Ohif to pass http auth for accessing Orthanc
Edit Nginx reverse proxy
```bash
nano ./ohif/nginx/ohif-nginx.conf
```
and paste base64-encoded user name and password

```conf
proxy_set_header Authorization "Basic aHlwZXI6bWFwZHI=";  # Replace with base64-encoded credentials
```
> Whenever you access OHIF it won't ask for Orthanc auth and password. 

### Spin up containers
```bash
docker compose up -d
```

### Run this command everytime you restart the containers to change the Viewer HTML Title
```bash
sudo docker exec -it --user root ohif sh -c "sed -i 's/<title>OHIF Viewer<\/title>/<title>Mediverse Viewer<\/title>/' /usr/share/nginx/html/index.html"
```
