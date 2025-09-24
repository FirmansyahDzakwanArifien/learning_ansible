# Practicum 4 - Container Deployment with Docker using Ansible

## Objective
- Learn how to install Docker automatically using Ansible.  
- Deploy and manage Docker containers using an Ansible playbook.  

---

## 1. Playbook for Docker Installation

File name: **`install_docker.yaml`**

### Task Explanations
1. **Update the repository cache**  
   Ensures the system has the latest package information.  
   ```yaml
   - name: Update apt cache
     apt:
       update_cache: yes

2. **Install required dependencies**
Required packages to enable HTTPS downloads.

- name: Install required dependencies
  apt:
    name:
      - apt-transport-https
      - ca-certificates
      - curl
      - software-properties-common
    state: present

3. **Add Docker’s GPG key**
Ensures package authenticity.

- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present


4. **Add the Docker repository**
Provides the official Docker source

- name: Add Docker repository
  apt_repository:
    repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
    state: present


5. **Install Docker Community Edition**
Installs Docker CE

- name: Install Docker CE
  apt:
    name: docker-ce
    state: latest


6. **Enable and start Docker service**
Makes Docker run automatically on boot

- name: Enable and start Docker
  service:
    name: docker
    state: started
    enabled: yes

## 2. Playbook for Container Management

File name: **`manage_container.yaml`**

### Task Explanations

1. **Create a Docker network**

- name: Create Docker network
  docker_network:
    name: my_network
    state: present

2. **Run a web server container (nginx)**

- name: Run nginx web server
  docker_container:
    name: web_server
    image: nginx
    state: started
    networks:
      - name: my_network
    ports:
      - "80:80"

3. **Run a database container (MySQL 5.7)**

- name: Run MySQL database
  docker_container:
    name: db_server
    image: mysql:5.7
    state: started
    networks:
      - name: my_network
    env:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    ports:
      - "3306:3306"

4. **Remove an old container if exists**

- name: Remove old db_server container if exists
  docker_container:
    name: db_server
    state: absent

## 3. Running the Playbooks

1. **Install Docker:**

ansible-playbook -i hosts install_docker.yaml

2. **Manage containers:**

ansible-playbook -i hosts manage_container.yaml

3. **Verify running containers:**

docker ps

4. **Verify Docker networks:**

docker network ls
