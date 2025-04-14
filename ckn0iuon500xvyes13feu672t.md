---
title: "IIS setup WCF with windows auth"
datePublished: Tue Oct 25 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iuon500xvyes13feu672t
slug: iis-setup-wcf-with-windows-auth
tags: authentication, security, services, auth

---


Recently I had to install a WCF service in my local IIS server so I could tests a specific bit of security that I had added. Since this is the first time I did that, I thought I'd document the process for posterity.

## Installing the development certificate

The first thing you need to do is to install the development certificate in your computers certificate store. Run the Microsoft Management Console (mmc) by pressing `Win + R` and typing mmc. In the mmc, click on the menu `File > Add/Remove snap-in`. Select the _Certificates_ option and press `Add`. A new window will pop up. Select the _Computer account_ option and press `Next` and `Finish`. Keep the _Local computer_ option. Click on `Ok` to close the other window.

![mmc-add-snapin.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391418254/51Mew4Qe-.png)

![certificate-snapin.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391434776/ANWvSXklK.png)

![select-computer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391445341/2jDE1pBkL.png)

Now expand the tree to the left so you see the path _Console root\\Certificates (Local computer)\\Personal\\Certificates_. Right click in the middle window and select the `More tasks > Import...` option. Click `Next`, select the development certificate, click `Next` again, choose the _Personal_ folder to install the certificate in, click `Next` a last time and click on `Finish`.

![certificate-location.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391469756/epIafERjm.png)

![certificate-import.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391479610/llKFd18jF.png)

![certificate-selection.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391494622/GI-f6cx-0.png)

![certificate-folder.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391505899/12G9QW5Lu.png)

![certificate-completion.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391520090/VxhXcmYFM.png)

## Configuring IIS

After this, you need to configure IIS. Open the `IIS Manager`, navigate to `ComputerName > Application pools` and click on the `Add Application Pool...` link to the right. Fill in a name you can remember, choose .NET 4.0.xxx as target framework and let the app pool start upon creation. Click `OK`. If the default app pool does not have sufficient rights, it's possible to give it another identity that has more rights on your machine. To do that, click on `Advanced settings...` and update the _Identity_ option.

![iis-app-pool.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391533068/ypGPPOl3B.png)

![create-app-pool.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391553949/2CnGJuzR8.png)

![app-pool-identity.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391563378/KHPV9yTd-.png)

Next you create a new site. Set the name and select the recently created app pool. Point the `Physical path` to the location of your WCF service code, point it to the base folder of your code project to have always the latest version of the API available. You could give it a custom host name and [add that name with it's IP to the hosts file](https://www.google.be/search?q=hosts+file+windows&oq=hosts&aqs=chrome.2.69i57j0l5.3338j0j7&sourceid=chrome&ie=UTF-8) to be able to reach it on a readable url.

![iis-sites.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391578138/d-9M6gelM.png)

![create-site.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391589517/ij4jwWaI9.png)

In the newly created site, double click on the authentication tab in the middle area. This will open all the authentication options available. Depending on what modules there are installed on the server, there will be more or less options available. Enable the options to provide support for them.

![site-overview.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391607360/LIVLJenUV.png)

![site-authentication.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1618391616681/3ESIbqp1B.png)

Enjoy your hosted WCF service. :)
