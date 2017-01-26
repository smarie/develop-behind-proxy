# develop-behind-proxy
Some configuration tricks for developers located behind a corporate proxy. Most of these have been tested under Windows behind a specific NTLM proxy (Zscaler), so it is far from being exhaustive! Linux testers, and other proxy users, welcome ;) 

## 1. HTTP/HTTPs Proxy basics

An HTTP/HTTPS proxy (such as *ZScaler*) sits between your computer and the internet. Its role is to check all outbound connections, possibly log them and even forbid some of them (+ other features depending on the proxy).

Therefore, all your applications that need to connect to the internet have to be informed that they need to `CONNECT` to that proxy first. For this, you will have to:
* declare the proxy's **host** and **port** for the two protocols (HTTP and HTTPs)
* optionally declare your credentials (**username** and **password**). Note that NTML proxies such as ZScaler do not require this - this is even dangerous in term of security to write your username and password somewhere so we don't recommend it
* declare the list of hosts that do not require a proxy. These are typically hosts located in the intranet, but also some hosts that have been put in a whitelist by your IT department. 


### Proxies that modify the SSL certification chain

#### a) Checking if your proxy's root certificate needs to be trusted

Some proxies such as *ZScaler* modify the SSL certification chain by replacing the root Certification Authority (CA)'s certificate signature with their own CA certificate. 

In these cases, you will have to tell **all your software applications** to **trust** this new CA certificate! We will see below how to do that, but first let's check if you need it. To check if your proxy changes the certification chain, here is the procedure using Firefox:
* connect to any https page such as [https://google.com](https://google.com)
* click on the small 'locker' icon at the left of the URL bar
* extend the popup. It should state "secure connection, verified by `xxx`"
* if `xxx` has the name of your proxy instead of being a known certification authority (such as DigiCert, Thawte, Verisign, Symantec, etc.), that means that your proxy has modified the certification chain. Otherwise, that means that either your proxy does not modify it, or bad luck : you picked a site that is in the whitelist :) select another https url and try again, to be sure. 

#### b) Downloading your proxy's root certificate

If the test was positive, you will need to download the proxy's CA certificate in order to be able to trust it. For this:
* Click on `More Information > View Certificate > Details`. 
* Select the ROOT certificate, at the top of the hierarchy - not the intermediate ones ! - and export it as a `*.crt` or `*.cer` file. If your browser asks you which certificate format is needed, select `base64`. This format is a string representation of the certificate's bytes, so you may open the file in Notepad and copy/paste the certificate easily if you need to.

You now have a `my_proxys_root_ca_cert.cer` file. Note: `.cer`, `.crt` and `.pem` are both valid extensions for such a `base64` encoded file, see [this article](https://support.ssl.com/Knowledgebase/Article/View/19/0/der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them). They can be opened with a text editor if needed.

#### c) Creating a certificate bundle including your proxy's root certificate

As we'll see below, some tools are able to support addition of trusted certificates one by one, while some others only support changing the whole trusted certificate bundle entirely. For the latter, you'll therefore need to build a new trusted certificate bundle by appending you proxy's root certificate at the end of an existing certificate bundle.

For example you may wish to get one from [certifi](https://certifi.io/en/latest/):
* download it from this adress: [certs.pem](https://mkcert.org/generate/).
* save the file somewhere on your computer, and rename it for example `160117-certifi-ca-bundle-with-proxy.pem`.
* make sure that only administrator account has modify permissions
* and open it in your favorite text editor to edit it. At the end of the file, append your Proxy's certificate (the contents of the `my_proxys_root_ca_cert.cer` file downladed in previous step). The end of the ca certs bundle file should look like this:
 
 ```text
(... end of the certs.pem file provided by certifi)
TkXWStAmzOVyyghqpZXjFaH3pO3JLF+l+/+sKAIuvtd7u+Nxe5AW0wdeRlN8NwdC
jNPElpzVmbUq4JUagEiuTDkHzsxHpFKVK7q4+63SM1N95R1NbdWhscdCb+ZAJzVc
oyi3B43njTOQ5yOf+1CceWxG1bQVs5ZufpsMljq4Ui0/1lvh+wjChP4kqKOJ2qxq
4RgqsahDYVvTH9w7jXbyLeiNdd8XM2w9U/t7y0Ff/9yi0GE44Za4rF2LN9d11TPA
mRGunUHBcnWEvgJBQl9nJEiU0Zsnvgc/ubhPgXRR4Xq37Z0j4r7g1SgEEzwxA57d
emyPxgcYxn/eR44/KJ4EBs+lVDR3veyJm+kXQ99b21/+jh5Xos1AnX5iItreGCc=
-----END CERTIFICATE-----

My Proxy's Root CA
==================
-----BEGIN CERTIFICATE-----
MIAE07CCA7ugAwIBAgIJANu+mC2Jt3uTMA0GCSqGSIb3DQEBCwUAMIGhMQswCQYD
VQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTERMA8GA1UEBxMIU2FuIEpvc2Ux
(... this is fairly long)
xFNjavxrHmsH8jPHVvgc1VD0Opja0l/BRVauTrUaoW6tE+wFG5rEcPGS80jjHK4S
pB5iDj2mUZH1T8lzYtuZy0ZPirxmtsk3135+CKNa2OCAhhFjE0xd
-----END CERTIFICATE-----
```

You are now all set :

* You already have a `my_proxys_root_ca_cert.crt` file containing your proxy's certificate only. This will be referred to as `<path_to_root_ca_cert.crt>` below
* You now have a `160117-certifi-ca-bundle-with-proxy.pem` file containing a bundle of trusted certificates, as well as your proxy's. This will be referred to as `<path_to_ca_bundle.pem>`

(Remember that `.crt`, `.cer` and `.pem` are both valid extensions for these files)


## 2. Configuring each application to use the proxy

This chapter helps you to configure the various applications that you may wish to use. Remember that the proxy prevents any application to access the internet, so any time an application does not work and you think it requires internet access, this might be it. If you don't find your tool in the list, please feel free to contribute with a pull request.


### Common settings for tools that support auto-config scripts (.pac)

Some tools are able to get their configuration with "auto-config" from a `.pac` script. If your organization provides such a script, simply use it to configure the tool and you're done.


### Common settings for several Unix/Linux inspired tools  

Curl, Git, Conda, Pip and many others rely on the following standard unix/linux environment variables that you should define in your OS : 

```bash
HTTP_PROXY      http://<username>@<proxy_host>:<proxy_port>  
HTTPS_PROXY     http://<username>@<proxy_host>:<proxy_port>
NO_PROXY        localhost,127.0.0.1,<local_addresses>  
CURL_CA_BUNDLE  <path_to_ca_bundle.pem>
```

Important note: storing the list of trusted certificates in a raw file such as here `<path_to_ca_bundle.pem>` is still a bit unsecure, unless you make sure that the file can not be corrupted by a third party (at least you may wish to ensure that only administrators can modify this file). But this is far more secure than disabling SSL certificate verification in your application :)


### b) Git

Git accepts the environment variables described earlier. There is no simple way to add a unique certificate to git's trust store (see [this discussion](http://stackoverflow.com/questions/23807313/adding-self-signed-ssl-certificate-without-disabling-authority-signed-ones)), but it seems to correctly manage the `CURL_CA_BUNDLE` one (see previous chapter) so if you've done the previous steps you should be all set. 


### c) Python

#### Any python code

If you rely on the python [`Requests`](http://docs.python-requests.org/en/master/) package for HTTP and HTTPs calls, all environment variables described earlier are supported. Concerning certificate trust, the package first looks at `REQUESTS_CA_BUNDLE` environment variable for trusted certificates, and defaults to `CURL_CA_BUNDLE` if it is not defined.

#### Conda (Anaconda Python distribution)

Conda accepts the environment variables described earlier. There is no simple way to add a unique certificate to its trust store, but it seems to correctly manage the `CURL_CA_BUNDLE` one (see previous chapter) so if you've done the previous steps you should be all set. 

#### Pip 

Depending on your version of pip you may either have 

* nothing to do (same than `Requests` and `Conda`), 
* or the following manual operation to perform to replace the trusted bundle of certificates: create a file under *%APPDATA%/pip/pip.ini* containing

```text
[global]
cert = <path_to_ca_bundle.pem>
```


#### PyCharm

By default PyCharm relies on the python distribution (e.g. conda) to install the packages. Therefore if you configured conda, PyCharm will be able to download packages from the web. 

In addition, in order for PyCharm to be able to download its own updates, it needs to be configured under *File > Settings > Appearance & Behaviour > System Settings > HTTP Proxy*:

* If possible, enable "Auto-detect Proxy settings" and possibly enter your automatic proxy configuration URL here.
* Otherwise, enable "Manual Proxy Configuration" .

(Tested with Community Edition 2016.2.3:)

You may then add the proxy's server certificate to the list of trusted servers, using *Tools > Server Certificates*


### d) Matlab

In MATLAB Settings (*Home > Preferences > Web*) you may configure the proxy host and port.

Alternatively you may set it using a script:

```matlab
com.mathworks.mlwidgets.html.HTMLPrefs.setUseProxy(true);
com.mathworks.mlwidgets.html.HTMLPrefs.setProxyHost('<proxy_host>');
com.mathworks.mlwidgets.html.HTMLPrefs.setProxyPort('<proxy_port>');
com.mathworks.mlwidgets.html.HTMLPrefs.setProxySettings();
```

There is a nice forum post here to help you trust an SSL certificate in Matlab's JRE : https://fr.mathworks.com/matlabcentral/answers/92506-how-can-i-configure-matlab-to-allow-access-to-self-signed-https-servers?requestedDomain=www.mathworks.com


### e) R

R provides several packages to perform http call (httr, Rcurl, curl). By default, the `HTTP_PROXY`, `HTTPS_PROXY` environment variables seem to be taken into account quite well.

Unfortunately, the R libraries don't seem to take into account the `CURL_CA_BUNDLE` environment variable. You may however trust a proxy by appending it's root certificate at the end of the following file : `<R_HOME>\etc\curl-ca-bundle.crt`. 


### f) .Net-based applications

Most .Net-based applications rely on Internet Explorer proxy settings and Windows trusted certificates store. Therefore you simply have two steps to perform 
* in IE Settings, declare the proxy : `Internet Options > Connections > Network Settings`
* on your windows desktop, trust the proxy's root certificate by right-clicking on the certificate, and selecting `install certificate`

You are then all set for most applications relying on the .Net framework..


### g) Java-based applications

Each Java Virtual Machine (JVM) relies on a file named `cacerts` where the bundle of trusted certificated is held. This file is encrypted (as opposed to the one that we created for `CURL_CA_BUNDLE`) and its default password is `changeit`. You may first wish to change the password:
 
 ```bash
 > keytool -storepasswd -keystore "%JAVA_HOME%/jre/lib/security/cacerts"
 ```
 
 And then add the trusted certificate:

```bash
> keytool -keystore "%JAVA_HOME%/jre/lib/security/cacerts" -importcert -alias <proxy_alias> -file <proxy_root_certificate.cer>
```

Note: [keytool](http://docs.oracle.com/javase/7/docs/technotes/tools/windows/keytool.html) ships with the JVM so if your JVM is on the system PATH it should be found.

Several Java-based applications ship with their own dedicated JVM: MATLAB, Rapidminer, etc. Therefore they don't benefit from this modification, you'll have to redo it for each of them individually (or to copy/paste the above `cacerts` file onto the application's to replace it).

Note that for MATLAB a specific more 'user-friendly' tool was provided by Mathworks, see corresponding chapter above.

### h) Browsers

Although browsers settings allow to modify the proxy more or less easily, there are useful plugins to switch even faster - typically between home and office.

* *Firefox*: 

    * the excellent [Proxy Switcher](http://firefox.add0n.com/proxy-switcher.html) add-on will save you time! It supports automatic configuration scripts (.pac) as well as manual configuration too.
    * in order to trust your proxy's root certificate, go to *Advanced Settings > Certificates > View Certificates > Authorities > Import* and import your proxy's root certificate here. Check the "Trust for websites" option, at least, and confirm.


# Additional readings

https://support.ssl.com/Knowledgebase/Article/View/19/0/der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them

http://devangst.com/death-by-proxy-tips-for-developing-behind-proxy/ 

https://mdeinum.wordpress.com/2013/07/01/yeoman-behind-a-corporate-proxy/ 

