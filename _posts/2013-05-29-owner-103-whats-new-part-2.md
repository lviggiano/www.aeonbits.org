---
layout: post
title: "OWNER 1.0.3 what’s new. Part 2."
description: ""
category: owner
tags: [annotations, bsd, configuration, examples, framework, free, library, maven, opensource, owner, owner api, properties, propertyeditor]
---
{% include JB/setup %}

For the ones who don't know what <a href="http://owner.aeonbits.org">OWNER library</a> is, you can read my previous article here: <a href="http://en.newinstance.it/2012/12/27/introducing-owner-a-tiny-framework-for-java-properties-files/" title="Introducing OWNER, a tiny framework for Java Properties files.">Introducing OWNER, a tiny framework for Java Properties files</a>.

It has been a long time since I posted the <a href="http://en.newinstance.it/2013/02/04/owner-1-0-3-whats-new-part-1-variable-expansion/">Part 1</a> of this article, and version 1.0.3 was just released. Now I should probably write about version 1.0.4 which is going to have some neat new features, and it will be released soon; but first I need to complete the overview of 1.0.3, and I will also mention new stuff coming in 1.0.4.

In February I wrote about variable expansion. Here I want to briefly introduce some other things that can be useful:
<ul>
	<li>Advanced type conversion</li>
        <li>LoadPolicies to load your property files using different strategies</li>
	<li>Wrapping your java.util.Properties instances (and java.util.Maps too...)</li>
	<li>Passing parameters to properties (and the wonder of @DefaultValue)</li>
</ul>


<h4>Advanced type conversion</h4>

In last post I showed that user can specify some basic Java types as return type for your <em>Properties Mapping Interface</em>, so this is what I am talking about:

<pre class="brush:java">
public interface ServerConfig extends Config {
    Integer port();
    String hostname();
    @DefaultValue("42")
    int maxThreads();
    File logFile();
}

...
// then you can do
public static void main(String[] args) {
    ServerConfig cfg = ConfigFactory.create(ServerConfig.class);
    System.out.println("Server " + cfg.hostname() + ":" + cfg.port() + " will run " + cfg.maxThreads());
}
</pre>

So if you have a file called ServerConfig.properties in the same package of the above class looking like the following one:
<pre class="brush:java">
port=80
hostname=foobar.com
maxThreads=100
logFile=/var/log/myapp.log
</pre>

The OWNER library will map the interface methods to the properties associated, making all the work to convert the String value to int, Integer, java.util.String or any other java primitive type, enums, wrapper classes. Plus some additional Java types are supported, like java.io.File, java.net.URL, java.net.URI, java.lang.Class etc. This is fully customizable, and fully documented <a href="http://owner.aeonbits.org">here</a>. But this is not new.

The new thing in version 1.0.3, is that you can map your property values to your own business objects. So, for instance, if you have class com.acme.wonderfulapp.User, you could define a <i>properties mapping interface</i> like this:

<pre class="brush:java">
public interface MyAppConfig { 
    //... other properties
    @Key("admin.user") // this associates the key for the property
    User adminUser();
}
</pre>

The associated file MyAppConfig.properties should define the above properties like:

<pre class="brush:java">
# MyAppConfig.properties
admin.user=admin 
</pre>

To allow the OWNER library to convert the property <tt>admin.user=admin</tt> to a com.acme.wonderfulapp.User there are many options:

 - You can declare a public Constructor taking java.lang.String or java.lang.Object:

<pre class="brush:java">
public class User {
    //...

    public User(String username) {
        this.username = username;
    }

    // or 
    public User(Object username) {
        this.username = username;
    }
}
</pre>

 - You can declare a public static method <tt>valueOf(String)</tt>:

<pre class="brush:java">
public class User {
    //...
    public static User valueOf(String username) {
        User user = new User();
        user.setUsername(username);
        return user;
    }
}
</pre>
 
 - You can define and register a <tt><a href="http://docs.oracle.com/javase/7/docs/api/java/beans/PropertyEditor.html">PropertyEditor</a></tt>

<pre class="brush:java">
public interface MyAppConfig extends Config {
    @DefaultValue("admin")
    User user();
}

public class UserPropertyEditor extends PropertyEditorSupport {
    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        User user = new User();
        user.setUsername(text);
        setValue(user);
    }
}

public static void main(String[] args) {
    // register the propertyEditor (consult PropertyEditorManager javadocs)
    PropertyEditorManager.registerEditor(User.class, UserPropertyEditor.class);
    MyAppConfig cfg = ConfigFactory.create(MyAppConfig.class);
    System.out.println("admin username is: " + cfg.user().getUsername());
}
</pre>
PropertyEditorManager resolves the editor using some naming convention, so if I rename the class <tt>UserPropertyEditor</tt> into <tt>UserEditor</tt> then there is no need to invoke the method <tt>PropertyEditorManager.registerEditor()</tt>: PropertyEditorManager tries to resolve an editor appending <tt>'-Editor'</tt> to your class name. The editor resolution is a little bit more complex than that; please consult <tt><a href="http://docs.oracle.com/javase/7/docs/api/java/beans/PropertyEditorManager.html">PropertyEditorManager javadoc</a></tt>.

In the above examples I just showed simple cases, but having the ability specify the logic to convert a java property value into a domain object can be quite handy and powerful I think.
The full list of supported automatic conversion is available on the <a href="http://owner.aeonbits.org/#type-conversion">documentation website (see paragraph "Type Conversion")</a>.

<div style="background-color: yellow; padding: 5px; border: 1px solid darkgray">
<div style="text-align: center; font-weight: bold">WARNING: SPOILER ALERT!</div>
With version 1.0.4 the OWNER library also supports arrays and Collections of all the already supported types. This topic will be covered in the next post, when version 1.0.4 will be officially released. So you could do:
<pre class="brush:java">
public interface MyAppConfig extends Config {
    @DefaultValue("admin,root")
    List&lt;User&gt; users(); // collections of user's domain classes
    @DefaultValue("8080,8090")
    int[] ports(); // also arrays and primitive are supported!
}

public static void main(String[] args) {
    PropertyEditorManager.registerEditor(User.class, UserPropertyEditor.class);

    MyAppConfig cfg = ConfigFactory.create(MyAppConfig.class);
    List&lt;User&gt; users = cfg.users();
    assertEquals("admin", users.get(0).getUsername());
    assertEquals("root", users.get(1).getUsername());
}
</pre>

Thanks goes to <a href="http://ffbit.com">Dmytro Chyzhykov</a> for the awesome code he contributed.

But... I'll tell you more after the 1.0.4 release.
</div>


<h4>LoadPolicies to load your property files using different strategies</h4>

By default you don't specify any source, OWNER API tries to load the properties from a file called like your class: if my class is called com.acme.MyAppConfig, then by default OWNER will try to load com.acme.MyAppConfig.properties from the classpath.

Of course this is not enough, and since the first release, OWNER is able to load your property files in a flexible way. You can specify the annotation <tt>@Source</tt> with several locations, in form of URLs, from where the properties must be loaded. 

<pre class="brush:java">
@Sources({ "file:~/.myapp.config", "file:/etc/myapp.config", "classpath:foo/bar/baz.properties" })
public interface ServerConfig extends Config {
    public String myProperty();
    ...
</pre>
This means that OWNER will fist try to load the file ".myapp.config" located in my home directory (the '~' is resolved as the user home), then if that file does not exists the file "/etc/myapp.conf" will be loaded, and as last resort the file "foo/bar/baz.properties" will be loaded from the class path. And, as if this was not enough, you can also specify a default value for every property with the annotation <tt>@DefaultValue</tt>.
All this is nice, but <strong>NOT NEW</strong>, since version 1.0.3 adds even more.

In fact, with version 1.0.3 you have two different "load policies". The default one, is the one exposed above. And we call this behavior <tt>LoadType.FIRST</tt>, since the first available resource specified by the <tt>@Source</tt> annotation is loaded and the remaining are ignored.
The new additional load policy is called <tt>LoadType.MERGE</tt>, and here is an example:

<pre class="brush:java">
@LoadPolicy(LoadType.MERGE)
@Sources({ "file:~/.myapp.config", "file:/etc/myapp.config", "classpath:foo/bar/baz.properties" })
public interface ServerConfig extends Config {
    public String myProperty();
    ...
</pre>
This means that, when I call the method <tt>ServerConfig.myProperty()</tt> <strong>ALL</strong> the sources will be considered in order. If myProperty is specified in "file:/etc/myapp.config" will be loaded from there, unless it is redefined in "file:~/.myapp.config". So basically we have a kind of "fallback" mechanism which allows the user to have a main configuration with lower priority (i.e. system level) and overriding configurations with higher priorities (i.e. user level).


<h4>Wrapping your java.util.Properties instances (and java.util.Maps too...)</h4>

If you have existing instances of Properties (or Maps), for example you have a legacy application that loads the property in some particular way, or just instantiates those properties programmatically, you can "wrap" those instances with OWNER. 
I called this mechanism "importing properties" since it is an *addition* to loading the properties from external resources, and you can use this feature in combination of it.
For instance you can wrap <tt>System.getProperties()</tt> or <tt>System.getenv()</tt> doing something like this:

<pre class="brush:java">

// wrapping System.getenv() 

interface EnvProperties extends Config {
    @Key("HOME")
    String home();

    @Key("USER")
    String user();
}

public static void main(String[] args) {
    SystemEnvProperties cfg = ConfigFactory.create(SystemEnvProperties.class, System.getenv());
    assertEquals(System.getenv().get("HOME"), cfg.home());
    assertEquals(System.getenv().get("USER"), cfg.user());
}

// wrapping System.getProperties()

interface SystemEnvProperties extends Config {
    @Key("file.separator")
    String fileSeparator();

    @Key("java.home")
    String javaHome();
}

public static void main(String[] args) {
    SystemEnvProperties cfg = ConfigFactory.create(SystemEnvProperties.class, System.getProperties());
    assertEquals(File.separator, cfg.fileSeparator());
    assertEquals(System.getProperty("java.home"), cfg.javaHome());
}

// you can also wrap them together in a single interface:

interface SystemEnvProperties extends Config {
    @Key("file.separator")
    String fileSeparator();

    @Key("java.home")
    String javaHome();

    @Key("HOME")
    String home();

    @Key("USER")
    String user();
}

public static void main(String[] args) {
    SystemEnvProperties cfg = ConfigFactory.create(SystemEnvProperties.class, System.getProperties(), System.getenv());
    assertEquals(File.separator, cfg.fileSeparator());
    assertEquals(System.getProperty("java.home"), cfg.javaHome());
    assertEquals(System.getenv().get("HOME"), cfg.home());
    assertEquals(System.getenv().get("USER"), cfg.user());
}
</pre>
In the same way, you can wrap any instance of Properties - and any subclass of java.util.Map - you already have in your project, in a convenient JavaBean-like interface (if you like JavaBeans), with the advantage of having the type conversion for free.

<div style="background-color: yellow; padding: 5px; border: 1px solid darkgray">
<div style="text-align: center; font-weight: bold">WARNING</div>
Notice that the wrapped object becomes unmodifiable (since it is internally copied), so if you modify the original objects, the change will not be reflected. The objects instantiated by OWNER, should be considered <strong>immutable</strong> (well, I am planning to implement the "hot reloading" soon, maybe already in 1.0.4, and being thread safe is a requirement).
</div>


<h4>Passing parameters to properties (and the wonder of @DefaultValue)</h4>

This feature is not really new and is more related to internationalization than on configuration. In any case, in Java interface method can accept parameters and I thought it may be useful to parametrize a property value. And it was easy and elegant to implement, so I did it.

<pre class="brush:java">
interface Sample extends Config {
    @DefaultValue("Hello Mr. %s!")
    String helloMr(String name);
}
public static void main(String[] args) {
    Sample sample = ConfigFactory.create(Sample.class);
    // this will print "Hello Mr. Luigi!"
    System.out.println(sample.helloMr("Luigi")); 
}
</pre>

This is not new in 1.0.3 but I feel like this is an unknown feature, and... in the above example I also show the usage of the <tt>@DefaultValue</tt> annotation. Sometime you just want to define some default values for your configuration, plus specifying a source from where to load the overriding values defined by the user. This is one of the things I like more about this library: it saves me to write properties file until they are really needed. Or if you want, you can write an example, put it on the classpath and let the user to override it placing the property in a conventional location.
The parameters, are just a plus and an excuse to talk again about <tt>@DefaultValue</tt>.

<div style="background-color: yellow; padding: 5px; border: 1px solid darkgray">
<div style="text-align: center; font-weight: bold">WARNING: SPOILER ALERT!</div>
Some people don't need this parameter formatting feature, or the variable expansion that has been introduced in the <a href="http://en.newinstance.it/2013/02/04/owner-1-0-3-whats-new-part-1-variable-expansion/">previous episode</a>. For example, your application may already implement variable expansion in a very similar way, and having an unwanted feature may lead to conflict with legacy features if you are trying to introduce OWNER in your project. For this reason, version 1.0.4 will have an annotation <tt>@DisableFeature</tt> which will allow the user to disable the variable expansion or the parameter formatting on class level and/or on method level.
</div>  
  
  
<h4>Conclusions</h4>

This is just a quick tour on the new things regarding the OWNER library. In 1.0.3 there were also some minor bugfixes and some code cleanup.
What we are trying to do with OWNER library, is to add beautiful features while keeping the code compact, elegant, documented and thoroughly tested. So far so good, I suppose. And more things are bubbling in the pan<sup>[1]</sup>.

<div>
<div>
<strong>Notes:</strong>
</div>
<sup>[1]</sup>:<em>"what's bubbling in the pan?" is the italian literal translation of "Cosa bolle in pentola?", which corresponds to "What's cooking?"</em>
</div>