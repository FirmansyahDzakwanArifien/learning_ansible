# Panduan Deployment Node.js Application dengan Ansible

# Sebelumnya pastikan node.js sudah terinstall di managed node

## 1. Struktur File Aplikasi

### app.js

```javascript
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;
const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hi Everyone, Dev Ops Group 1 NFA Batch 3\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

### package.json

```json
{
    "name": "my-node-app",
    "version": "1.0.0",
    "description": "A simple Node.js Practice application",
    "main": "app.js",
    "scripts": {
        "start": "node app.js",
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "Your Name",
    "license": "ISC",
    "dependencies": {}
}
```

---

## 2. Inventory Ansible

Buat file `inventory.ini` untuk mendefinisikan server target (misalnya bernama group `NFA`):

```ini
[nama-hosts]
ip_managed_node ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa=host
```

> Ganti `192.168.1.100` dengan IP server target dan sesuaikan dengan user serta key SSH yang kamu gunakan.

---

## 3. Playbook Ansible

Buat file `deploy_nodejs_app.yaml` seperti berikut:

```yaml
- name: Deploy Node.js Application
  hosts: NFA
  tasks:
    - name: Install Node.js & npm
      apt:
        name: 
          - nodejs
          - npm
        state: present
      become: yes

    - name: Copy App Files
      copy:
        src: /path/to/local/app/app.js
        dest: /var/www/myapp/app.js
      become: yes

    - name: Copy Package.json
      copy:
        src: /path/to/local/app/package.json
        dest: /var/www/myapp/package.json
      become: yes

    - name: Install App Dependencies
      npm:
        path: /var/www/myapp
        state: present
      become: yes

    - name: Start Application
      command: node /var/www/myapp/app.js
      async: 10
      poll: 0
```

---

## 4. Catatan Penting

- **Path Lokal (`src`)**  
  Sesuaikan dengan lokasi file aplikasi di Ansible controller.  
  Contoh: `/home/user/projects/myapp/app.js`

- **Path Remote (`dest`)**  
  Sesuaikan dengan lokasi target di server managed node.  
  Contoh: `/var/www/myapp/app.js`

- **Menjalankan Playbook**  
  Gunakan perintah berikut untuk menjalankan playbook:  
  ```bash
  ansible-playbook -i inventory.ini deploy_nodejs_app.yaml
  ```

- **Verifikasi**  
  Setelah playbook selesai, cek aplikasi dengan membuka browser atau curl:  
  ```bash
  curl http://<server-ip>:3000
  ```

Jika berhasil, output yang muncul:  
```
Hi Everyone, Dev Ops Group 1 NFA Batch 3
```
