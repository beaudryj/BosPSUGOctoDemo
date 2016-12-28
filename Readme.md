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

## Spin up VMs 

 If you haven't already clone down this Repository to acquire the Website, and vagrant file for spinning up your VMs. 

Open a powershell Session and run 

```
Vagrant up
```
---
## Generate Package for deployment

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

---
## Configure Web Server
At this point Octopus is most likely still installing SQL, but our Servers should be up. Now we should make sure the webserver has IIS Setup on it so that we can deploy code to it. 

Open up your virtual box instance and log in to the vm 
    
    U: Vagrant - p: vagrant

Once logged in there will be a command prompt, just type powershell to kick off a powershell session and run the code below.

```
import-module servermanager
add-windowsfeature Web-Server, Web-WebServer, Web-Security,Web-Filtering
````

We Should be able to hit the [Test Site](http://localhost:8081) on IIS 

We are also going to install the octopus tentacle now. 
Go back on to your webserver and go to your powershell session and run the command below. 

### * if it fails just run again
```
choco install octopusdeploy.tentacle -confirm
```





---

## Getting Octopus Going

### Logging in 
At this point we should be able to access [Octopus](http://localhost:8080).

    U: Admin - p: Vagrant!

### Generating API Key
Once we have logged in we will generate ourselves an API key configuring our tentacle. 

This can be done by clicking your user in the top right, and then clicking on the API Key tab. you will then click new API key and it will prompt you for a purpose. Be sure to document your API Key for after.


![](http://i.imgur.com/uS3MTFP.gif)

___

### Creating Environment

Now we will create our "Environment" to place our Web Server in. This can be done by clicking environment in the top header. Once there you will click on the Green Add Environment button in the right. 
Here you will name it, and I would recommend checking Use Guided Failure mode by default. 
    
    *Guided failure allows for you to kick off a job where it left off if it failed on a step. 

![](http://i.imgur.com/kIHMaBE.gif)

___

### Adding a Machine

On our newly created environment we will click on Add a deployment target. 

- There we will select Listening tentacle 
- This will provide us with a server thumbprint that we will use on our Tentacle so be sure to document that 
- Configure hostname to be '192.168.245.11'
- Leave Port and Proxy default

Once you click add this might hang... Don't worry its due to the fact that we have not run the listener configuration on the tentacle just yet. 
![](http://i.imgur.com/uWSWCw6.gif)

 Then on the Web Server VM open up a powershell instance and copy and paste the script below, Once completed go back to octopus and finish applyinh the Listener settings which should then add your new server to the environment for us to interact with. 

#### Make sure to Apply your thumbprint below! 
#### May have to re-enter powershell after installing tentacle and webserver

```
cd "C:\Program Files\Octopus Deploy\Tentacle"
Start-Process "Tentacle.exe" -ArgumentList "create-instance --instance `"Tentacle`" --config `"C:\Octopus\Tentacle.config`" --console" -wait -nonewwindow
Start-Process "Tentacle.exe" -ArgumentList "new-certificate --instance `"Tentacle`" --if-blank --console" -wait -nonewwindow
Start-Process "Tentacle.exe" -ArgumentList "configure --instance `"Tentacle`" --reset-trust --console" -wait -nonewwindow
Start-Process "Tentacle.exe" -ArgumentList "configure --instance `"Tentacle`" --home `"C:\Octopus`" --app `"C:\Octopus\Applications`" --port `"10933`" --console" -wait -nonewwindow
Start-Process "Tentacle.exe" -ArgumentList "configure --instance `"Tentacle`" --trust `"YourThumbPrintHere`" --console" -wait -nonewwindow
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
Start-Process "Tentacle.exe" -ArgumentList "service --instance `"Tentacle`" --install --start --console" -wait -nonewwindow
```

Once this Script has been run we will finsih applying our settings

![](http://i.imgur.com/GA0IrZt.gif)


### Creating our First Project