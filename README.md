
# Malicious Extension Investigation

Sophos EDR trigger an alert from one of our customer regarding initiate C2 (Command and Control) connection to the malicious domain (admitab.com) from one of the endpoints where it installed malicious extension.




### Sophos EDR Alert

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/SophosAlertC2.png)

Sophos EDR detect endpoint Source IP (10.1.3.193) initiate connection C2 to domain (admitab.com) through dns query (port 53). Sophos firewall able to detected and dropped the connection before being able going out of network.






### Investigate The Domain

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/admitab.comVirustotalCheck.png)

Investigate the domain (admitab.com) that endpoint trying to reachout. From virustotal its shown the domain being flagged as malicious by 12 vendors. This should drawn our attention on this matter. Due to the domain being flagged is the reason why the firewall drop the connection.

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/admitab.comVirustotalCommunity.png)

So we investigate further about this domain, In community tab shown that this domain associated with malware that had happen before.

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/admitab.comArticle.png)

Checking the article link that provided in community tab, it an article about malicious extension and domain that associate with it. Which we can conclude that the user installed malicious extension in endpoint.

The way the malicious extension work:
```bash
1. Capture the URL of the page victim visiting.
2. Sends it to the remote server with victim unique tracking ID.
3. Receives potential redirect URLs from the command and control server.
4. Automatically redirects victim to bogus browser.
```
### Investigate The Endpoint

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/DCserver.png)

Investigating the source ip that initiate request dns query to the domain shown that it from Datacenter server since the connection already being dropped by firewall. We don't need to isolate the Datacenter server because it will disrupt the services. 

We continue investigate does Datacenter server have malicious extension installed.

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/DCserverChromeExtension.png)

The Datacenter server do have chrome installed but after inspecting, it doesn't have any extension installed. Which mean the initiate request doesn't coming from Datacenter server.

Since we know the request is using dns query (port 53) as state in alert, we investigate further about anything related with dns.

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/DCserverConfiguration.png)

Inspecting Datacenter server configuration, we can see the configuration for DNS server is 127.0.0.1 the first IP in DNS servers section which mean its loopback address so the Datacenter server is the DNS server. 

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/DCnslookup.png)

For more confirmation, execute "nslookup admitab.com" shows that the Datacenter server resolve the domain itself when its shows loopback address.

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/DCdnsState.png)

Another way to check whether if Datacenter server is DNS server is by checking dns configuration as screenshot above. DNS state as running meaning it is DNS server.

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/EndpointIpconfig.png)

We can also check the configuration of any endpoint for its DNS server, here we see the DNS server is 10.1.3.193 (Datacenter server).

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/EndpointNslookup.png)

Also from "nslookup google" on endpoint, its shows the DC server resolve it.

From this, we know Datacenter server is being setup as DNS server. Endpoint in the organization is requesting domain name will resolve into IP address by DC server. Which is why we see Source IP address requesting to admitab.com is DC server IP rather than endpoint IP.


### Finding Extension in Endpoint 

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/ArticleExtensionID.png)

Search any information regarding this malicious extension, we found an article that provide the extension id regarding malicious extension.

![App Screenshot](https://github.com/Muaz1425/Malicious-Extension-admitab.com-/blob/main/Images/RemovingExtensionOnEndpoint.png)

Using Sophos EDR, we can extract all the endpoints that have chrome extension and from article that provide extension id, we can find the malicious extension that have being installed.

With this we able to remove the malicious extension from the endpoints when Source IP Address is not available.
