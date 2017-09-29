# Configuring each application to use the proxy

In the previous sections you managed to collect information about your proxy. Here is what you should have collected so far:

* the proxy's **host** and **port** for HTTP, referred to as `<http_host>` and `<http_port>` below (e.g. `proxyhost`, `8080`)
* the proxy's **host** and **port** for HTTPS, referred to as `<https_host>` and `<https_port>` below (e.g. `proxyhost`, `4443`)
* the **list of hosts that do not require a proxy**, referred to as `<no_proxy_hosts>` below (e.g. `localhost,127.0.0.1,mylocalserver`)
* (optional) your credentials (`<user>` and `<pwd>`). We don't recommend that you use these.

If your IT department provides an auto-configuration script, a `.pac` file.

And, if your proxy changes the certification chain: 

* a `my_proxys_root_ca_cert.crt` file containing your proxy's certificate only. This will be referred to as `<path_to_root_ca_cert.crt>` below
* a `160117-certifi-ca-bundle-with-proxy.pem` file containing a bundle of trusted certificates, as well as your proxy's. This will be referred to as `<path_to_ca_bundle.pem>` below

This chapter helps you to configure various applications to use your proxy. Remember that the proxy prevents any application to access the internet, so any time an application does not work and you think it requires internet access, this might be the cause. If you don't find your tool in the list, please feel free to contribute with a pull request on the [github project](https://github.com/smarie/develop-behind-proxy).



## Common settings for several Unix/Linux inspired tools  

`curl`, `git`, `conda`, `pip`, `NodeJS`  and many other tools rely on the following standard UNIX/Linux environment variables that you should define in your OS (also on windows): 

```bash
http_proxy      http://[<user>:<pwd>@]<http_host>:<http_port>  
https_proxy     http://[<user>:<pwd>@]<https_host>:<https_port>
no_proxy        <no_proxy_hosts>
curl_ca_bundle  <path_to_ca_bundle.pem>
```

Important note: storing the list of trusted certificates in a raw file such as here `<path_to_ca_bundle.pem>` is still a bit unsecure, unless you make sure that the file can not be corrupted by a third party (at least you may wish to ensure that only administrators can modify this file). But this is **far** more secure than disabling SSL certificate verification in your application :)

To set these environment variables in Linux you can either use the user-specific ~/.bashrc file or configure them in the global environment. However, this depends on the distribution you are running:
* Ubuntu: Create or append to `/etc/environment`
* Gentoo Linux: Create a new file in `/etc/env.d` such as `/etc/env.d/99proxy`

On Windows, one configures the environment variables in the **Advanced System Settings**:
* Press Windows-Key + R, enter `sysdm.cpl ,3` (mind the space before the comma) and press Enter
* Click the **Environment variables** button
* In either of the fields (**User variables** or **System variables**), add the four variables



## Git

Git accepts the environment variables described earlier. There is no simple way to add a unique certificate to git's trust store (see [this discussion](http://stackoverflow.com/questions/23807313/adding-self-signed-ssl-certificate-without-disabling-authority-signed-ones)), but it seems to correctly manage the `curl_ca_bundle` one (see previous chapter) so if you've done the previous steps you should be all set. 

Edit : apparently this is not true, you have to add a specifig configuration option in one of your git configuration files:

```
[http]
  sslCAInfo = <path_to_ca_bundle.pem>
```

[Thanks, source!](https://blogs.msdn.microsoft.com/phkelley/2014/01/20/adding-a-corporate-or-self-signed-certificate-authority-to-git-exes-store/)

Not directly related to the Proxy but if your organization's firewall should block outgoing connections to TCP port 9418 (used by the git:// protocol) and some software automatically clones from those URLs without any method to change the URLs, you can tell git to use the HTTPS protocol instead of the faster GIT protocol by adding these lines to you git config file:
```
[url "https://"]
    insteadOf = git://
```
The same goes equivalently for the SSH protocol on TCP port 22.


## Python

### Any python code

If you rely on the python [`Requests`](http://docs.python-requests.org/en/master/) package for HTTP and HTTPs calls, all environment variables described earlier are supported. Concerning certificate trust, the package first looks at `requests_ca_bundle` environment variable for trusted certificates, and defaults to `curl_ca_bundle` if it is not defined.

### Conda (Anaconda Python distribution)

Conda accepts the environment variables described earlier. There is no simple way to add a unique certificate to its trust store, but it seems to correctly manage the `curl_ca_bundle` one (see previous chapter) so if you've done the previous steps you should be all set. 

### Pip 

Depending on your version of pip you may either have 

* nothing to do (same than `Requests` and `Conda`), 
* or the following manual operation to perform to replace the trusted bundle of certificates: create a file under *%APPDATA%/pip/pip.ini* containing

```text
[global]
cert = <path_to_ca_bundle.pem>
```


### PyCharm

By default PyCharm relies on the python distribution (e.g. conda) to install the packages. Therefore if you configured conda, PyCharm will be able to download packages from the web. 

In addition, in order for PyCharm to be able to download its own updates, it needs to be configured under *File > Settings > Appearance & Behaviour > System Settings > HTTP Proxy*:

* If possible, enable "Auto-detect Proxy settings" and possibly enter your automatic proxy configuration URL here.
* Otherwise, enable "Manual Proxy Configuration" .

(Tested with Community Edition 2016.2.3:)

You may then add the proxy's server certificate to the list of trusted servers, using *Tools > Server Certificates*


## MATLAB

In MATLAB Settings (*Home > Preferences > Web*) you may configure the proxy host and port.

Alternatively you may set it using a script:

```matlab
com.mathworks.mlwidgets.html.HTMLPrefs.setUseProxy(true);
com.mathworks.mlwidgets.html.HTMLPrefs.setProxyHost('<proxy_host>');
com.mathworks.mlwidgets.html.HTMLPrefs.setProxyPort('<proxy_port>');
com.mathworks.mlwidgets.html.HTMLPrefs.setProxySettings();
```

There is a nice forum post here to help you trust an SSL certificate in Matlab's JRE : https://fr.mathworks.com/matlabcentral/answers/92506-how-can-i-configure-matlab-to-allow-access-to-self-signed-https-servers?requestedDomain=www.mathworks.com


## R

R provides several packages to perform http call (httr, Rcurl, curl). By default, the `http_proxy`, `https_proxy` environment variables seem to be taken into account quite well.

Unfortunately, the R libraries don't seem to take into account the `curl_ca_bundle` environment variable. You may however trust a proxy by appending it's root certificate at the end of the following file : `<R_HOME>\etc\curl-ca-bundle.crt`. 


## .Net-based applications

Most .Net-based applications rely on Internet Explorer proxy settings and Windows trusted certificates store. Therefore you simply have two steps to perform 
* in IE Settings, declare the proxy : `Internet Options > Connections > Network Settings`
* on your windows desktop, trust the proxy's root certificate by right-clicking on the certificate, and selecting `install certificate`

You are then all set for most applications relying on the .Net framework..


## Java-based applications

Each Java Virtual Machine (JVM) relies on a file named `cacerts` where the bundle of trusted certificated is held. This file is encrypted (as opposed to the one that we created for `curl_ca_bundle`) and its default password is `changeit`. You may first wish to change the password:
 
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

If you have any Java software or daemons/services running (for example the Gradle deamon used for Android development and which is automatically started) those will need to be killed and restarted before they start accepting the proxy's certificate.


## Web Browsers

Although browsers settings allow to modify the proxy more or less easily, there are useful plugins to switch even faster - typically between home and office.

### Firefox 

* the excellent [Proxy Switcher](http://firefox.add0n.com/proxy-switcher.html) add-on will save you time! It supports automatic configuration scripts (`.pac`) as well as manual configuration too.

* in order to trust your proxy's root certificate, go to *Advanced Settings > Certificates > View Certificates > Authorities > Import* and import your proxy's root certificate here. Check the "Trust for websites" option, at least, and confirm.

### Chrome

Chrome relies on the OS to get its proxy configuration. On windows this is similar to Internet Explorer, see below.

### Internet Explorer

Internet Explorer relies on the Windows OS to get its proxy configuration. You may either choose a manual configuration or an automatic configuration script (`.pac`) in *Internet Options > Connections > Network Settings *

In order to trust your proxy's root certificate, right click on your proxy's certificate file (not the bundle file) in the windows file explorer and select *Install certificate*.

### OpenSSL-based applications (such as wget)

OpenSSL uses a directory-based certificate store, usually in `/etc/ssl/certs`. To make OpenSSL-based applications accept your proxy's certificate, you need to copy the Base64-encoded crt file to that folder and create a symlink with a specific name to that file. That process is described at (http://gagravarr.org/writing/openssl-certs/others.shtml#ca-openssl)

### APT (Debian Linux and Ubuntu Linux packet manager)

`apt` unfortunately  does not honor the environment variables mentioned in section a) and needs  the proxy configured in a dedicated configuration file. As root, create a new file named `/etc/apt/apt.conf.d/99proxy` and the following content:
```
Acquire::http::proxy "http://proxy.company.com:80/";
Acquire::https::proxy "https://proxy.company.com:80/";
Acquire::ftp::proxy "ftp://proxy.company.com:80/";
```

### NodeJS / npm

npm honors the environment variables as described in section a). These should definitely be set even if you configure npm to use a proxy via npm's config file because any post-install hooks will NOT honor npm's config. If the environment variables are set, all post-install hooks should work properly. Certificate errors can be avoided by running `npm config set strict-ssl false`.

### Apache cordova projects for Android / Gradle based projects

Gradle-based Android projects (and as such those created by Apache cordova) need to have the proxy server configured via a specific file called `gradle.properties` in the project's root folder. It should have the following content:

```
systemProp.http.proxyHost=proxy.company.com
systemProp.http.proxyPort=80
systemProp.https.proxyHost=proxy.company.com
systemProp.https.proxyPort=80
```

### Android Studio

The proxy certificate needs to imported via the settings dialog: `Settings → Appereance & Behavior → System Settings → HTTP Proxy and Settings → Tools → Server Certificates`

### Android SDK Manager (standalone tool)

While the Android SDK downloads can be managed via the Android Studio, there is a standalone-tool for that task as well. If you still get certificate errors here, option the `Options` dialog and check the checkbox `Force https://... sources to be fetched using http://...`

### WinHTTP service (used for Windows Update downloads)

Sometimes the WinHTTP service doesn't pick up the proxy configured via the **Internet Options** dialog. If your windows installation fails to download updates and takes very long before producing an error, try configuring the WinHTTP service manually as described at http://www.winplat.net/post/2012/04/06/Configure-Proxy-settings-for-WinHttp-on-Windows-2008-R2-and-Windows-7.aspx
