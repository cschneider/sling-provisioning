# sling-provisioning
Experimental project to explore a modular way to package and deploy sling

## Current state

The sling distribution is currently defined in launchpad/builder. There are txt file for each "feature".
Each such file can define variables, artifacts, configurations, repoinit commands. Some of the definitions only apply to certain runlevels.

* Variables define replacements inside the file
* Artifacts defines bundles to load per start level
* Configuration defines defult configs to apply to config admin
* Repoinit commands are executed on the oak repo. (Not sure how exactly)

## Problems with current approach

* The approach is quite exotic. There is no other software that is defined in this way. 
* Another problem seems to be that sling as well as AEM are currently only built in one way. So it is not easy to define smaller deployments with only a subset of the features.
* Each feature must list all bundles it needs. There is no good way to check if a feature or planned packaging should work before actually starting it.

## Proposed approach

I propose an approach that is very near to the OSGi standards and is common in the bndtools community.
The idea is to describe the deployment by using one or more backing repositories and requirements and use the resolver to compute a closure around the requirements. Now the question is of course how can we translate the specific descriptions from sling the sling builder into standard OSGi descriptions of a packaging.

### Repository

The easiest way to create a repository is to use a simple pom and the bnd-indexer-plugin. This approach is very familiar to anyone who knows maven and is also supported by eclipse tooling (context based search in maven repos, incremental build integration).

### Features

The sling features can be described as bundles. So the idea is to translate the different descriptions in the current txt feature files into contents of a bundle.

* Feature identification: Currently each feature has a name in sling. We can tranlate this into a naming scheme for the symbolic name of the feature. Like: "feature.sling.event". Additionally we can define a special capability to formally name the feature.

* Configuration: The OSGi configirator spec defines that configuration in a special place in a bundle is automatically applied at runtime. Se we simply need to place the config in the correct location.

* Artifacts: Instead of listing each artifact of a feature we would only list the top level bundles and let the resolver compute the rest. In many cases a feature would then only need to require one single top level bundle. In this case we could then even combine feature description and bundle. So the feature does not even need an extra artifact.

* Repoinit: There is no existing solution to this but we can try to apply the extender appraoch. Like for the configurator we can define a special place in the bundle. /OSGI-INF/oak/* might be a good place. Each bundle/feature with a repoinit script would define a requirement for a repoinit extender that applies the scripts to the repository.

# Advantages of the new model

* Uses only OSGi standard artifacts to describe features (bundles, requirements, capabilities, repository)
* We can start an debug sling and even AEM from bndtools. A simply bndrun file with an index and requirements for the features should be enough to get a custom sling or AEM distro running.
* We can apply the same approach for itests. An itest would use the same combination of requirements and repository to describe a test instance
* Pax exam will get support for such deployments. So we also have a good test tooling without additional effort

# Possible problems

* Resolving very many bundles can take long. We have to test how the approach scales to the size of AEM
