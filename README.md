
# OHIF v3 with Orthanc

## Intro:
> **🩻 Ohif v3:** (OHIF) Viewer is an open source, web-based, medical imaging platform.

> **📀 Orthanc:** Its free and open-source, lightweight DICOM server for medical imaging.

*** Create new docker network =pacs=
#+begin_src bash
docker network create pacs
#+end_src

*** Clone This Repo
#+begin_src bash
git clone https://github.com/imodoye94/pacs-deploy.git
cd pacs-deploy
docker-compose up -d
#+end_src

It will also create a new /bridge-network/ pacs.

*** Access OHIF and Orthanc

1. To access =OHIF= go to address `http://server-ip:3000`.
2. To access =ORTHANC= webpage go to address `http://server-ip:8042`

Default Username is =user= & password is =@password1234=


*** Import DICOM Studies
1. You can directly Import DICOM files or Folder from OHIF.
2. You can directly send images from existing PACS/Machine
 - AE: MEDIVERSE
 - PORT: 4242
 - IP: your-server-ip
3. If you have file locally install httplib2 with pip =python ImportDicomFiles.py localhost 8042 /path/to/input/dir=
