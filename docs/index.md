# develop-behind-proxy
Some configuration tricks for developers located behind a corporate proxy. Most of these have been tested under Windows behind a specific NTLM proxy (Zscaler), so it is far from being exhaustive! Linux testers, and other proxy users, welcome ;) 

This documentation is made of three parts:

 * In [HTTP/HTTPs Proxy basics](basics) we remind the concept of HTTP/HTTPs proxy
 * In [Know your proxy](know_your_proxy) we go through all the information that you have to collect and explain how to retrieve the proxy's root SSL certificate if needed.
 * In [Configure your applications](configure_apps) we explain for a list of popular applications, how to configure it to work with your proxy.
 * In [Switching tools](switching) we provide a list of tools to switch between different network environments

# Additional readings

You may find the following articles interesting:

 * [https://support.ssl.com/Knowledgebase/Article/View/19/0/der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them](https://support.ssl.com/Knowledgebase/Article/View/19/0/der-vs-crt-vs-cer-vs-pem-certificates-and-how-to-convert-them)
 * [http://devangst.com/death-by-proxy-tips-for-developing-behind-proxy/](http://devangst.com/death-by-proxy-tips-for-developing-behind-proxy/)
 * [https://mdeinum.wordpress.com/2013/07/01/yeoman-behind-a-corporate-proxy/](https://mdeinum.wordpress.com/2013/07/01/yeoman-behind-a-corporate-proxy/) 

