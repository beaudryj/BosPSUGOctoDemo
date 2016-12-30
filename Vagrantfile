Vagrant.configure("2") do |config|
    config.vm.define "octo" do |octo|
      octo.vm.box = "jbeaudry/Octobox"
      octo.vm.network "forwarded_port", guest: 80, host:8080,
        auto_correct: true
      config.vm.network "private_network", ip: "192.168.245.10"
    end
    config.vm.define "web" do |web|
      web.vm.box = "MattHodge/Windows2016StdCore-WMF5-NOCM"
      web.vm.network "forwarded_port", guest: 80, host:8081,
        auto_correct: true
      config.vm.network "private_network", ip: "192.168.245.11"
    end
end
