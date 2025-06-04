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
        # ��������� ����������� �� ������
        sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
        systemctl restart sshd.service

        # ������� ������������� otusadm � otus
        useradd otusadm
        useradd otus

        # ������������� ������ ��� �������������
        echo "otusadm:Otus2022!" | chpasswd
        echo "otus:Otus2022!" | chpasswd

        # ������� ������ admin
        groupadd -f admin

        # ��������� ������������� � ������ admin
        usermod -a -G admin otusadm
        usermod -a -G admin root
        usermod -a -G admin vagrant

        # ������� ������ �������� ������� /usr/local/bin/login.sh
        cat << 'EOF' > /usr/local/bin/login.sh
#!/bin/bash

# ���������, �������� �� ����
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
  # ���������, ������ �� ������������ � ������ admin
  if getent group admin | grep -qw "$PAM_USER"; then
    exit 0
  else
    exit 1
  fi
fi

# ���� ���� �� ��������, ��������� ������
exit 0
EOF

        # ������ ���� �����������
        chmod +x /usr/local/bin/login.sh

        # ��������� pam_exec � ���� /etc/pam.d/sshd
        echo 'auth required pam_exec.so debug /usr/local/bin/login.sh' | tee -a /etc/pam.d/sshd

        # ������������� ���� �� 16 ������ 2014 ����, 11:00
        date -s "2014-11-16 11:00:00"

        # ������������� Docker
        apt update && apt install -y docker.io

        # ��������� ������������ otus � ������ docker
        usermod -aG docker otus

        # ��������� ������������ otus ������������� Docker-������
        echo 'otus ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart docker' | tee /etc/sudoers.d/docker
      SHELL
    end
  end
end