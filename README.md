
# 🐧IncuSpeed : Linux Virtualization RestAPI Server and Front-End App

> **A super-lightweight LXD/Incus container management GUI for Linux systems**  
> Current Host Distro: **🟣 Ubuntu 24.04**

[![DeepSource](https://app.deepsource.com/gh/gg582/incuspeed.svg/?label=code+coverage&show_trend=true&token=eghapy87jjnelBbEvOL0DkW4)](https://app.deepsource.com/gh/gg582/incuspeed/)
[![DeepSource](https://app.deepsource.com/gh/gg582/incuspeed.svg/?label=active+issues&show_trend=true&token=eghapy87jjnelBbEvOL0DkW4)](https://app.deepsource.com/gh/gg582/incuspeed/)
[![DeepSource](https://app.deepsource.com/gh/gg582/incuspeed.svg/?label=active+issues&show_trend=true&token=eghapy87jjnelBbEvOL0DkW4)](https://app.deepsource.com/gh/gg582/incuspeed/)

## Overview

### Purpose of the Project
Incus is a powerful manager for system containers and VMs, but it typically operates through a text-based user interface (TUI), and all tasks need to be executed through shell commands. This presents a challenge for developers who want to easily set up environments but don't always have access to a full development setup (e.g., on a subway).

This project aims to solve that problem by providing a management app for Incus containers that makes it easier to set up, manage, and test containers on the go. Whether you're developing, testing, or just want to manage your Linux containers from anywhere, this app makes Incus more accessible, especially for developers.
### Strength compared to Well-known Apps

Many kinds of Incus management app is constructed as Web App Dashboard. Although it supports strong, and detailed setup for managing Incus container, too many settings and details are not necessary for creating simple containers.
Many of developers, and amateur programmers may face a clear fact: our habits for testing containers are, basically similar to CRUD.

If someone needs to deploy huge web services, one has better solution: Podman-Compose/Docker-Compose, Kubernetes.

Also, there are many ready-made solutions, like AWS, GKE. For production use, one can use them.

This project aims for **dumb, but prominent container works**. With this Desktop/Mobile app and server set, you can manage it simpler.

Supports major distros including RHEL-forks(alma, rocky), Ubuntu(20.04, 22.04, 24.04), Debian(10, 11, 12), Slackware(14.2, 15.0, current), Archlinux.

**You can add custom distro via incus publish, image name should be in a format, with a single hyphen(distro-version).**
For example, ou have ubuntu 24.04 container that contains nginx v1.24.0 installation.
Then, you should follow this:
```bash
incus snapshot create my-nginx-container nginx
incus publish my-nginx-container/nginx --alias nginx-1.24.0
```
Then, you can use this image as nginx:1.24.0.
#### Test for custom NGINX container

```bash
git submodule init
git submoule update
cd examples/nginx-examples
```
And see README.md text for information.

#### Questions & Answers

- Q. How about monitoring metrics and logger of server?
- A. Use Kibana, Java Resource Monitoring functions, Grafana, etc. Those are not suitable for **dumb application**.

- Q. How can I use spare port for web server?
- A. 30001, 30002 in container, are mapping into SSH port +1, and SSh port +2. Use each port and keep connecting.

- Q. Can I transfer big files into container?
- A. Recently updated. Check for v0.2.7.

- Q. How can I specify filename?
- A. Add file extension at your destination.

- Q. What languages does it support?
- A. I am from South Korea. So, CJK + English, and some well-known European alphabets are supported. Check the font that I provided.

- Q. Why did you select Go, and Python3?
- A. Go is easy to learn. Also, Incus natively support Go(since it is implemented in Go). Python3 + KivyMD is great choice for fast prototyping.

- Q. Why all POST methods? It is basically CRUD.
- A. To write consistent and simple code, every tasks are managed as POST; Simple information is handled by plain text, detailed one is managed by JSON.  

- Q. How can I use it on my server?
- A. You don't need to edit REST Server. Just edit your client app, make your openssl configuration, and certificates too.

### REST API Structure

![RestAPI structure](assets/RestAPIStructure.png)

The back-end of the project is built using **Go**, and it interacts with Incus containers through API bindings. The system includes basic requests for managing container states and creating new containers. It operates with a simple API structure that manages container tags and allocates necessary ports for each container.

- **Ports:** 
  - First port is used for SSH access (OpenBSD Secure Shell).
  - Two additional ports are available for other services like MySQL, XRDP, etc. (30001, 30002)

### Secure Shell Reverse Proxy

![Secure Shell Proxy](assets/SSHConnectionRevProxy.png)

SSH access to containers is managed by an Nginx reverse proxy. When a container starts up, the reverse proxy configuration is automatically updated to route SSH traffic to the container's allocated SSH port.

---

## 🚀 Getting Started – Server Setup

### 📦 Container State Change API

These endpoints allow you to change the state of a container instance managed by the virtualization unit.

#### Available Endpoints for Status Change
Other endpoints such as `/delete`, `/create`, and `/request` are detailed in the Swagger docs. Please visit `https://yourserverdomain:32000/swagger/index.html` for the full API reference.

| Method | Endpoint     | Description               |
|--------|--------------|---------------------------|
| POST   | `/start`     | Start a container         |
| POST   | `/pause`     | Pause a running container |
| POST   | `/resume`    | Resume a paused container |
| POST   | `/restart`   | Restart a container       |
| POST   | `/upload`    | Upload a file to container|

#### Request Body

All endpoints require a plain text body with the container's tag.

```text
  container-name
```

- **tag** (string, required): The unique identifier (name or tag) of the container you want to target.

#### Responses

| Code | Meaning                      |
|------|------------------------------|
| 200  | State changed successfully   |
| 202  | File Push Accepted       g   |
| 400  | Bad request (e.g. missing tag) |
| 500  | Internal server error        |

#### Example Curl

```bash
curl -X POST https://<host>:<port>/start   -d 'my-container --cacert ca.crt'
```

---
## Setting Up an initial Master Node
*warning: you should change the domain to your server, at linux_virt_unit/linux_virt_unit.go and app/main.py.*
### 📦 Installation Steps

1. **Clone this repository**
   ```bash
   git clone https://github.com/gg582/incuspeed
   cd incuspeed
   ```

2. **Run installation commands**
   ```bash
   git clone https://github.com/gg582/linux_virt_unit
   make
   ./initial_setup.sh --reconfigure-incus
   systemctl start --now incuspeed
   ```

   > ⚠ **WARNING:**  
   > This process **overwrites your Nginx configuration**.  
   > Be sure to modify `nginx.conf` from this repo before running the setup script.

3. **After setup**
   - Default SSH & Spare ports will be assigned automatically.
   - Incus containers' port connection will be managed via a reverse-proxy (Nginx).
### 🔐Generating Certificates
1. **Load Management Tools**
    ```bash
    source ./utils/management_tools.sh
    ```
2. **Run Keygen***
    ```bash
    source ./utils/keygen.sh
    ```
This certification will be included when building mobile apps.
### 📱Building Your App
1. **Install OpenJDK**
2. **Install Required Packages**
    ```bash
    pip3 install -r requirements.txt
    ```
3. **Install extra dependencies**
    libffi-dev is required for Cythonize
4. **Build your app**
    There are long integer bugs in pyjnius. Use cython version that kivy team has suggested



### Installation Steps
---

## 🖥 GUI Application Usage

### 📷 Screenshots

![ContList](./assets/containerlist.png)
![MainScreen](./assets/mainscreen.png)

### ✅ Quick Steps

1. **Navigate to the application directory**
   ```bash
   cd app
   ```

2. **Run the GUI application**
   ```bash
   python3 main.py
   ```

3. **Login with default credentials**
   - **Username:** *username from Front-end app*
   - **Password:** *password from Front-end app*  
     > ⚠ Change this password immediately after your first login!

---

## 💡 How to Use
Go to your application, and manage your container by buttons and entries.

## 🧠 Back-End Information

- Written in **Go**, optimized for Linux server orchestration.
- All container operations (create, stop, start, etc.) are securely managed via the back-end.

### 🔧 Build the back-end binary

```bash
make
```

---

## 🧱 Virtual Machine Management

- Powered by **LXD/Incus containers**
- Integrated with Nginx reverse-proxy (experimental!)
- Containers are isolated, and each is assigned its own port.

> ⚠ **Note:** Reverse proxy logic is still under active development. You may experience unstable behavior when using multiple container ports simultaneously.

---

## 📁 Directory Structure Overview

```text
linuxVirtualization/
├── app
│   ├── bin
│   │   ├── ##your compiled apks
│   ├── buildozer.spec #buildozer config file
│   ├── certs
│   │   └── ca.crt # client cert (auto-generated)
│   ├── icon.png
│   ├── main.py # kivy client 
│   ├── README.md
│   └── requirements.txt
├── ca.srl
├── certs # server certs (auto-generated)
│   ├── server.crt
│   └── server.key
├── conSSH.sh # ssh initialization script
├── container
│   └── latest_access
├── docs # swagger docs
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── drop_all.props # force drop all mongo props
├── go.mod 
├── go.sum
├── initial_setup.sh # initial setup script
├── install_svc.sh # install daemon service script
├── killall.sh # force delete all information
├── kill_for_reload.sh 
├── kill.sh # systemctl stop command
├── linuxVirtualizationServer # go compiled binary
├── linuxVirtualization.service # daemon service file
├── linux_virt_unit
│   ├── crypto
│   │   └── crypto.go # encryption logics
│   ├── go.mod
│   ├── go.sum
│   ├── http_request
│   │   └── http_request.go # RestAPI Endpoints
│   ├── incus_unit
│   │   ├── base_images.go # auto-generated base image fingerprints
│   │   ├── change_container_status.go # state change logic
│   │   ├── create_containers.go # container creation logic
│   │   ├── get_info.go # get miscellanous information
│   │   ├── handle_container_state_change.go # handle state change endpoints
│   │   ├── handle_user_info.go # securely handle user auth
│   │   └── worker_pool.go # multi-processing worker pool
│   ├── linux_virt_unit.go # shared structure definitions
│   ├── mongo_connect
│   │   └── mongo_connect.go # mongoDB client initialization
│   └── README.md
├── main.go # main function of this server
├── Makefile
├── mongo.props # create specified mongoDB admin user
├── nginx.conf  # default nginx configuration (if you have pre-configured Nginx config, place here)
├── openssl.cnf # openssl configuration for self-signing
├── README.md
├── remove-service.sh # daemon service uninstallation
├── server_reload.sh  # systemctl restart 
├── server.sh # execute server as nohup
└── utils
    ├── keygen.sh # generate self-signed certificate
    ├── make_base_images.sh # create base image
    ├── make_incus_units.sh # create base_images.go
    └── management_tools.sh # bash alias for convenient management
```

## 🧩 Architecture

```
[Client (KivyMD)] ⇄ [REST API (Go)] ⇄ [linux_virt_unit] ⇄ [Incus API]
                                       ⇅
                                   [MongoDB]
```

---

## 📜 TODO

- Support for other distributions
- Incus integration for RestAPI `/create` path

## 📜 NOTE

- Default domain is `hobbies.yoonjin2.kr`.
- If you are installing this, please change the URL prefix.
- You can find prefixes by running this command:
```bash
grep yoonjin2 $(find . -type f)
```
Additional images are supported.
See `utils/additional_image/README.md` for more.
[Additional README](./utils/additional_image/README.md)

---

## 🤝 Contributing

Pull requests are welcome! If you find any issues or have improvements, feel free to open a PR or issue.

---

## 📜 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

