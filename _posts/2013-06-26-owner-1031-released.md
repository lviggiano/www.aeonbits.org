---
layout: post
title: "OWNER 1.0.3.1 released"
description: "first maintenance release for OWNER"
category: owner 
tags: [annotations, bsd, configuration, free, library, maven, owner, owner api, properties, release, announcement]
---
{% include JB/setup %}

Yesterday night I got a bug report by email. Ivan defined his `BechmarkConfig` class with an associated `BenchmarkConfig.properties` file in the classpath, but he wanted to be able to specify a properties file at command line to override its contentent. So he used the imported properties feature, but surprise... the imported properties have the lowest priority, so the BenchmarkConfig.properties was prevailing.
I filled the bug report [35](https://github.com/lviggiano/owner/issues/35) yesterday night, and started immediately to work to help.

I implemented the imported properties to have the lowest priority by purpose, but when I read the use case that Ivan was trying to implement, it was so obvious that what I supposed to be the correct behavior was in fact wrong. That behavior was not documented anywhere, but there was a unit test ensuring that the imported properties had the lowest priority in overriding the properties values. But I feel that it was a mistake, so I changed it and released the first maintenance release, the 1.0.3.1. After getting the confirmation from Ivan that the new library jar was working as expected, I released the new version in the maven central repository. 
Git branching was helpful to make things easier and fast, and the Sonatype system to release on maven was incredibly efficient, to allow Ivan to get his fixed jar from maven directly, two hours after he sent me the email to report his problem.

Now... what if somebody comes out expecting the opposite behavior? (the imported properties having the lower priority as before) 
Well, in this case I need to implement some mechanism to allow the user to specify when imported properties should override - or not - the files loaded with the `@Sources` annotation or by default the corresponding file from the classpath. I have already a couple of different ideas on how this can be implemented. But... I will not add this until I find a somebody who actually needs it. And like it happened with Ivan, I will be happy to provide a solution as quickly as I can.

OWNER version 1.0.4 (or maybe be 1.1.0) will be available soon. Many things have been added and the source code has been deeply restructured. Some interesting additions have been made to the public interfaces, but everything should work as before for the people who are already using it.
Yesterday I added Sonar to the development process to keep eyes open over code quality and the growth of the codebase (reports [here](sheldon.dyndns.tv:9000/dashboard/index/1)). Surprisingly, even though more feature have been added since 1.0.3, the master branch is not growing very much, and the code base has been kept slim. 1.0.4 will be a major addition to the features in OWNER APIs.

