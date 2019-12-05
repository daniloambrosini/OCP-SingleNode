OC_DOMAIN = "oc-chewie.test"
OC_IP = "172.16.245.100"
OC_PUB_IP = "192.168.0.100"
OC_NAME = "oc-chewie"

BOX_IMAGE = "bento/ubuntu-16.04"
VERSION = "1.0"

Vagrant.configure("2") do |config|
  config.vm.box = BOX_IMAGE

  ENV['LC_ALL']="en_US.UTF-8"

  config.vm.define "#{OC_NAME}" do |node|

    node.vm.hostname = "#{OC_NAME}"
    
    # perhaps you will need to adapt the bridge, it's relate to your material
    # usually on OSX, its `bridge: "en0: Wi-Fi (AirPort)"`
    
    node.vm.network "public_network", ip:"#{OC_PUB_IP}",  bridge: "wlp58s0"
    node.vm.network "private_network", ip: "#{OC_IP}"

    node.vm.network "forwarded_port", guest: 8443, host: 8443

    node.vm.provider "virtualbox" do |vb|
      vb.memory = 10240
      vb.cpus = 4
      vb.name = "#{OC_NAME}"
    end

    node.vm.provision :docker

    node.vm.provision :shell, inline: <<-SHELL
      apt-get update
      apt-get install -y curl

      # openshift cluster
      cd /tmp/
      curl -Lso oc-cli.tgz  https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
      tar -xvzf oc-cli.tgz
      cp openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit/oc /usr/local/bin/
      cp openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit/kubectl /usr/local/bin/
      chmod +x /usr/local/bin/oc
      chmod +x /usr/local/bin/kubectl

      # Optionally add what you want:
      echo "" >> /etc/hosts
      echo '#{OC_PUB_IP} #{OC_DOMAIN}' >> /etc/hosts
      echo "" >> /etc/hosts

      # Add unsecure registry (this is a prerequisite - see the official documentation)
      echo "" >> /etc/docker/daemon.json
      echo '{' >> /etc/docker/daemon.json
      echo '  "insecure-registries" : ["172.30.0.0/16"]' >> /etc/docker/daemon.json
      echo '}' >> /etc/docker/daemon.json
      echo "" >> /etc/docker/daemon.json

      service docker restart

    SHELL

    # start the cluster
    node.vm.provision "start", type: "shell" , run: "always" , inline: <<-SHELL
      oc cluster up --public-hostname=#{OC_PUB_IP}
    SHELL

  end
end
