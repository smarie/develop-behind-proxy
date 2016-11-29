# develop-behind-proxy
Some configuration tricks for developers located behind a corporate proxy. Most of these have been tested under Windows behind a specific NTLM proxy (Zscaler), so it is far from being exhaustive! Linux testers, and other proxy users, welcome ;) 

## 0. NTLM Proxy preamble

With a NTLM HTTP/HTTPS proxy (such as ZScaler), there are a couple things that need to be configured on all tools that require access the outside internet:

* declare the proxy host and port (but not the login and password)
* declare the list of hosts that are located in the intranet and therefore do not require a proxy
* potentially there is a need to add the proxy's root SSL certificate to the list of trusted certificates. 
To check this, here is the procedure using Firefox:
    * connect to an https page such as https://google.com
    * click on the small 'locker' icon at the left of the URL bar
    * extend the popup. It should state "secure connection", verified by xxx
    * if xxx has the name of your proxy instead of being a known certification authority (such as DigiCert, Thawte, Verisign, Symantec, etc.), then you'll have to trust it. First let's retrieve it:
        * Click on More Information > View Certificate > Details. 
        * Select the ROOT certificate, at the top of the hierarchy, and export it as a *.crt file. 
    * Warning: if xxx does not have the name of your proxy, check that this is the case on several sites. Indeed some corporate network policies might setup "white lists" of hosts for which the connection does not go through the proxy but is direct instead.

The guide below helps you to perform these 3 steps in the various tools. If you don't find your tool in the list, please feel free to contribute with a pull request.


## 1. Git

Git relies on the following standard unix/linux environment variables that you should define in your OS : 

    HTTP_PROXY      http://<proxy_host>:<proxy_port>  
    HTTPS_PROXY     http://<proxy_host>:<proxy_port>
    NO_PROXY        localhost,127.0.0.1,<local_addresses>

In addition, there is no handy way to always trust the proxy certificate, without disabling the one from the other authorities or without a tideous addition into the certificate store (see http://stackoverflow.com/questions/23807313/adding-self-signed-ssl-certificate-without-disabling-authority-signed-ones). For this reason I currently use the following *BAD* habit of disabling SSL cert validation for Git:

* Edit your Global git config file. For that you have two options
    * Right click > TortoiseGit > Settings > Git > "Edit Global .gitconfig" button
    * Or manually create/edit a file located at <USER_HOME>/.gitconfig
* Add the following :

        [http]
            sslVerify = false

Note (optional) that you may also wish to add information about you in this same file, that will be provided at every commit.

    [user]
        name = <your name>
        email = <your email address>

## 2. Python

### Conda

Tested with Anaconda3 4.2.0. Run this command in any windows commandline:

    conda config --set ssl_verify <path_to_proxy_cert_file.crt>

### Pip

Create a file under *%APPDATA%/pip/pip.ini* containing

    [global]
    cert = <path_to_proxy_cert_file.crt>

### PyCharm

By default PyCharm relies on the python distribution (e.g. conda) to install the packages. Therefore if you configured conda, PyCharm will be able to download packages from the web. 

In addition, in order for PyCharm to be able to download its updates, it needs to be configured:
Enable "Manual Proxy Configuration" under *File > Settings > Appearance & Behaviour > System Settings > HTTP Proxy*.
(Tested with Community Edition 2016.2.3:)

You may then add the proxy's server certificate to the list of trusted servers, using *Tools > Server Certificates*

## 3. Matlab

In MATLAB Settings (*Home > Preferences > Web*) you may configure the proxy host and port.

Alternatively you may set it using a script:

    com.mathworks.mlwidgets.html.HTMLPrefs.setUseProxy(true);
    com.mathworks.mlwidgets.html.HTMLPrefs.setProxyHost('<proxy_host>');
    com.mathworks.mlwidgets.html.HTMLPrefs.setProxyPort('<proxy_port>');
    com.mathworks.mlwidgets.html.HTMLPrefs.setProxySettings();

There is a nice forum post here to help you trust an SSL certificate in Matlab's JRE : https://fr.mathworks.com/matlabcentral/answers/92506-how-can-i-configure-matlab-to-allow-access-to-self-signed-https-servers?requestedDomain=www.mathworks.com

# Additional readings

http://devangst.com/death-by-proxy-tips-for-developing-behind-proxy/ 
https://mdeinum.wordpress.com/2013/07/01/yeoman-behind-a-corporate-proxy/ 

