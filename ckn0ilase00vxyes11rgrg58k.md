---
title: "Adding health monitoring to services"
datePublished: Tue Nov 22 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ilase00vxyes11rgrg58k
slug: adding-health-monitoring
tags: monitoring, services

---


In the previous project I worked on, the team needed a way to see if our services were still up and running. Thus health monitoring was introduced. Read on to see the options that the team discussed.

Health monitoring is a way of checking wether a service, site or application is working. The results can be displayed on a screen so the team can be notified when a service stops working. Advanced monitoring implementations could show when important, but non-fatal errors occur that still need a fast response.

Since there were two kinds of services running in the project, we discussed solutions for web and Windows services.

## Web service monitoring

As a starting point, we would do a HTTP GET request for the WSDL every 10 minutes. The interval can be shorter or longer depending on how important the service is. We could also request any random end point on the webservice to get the status back, but the WSDL is a default endpoint on any .NET service. It is also quite reusable. When we want to monitor other services, we could supply the base address of another service and the WSDL endpoint can be automatically discovered. When the response of the WSDL is a `200 OK`, we know the service is up and running. When the response is not a `200 OK` response, we can flag it as unavailable.

If we want to receive a more detailed report for the health monitoring dashboard, then it is advisable to implement a custom endpoint that can supply detailed information. The contract to implement on the service should be defined in the framework that will handle the response. This can then be easily shared among multiple services to promote reuse. The `2oo OK` repsonse does not just indicate a service that is up and running, but can contain additional information, such as important errors that need to be checked out or other custom information that is important for the support staff. To determine the unavailability, we would still listen to the custom endpoint for anything else than a `200 OK` response message.

## Windows service monitoring

The other service we need to monitor is a Windows service. This is a little bit more tricky as we cannot directly invoke such a service. The solution we devised is to let the service update a file that we can access (i.e. on a network share, if that is allowed within the security context of the service) or update a record in a special database or table. We could write a custom message or exceptions that need attention. I will call it a file, since that is what we chose, but know that a database is equally valid.

The service would update the file every 10 minutes (again, determine what works best) with the date and time, so we know approximatly when the last time the service was active. In addition we would log the most ciritcal errors that did not crash the application, but need immediate attention.

To monitor it, we also read the file every 10 minutes from the service that listens to the web services. When the date and time in the file is the same as the date and time currently displayed, then we know something is wrong. The complication here is that we don't know whether the problem is with the Windows service, with the connection to the file location (or the database connection) or the account with which the file is accessed. The most important thing is that we know something is wrong so we can investigate.

## Conclusion

Health monitoring is not difficult, but it does take some work to get it up and running decently and accuratly. It provides a valuable tool in monitoring the availability of our (and your) services.
