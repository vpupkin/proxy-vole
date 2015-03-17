## Using the default strategy to find the settings ##

Here an example on how to detect the proxy settings by using the predefined search strategies.

```
ProxySearch proxySearch = ProxySearch.getDefaultProxySearch();
ProxySelector myProxySelector = proxySearch.getProxySelector();
		
ProxySelector.setDefault(myProxySelector);
```

In line 1 we use the static factory method getDefaultProxySearch to create a proxy search object with default settings.
The default settings choosen  depend on the platform that we are currently running on. Normally it is something like this.
Try Java Proxy System Properties if not available try browser settings, if not available try global desktop settings and finally try to detect proxy settings in an environment variable.

In line 2 we invoke the proxy search. This will try to detect the proxy settings as explained above. This will create a ProxySelector for us that will use the detected proxy settings.

The last line installs this ProxySelector as default ProxySelector for all connections.

## Modifying the Search Strategy ##
This example will show you how you can change the default proxy search strategy.

```
ProxySearch proxySearch = new ProxySearch();
		
if (PlatformUtil.getCurrentPlattform() == Platform.WIN) {
  proxySearch.addStrategy(Strategy.IE);
  proxySearch.addStrategy(Strategy.FIREFOX);
  proxySearch.addStrategy(Strategy.JAVA);
} else 
if (PlatformUtil.getCurrentPlattform() == Platform.LINUX) {
  proxySearch.addStrategy(Strategy.GNOME);
  proxySearch.addStrategy(Strategy.KDE);
  proxySearch.addStrategy(Strategy.FIREFOX);
} else {
  proxySearch.addStrategy(Strategy.OS_DEFAULT);
}

ProxySelector myProxySelector = proxySearch.getProxySelector();
		
ProxySelector.setDefault(myProxySelector);
```
Here we use a proxy search strategy customized for different platforms. On windows we try InternetExplorer, Firefox and finally the Java System properties (lines 3-6).

On Linux systems we try to detect the desktop settings first for Gnome then for KDE and if both are not set we try Firefox settings (line 8-11).

For all other platforms we will use the default search strategy (line 13).

And again we invoke the getProxySelector method to let the ProxySearch build a ProxySelector for us that we install as default selector.

## Improving PAC performance ##
When your system uses an automatic procy script (PAC) then a javascript is executed to determine the actuall proxy. The script is interpreted by the Rhino Javascript engine. This one is really good at doing the job but never the less it is slower than compiled java code. If your program needs to access a lot of HTTP urls then this might become a performance bottleneck. To speed things up a little bit you can activate a cache  that will store already processed URLs and when the same host is accessed later on again it will skip the script and return the cached proxy. The following code shows how to activate and configure the proxy cache.

```
ProxySearch ps = ProxySearch.getDefaultProxySearch();

ps.setPacCacheSettings(32, 1000*60*5); // Cache 32 urls for up to 5 min. 
		
ProxySelector myProxySelector = ps.getProxySelector();
ProxySelector.setDefault(myProxySelector);
```
In line 3 we see the setting for the PAC proxy selector cache. The cache will only be used when a PAC proxy selector is created. All other proxy selectors will not be affected.
If you want to setup a cache manually you can do so by using the BufferedProxySelector that is implemented as "Facade".

```
ProxySearch ps = ProxySearch.getDefaultProxySearch();
ProxySelector myProxySelector = ps.getProxySelector();
	
BufferedProxySelector cachedSelector 
    = new BufferedProxySelector(32, 1000*60*5, myProxySelector);

ProxySelector.setDefault(cachedSelector);
```
In the example above we see the usage of a BufferedProxySelector (line 4 and 5) that is wrapped around our original selector.

## How to handle proxy authentication ##
Some proxy servers request a login from the user before they will allow any connections. Proxy-Vole has no support to handle this automatically. This needs to be done manually because there is no way to read the login and password. These settings are stored encrypted. So you need to install an authenticator in your program manually and e.g. ask the user in a dialog to enter a username and password. The following code demonstrates how to setup an network authenticator in Java:

```
Authenticator.setDefault(new Authenticator() {
  @Override
  protected PasswordAuthentication getPasswordAuthentication() {
    if (getRequestorType() == RequestorType.PROXY) {
      return new PasswordAuthentication("proxy-user", "proxy-password".toCharArray());
    } else { 
      return super.getPasswordAuthentication();
    }
  }
		
});
```
Note that we test who is requesting the password. In this case we only handle requests from a proxy server (line 4). You could also check for the hostname or the protocol or use a dialog to ask a user to enter a password instead of returning a hardcoded value.

## Some notes on PAC support and Java 1.5 ##
proxy-vole is compatible with Java 1.5 with a small exception. We have a few classes (JavaxPacScriptParser.java and some corresponding unit tests) that will only compile on Java 1.6. If you need to compile the source under Java 1.5, just remove these classes and and all should compile.

Without these classes you need to ship the Rhino javascript engine (js.jar) with your application if you need PAC support in Java 1.5.

On Java 1.6 all will compile out of the box and you do not ship the js.jar at all.

The build binary release of proxy-vole is compatible with Java 1.5 and this is done with the following trick.

  * If we run under Java 1.6 or higher use the build in javascript support
  * If we do not have build in javascript support but the Rhino engine is on the  classpath use that one.
  * If no javascript engine is found then PAC script parsing will not be supported.

## Editing debugging and testing ##
If you need to know what is going on inside of the library you may want to install a logger on proxy-vole.

```
   Logger.setBackend(new MyLogger());
```
proxy-vole does not use Log4J, `LogBack` or SLF4J because I want to make sure that the library is as light weight as possible with no external dependencies.

If you want to test the PAC parser then you may face the problem that the myIPAddress() method may return you different results on different machines. For unit testing you can set the system property **com.btr.proxy.pac.overrideLocalIP** to a value and then this value will be always used as myIPAddress in all PAC scipts.

Example:
```
   System.setProperty(PacScriptMethods.OVERRIDE_LOCAL_IP, "123.123.123.123");
```



## Summary ##
The examples above show pretty much everything that you can do with the Proxy - Vole library at the moment. To find out more about the available search strategies please consult the [JavaDoc](http://proxy-vole.kenai.com/release/current/javadoc/).

Have fun,

- Bernd Rosstauscher