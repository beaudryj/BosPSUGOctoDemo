Vagrant.configure("2") do |config|
    config.vm.define "octo" do |octo|
      octo.vm.box = "jbeaudry/Octobox"
      octo.vm.network "forwarded_port", guest: 80, host:8080
    end
    config.vm.define "web" do |web|
      web.vm.box = "MattHodge/Windows2012R2Core-WMF5-NOCM"
      web.vm.network "forwarded_port", guest: 81, host:8081
    end
end
