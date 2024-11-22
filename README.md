
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

### Spin up containers
```bash
docker compose up -d
```

### Run this command everytime you restart the containers to change the Viewer HTML Title
```bash
sudo docker exec -it --user root ohif sh -c "sed -i 's/<title>OHIF Viewer<\/title>/<title>Mediverse Viewer<\/title>/' /usr/share/nginx/html/index.html"
```
