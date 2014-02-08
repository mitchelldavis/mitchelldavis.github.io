---
layout: post
title:  "Setting Up SSL On WinRM"
date:   2014-02-08
categories: SSL WinRM
---

I started looking into this when it became apparent that some automated tasks needed to be run on separate machines, but initiated by our Build server. (I.E. Jenkins)

A great mechanism for remoting into windows machines is WinRM and is already abstracted into PowerShell cmdlets for us. (See New-PsSession or Enter-PSSession)

A few things are assumed in this write up:

1.) We assume the WinRM service is running on the remote machine and that HTTPS is not already configured for the service.

To setup PowerShell remoting on the machine, type 

	Enable-PSRemoting

into a PowerShell prompt. It'll ask you if you really want to enable it. Make sure you do.

To Check if HTTPS is enabled on the service, type 

	dir WSMan:\localhost\listener 

into a PowerShell console. If any of the listeners listed have an HTTPS transport protocol, then HTTPS might have already been setup on this machine.

If you want to get rid of the HTTPS protocol, or you make a mistake later on in this document, you can type the following into a command or PowerShell prompt to remove the HTTPS listener: 

	winrm delete winrm/config/Listener?Address=*+Transport=HTTPS

2.) We assume the client machine (The one making the remote calls) has the remote machine setup in its TrustedHosts.

_I've noticed that some hairy things happen when the client is the same as the remote machine. Yes, you can remotely connect to your current workstation, however, I wasn't able to get this to work correctly. So, I suggest, just separating the two out._

The client machine needs to signify that it trusts the remote machine it is communicating with. To do this, you can type the following into a PowerShell prompt: 

	Set-Item WSMan:\localhost\client\TrustedHosts *

This means that we trust all computers, but if you want to specify the host, you can replace the asterisks with the name or IP of the machine.

3.) We assume that you're able to communicate with port 5986 of the target machine.  That is the SSL protocol port for WinRM.  The non-SSL port is 5985.

**Create The Certificate**

First, we need to create a certificate. The tool we’re going to use is MakeCert.exe that is readily available in any Windows SDK. ([http://www.microsoft.com/en-us/download/details.aspx?id=8279](http://www.microsoft.com/en-us/download/details.aspx?id=8279))

Once you have access to MakeCert, copy it to the remote machine. We need to run it on the remote machine because the command we’re going to run will automatically do some stuff for us.

Open up a PowerShell prompt and navigate to where the MakeCert.exe executable is then run the following command:

	makecert.exe -len 2048 -r -pe -n "CN=<ip of the remote machine>" -eku 1.3.6.1.5.5.7.3.1 -ss my -sr localMachine -sky exchange -sp "Microsoft RSA SChannel Cryptographic Provider" -sy 12 <file name of the certificate>.cer

A long command, and doing a lot. Two things of note:

- You’ll have to supply the IP of the remote machine after the “CN=”. In a website, this would be the hostname of the site. For example, www.cnn.com has a hostname of cnn.com. However, we aren’t going to access this machine via a hostname, so an IP will have to do.
- You’ll have to supply the name of the certificate to save to file. This is actually kind of funny, because once you run this command, you need to delete that certificate. That’s because the MakeCert executable is already installing the certificate into the machine’s certificate store, so we’re going to use the copy that’s already installed instead of the one put to disk.

**Make sure the Certificate is in all the right places**

- On the Remote Machine Type “MMC” into a PowerShell prompt and after the MMC window pops up, go to _File->Add/Remote Snap In_
- Choose *Certificates* from the list and *Add* it into the *Selected snap-ins* window.
- In the Certificates snap-in window, choose *Computer Account*
- In the Select Computer window, choose *Local computer*.
- Back on the Add or Remove Snap-ins window, click *Ok*.
- On the left of the MMC Main window, there will be a tree-view that allows you to navigate the various Certificate Stores for the Local Computer (since you chose local computer). The certificate you created in the last step should already be in *Console Root -> Certificates -> Personal -> Certificates*.

  We’re going to need to export that certificate and import it into *Console -> Trusted Root Certification Authorities -> Certificates*.

  Right-Click on the certificate and choose *All Tasks -> Export*.
- Choose *Yes, export the private key* when asked.
- Provide a temporary password to encrypt the key. Remember this, because you’ll need to 
provide it when you export this certificate back into the store.
- Supply a file name so you can find it on your hard drive when asked.
- Click *Finish* when you get to it.
- Navigate to *Console Root -> Certificates -> Personal -> Certificates* node in the tree-view on the right, and right-click on the *Certificates* folder and choose *All Tasks -> Import*.
- *Browse* to the file you have just exported.
- Provide the password when asked, and keep hitting *Next* until you are able to hit *Finish*.

Once you finish the import, the certificate is in all the appropriate places to continue on. However, we need a little information from that certificate in the later steps, so we’ll grab that now.

- In either the *Personal* or *Trusted Root Certification Authorities* location, double-click on the certificate to open up the *Certificate Detail*
- On the *Detail* tab, scroll to and click on the *Thumbprint*.
- The window below the field list, will have the thumbprint printed out, copy that and save it for later. We’ll refer to it as the Certificate Thumbprint from now on.

**Create the HTTPS WinRM Listener**

On the remote machine, we need to run the following command. However, you can’t execute it in PowerShell, so you’ll have to open a Command Prompt:

	winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname="<Machine IP>";CertificateThumbprint="<Certificate Thumbprint>"}

This command will create the listener and if you get any errors, you’ll need to go back and make sure that you’ve followed all the steps up to this point correctly. If the certificate is not correct, this step will error.

**Connecting to the Remote Machine**

You should now be able to connect to the remote machine via PowerShell. There is a nuance though. Since we’re using a Self-Signed Certificate, we need to tell PowerShell to not worry about a Certificate Authority. Here is how we do it:

	$opt = New-PSSessionOption –skipcacheck
	$cred = Get-Credential #This will ask you for credentials.
	Enter-PsSession <machine ip> -usessl -SessionOption $opt –cred $cred

You need to supply the –skipcacheck as a session option to tell PowerShell to not worry about the Certificate Authority. Otherwise, it’ll fail because we’re using a self-signed certificate. However, this is still better than using HTTP and just as cheap.

**References**

- [http://social.technet.microsoft.com/Forums/en-US/ITCG/thread/817f788a-4efb-455c-9b59-a6f3aa92ad03](http://social.technet.microsoft.com/Forums/en-US/ITCG/thread/817f788a-4efb-455c-9b59-a6f3aa92ad03)
- [http://blogs.technet.com/b/christwe/archive/2012/06/20/what-port-does-powershell-remoting-use.aspx](http://blogs.technet.com/b/christwe/archive/2012/06/20/what-port-does-powershell-remoting-use.aspx)