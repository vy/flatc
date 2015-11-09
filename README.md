According to the [official FlatBuffers project](https://github.com/google/flatbuffers),

> FlatBuffers is a serialization library for games and other memory
> constrained apps. FlatBuffers allows you to directly access serialized
> data without unpacking/parsing it first, while still having great
> forwards/backwards compatibility.

Unfortunately, FlatBuffers project does not have a native Java compiler.
This project provides a set of Maven artifacts containing platform-dependent
`flatc` binaries. The list of supported platforms are as follows:

- Linux (x86-64)
- OSX (x86-64)

Usage
=====

Following POM content shows how you can purpose `flatc` artifacts to compile
`*.fbs` files under `src` directory.

```xml
<properties>

    <!-- fbs paths -->
    <fbs.input.directory>src</fbs.input.directory>
    <fbs.output.directory>${project.build.directory}/generated-sources</fbs.output.directory>

    <!-- library versions -->
    <build-helper-maven-plugin.version>1.9.1</build-helper-maven-plugin.version>
    <fbs.version>1.2.0-3f79e055</fbs.version>
    <maven-antrun-plugin.version>1.8</maven-antrun-plugin.version>
    <maven-compiler-plugin.version>3.3</maven-compiler-plugin.version>
    <maven-dependency-plugin.version>2.10</maven-dependency-plugin.version>
    <os-maven-plugin.version>1.4.1.Final</os-maven-plugin.version>

</properties>

<dependencies>

    <!-- provides com.google.flatbuffers package -->
    <dependency>
        <groupId>com.vlkan</groupId>
        <artifactId>flatbuffers</artifactId>
        <version>${fbs.version}</version>
    </dependency>

</dependencies>

<build>

    <extensions>

        <!-- provides os.detected.classifier (i.e. linux-x86_64, osx-x86_64) property -->
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>${os-maven-plugin.version}</version>
        </extension>

    </extensions>

    <plugins>

        <!-- copy flatc binary into build directory -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>${maven-dependency-plugin.version}</version>
            <executions>
                <execution>
                    <id>copy-flatc</id>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>copy</goal>
                    </goals>
                    <configuration>
                        <artifactItems>
                            <artifactItem>
                                <groupId>com.vlkan</groupId>
                                <artifactId>flatc-${os.detected.classifier}</artifactId>
                                <version>${fbs.version}</version>
                                <type>exe</type>
                                <overWrite>true</overWrite>
                                <outputDirectory>${project.build.directory}</outputDirectory>
                            </artifactItem>
                        </artifactItems>
                    </configuration>
                </execution>
            </executions>
        </plugin>

        <!-- compile fbs files using copied flatc binary -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>${maven-antrun-plugin.version}</version>
            <executions>
                <execution>
                    <id>exec-flatc</id>
                    <phase>generate-sources</phase>
                    <configuration>
                        <tasks>
                            <property name="flatc.filename" value="flatc-${os.detected.classifier}-${fbs.version}.exe"/>
                            <property name="flatc.filepath" value="${project.build.directory}/${flatc.filename}"/>
                            <chmod file="${flatc.filepath}" perm="ugo+rx"/>
                            <mkdir dir="${fbs.output.directory}" />
                            <path id="fbs.path">
                                <fileset dir="${fbs.input.directory}">
                                    <include name="**/*.fbs"/>
                                </fileset>
                            </path>
                            <pathconvert pathsep=" " property="fbs.files" refid="fbs.path"/>
                            <exec executable="${flatc.filepath}" failonerror="true">
                                <arg value="-o"/>
                                <arg value="${fbs.output.directory}"/>
                                <arg value="-I"/>
                                <arg value="${fbs.input.directory}"/>
                                <arg value="-j"/>
                                <arg line="${fbs.files}"/>
                            </exec>
                        </tasks>
                    </configuration>
                    <goals>
                        <goal>run</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- add generated flat buffer classes into the package -->
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>build-helper-maven-plugin</artifactId>
            <version>${build-helper-maven-plugin.version}</version>
            <executions>
                <execution>
                    <id>add-classes</id>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>add-source</goal>
                    </goals>
                    <configuration>
                        <sources>
                            <source>${fbs.output.directory}</source>
                        </sources>
                    </configuration>
                </execution>
            </executions>
        </plugin>

    </plugins>

</build>
```

A Note on Versioning
====================

Official FlatBuffers project does not follow a versioning scheme as
of this writing. In order to the report the version of the used clone,
I used the following fields:

- the most recent Java API version (1.2.0-SNAPSHOT)
- the first 8 digits of the clone's last commit I had used to compile the binaries

License
=======

As is the case for the official FlatBuffers repository, this fork is also
licensed under the terms of Apache License, Version 2.0.
