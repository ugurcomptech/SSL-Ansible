#  Ansible ile SSL Kurulumu

Bu döküman da ansible aracı ile nasıl belirlemiş olduğunuz sunuculara SSL sertifikası kurabileceğinizi anlatmaktadır. Gerekli adımları aşağıda bulabilirsiniz.


## Ansible Download

Aşağıdaki komutu yazarak Ubuntu sunucumuza ansibleyi indiriyoruz.

```
sudo apt install ansible
```

Ansible indirdikten sonra servisimizi  `systemctl status ansible` ile kontrol edelim.



## inventory.ini

Sunucularımızı statik olarak belirtmemiz için bir .ini dosyasına ihtiyacımız bulunmaktadır. Bu dosyaya istediğiniz ismi verebilirsiniz.

```
[test-server]
192.168.1.1
```


Burada yazmış olduğumuz `test-server` .yml dosyasında kullanacağımız hosttur. O yüzden burada isim belirlerken aklınızda kalabilecek veya kullanmış olduğunuz servera uygun bir isim vermeniz daha sağlıklı olacaktır.

**Not:** *Eğer sunucunuzda bir ssh-key veya ssh portunuz farklı ise .ini dosyanızda bunu da belirtmeniz gerekmektedir. Örnekleri aşağıda bulabilirsiniz.*


```
[waf-server]
192.168.1.2 ansible_port=4478

[mail-server]
162.168.1.3 ansible_port=22 ansible_ssh_private_key_file=/root/private.key
```


## Yml Dosyası Oluşturma

Ansible ile çalışırken klasör olarak farklı farklı ayırmak her zaman daha sağlıklıdır.

Çalışma ortamımız için klasörümüzü oluşturalım:

```
mkdir -p ssl-ansible
```

.yml dosyamızı oluştururken isimlendirmek önemlidir. Mutlaka aklınızda kalabilecek veya kullanmış olduğunuz yapılara uygun bir isim veriniz.

**Not:** *Scriptin tamamını repo üzerinde bulabilirsiniz. Bu dökümanda  .yml dosyasında bulunan taskları adım adım açıklayarak gideceğiz.*


Burada belirtmiş olduğumuz `hosts` .ini dosyamızda bulunan sunucumuzdur. `become` komutu ise yapacağımız işlemlerde eğer root yetkisi gerektiriyorsa buna izin vermektedir.

```
- name: Install and Configure SSL with Certbot
  hosts: test-server
  become: yes
```

İleri ki süreçlerde domain ve e-mail kısmını değiştirmek isterseniz script üzerinde ilgili yerleri aramamak için değişkenler tanımlıyoruz.

```
  vars:
    domain_name: test.com
    email: test@test.com  
```

Gerekli işlemlerden önce sistemi update ediyoruz.

```
  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes
```

Buradaki işlemde ise cerbotu ve web serverımıze uygun plugini indiriyoruz. Eğer birden fazla servis ismi belirticekseniz aşağıdaki gibi madde madde olmalıdır. Belirtmiş olduğumuz `present` değeri ise sistemde eğer cerbot veya nginx-cerbot bulunmuyor ise yüklemesini ifade eder. Zaten bu servisler var ise hiçbir işlem sağlamaz.

```
    - name: Install Certbot and nginx plugin
      ansible.builtin.apt:
        name:
          - certbot
          - python3-certbot-nginx
        state: present
```

 
Son işlemimizde ise `ansible` ait olan `ansible.builtin.command` modülünü kullanıyoruz. Bu modül sayesinde terminal üzerinde belirteceğimiz kodlar çalışacaktır. Argv ise bu komutları belirtmemize yaramaktadır. 

Buradaki değişkenler ise daha önceden belirtmiş olduğumuz domain adresimiz ve e-mail adresimizdir.

```
 - "{{ email }}"
 - "{{ domain_name }}"
```

İlgili SSL sertifikasını gerekli yere oluşturur.

```
creates: "/etc/letsencrypt/live/{{ domain_name }}/fullchain.pem"
```

```
    - name: Obtain SSL certificate using Certbot
      ansible.builtin.command:# Ansible ile SSL Sertifikası Kurulumu

Bu dökümanda, Ansible aracı ile bir veya birden fazla uzak sunucuya SSL sertifikası kurulumunun nasıl yapılacağını adım adım öğrenebilirsiniz.

---

## 📦 Ansible Kurulumu

Ubuntu sunucunuza Ansible kurmak için aşağıdaki komutu kullanın:

```bash
sudo apt update && sudo apt install ansible -y
```

Kurulumdan sonra aşağıdaki komutla servisin çalıştığını kontrol edebilirsiniz:

```bash
ansible --version
```

---

## 🗂️ inventory.ini Dosyası

Sunucularınızı Ansible’a tanıtmak için bir envanter dosyası (`inventory.ini`) oluşturmalısınız:

```ini
[test-server]
192.168.1.1
```

Grup adı (`test-server`) `.yml` dosyasında kullanılacak olan host ismidir. Daha karmaşık bağlantılar için:

```ini
[waf-server]
192.168.1.2 ansible_port=4478

[mail-server]
192.168.1.3 ansible_port=22 ansible_ssh_private_key_file=/root/private.key
```

---

## 📁 Proje Dizini ve Playbook Oluşturma

Çalışma dizininizi oluşturun:

```bash
mkdir -p ssl-ansible
cd ssl-ansible
```

Playbook dosyasını oluşturun (örneğin `ssl-hook.yml`). İçeriği aşağıdaki gibi olabilir:

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

    - name: Install Certbot and Nginx plugin
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

### Açıklamalar

- `become: yes` → root yetkisi gerektiren işlemler için kullanılır.
- `vars` → domain ve e-posta gibi değişkenleri merkezi şekilde tanımlamanızı sağlar.
- `apt` modülü ile Certbot ve Nginx plugin yüklenir.
- `state: present` → eğer paketler yüklü değilse yükler, yüklüyse işlem yapmaz.
- `command` modülü ile doğrudan terminal komutları çalıştırılır.
- `creates` → Eğer sertifika dosyası varsa, komut yeniden çalıştırılmaz.

---

Bu yapı sayesinde aynı anda birden fazla sunucuya otomatik şekilde SSL sertifikası kurulabilir.

---

📌 Daha fazla bilgi ve katkı için lütfen bu repoyu forkladıktan sonra PR gönderin.
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
Bu script sayesinde dilerseniz birden fazla sunucuya SSL sertifikası kurabilirsiniz. 






