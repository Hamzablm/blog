---
title: Apache Maven&#58; Working With Build Profiles
author: Hamza Belmellouki
categories: [Build Tools]
tags: [maven]
comments: true
--- 

In this article, we’re going to take a peek at Maven Build Profiles. Build profiles are often underestimated yet they have a lot of capabilities, and we are going to explore some of its capabilities. How we can use it? how we can leverage build profiles to make our build flexible?

## 2. Overview of Maven Build Profiles

The power of a profile comes from its ability to modify the basic POM only under certain circumstances. A profile is an alternative set of values that set or override default values.

### 2.1 Why Use Profiles?

* Profiles allow you to customize a particular build for a particular environment (development, testing, and production).

* Profiles facilitate portability between different build environments.

* It may also be used in a situation where you need a different build based on the OS, JDK.

* To activate specific plugins

* To provide alternative build-time configuration values

### 2.2 Declaring Profiles

You can declare build profiles in one of the following ways:

* In pom.xml (Use this when you want your build to be portable between different computers)

* Declared In `<user-home>/.m2/settings.xml` (use this when you want the profile to be available to many projects)

## 3. Simple Example

This example demonstrates how to declare and leverage profiles in your pom.xml file. You declare a profile inside `<profile>` element like so:

```xml
<profiles>

        <profile>
            <id>prod</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <version>2.22.2</version>
                        <configuration>
                            <skipTests>true</skipTests>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>3.8.1</version>
                        <configuration>
                            <debug>false</debug>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>

        <profile>
            <id>itTest</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-failsafe-plugin</artifactId>
                        <version>2.22.2</version>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>integration-test</goal>
                                    <goal>verify</goal>
                                </goals>
                                <configuration>
                                    <includes>
                                        <include>**/*IT.java</include>
                                    </includes>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>

    </profiles>
```

This block of code declares 2 profiles: *prod*(production) and itTest(integration testing) and declares and configures plugins for production and integration testing (of course this is just a simple example to demo profiles. Don’t take it as a robust configuration). Now each profile must have an `<id>` element. By default, *prod* profile is *not* active and *itTest* profile is active.

To view active profiles run this command:

```console
mvn help:active-profiles
```

You can activate them by using one of the following ways (there is a lot of options):

* Using a command-line argument

* In settings.xml in `<activeProfiles>` element

* Using `<activeByDefault>` element inside the `<profile>` element

* Activation based on JDK versions, OS

* Present or missing files

* Environment variable

### 3.1 Command-Line

You activate profiles by using the command-line *-P* option, for example, here I am running default lifecycle until package phase with *prod* as the active profile.

If you run this command you’ll notice that the tests did not run because we’re skipping them in this build, also we’ve set `<debug>` element to ‘true’ in the compiler plugin to *not* include debugging information in the compiled class files:

```console
mvn package -P prod
```

You can deactivate profiles by using an exclamation mark(!) or a dash(-), for example here I am deactivating *itTest* profile (Remember *itTest* is set up active by default):

```console
mvn verify -P -itTest
```

This command deactivates *itTest* profile, thus it will not run integration tests.

### 3.2 settings.xml

You can activate profiles in *settings.xml* (located under `/Users/<user-home>/.m2`) by using `<activeProfiles\>` element. For example, here I am activating *prod* profile. By activating that profile unit tests will be skipped and there will be no debugging information in the bytecode:
```xml
<activeProfiles>
    <activeProfile>prod</activeProfile>
</activeProfiles>
```

### 3.3 <*activeByDefault*> element

Another useful trick to activate profiles is to use `<activeByDefault\>` element right in the pom.xml. We’ve already used that in *itTest* profile to activate it by default:
```xml
<activation>
    <activeByDefault>true</activeByDefault>
</activation>
```

### 3.5 OS settings

Use `<os>` element if you want the profile build to be activated based on the OS. I’ve never used this feature before, but I think it would be good to know it. This *windows* profile will trigger when the system is windows XP:
```xml
<profile>
    <id>windows</id>
    <activation>
        <os>
            <name>Windows XP</name>
            <family>Windows</family>
            <arch>x86</arch>
            <version>5.1.2600</version>
        </os>
    </activation>
</profile>
```

### 3.6 Files

Use `<file>` element if you want to activate a profile based on the absence or presence of a file. For example, this profile will be active if data.sql exists If not the profile will not activate:
```xml
<profile>
    <id>file-present</id>
    <activation>
        <file>
            <exists>target/resources/data.sql</exists>
        </file>
    </activation>
</profile>
```

### 3.7 Environment Variable

Use `<property>` element if you want the profile build to be activated based on environment variables. For example:
```xml
<profile>
    <id>test</id>
    <activation>
        <property>
            <name>env.FOO</name>
            <value>test</value>
        </property>
    </activation>
</profile>
```

The test profile will trigger when the system property “FOO” is specified with the value “test”. Create an environment variable “FOO” and set its value as “test” if you want to trigger the profile.

## 4. Conclusion

This article was a quick overview of Maven profiles, we talked about why to use them, how to declare them, and how to activate them. In the next article, we’ll step back from Maven and talk about Core Java. Stay tuned!

Let us know what you think in the comments below and don’t forget to share!
