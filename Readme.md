# Octopus Deploy Demonstration 

## Pre-Requisets 


- Git Client to pull down Repository 
- Vagrant to spin up Boxes 
    - I am currently using Vagrant/VirtualBox  (If you have hyper-v installed you may have to disable that to have Vagrant/Virtualbox work)
    - Clone this Repo to follow along https://github.com/beaudryj/BosPSUGOctoDemo
- Chocolatey

# Introduction 
The Purpose for this demonstration is to show people the simplicity of using Octopus for daily deploy tasks. 
In this demonstration octopus will be used to deploy a powershell module from the Powershell Gallery, deploying a basic website, and configuring IIS Settings. 

# Demo 

## Spin up VMs 

To get started use git to clone down [this repository](https://github.com/beaudryj/BosPSUGOctoDemo) on your local machine. 

On your local machine open a powershell Session navigate into the cloned repository and then run the command below. (This step requires you have vagrant installed)

```
Vagrant up
```
---

## Generate Package for deployment

The Vagrant up will spin up the 2 Virtual Machines for the purpose of our demo. 

While the 2 VMs spin up, we will start to work on generating our Nuget file to be used by our Octopus Server to Deploy to our WebServer. 

Open up another powershell session on your local machine while vagrant runs and navigate to the working directory of the repository. From there we will navigate to the Blue_skies directory to generate our nuget package. (Step Requires you to have Chocolatey to install nuget or have nuget.exe installed) 
 

```
cd blue_skies

choco install nuget.commandline -confirm

```

Now that we have nuget installed we can start working on generating our nuget package. 
Nuget packages are generated via nuspec files in which I have already provided one. They are effectively zip files with useful versioning and metadata. 


```
nuget pack .\blue_skies.nuspec
```

you should now have a nupkg file that we can use push to our octopus server once that has come up. 

---
## Configure Web Server
At this point Octopus is most likely still installing SQL, but our web server should be up. Now we should make sure the webserver has IIS Setup on it so that we can deploy code to it. 

Open up your virtual box instance for "Web" and log in to the vm 
    
    U: Vagrant - p: vagrant

Once logged in there will be a command prompt, just type powershell to kick off a powershell session and run the code below.

```
import-module servermanager
add-windowsfeature Web-Server, Web-WebServer, Web-Security,Web-Filtering
````

We Should be able to hit the [Test Site](http://localhost:8081) on IIS on our Local Machine

We are also going to install the octopus tentacle now. 
Go back on to your webserver vm,  and go to your powershell session and run the command below. 

### * if it fails just run again
```
choco install octopusdeploy.tentacle -confirm
```

---

## Getting Octopus Going

### Logging in 
At this point we should be able to access our octopus server on our local machine using [Octopus](http://localhost:8080).

    U: Admin - p: Vagrant!

### Generating API Key
Once we have logged in we will generate ourselves an API key for publishing our nupkg. 

This can be done by clicking your user in the top right, and then clicking on the API Key tab. you will then click new API key and it will prompt you for a purpose. Be sure to document your API Key for after.


![](https://i.imgur.com/uS3MTFP.gif)

---

### Creating Environment

Now we will create our "Environment" to place our Web Server in. This can be done by clicking environment in the top header on our octopus dashboard. Once there you will click on the Green Add Environment button in the right. 
Here you will name it, and I would recommend checking Use Guided Failure mode by default. 
    
    *Guided failure allows for you to kick off a job where it left off if it failed on a step. 

![](https://i.imgur.com/kIHMaBE.gif)

---

### Adding a Machine

On our newly created environment we will click on Add a deployment target. 

- There we will select Listening tentacle 
- This will provide us with a server thumbprint that we will use on our Tentacle so be sure to document that 
- Configure hostname to be '192.168.245.11'
- Leave Port and Proxy default

Once you click add this might hang... Don't worry its due to the fact that we have not run the listener configuration on the tentacle just yet. 
![](https://i.imgur.com/uWSWCw6.gif)

 Then on the Web Server VM open up a powershell instance and copy and paste the script below, Once completed go back to octopus and finish applyinh the Listener settings which should then add your new server to the environment for us to interact with. 

#### Make sure to Apply your thumbprint below! 
#### May have to re-enter powershell after installing tentacle and webserver

```
cd "C:\Program Files\Octopus Deploy\Tentacle"
Start-Process ".\Tentacle.exe" -ArgumentList "create-instance --instance `"Tentacle`" --config `"C:\Octopus\Tentacle.config`" --console" -wait -nonewwindow
Start-Process ".\Tentacle.exe" -ArgumentList "new-certificate --instance `"Tentacle`" --if-blank --console" -wait -nonewwindow
Start-Process ".\Tentacle.exe" -ArgumentList "configure --instance `"Tentacle`" --reset-trust --console" -wait -nonewwindow
Start-Process ".\Tentacle.exe" -ArgumentList "configure --instance `"Tentacle`" --home `"C:\Octopus`" --app `"C:\Octopus\Applications`" --port `"10933`" --console" -wait -nonewwindow
Start-Process ".\Tentacle.exe" -ArgumentList "configure --instance `"Tentacle`" --trust `"YourThumbPrintHere`" --console" -wait -nonewwindow
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
Start-Process ".\Tentacle.exe" -ArgumentList "service --instance `"Tentacle`" --install --start --console" -wait -nonewwindow
```

Once this Script has been run we will finsih applying our settings

![](https://i.imgur.com/GA0IrZt.gif)


### Creating our First Project

Creating our first project is quite easy. Navigate to projects at the top and click to all proects. You will then see an Add project button in the top right where you can click to generate a new project.

![](http://imgur.com/fQWuvn9.gif)

Before we get deep into creating our project we need to publish our Nuget Package we generated at the beginning. 

We will need the API Key we Generated previously. 

On YOUR machine navigate to your Blue Skies Directory if you arent already there and run. 

```
Nuget push blue_skies.1.0.0.nupkg -ApiKey "yourApiKey" -Source http://localhost:8080/nuget/packages
```

Now Let's validate it uploaded. 

On the Top Right of your dashboard on Octopus Click on Library, and you should see "Blue_Skies" under available packages. 

![](https://i.imgur.com/4vY6rOz.gif)


Now That we have a package to deploy let's add a step to our Demo Project! 

On your dashboard click on the Demo Deploy we created, then click on process on the left hand side and then on the middle of the screen click on add step. We will then click on Deploy a package. 
We will then name the step. Define the Target to run on `Web-Server`

Once that is set we will search for our package.  Type the package name in the packageID field. For this demo we will not have any Configuration Variables or XML Transforms so we can uncheck those. 

However we do want to create an IIS WebSite and AppPool. So at the bottom we will click on configure features and then click on `IIS Web Site and Application Pool`

Once you select that the page will reload with IIS Configuration Settings. 

Since we did not delete the default site we are just going to re-configure that. So fill in the Website name with `Default Web Site`

Same for the Application Pool so name the Application Pool `DefaultAppPool` 

Make sure to enable Anonymous `Authentication` and disable `Windows Authentication`

Then scroll to the bottom and save. 

![](https://i.imgur.com/wGU7bGf.gif)


### Let's Run it! 

Once the project has been created click on Create Release. 

Click on Save. 

And then Deploy to Demo, where you can then click on Deploy release again. You will then be brough to a deploy page where you can see the packages get Deployed. 

![](https://i.imgur.com/XAndh5P.gif)

Check on your [Site](http://localhost:8081/) and you should now be on the Blue Skies test site 

---
## Bonus Section 

Deploying Powershell Modules from Powershell Gallery

So we have our site deployed, but now we want some pester tests, how would we handle getting pester on this node if it isnt already installed? 

On the top left click on Library and navigate to external feeds. 

In the top right we will then `Add Feed` 

We will name it, and then paste in `https://www.powershellgallery.com/api/v2/` to our URL field, there should be no username or password

You will then click Save and test and we can use `Pester` for the test package name. 

![](https://i.imgur.com/ACEaL7V.gif)

Now that we can access the powershell Gallery let's create a new project for deploying pester 

We will click on Projects > All Projects > Add a project. 

The process will be similar. We will Add A step to deploy a package except this time instead of Selecting IIS Web Site and Application Pool we will configure the process for `Custom Installation Directory`

First we Will need to change our Package Feed to Powershell Gallery, and then we can use our package ID of Pester.  

We will still use the `Web-Server` target for our Job. 

     In this Example we are going to want to install to the whole system. 

#### MAKE SURE TO NOT USE PURGE THIS DIRECTORY BEFORE INSTALLTION THIS WILL DELETE ALL OF YOUR SYSTEM MODULES! (IF using in your own environment)

For our install to location we will use `C:\Windows\System32\WindowsPowerShell\v1.0\Modules`

![](https://i.imgur.com/9QGNECK.gif)


Before we run the release we are going to take a look at `Channels` and lock our pester to a prior version. 

Example Say we have a public package and want to make sure we arent accidentally deploying the latest. So We will want to set a maximum version. 
To do this click on channels on the left, we dont have any other channels at this time, so we will click on the default which is being used. We will then click on Add Version Rule, apply it to our Deploy Pester Step. 
We will then set a maximum version by following these [Rules](https://docs.nuget.org/ndocs/create-packages/dependency-versions) for our use case `(,3.4.0]`

![](https://i.imgur.com/LwC543t.gif)

From here let's create our release and Deploy it out! When you go to create your release notice it's set to "Latest" using `3.4.0` not `3.4.3` (At time of writing this) This is a great way to make sure new not vetted versions are deployed. 

![](https://i.imgur.com/MWxcP0m.png)


### Retention 

Lastly we want to make sure we arent excessively deploying things and over loading the Hard drive, so under library and lifecycles let's click on our default lifecycle and setup some retention policies. This will make sure we don't hold onto too many release or packages are held on the server. 

Select your lifecycle
Click on change on the default retention policy and set as you see fit. 

![](https://i.imgur.com/0sLwKu8.gif)
