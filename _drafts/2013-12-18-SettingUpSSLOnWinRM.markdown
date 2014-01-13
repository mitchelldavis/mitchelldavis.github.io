---
layout: post
title:  "Setting Up SSL On WinRM"
date:   2013-12-18
categories: SSL WinRM
---

I started looking into this when it became apparent that some automated tasks needed to be run on separate machines, but initiated by our Build server. (I.E. Jenkins)

A great mechanism for remoting into windows machines is WinRM and is already abstracted into PowerShell cmdlets for us. (See New-PsSession or Enter-PSSession)

A few things are assumed in this write up:

- We assume the WinRM service is running on the remote machine and that HTTPS is not already configured for the service.

	To setup PowerShell remoting on the machine, type 

	{% highlight bash %}
	Enable-PSRemoting
	{% endhighlight %} 

	into a PowerShell prompt. It’ll ask you if you really want to enable it. Make sure you do.

	To Check if HTTPS is enabled on the service, type `dir WSMan:\localhost\listener` into a PowerShell console. If any of the listeners listed have an HTTPS transport protocol, then HTTPS might have already been setup on this machine.

	If you want to get rid of the HTTPS protocol, or you make a mistake later on in this document, you can type the following into a command or PowerShell prompt to remove the HTTPS listener: `winrm delete winrm/config/Listener?Address=*+Transport=HTTPS`

- We assume the client machine (The one making the remote calls) has the remote machine setup in its TrustedHosts.

	Note: I’ve noticed that some hairy things happen when the client is the same as the remote machine. Yes, you can remotely connect to your current workstation, however, I wasn’t able to get this to work correctly. So, I suggest, just separating the two out.

	The client machine needs to signify that it trusts the remote machine it is communicating with. To do this, you can type the following into a PowerShell prompt: `Set-Item WSMan:\localhost\client\TrustedHosts *`

	This means that we trust all computers, but if you want to specify the host, you can replace the asterisks with the name or IP of the machine.

- We assume that you're able to communicate with port 5986 of the target machine.  That is the SSL protocol port for WinRM.  The non-SSL port is 5985.

**Create The Certificate**

First, we need to create a certificate. The tool we’re going to use is MakeCert.exe that is readily available in any Windows SDK. ([http://www.microsoft.com/en-us/download/details.aspx?id=8279](http://www.microsoft.com/en-us/download/details.aspx?id=8279))

Once you have access to MakeCert, copy it to the remote machine. We need to run it on the remote machine because the command we’re going to run will automatically do some stuff for us.

Open up a PowerShell prompt and navigate to where the MakeCert.exe executable is then run the following command:

{% highlight bash %}
makecert.exe -len 2048 -r -pe -n "CN=_ip of the remote machine_" -eku 1.3.6.1.5.5.7.3.1 -ss my -sr localMachine -sky exchange -sp "Microsoft RSA SChannel Cryptographic Provider" -sy 12 _file name of the certificate_.cer
{% endhighlight %}

