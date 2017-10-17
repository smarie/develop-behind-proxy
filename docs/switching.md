# Switching tools

If you find yourself switching frequently between proxy / non-proxy configurations, you might wish to use these tools.

## Firefox ProxySwitcher

The excellent [Proxy Switcher](http://firefox.add0n.com/proxy-switcher.html) add-on will save you time! It supports automatic configuration scripts (`.pac`) as well as manual configuration too.

![ProxySwitcher](proxyswitcher.png)


## Envswitcher

I created [envswitch](https://smarie.github.io/env-switcher-gui/) because I did not find anything like Firefox ProxySwitcher to switch the environment variables used by `curl`, `git`, `conda`, `pip` and many other tools.

This is a general-purpose environment switcher, so you have to create an initial configuration file for your needs. You can use the following [configuration file](network_config.yml) with two environments (no proxy / proxy) to get started.

![envswitch](envswitcher.png)
