
# OHIF v3 with Orthanc

## Intro:
> **🩻 Ohif v3:** (OHIF) Viewer is an open source, web-based, medical imaging platform.

> **📀 Orthanc:** Its free and open-source, lightweight DICOM server for medical imaging.


# Setting Up PACS and Integrating Local Machines with Cloud PACS

## Introduction

This document outlines the process of setting up a cloud-based PACS and viewer, deploying a local Orthanc instance to act as a gateway, and integrating imaging machines to connect seamlessly with the cloud PACS. This setup ensures that patient data is securely stored, and imaging workflows are automated, providing efficiency and compliance with data protection regulations.

---

## Prerequisites

- **Cloud Service Provider**: Access to a cloud service provider (e.g., Google Cloud Platform or DigitalOcean).
- **Domain Name**: Managed through a DNS provider like Namecheap.
- **Docker**: Installed on the cloud server (or a Docker-ready droplet).
- **Secure Credential Storage**: A secure method to store sensitive credentials (e.g., Google Cloud Secret Manager).
- **Company Flash Drive**: Contains the following:
  - Windows installers for Orthanc and Syncthing.
  - The `upload_to_cloud.lua` script.
  - This setup guide (README.md).

---

## Phase 1: Setting Up the Cloud PACS and Viewer

### Step 1: Provision a Cloud Server

- **For DigitalOcean**:
  - Create a Docker Droplet (choose appropriate specifications based on expected load).
- **For Google Cloud Platform**:
  - Create a Compute Engine instance.
  - Install Docker on the instance:

    ```bash
    sudo apt-get update
    sudo apt-get install docker.io
    ```

### Step 2: Clone the PACS Deployment Repository

1. **SSH into the cloud server**:

    ```bash
    ssh username@server_ip
    ```

2. **Clone the repository**:

    ```bash
    git clone https://github.com/imodoye94/pacs-deploy.git
    ```

3. **Change directory to the cloned repository**:

    ```bash
    cd pacs-deploy
    ```

### Step 3: Configure Orthanc

1. **Edit the `orthanc.json` file**:

    ```bash
    nano orthanc.json
    ```

2. **Update the `Username` and `Password` fields** with the secure Orthanc credentials retrieved from your secure credentials manager:

    ```json
    {
      "Name": "Orthanc",
      "AuthenticationEnabled": true,
      "RegisteredUsers": {
        "username": "password"
      }
      // Other configurations...
    }
    ```

### Step 4: Configure Nginx for the OHIF Viewer

1. **Generate the Base64 encoded string** of the `username:password` combination:

    ```bash
    echo -n 'username:password' | base64
    ```

2. **Edit the Nginx configuration file for OHIF**:

    ```bash
    nano nginx_ohif.conf
    ```

3. **Replace the existing `Authorization` header's Base64 string** with the one you just generated:

    ```nginx
    proxy_set_header Authorization "Basic base64_encoded_credentials";
    ```

### Step 5: Deploy the Docker Containers

1. **Create a Docker network**:

    ```bash
    docker network create pacs
    ```

2. **Start the containers using Docker Compose**:

    ```bash
    docker compose up -d
    ```

### Run this command everytime you restart the containers to change the Viewer HTML Title
```bash
sudo docker exec -it --user root ohif sh -c "sed -i 's/<title>OHIF Viewer<\/title>/<title>Mediverse Viewer<\/title>/' /usr/share/nginx/html/index.html"
```

### Step 6: Configure DNS Records

1. **Log in to your DNS provider** (e.g., Namecheap).
2. **Create `A` records pointing to your server's IP address** for:

    - `siteid.viewer.mediverse.ai`
    - `siteid.pacs.mediverse.ai`

3. Replace `siteid` with the actual site identifier and `example.com` with your domain.

### Step 7: Set Up Nginx Proxy Manager

1. **Access Nginx Proxy Manager** at `http://<server-ip>:81`.
2. **Log in with default credentials** (update if necessary).
3. **Create two Proxy Hosts**:
    - **For the Viewer**:
        - **Domain Names**: `siteid.viewer.mediverse.ai`
        - **Forward Hostname / IP**: `ohif`
        - **Forward Port**: `80`
        - **Enable Websockets Support**
        - **Request a Let's Encrypt SSL certificate`
    - **For the PACS**:
        - **Domain Names**: `siteid.pacs.mediverse.ai`
        - **Forward Hostname / IP**: `orthanc`
        - **Forward Port**: `8042`
        - **Request a Let's Encrypt SSL certificate`
4. **Save and apply the configurations.**

### Step 8: Verify the Setup

- **Access the OHIF Viewer** at `https://siteid.viewer.mediverse.ai`.
- **Access the Orthanc PACS** at `https://siteid.pacs.mediverse.ai`.
- **Ensure both services are running securely over HTTPS.**

---

## Phase 2: Setting Up the Local Orthanc Gateway

### 2a. Installing Orthanc and Syncthing on the Local Machine

1. **Prepare the Company Flash Drive**:
    - Ensure it contains:
        - Windows installer for Orthanc.
        - Windows installer for Syncthing.
        - The `upload_to_cloud.lua` script.
        - This setup guide (README.md).

2. **Install Orthanc**:
    - Run the Orthanc Windows installer from the flash drive.
    - Follow the installation prompts to complete the setup.

3. **Install Syncthing**:
    - Run the Syncthing Windows installer from the flash drive.
    - Follow the installation prompts to complete the setup.

4. **Copy the Lua Script**:
    - Create a directory for Orthanc scripts, e.g., `C:\\Users\\HP\\Documents\\OrthancScripts`.
    - Copy `upload_to_cloud.lua` into this directory.

---

## Connecting the Scanner to the Local Orthanc and Automation

Complete automation logic and connectivity details for the scanner continue.


# Setting Up DICOM Worklist Synchronization and Image Forwarding

## 2b. Setting Up DICOM Worklist Synchronization via Syncthing

### Configure Orthanc

1. **Open the `orthanc.json` configuration file** (usually located in `C:\\Program Files\\Orthanc Server\\Configuration`).

2. **Update the `Name` and `DicomAet` to your desired AE Title**, e.g., "Mediverse":

    ```json
    {
      "Name": "Mediverse",
      "DicomAet": "MEDIVERSE",
      // Other configurations...
    }
    ```

3. **Add the path to the Lua scripts**:

    ```json
    "LuaScripts": [ "C:\\Users\\HP\\Documents\\OrthancScripts\\upload_to_cloud.lua" ],
    ```

4. **Define the Orthanc peer (the cloud PACS)**:

    ```json
    "OrthancPeers": {
      "MediverseCloud_siteid": {
        "Url": "https://siteid.pacs.example.com",
        "Username": "your_orthanc_cloud_username",
        "Password": "your_orthanc_cloud_password"
      }
    },
    ```

5. **Enable the Worklist feature**:

    ```json
    "Worklists": {
      "Enabled": true,
      "Database": "C:\\Users\\HP\\Documents\\PACSWorklists"
    },
    ```

6. **Save the `orthanc.json` file**.

---

### Restart Orthanc Service

1. **Open the Services management console** (`services.msc`).
2. **Find the Orthanc service**.
3. **Restart the service** to apply the new configuration.

---

### Set Up the Worklist Directory

1. **Create the directory specified in the `Database` path**, e.g., `C:\\Users\\HP\\Documents\\PACSWorklists`.

2. **Grant full permissions to the Syncthing service account**:

   - **Right-click the folder**, select **Properties**.
   - Go to the **Security** tab.
   - Click **Edit**, then **Add**.
   - Enter `Syncthing Service Account` (usually `syncthingserviceacct`).
   - Grant **Full Control** permissions.

---

### Configure Syncthing on the Local Machine

1. **Open Syncthing**.

2. **Add a folder to share**:

   - **Folder Path**: `C:\Users\HP\Documents\PACSWorklists`.
   - **Folder ID**: A unique identifier (e.g., `siteid-worklists`).
   - **Folder Label**: `Orthanc Worklists`.

3. **Connect Syncthing to the Cloud Instance**:

   - **Obtain the Device ID** of the cloud Syncthing instance.
   - **Add the remote device** in the local Syncthing:
     - **Device ID**: Cloud Syncthing Device ID.
     - **Device Name**: `CloudSyncthing`.
   - **Share the `Orthanc Worklists` folder** with the cloud device.
   
4. **Set up the folders in the running container where .wl files from the LIMS will be written**:

   - **Login to the CLFY backend** of mediverse cloud.
   - **SSH into the running n8n container** here create a single folder for that sites's .wl files:
     - **Create a folder**:

		 ```bash
		 mkdir /files/worklists_wl/siteid
		 ```
   - **SSH into the CLFY host itself** grant ownership and permission of the site folder to the node user.
        - **Change ownership of folder**:


		```bash
		sudo docker exec -it --user root <n8n container id> chown -R node /files/worklists_wl/siteid
		sudo docker exec -it --user root <n8n container id> chmod -R u+rwx /files/worklists_wl/siteid
		```
---

## 2c. Automating Image Forwarding Using Lua Scripts

### Ensure the `upload_to_cloud.lua` Script is Correct:

```lua
function OnStoredInstance(instanceId, tags, metadata, origin)
  -- Avoid infinite loops
  if origin['RequestOrigin'] ~= 'Lua' then
    local peerName = 'MediverseCloud_siteid'
    local resourceType = 'Instance'

    -- Send the instance to the cloud PACS
    local success, result = pcall(function()
      return SendToPeer(instanceId, peerName, resourceType)
    end)

    if success then
      print("Successfully sent instance " .. instanceId .. " to " .. peerName)
    else
      print("Failed to send instance " .. instanceId .. " to " .. peerName)
      print("Error: " .. result)
    end
  end
end
```

---

### Verify the Script Path in `orthanc.json`:

```json
"LuaScripts": [ "C:\\Users\\HP\\Documents\\OrthancScripts\\upload_to_cloud.lua" ],
```

---

### Restart Orthanc Service Again:

- Restart the Orthanc service to ensure the Lua script is loaded.

---

## Connecting the Scanner to the Local Orthanc

### Configure the Local Network Adapter

1. **Set a Static IP Address** on the Ethernet adapter that connects to the scanner:
   - **IP Address**: An IP in the same subnet as the scanner (e.g., if the scanner is `192.168.0.10`, set the computer to `192.168.0.20`).
   - **Subnet Mask**: Match the scanner's subnet mask (usually `255.255.255.0`).
   - **Default Gateway**: If required, set to the network's gateway or leave blank if not used.

2. **Disable Other Network Adapters** if necessary to avoid network conflicts.

---

### Register the Scanner in Orthanc

1. **Edit the `orthanc.json` configuration file**.

2. **Add the scanner to the `DicomModalities` section**:

    ```json
    "DicomModalities": {
      "Scanner": {
        "Aet": "SCANNER_AE_TITLE",
        "Host": "192.168.0.10",
        "Port": 104,
        "AllowEcho": true,
        "AllowStorage": true
      }
    },
    ```

   - Replace:
     - `SCANNER_AE_TITLE` with the scanner's AE Title.
     - `192.168.0.10` with the scanner's IP address.
     - `104` with the scanner's DICOM port (default is usually `104`).

3. **Restart Orthanc Service** to apply changes.

---

### Configure the Scanner's DICOM Settings

1. **Set the Destination (Store SCP) Configuration**:
   - **AE Title**: `MEDIVERSE` (as set in Orthanc's `DicomAet`).
   - **Host IP**: The static IP of the local machine (e.g., `192.168.0.20`).
   - **Port**: `4242` (Orthanc's default DICOM port).

2. **Set the Worklist Server (Query SCP) Configuration**:
   - **AE Title**: `MEDIVERSE`.
   - **Host IP**: The static IP of the local machine.
   - **Port**: `4242`.

3. **Save the Configuration** on the scanner.

---

### Test the Connection

1. **Perform a DICOM Echo (C-ECHO)**:
   - From the scanner, send a C-ECHO request to the local Orthanc.
   - Verify that the response is successful.

2. **Test the Worklist Query**:
   - On the scanner, perform a Worklist query.
   - Verify that scheduled patients appear (assuming worklists are available).

3. **Test Image Transmission**:
   - Acquire a test image or study.
   - Send the images to the local Orthanc.
   - Verify that the images appear in Orthanc and are forwarded to the cloud PACS.

---

## Conclusion

By following these steps, you have successfully set up a cloud PACS and viewer, configured a local Orthanc gateway, and connected a scanner to integrate seamlessly with the cloud infrastructure. This setup ensures secure, efficient, and automated handling of imaging data, streamlining workflows and enhancing patient care.

- **Secure Data Handling**: Patient data is securely transferred and stored.
- **Automated Workflows**: DICOM Worklists and image forwarding are automated.
- **Scalable Architecture**: Each customer has isolated resources, enhancing security and scalability.
- **Regulatory Compliance**: The setup adheres to data protection regulations like HIPAA.
