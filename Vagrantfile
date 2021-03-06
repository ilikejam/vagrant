# Master Vagrantfile
# Install dummy vagrant box:
# vagrant box add http://vdcpuppet/pub/vagrant/vrealize.box --name vrealize

email = 'chunkylover53@aol.com'

Vagrant.configure("2") do |config|

# ---------------------------------- Defaults ----------------------------------

  config.vm.provision "shell", inline: "echo 'Welcome to VDC Vagrant'"

  config.vm.provider :vrealize do |vrealize|
    vrealize.vra_username  =  ENV['USER']
    vrealize.vra_password  =  ENV['PASS']
    vrealize.vra_tenant    = 'vsphere.local'
    vrealize.vra_base_url  = 'https://e2portal'
    vrealize.requested_for =  ENV['USER'] + '@bskyb.com'
    vrealize.subtenant_id  = 'f04f060d-73be-48a3-b82c-20cb98efd2d2'
    vrealize.cpus          = 1
    vrealize.memory        = 1024
    vrealize.lease_days    = 5
  end

  config.vm.provider :virtualbox do |vbox|
    vbox.customize [ "modifyvm", :id, "--cpus", "1" ]
    vbox.customize [ "modifyvm", :id, "--memory", "2048" ]
    vbox.customize [ "modifyvm", :id, "--nictype1", "virtio" ]
    vbox.customize [ "modifyvm", :id, "--nictype2", "virtio" ]
    vbox.gui = true
  end

# ------------------------------------ VDC -------------------------------------

  config.vm.define "vdc-centos6" do |node|
    node.vm.provider "vrealize" do |v|
      v.catalog_item_id = '3bd374bc-ca1a-4b81-b3a0-0665cca35355'
    end
    node.vm.box = "vrealize"
  end

  config.vm.define "vdc-centos7" do |node|
    node.vm.provider "vrealize" do |v|
      v.catalog_item_id = 'fa58e74c-825e-418d-9c7a-e0cf9ef703c9'
    end
    node.vm.box = "vrealize"
  end

  config.vm.define "vdc-ubuntu14" do |node|
    node.vm.provider "vrealize" do |v|
      v.catalog_item_id = '6720b624-3ff4-473b-91cd-769e9dd7771d'
    end
    node.vm.box = "vrealize"
  end

  config.vm.define "vdc-ubuntu15" do |node|
    node.vm.provider "vrealize" do |v|
      v.catalog_item_id = '5d4f3ef2-ad41-4f68-b5f7-ba5ba946ee3d'
    end
    node.vm.box = "vrealize"
  end

  config.vm.define "vdc-ubuntu16" do |node|
    node.vm.provider "vrealize" do |v|
      v.catalog_item_id = 'a0e65937-bbbd-4228-8c36-280c21cf4f4e'
    end
    node.vm.box = "vrealize"
  end

  config.vm.define "build-govc" do |node|
    # Centos7, 4 gig big for go compiler
    node.vm.provider "vrealize" do |v|
      v.catalog_item_id = 'fa58e74c-825e-418d-9c7a-e0cf9ef703c9'
      v.memory          = 4096
    end
    $govcscript = <<-_EOF
      yum -y install mutt wget git
      wget https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz
      tar xzvf go1.6.2.linux-amd64.tar.gz
      export GOROOT=`pwd`/go
      export GOPATH=`pwd`/mygo
      export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
      mkdir -p $GOPATH/src/github.com/vmware
      cd $GOPATH/src/github.com/vmware
      git clone https://github.com/ilikejam/govmomi.git
      cd govmomi
      make
      cd govc
      go build -o govc main.go
      bzip2 govc
      echo "govc build artefacts attached" | mutt -a govc.bz2 -s "govc build artefacts" -- #{email}
      _EOF
    node.vm.provision "shell", inline: $govcscript
    node.vm.synced_folder ".", "/vagrant", disabled: true
    node.vm.box = "vrealize"
  end

  config.vm.define "quick" do |node|
    node.vm.provider "vrealize" do |v|
      v.catalog_item_id = 'fa58e74c-825e-418d-9c7a-e0cf9ef703c9'
    end
    node.vm.box = "vrealize"
    node.vm.synced_folder ".", "/vagrant", disabled: true
  end

# --------------------------------- Virtualbox ---------------------------------

$networkscript = <<-_EOF
  ip route add 10.105.16.253/32 via `ip route | grep eth1 | grep via | awk '{print $3}' | head -1` dev eth1; true
  ip route add 10.105.16.64/32  via `ip route | grep eth1 | grep via | awk '{print $3}' | head -1` dev eth1; true
  _EOF
$puppetlocalscript = <<-_EOF
  cp -f /vagrant/puppet.conf /etc/puppet/puppet.conf
  cp /vagrant/csr_attributes.yaml /etc/puppet/csr_attributes.yaml
  _EOF
$cleanscript = <<-_EOF
  subscription-manager unregister; true
  rm -f /etc/facter/facts.d/katello*
  rpm -qa | grep katello | xargs rpm -e; true
  _EOF

  config.vm.define "moduletest7" do |node|
    node.vm.box = "bskyb_base_rhel7.1_v4"
    node.vm.box_url = "http://vdcpuppet/pub/vagrant/bskyb_base_rhel7.1_v3.box"
    node.vm.synced_folder "hieradata/", "/etc/puppet/hieradata"
    node.vm.hostname = "vagrant.bskyb.com"
    node.ssh.insert_key = false
    node.vm.provision "shell", inline: $networkscript
    node.vm.provision "shell", inline: $puppetlocalscript
    node.vm.provision "puppet" do |puppet|
      puppet.options           = [ "--evaltrace --debug --trace --verbose --disable_warnings=deprecations --graph" ]
      puppet.manifests_path    = "manifests"
      puppet.manifest_file     = "test.pp"
      puppet.module_path       = [ "modules" ]
      puppet.hiera_config_path = "hieradata/hiera.yaml"
    end
    node.vm.provision "shell", inline: $cleanscript
    node.vm.network "public_network"
  end

  config.vm.define "moduletest6" do |node|
    node.vm.box = "bskyb_base_rhel6.7_v0"
    node.vm.box_url = "http://vdcpuppet/pub/vagrant/bskyb_base_rhel6.7_v0.box"
    node.vm.synced_folder "hieradata/", "/etc/puppet/hieradata"
    node.vm.hostname = "vagrant.bskyb.com"
    node.ssh.insert_key = false
    node.vm.provision "shell", inline: $networkscript
    node.vm.provision "shell", inline: $puppetlocalscript
    node.vm.provision "puppet" do |puppet|
      puppet.options           = [ "--evaltrace --debug --trace --verbose --disable_warnings=deprecations --graph" ]
      puppet.manifests_path    = "manifests"
      puppet.manifest_file     = "test.pp"
      puppet.module_path       = [ "modules" ]
      puppet.hiera_config_path = "hieradata/hiera.yaml"
    end
    node.vm.provision "shell", inline: $cleanscript
    node.vm.network "public_network"
  end
end

# ----------------------------------- Certs ------------------------------------

OpenSSL::SSL::SSLContext::DEFAULT_CERT_STORE.add_cert(OpenSSL::X509::Certificate.new(<<-CERT))
-----BEGIN CERTIFICATE-----
MIIFCzCCAvOgAwIBAgIQCtdItpDe64pCJPBnQtRFnDANBgkqhkiG9w0BAQUFADAY
MRYwFAYDVQQDEw1CU2t5QiBSb290IENBMB4XDTA3MTExNjE0MTIwNVoXDTI3MTEx
NjE0MTgxMlowGDEWMBQGA1UEAxMNQlNreUIgUm9vdCBDQTCCAiIwDQYJKoZIhvcN
AQEBBQADggIPADCCAgoCggIBAKMJcMkR7W150rIvCVlRGQ8QtAJF2oFo8KASIvjq
eAy2kc7K9jFUBj4Q2SfTmYNNbKdiJiwOifuatyBnAxnPX3GyEdYCeH3QS2aDfH62
Rq/o234nagEqE2e80WExVKa6uX7XI17eDggFdPuCg1Iznozyf8K0UzFlsCvanHA1
WEB9XNJtYB7cnZMHfRNzTj/X0rneCcAoHMaIcU9lU0O2/n7n1v7fDBtZYpusbKBy
Mq8n6WSj/ToHzBN+kUbodTRG8SNib8+W51wYdX10DdYhRaUNISp9rg99a9TA4bto
85k8UUDrtF2jFC/tSNjTfaIB0EulCx/lheSHi6HIxo4ULjd26+OukooO0aiLhqAl
gFEEK+Ro0JcTbPJoqMx9CF1eBlsX2+zT14oFInIBYDME/kUKtY7M/Vw67Li5ZZBz
Tg9iLzoV/O0fbXQYmkYq7+B7y7gdaVtMubjjHqtR6JXQSgT5uVXw4Xy8kRdS8wYk
nQtEBnGLiFKGCi0IOdxi3acRZ7dX7iWgeW8IehYK90uyAWdhCZtLwEbTGFBJz+fF
GKNkypefHBlbgVA644oFSqUYIH2xm7+ZVnMG+DZt39VMqcK+xSRVSTXOpka2uTXB
WiTvk/zhJDb626KZNw0BWr43S5Hq7eVo1VZLsEQDFE7LYa+bMyAuXbcMEoxNfoJ0
PWoBAgMBAAGjUTBPMAsGA1UdDwQEAwIBhjAPBgNVHRMBAf8EBTADAQH/MB0GA1Ud
DgQWBBSfPmNTKs0OcGbbq2od0irvMY+8ljAQBgkrBgEEAYI3FQEEAwIBADANBgkq
hkiG9w0BAQUFAAOCAgEAhkyOviEdjHDO/yZ+kkHORgCcha7B6QLJdsU7hJT6dPcB
nIS3aCc/w10nD/H4sRJkhb8A4N1oEw11iboIhrnE4CnxhutHcGLdCG6mRkjVG7jZ
UN5R95WKU5VMhBmCvXvXXoQyeR9r+2ToedcZAuot+X8a33rH2OKRi1EnqFb5IyXx
PAOg2xOAhfkP9wGtQZT6QCH32OPILglWDzQfmgMcBR4sp/EYHjdlxGjX9nUC7j+9
CHige18w4hg05v0wS/WDVYtyXrs6SPJJ5dQ4wRBM+UXOOjPuwoJAdhkreGlVX2jH
UCa/c3m9vkkTRgXnhmf/62DGP95q2R1w+57pcT4K/SbXsUZNWfAyZSc1B//edUVw
/HcIX8KSjrOJPvQJ8VxEGAxMzZdR16Y2fbs+f417bkWhnGtuu67AkwUVy3fX/4t5
/zY8qG4YmKxsPlDpw5li6oTeCQ6OAA6AAWApjtpp8OP9V0yAyTz5gK6nFse6LRre
62QUMK8fbk4vcaHNhDUGnfecL3ajR9h1GD6spP9LfXWnxlpD7L2XNvUBNKDjLSjn
wMUEhEnggyVhp1bob+I641w/ZltPMqfqyn7fDGPjRep+YDAKNug7MYiJB0Q51QkW
eK2AVpl7g3BFaZW16H6LyaO3OfHgjsrh8zNhd5IB5vv7fC6QKMb6ZeGzC2263wA=
-----END CERTIFICATE-----
CERT
