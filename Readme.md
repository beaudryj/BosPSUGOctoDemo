# Octopus Deploy Demonstration 

## Pre-Requisets 

- Git Client to pull down Repository 
- Vagrant to spin up Boxes 
    - I am currently using Vagrant/VirtualBox  (If you have hyper-v installed you may have to disable that to have Vagrant/Virtualbox work)
- Chocolatey

# Introduction 
    The Purpose for this demonstration is to show people the simplicity of using Octopus for daily deploy tasks. 
    In this demonstration octopus will be used to deploy a powershell module from the Powershell Gallery, deploying a basic website, and configuring IIS Settings. 

# Demo 

 If you haven't already clone down this Repository to acquire the Website, and vagrant file for spinning up your VMs. 

Open a powershell Session and run 

```
Vagrant up
```

This will spin up the 2 Virtual Machines for the purpose of our demo. 

While those spin up, we will start to work on generating our Nuget file to be used by our Octopus Server to Deploy to our WebServer. 

Open up another powershell while vagrant runs and navigate to the working directory. 

```
cd blue_skies

#if chocolatey is installed 
#Only run if you need to install nuget 

choco install nuget.commandline -confirm

```

Now that we have nuget installed we can start working on generating our nuget package. 
Nuget packages are generated via nuspec files in which I have already provided one. They are effectively zip files with useful versioning and metadata. 


```
nuget pack .\blue_skies.nuspec
```

you should now have a nuget file installed that we can use push to our octopus server once that has come up. 


