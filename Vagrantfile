MACHINES = {
  :"pam" => {
    :box_name => "ubuntu/jammy64",
    :cpus => 2,
    :memory => 1024,
    :ip => "192.168.57.10",
  }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.network "private_network", ip: boxconfig[:ip]
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
      box.vm.provision "shell", inline: <<-SHELL
        # Разрешаем подключение по паролю
        sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
        systemctl restart sshd.service

        # Создаем пользователей otusadm и otus
        useradd otusadm
        useradd otus

        # Устанавливаем пароли для пользователей
        echo "otusadm:Otus2022!" | chpasswd
        echo "otus:Otus2022!" | chpasswd

        # Создаем группу admin
        groupadd -f admin

        # Добавляем пользователей в группу admin
        usermod -a -G admin otusadm
        usermod -a -G admin root
        usermod -a -G admin vagrant

        # Создаем скрипт контроля доступа /usr/local/bin/login.sh
        cat << 'EOF' > /usr/local/bin/login.sh
#!/bin/bash

# Проверяем, выходной ли день
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
  # Проверяем, входит ли пользователь в группу admin
  if getent group admin | grep -qw "$PAM_USER"; then
    exit 0
  else
    exit 1
  fi
fi

# Если день не выходной, разрешаем доступ
exit 0
EOF

        # Делаем файл исполняемым
        chmod +x /usr/local/bin/login.sh

        # Добавляем pam_exec в файл /etc/pam.d/sshd
        echo 'auth required pam_exec.so debug /usr/local/bin/login.sh' | tee -a /etc/pam.d/sshd

        # Устанавливаем дату на 16 ноября 2014 года, 11:00
        date -s "2014-11-16 11:00:00"

        # Устанавливаем Docker
        apt update && apt install -y docker.io

        # Добавляем пользователя otus в группу docker
        usermod -aG docker otus

        # Разрешаем пользователю otus перезапускать Docker-сервис
        echo 'otus ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart docker' | tee /etc/sudoers.d/docker
      SHELL
    end
  end
end