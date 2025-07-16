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

    win10.vm.provision "reload"
    win10.vm.provision "shell", inline: "Get-NetConnectionProfile"
    win10.vm.provision "shell", inline: "Set-NetConnectionProfile -name \"Unidentified network\" -NetworkCategory Private"
    win10.vm.provision "shell", inline: "Set-NetConnectionProfile -name \"network\" -NetworkCategory Private"
    #win10.vm.provision "shell", type: "powershell", inline: "Get-NetConnectionProfile"
    #win10.vm.provision "shell", type: "powershell", inline: "Set-NetConnectionProfile -name \"Unidentified network\" -NetworkCategory Private"
    #win10.vm.provision "shell", type: "powershell", inline: "Set-NetConnectionProfile -name \"network\" -NetworkCategory Private"

    win10.vm.provision "shell", inline: "winrm quickconfig -q"
  end

  ############################
  # Orquestador Debian       #
  ############################

  # Maquina atacante/orquestador
  config.vm.define "orchestrator" do |orc|
    orc.vm.box = "debian/bookworm64"
    orc.vm.hostname = "orchestrator"
    orc.vm.network "private_network",
                   ip: "10.13.37.5",
                   netmask: "255.255.255.0"

    orc.vm.provider "virtualbox" do |vb|
      vb.name = "ADLab_orchestrator"
      vb.gui = false
      vb.cpus = 2
      vb.memory = 2048
    end

    # Aprovisionamiento
    orc.vm.provision "shell", privileged: true, inline: <<-SHELL
 
        echo "[orchestrator] Actualizando e instalando dependencias..."
        apt-get update -y
        apt-get install -y ansible git sshpass  # incluye python3-winrm
 
        echo "[orchestrator] Ejecutando playbook de Ansible como usuario vagrant..."
        # Ejecutamos el playbook como el usuario 'vagrant' para que tenga
        # acceso al directorio compartido /vagrant
        #sudo -u vagrant bash -c 'cd /vagrant/Ansible && ansible-playbook cloudlabs.yml'  #deprecado porque el directorio tiene permisos de escritura cuando se provisiona.
        #
 	sudo -u vagrant bash -c 'cd /vagrant/Ansible && ANSIBLE_CONFIG=ansible.cfg ansible-playbook -i inventory.yml cloudlabs.yml'
    SHELL
  end
end
