# 🌐 Serverless Image Processing with Multi-User Concurrency and Distributed Computing

## 📝 Project Overview  
This project presents a scalable, intelligent image processing system designed to support **multiple concurrent users**. It leverages a **hybrid architecture** that integrates a Windows-based host machine, Linux-based local virtual machines (VMs), and serverless cloud containers deployed on **Google Cloud Run**. The system is built with modular microservices, real-time monitoring, and a lightweight custom load balancer that dynamically routes user tasks to the most optimal resource based on live system metrics.

---

## ⚙️ Key Features  
- 🧩 **Modular microservices** for:
  - Image-to-sketch transformation ✏️
  - Background removal 🖼️
  - Image captioning using Salesforce BLIP 🧠
- 📡 Real-time CPU and memory monitoring of VMs via socket communication
- ⚖️ Load balancing based on current RAM and CPU utilization
- ☁️ Fallback to **Google Cloud Run** when both VMs are under high load
- 🔗 RESTful APIs and lightweight socket protocols for inter-component communication
- 🎛️ Streamlit-based interactive dashboard and user interface
- 👥 Support for multiple concurrent users with minimal latency

---

## 🧱 System Architecture  

### 1️⃣ **Windows Host Controller (`windows-host-app/`)**
This is the central component coordinating user interactions, resource monitoring, and task routing.

- `homepage.py`: Streamlit frontend for user task selection and image uploads.
- `backend.py`: Python backend that:
  - Reads `choice.txt` to determine the target VM or GCP.
  - Sends image requests to the selected image processing API endpoint.
  - Uses hardcoded IPs:
    ```python
    VM1_IP = "192.168.56.101"
    VM2_IP = "192.168.56.103"
    ```
- `load_balancer.py`: Reads latest CPU and RAM values from the `vm_usage/` folder. If:
  - Any VM has < 40% load → writes `VM1` or `VM2` to `choice.txt`.
  - Both VMs exceed the threshold → writes `GCP` to `choice.txt`.
- `parallel_monitor.py`: Uses multithreaded sockets to poll `monitor.py` running on both VMs. Updates the following in real-time:
  - `vm_usage/vm1/cpu.txt` and `ram.txt`
  - `vm_usage/vm2/cpu.txt` and `ram.txt`
- `dashboard.py`: Displays live graphs 📊 of CPU and RAM usage using Streamlit.
- `vm_usage/`: Stores recent CPU and RAM usage metrics for each VM in `.txt` files.

---

### 2️⃣ **Linux Virtual Machines (`vm-backend/`)**
Each VM hosts the actual image processing services and exposes usage data to the host.

- `monitor.py`: Socket server that:
  - Listens for usage queries from the Windows host.
  - Responds with current RAM and CPU stats using `psutil`.
- 🐳 **Dockerized Microservices**:
  - `sketch-app/`: Sketch converter
  - `remove-bg/`: Background remover
  - `caption-service/`: Image captioning using BLIP
  - Each service contains:
    - `app.py`: Flask API server
    - `requirements.txt`: Dependencies
    - `Dockerfile`: For containerization

---

### 3️⃣ **Google Cloud Run (☁️ Fallback)**
When both VMs exceed the usage threshold, the host system routes tasks to identical microservices deployed as **auto-scaling containers** on **Google Cloud Run**. These services have the same API endpoints and behavior as the local ones.

---

## 🔄 System Workflow  
1. 🖼️ User opens the Streamlit UI (`homepage.py`) and uploads an image.
2. 🧠 `load_balancer.py` reads real-time metrics and writes a target to `choice.txt`.
3. 🚀 `backend.py` forwards the image to the selected API endpoint.
4. 🧪 The chosen service processes the image.
5. 📷 The output is displayed on the Streamlit frontend.
6. 📊 Admins or users monitor live system usage via `dashboard.py`.

---

## 🧰 Technologies Used  
- Python, Flask, Streamlit  
- Docker 🐳  
- Google Cloud Run ☁️  
- `psutil`, `socket`, `OpenCV`, `rembg`, `transformers` (Salesforce BLIP)

---

## ✅ Prerequisites  
- Python 3.8 or higher  
- Docker installed and configured  
- Google Cloud CLI authenticated  
- A VM environment (VirtualBox/VMware) with Ubuntu instances

---

## 📈 Outputs  
- Distributed, load-aware image processing across local and cloud resources  
- Real-time visualization of VM metrics  
- Fallback to cloud under heavy load  
- Accurate and responsive user experience under concurrency

---

## 🔮 Future Scope  
- 🔄 Horizontal scaling of VMs using infrastructure-as-code (Terraform, Ansible)  
- 🧠 ML-enhanced predictive load balancing based on task complexity  
- ☁️ Multi-cloud support to avoid vendor lock-in (AWS Lambda, Azure Container Apps)  
- 🔐 Secure API tokens and encryption for inter-process communication  

---

## 📽️ Demonstration Link  
- [Watch Demo](https://drive.google.com/file/d/1sRaDb7yy15HOlgxpVbJoUfwvv8cxEkjL/view?usp=sharing)
