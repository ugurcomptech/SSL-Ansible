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


```

```







