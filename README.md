# VagrantChallenge
Vagrant Challenge Python Flask app Ansible Gunicorn

## Vagrant Nedir?

Vagrant, Yazılım geliştirme projeleri için hızlı ve tekrarlanabilir bir şekilde sanal makineler oluşturulmasını sağlayan araçtır.


## Proje Adımları

1. GitHub'da repo oluşturuldu ve lokale clone edildi.
2. Lokalde ilgili klasörün içine gidildi:

    ```bash
    cd Desktop/VagrantChallenge/
    ```

3. Vagrant dosyası (`Vagrantfile`) oluşturuldu:

    ```bash
    vagrant init ubuntu/focal64
    ```
    vagrant init komutu, bulunduğunuz konumda örnek bir vagrantFile oluşturur. Bu örnekte, Ubuntu 20.04 LTS sanal makinesi kullanıldı.
    Bir diğer yöntem olarak Vagrantfile'ı kendiniz oluşturabilirsiniz, bunun için aşağıdaki komutları girebilirsiniz.

    ```ruby
    Vagrant.configure("2") do |config|
      config.vm.box = "ubuntu/focal64"
    end
    ```

## Vagrantfile Dosyası

```ruby
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/focal64"
  
  config.vm.network "public_network"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision.yml"
  end
  
end
```
DHCP den ip alabilmesi için sanal makinenin network ayarı public olarak yapıldı.

```ruby
config.vm.network "public_network"
```

Sanal makineye otomatik yapılandırma sağlamak için Ansible kullanıldı. ansible.playbook ifadesi ile hangi Ansible playbook'unun kullanılacağını belirtir. Bu örnekte, provision.yml isimli playbook belirtildi.

```ruby
config.vm.provision "ansible" do |ansible|
  ansible.playbook = "provision.yml"
end
```

## provision.yml dosyası

```yaml
- hosts: all
  become: yes
  tasks:
    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install Python 3 and pip
      apt:
        name: "{{ item }}"
        state: present
      with_items:
        - python3
        - python3-pip

    - name: Install Flask and Gunicorn
      pip:
        name: "{{ item }}"
        state: present
      with_items:
        - flask
        - gunicorn

    - name: Copy Flask application
      copy:
        src: ./app/
        dest: /home/vagrant/
```

Uygulama python üzerinden geliştirildiğinden ilk olarak çalıştırılacak olan makinede `apt` paket yöneticisi ile `python3` ve `pip` uygulamaları yüklendi. Projenin kullandığı kütüphane ve bağımlılıkları (`flask`, `gunicorn`) için `pip` ile yüklenmesi sağlandı. Son olarak `copy` komutu kullanılarak proje dosyalarının hedef makineye gönderildi.

Gunicorn, Python uygulamalarının web sunucusu olarak çalıştırılmasını sağlar ve bu durumda bir Flask uygulamasını çalıştırmak için kullanılmaktadır.

1. Sanal makineyi başlatmak için bu komut çalıştırılır

    ```bash
    vagrant up
    ```
    
    ![](./doc/vm.png)

2. Sanal makineye SSH ile bağlanmak için:

    ```bash
    vagrant ssh
    ```

3. Uygulama dizinine gidilerek, uygulama Gunicorn ile çalıştırılır.

   ```bash
   gunicorn --bind 0.0.0.0:8000 app:app
   ```
	--bind 0.0.0.0:8000 Bu, Gunicorn sunucusunun hangi IP adresi ve port üzerinde dinleyeceğini belirtir. app:app Bu, Gunicorn sunucusunun çalıştırılacak uygulamayı belirtir. 
    

4. Sanal makine ip'sini öğrenmek için `ifconfig` komutu kullanılır. İlgili ip ve port numarası web browser üzerinden girilerek test edilir.

    ![Hello World](./doc/app.png)

5. Sanal makineyi kapatmak için:

    ```bash
    vagrant halt
    ```

6. Sanal makinayı tamamen silmek için:

    ```bash
    vagrant destroy
    ```

