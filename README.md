
# Ansible ile SSL Sertifikası Kurulumu

Bu dökümanda, Ansible kullanarak uzak sunuculara nasıl otomatik olarak SSL sertifikası kurulacağı adım adım anlatılmaktadır.

## 🔧 Ansible Kurulumu

Ubuntu sunucunuza Ansible kurmak için aşağıdaki komutu çalıştırın:

```bash
sudo apt install ansible
```

Kurulumdan sonra servisin durumunu kontrol etmek için:

```bash
systemctl status ansible
```

## 📁 inventory.ini Oluşturma

Sunucularınızı tanımlamak için bir `inventory.ini` dosyası oluşturmanız gerekir:

```ini
[test-server]
192.168.1.1
```

Sunucunuza özel SSH portu veya özel anahtar kullanıyorsanız:

```ini
[waf-server]
192.168.1.2 ansible_port=4478

[mail-server]
192.168.1.3 ansible_port=22 ansible_ssh_private_key_file=/root/private.key
```

## 📂 Proje Klasörü ve Playbook Dosyası

Önce bir klasör oluşturalım:

```bash
mkdir -p ssl-ansible
cd ssl-ansible
```

Sonrasında `ssl-install.yml` adında bir playbook dosyası oluşturarak şu içeriği ekleyin:

```yaml
- name: Install and Configure SSL with Certbot
  hosts: test-server
  become: yes
  vars:
    domain_name: test.com
    email: test@test.com

  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes

    - name: Install Certbot and nginx plugin
      ansible.builtin.apt:
        name:
          - certbot
          - python3-certbot-nginx
        state: present

    - name: Obtain SSL certificate using Certbot
      ansible.builtin.command:
        argv:
          - certbot
          - --nginx
          - -n
          - --agree-tos
          - --redirect
          - --email
          - "{{ email }}"
          - -d
          - "{{ domain_name }}"
      args:
        creates: "/etc/letsencrypt/live/{{ domain_name }}/fullchain.pem"
```

### Açıklamalar:

- `become: yes`: Root yetkisi gerektiren işlemler için.
- `vars`: Değişkenleri tanımlıyoruz (domain ve email).
- `state: present`: Paket zaten yüklü değilse yüklenmesini sağlar.
- `argv`: Terminalde çalışacak komutları listeler.
- `creates`: Eğer belirtilen dosya zaten varsa komut çalıştırılmaz.

## ✅ Çalıştırma

Playbook'u çalıştırmak için:

```bash
ansible-playbook -i inventory.ini ssl-install.yml
```

Bu playbook sayesinde bir veya birden fazla sunucuya otomatik olarak SSL sertifikası kurulabilir.

---

💡 Sorularınız için PR veya issue açabilirsiniz.
