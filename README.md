# Ansible ile Otomatik SSL Sertifikası Kurulumu

Bu döküman, Ansible kullanarak uzak sunucularınıza otomatik şekilde Let's Encrypt SSL sertifikası kurulumunu nasıl gerçekleştireceğinizi detaylı bir şekilde açıklar.

## Ansible Nedir?

Ansible; sistem yönetimini, konfigürasyon yönetimini ve uygulama dağıtımını kolaylaştıran açık kaynaklı bir otomasyon aracıdır. Agent (ajan) gerektirmeyen yapısıyla, SSH üzerinden sunuculara bağlanarak işlemleri gerçekleştirir. Özellikle birden fazla sunucuda aynı işlemi tekrarlamak isteyen sistem yöneticileri için oldukça idealdir.

## Ansible Kurulumu

Ubuntu sistemlerde aşağıdaki komut ile Ansible kurulabilir:

```bash
sudo apt update
sudo apt install ansible
```

Kurulum sonrası servisin çalıştığını kontrol etmek için:

```bash
ansible --version
```

## inventory.ini Dosyası Oluşturma

Ansible ile çalışırken, sunucularınızı tanımlamak için bir envanter (inventory) dosyasına ihtiyaç vardır. Bu dosya `.ini` formatında olabilir ve örnek olarak aşağıdaki gibi yapılandırılabilir:

```ini
[test-server]
192.168.1.10

[waf-server]
192.168.1.2 ansible_port=4478

[mail-server]
162.168.1.3 ansible_port=22 ansible_ssh_private_key_file=/root/private.key
```

- `ansible_port`: Sunucu farklı bir SSH portu kullanıyorsa burada belirtilir.
- `ansible_ssh_private_key_file`: Oturum açmak için özel anahtar kullanılıyorsa bu parametreyle yol belirtilmelidir.

## Çalışma Dizini ve Dosya Yapısı

Proje yapınızı düzenli tutmak için aşağıdaki gibi bir dizin oluşturabilirsiniz:

```bash
mkdir -p ssl-ansible
cd ssl-ansible
```

## Playbook: SSL Kurulumu

Aşağıda Ansible ile Let's Encrypt SSL sertifikası kurulumunu yapan bir `ssl-setup.yml` örneği verilmiştir:

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
```

### Açıklama:

Bu blok, APT paket yöneticisinin önbelleğini güncelleyerek sistemin en güncel paket listelerine sahip olmasını sağlar.

```yaml
    - name: Install Certbot and nginx plugin
      ansible.builtin.apt:
        name:
          - certbot
          - python3-certbot-nginx
        state: present
```

### Açıklama:

Bu adım, SSL sertifikası almak için kullanılan `certbot` ve nginx web sunucusu ile entegre çalışmasını sağlayan eklentisini kurar. `state: present` parametresi, bu paketlerin sistemde yoksa kurulmasını, varsa işlem yapılmamasını sağlar.

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

### Açıklama:

Bu adımda `certbot` komutu çalıştırılır. Parametrelerin açıklaması:

- `--nginx`: Nginx için yapılandırma yapılacak.
- `-n`: Etkileşimsiz mod, kullanıcıdan giriş beklenmez.
- `--agree-tos`: Let's Encrypt hizmet şartları otomatik kabul edilir.
- `--redirect`: HTTP'den HTTPS'e yönlendirme yapılır.
- `--email`: Sertifika bildirimleri için kullanılacak e-posta adresi.
- `-d`: Sertifika alınacak domain adı.

`creates` argümanı sayesinde bu dosya zaten varsa bu adım atlanır.

## Sonuç

Bu yapı sayesinde bir veya birden fazla sunucuya tek bir komutla SSL kurulumu yapabilirsiniz. Özellikle aynı domain yapısında çalışan sunucular için ideal ve zaman kazandıran bir çözümdür.

---

**Not:** Bu playbook yalnızca Nginx web sunucusu için örneklendirilmiştir. Apache gibi farklı sunucular için `certbot`'un uygun plugin'i yüklenmeli ve komutlar değiştirilmelidir.
