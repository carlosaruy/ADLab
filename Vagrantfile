# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Tiempo extra para el arranque de Windows (instalación de drivers, etc.)
  config.vm.boot_timeout = 600

  ############################
  # Guests Windows           #
  ############################

  # --- Controlador de Dominio (Windows Server 2019) ---
  config.vm.define "dc" do |dc|
    dc.vm.communicator = "winrm"
    dc.winrm.timeout   = 600
    dc.winrm.username  = "vagrant"
    dc.winrm.password  = "vagrant"

    dc.vm.box       = "gusztavvargadr/windows-server-2019-standard"
    dc.vm.hostname  = "dc"
    dc.vm.network "private_network",
                  ip: "10.13.37.10",
                  netmask: "255.255.255.0",
                  nic_type: "82540EM"

    dc.vm.provider "virtualbox" do |v|
      v.gui    = true
      v.memory = 2048
      v.cpus   = 2
    end

    # Reinicio limpio SOLO la primera vez → evita loop infinito
    dc.vm.provision "reload", run: "once", provision: true
    dc.vm.provision "shell", inline: "winrm quickconfig -q"
  end

  # --- Servidor secundario (Windows Server 2019) ---
  config.vm.define "winserv2019" do |web|
    web.vm.communicator = "winrm"
    web.winrm.timeout   = 600
    web.winrm.username  = "vagrant"
    web.winrm.password  = "vagrant"

    web.vm.box      = "gusztavvargadr/windows-server-2019-standard"
    web.vm.hostname = "winserv2019"
    web.vm.network "private_network",
                  ip: "10.13.37.100",
                  netmask: "255.255.255.0",
                  nic_type: "82540EM"

    web.vm.provider "virtualbox" do |v|
      v.gui    = true
      v.memory = 2048
      v.cpus   = 1
    end

    # Un solo reload para estabilizar la red
    web.vm.provision "reload", run: "once", provision: true
    web.vm.provision "shell", inline: "winrm quickconfig -q"
  end

  # --- Estación de trabajo (Windows 10) ---
  config.vm.define "win10" do |win10|
    win10.vm.communicator = "winrm"
    win10.winrm.timeout   = 600
    win10.winrm.username  = "vagrant"
    win10.winrm.password  = "vagrant"

    win10.vm.box      = "gusztavvargadr/windows-10"
    win10.vm.hostname = "win10"
    win10.vm.network "private_network",
                     ip: "10.13.37.150",
                     netmask: "255.255.255.0",
                     nic_type: "82540EM"

    win10.vm.provider "virtualbox" do |v|
      v.gui    = true
      v.memory = 2048
      v.cpus   = 2
    end

    # Sin reload automático para evitar perder la configuración de red
    win10.vm.provision "shell", inline: "winrm quickconfig -q"
  end

  ############################
  # Orquestador Debian       #
  ############################

  config.vm.define "orchestrator" do |orc|
    orc.vm.box      = "debian/buster64"
    orc.vm.hostname = "orchestrator"
    orc.vm.network "private_network",
                   ip: "10.13.37.5",
                   netmask: "255.255.255.0"

    # Monta el proyecto completo para playbooks, scripts, etc.
    orc.vm.synced_folder ".", "/vagrant", disabled: false

    # Aprovisionamiento: instala Ansible y dependencias para gestionar WinRM
    orc.vm.provision "shell", inline: <<-SHELL
      set -e
      echo "[orchestrator] Actualizando mirrors y paquetes …"
      cat > /etc/apt/sources.list <<EOF
      deb https://deb.debian.org/debian buster main
      deb https://security.debian.org/debian-security buster/updates main
      deb https://deb.debian.org/debian buster-updates main
      EOF
      apt-get update -y
      apt-get install -y python3-pip git sshpass
      pip3 install --upgrade ansible 'pywinrm>=0.3.0'
    SHELL

    # Ejecuta playbooks cada vez que arranca la VM
    orc.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "[orchestrator] Ejecutando playbook de Ansible…"
      cd /vagrant/Ansible
      ansible-playbook -i inventory.yml cloudlabs.yml || true
    SHELL
  end
end
