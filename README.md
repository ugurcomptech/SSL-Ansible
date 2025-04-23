# Ansible ile SSL Sertifikası Kurulumu

Bu döküman, Ansible aracı ile belirlemiş olduğunuz sunuculara nasıl otomatik olarak SSL sertifikası kurulabileceğini adım adım anlatmaktadır.

## Ansible Playbook Açıklamaları

### 🔹 Genel Başlık

```yaml
- name: Install and Configure SSL with Certbot
  hosts: test-server
  become: yes
```

**Açıklama:**
- `name`: Playbook'un başlığıdır.
- `hosts`: `inventory.ini` dosyasındaki grup adıdır.
- `become: yes`: Root yetkisi gerektiren işlemler için sudo kullanılır.

---

### 🔹 Değişken Tanımlamaları

```yaml
  vars:
    domain_name: test.com
    email: test@test.com
```

**Açıklama:**
- SSL kurulumu için gerekli domain ve e-posta değişkenleri tanımlanır.
- İleride komutlarda `{{ domain_name }}` ve `{{ email }}` olarak kullanılırlar.

---

### 🔹 APT Cache Güncelleme

```yaml
  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes
```

**Açıklama:**
- Sunucunun paket listesi güncellenir.
- Bu işlem, sistemin en güncel paket bilgisiyle çalışmasını sağlar.

---

### 🔹 Certbot ve Nginx Plugin Kurulumu

```yaml
    - name: Install Certbot and nginx plugin
      ansible.builtin.apt:
        name:
          - certbot
          - python3-certbot-nginx
        state: present
```

**Açıklama:**
- `certbot`: SSL sertifikası almak için kullanılır.
- `python3-certbot-nginx`: Nginx ile Certbot'u entegre eder.
- `state: present`: Paketler eksikse yüklenir, zaten varsa işlem yapılmaz.

---

### 🔹 SSL Sertifikası Alınması

```yaml
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

**Açıklama:**
- Certbot komutu çalıştırılır.
- `--nginx`: Nginx yapılandırması kullanılır.
- `--redirect`: HTTP'den HTTPS'e yönlendirme yapılır.
- `--agree-tos`: Kullanım şartları kabul edilir.
- `creates`: Bu dosya varsa komut tekrar çalışmaz.

---

## Örnek `inventory.ini`

```ini
[test-server]
192.168.1.10 ansible_port=22 ansible_ssh_private_key_file=/root/private.key
```

Bu dosyada sunucularınız tanımlanır ve `hosts:` parametresiyle eşleşir.

---

Bu yapı sayesinde birden fazla sunucuya otomatik SSL kurulumu gerçekleştirebilirsiniz.
