---
title: Quick Start  
layout: toc-guide-page
lprev: 023-examples-microservice-jdbc.html 
lnext: 020-examples-microservice.html
summary: A walkthrough of the Quick Start example
author: enRoute@paremus.com
sponsor: OSGi™ Alliance 
---

The quick start example is a simple OSGi enRoute application created from the [enRoute project archetype](../about/112-enRoute-Archetypes.html#the-project-archetype). Quick start contains a single OSGi service which exposes a REST endpoint and some static files. The application project then packages up the service and its dependencies into a runnable JAR file.

As the quick start example project uses Apache Maven as a build tool, at the root of the project we have a _reactor pom.xml_.

    $ pwd
    ~/quickstart 
    $ ls 
    app     impl     pom.xml
{: .shell }


## The Reactor POM

The reactor POM is responsible for setting common configuration for the build plugins used by modules in the build, and for setting the scopes and versions of common dependencies. 

As the enRoute example projects all live in a single workspace each of their reactor poms inherit configuration from this root reactor. In scenarios where application projects have their own dedicated workspaces, then the following items would be included directly in each of their reactor poms.

The root reactor pom defines configuration for the [bnd plugins](/about/115-bnd-plugins.html) used by enRoute, and the following common dependencies.

### APIs
The OSGi and Java EE APIs commonly used in OSGi enRoute applications are included at provided scope. This is because they should not be used at runtime, instead being provided by implementations, such as the OSGi framework.

### Implementations
The OSGi reference implementations used in OSGi enRoute are included at runtime scope so that they are eligible to be selected for use by the application at runtime, but not available to compile against.

### Debug and Test
The remaining dependencies are made available at test scope so that they may be used when unit testing, integration testing, or debugging OSGi enRoute applications.


## The REST Module

The REST module contains the main bundle for the application. It contains a simple declarative services component in under `src/main/java`, and a unit test in `src/test/java`. The `src/main/resources` folder contains files contributing a Web User Interface for the component.

### The POM

The pom.xml is simple, it includes the OSGi API, enterprise API and testing dependencies required by the module, and it activates the `bnd-maven-plugin` which will generate the OSGi manifest, and a Declarative Service descriptor for the component.

### The DS Component
The DS component contains a number of important annotations.

<p>
  <a class="btn btn-primary" data-toggle="collapse" href="#Upper" aria-expanded="false" aria-controls="Upper">
    Upper.java
  </a>
</p>
<div class="collapse" id="Upper">
  <div class="card card-block">
{% highlight java tabsize=4 %}
{% include osgi.enroute/examples/quickstart/rest/src/main/java/org/osgi/enroute/examples/quickstart/rest/Upper.java %}
{% endhighlight %}

  </div>
</div>

* **@Component** - This annotation indicates that `Upper` is a [Declarative Services](../FAQ/300-declarative-services) component. The `service` attribute means that even though `Upper` does not directly implement any interfaces it will still be registered as a service. The `@Component` annotation also acts as a runtime _Requirement_; prompting the host OSGi framework to automatically load a Declarative Services implementation.

* **@JaxrsResource** - This annotation marks the `Upper` service as a JAX-RS resource type that should be processed by the [JAX-RS whiteboard](../FAQ/400-patterns.html#whiteboard-pattern). It also acts as a runtime _Requirement_; prompting the host OSGi framework to automatically load a JAX-RS Whiteboard implementation.

* **@HttpWhiteboardResource** - This annotation indicates to the [Http Whiteboard](../FAQ/400-patterns.html#whiteboard-pattern) that the `Upper` bundle contains one or more static files which should be served over HTTP. The pattern attribute indicates the URI request patterns that should be mapped to resources from the bundle, while the prefix attribute indicates the folder within the bundle where the resources can be found.

The @Path, @GET and @PathParam annotations are defined by JAX-RS, and used to map incoming requests to the resource method “toUpper(String)” which converts the incoming String to upper case and then returns it.

### The Unit Test

The unit test makes use of JUnit 4 to perform a basic test on the Upper service.

## The app module

The app module is responsible for gathering together the rest module and all of its dependencies into a runnable OSGi application.

### The POM

The pom.xml is simple, it references the rest module, the OSGi reference implementations, and the OSGi debug bundles.

To create the exported application requires an OSGi repository index, so this module enables the `bnd-indexer-maven-plugin`, and it uses the `bnd-export-maven-plugin` to export the `app.bndrun` file.

Finally this project enables the `bnd-resolver-maven-plugin` for the `app.bndrun` and `debug.bndrun`, so that these files may be resolved from the command line.

### The app.bndrun

The `app.bndrun` sets up the requirements and launch parameters for the quickstart application.

Firstly the file defines the index that will be used by this bndrun, and uses it to configure this as a standalone bndrun file (meaning that it does not inherit from the rest of the workspace) so that only items from the index will be used when resolving.

Next the file defines the run requirements using `-runrequires`. This uses the OSGi requirement syntax to select the rest module.
Finally the file declares the OSGi framework implementation that should be used, and the Java version that should be assumed when resolving.

The remaining section `-runbundles` is automatically when resolving. It contains the complete list of bundles that have been determined to be needed to run the application.

If you are in an IDE then this file can be run and resolved directly, otherwise the file can be resolved from the command line and run as an executable JAR file.

### The debug.bndrun

The `debug.bndrun` inherits the requirements and launch parameters from the `app.bndrun`, and adds:
* An additional repository index for the test bundles
* Requirements for the Felix Gogo shell and the Felix web console

As a result when this bndrun file is resolved and run it includes the main application and the debug bundles, making this an easy way to debug the application when running in an IDE. If command line debugging is preferred then the export maven plugin can be reconfigured to export the `debug.bndrun`, rather than the `app.bndrun`.
