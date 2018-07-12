### resources

>- ​

### filter

>Sometimes a resource file will need to contain a value that can only be supplied at build time. To accomplish this in Maven, put a reference to the property that will contain the value into your resource file using the syntax `${<property name>}`. The property can be one of the values defined in your pom.xml, a value defined in the user's settings.xml, a property defined in an external properties file, or a system property.
>
>To have Maven filter resources when copying, simply set `filtering` to true for the resource directory in your `pom.xml`:
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
>  <modelVersion>4.0.0</modelVersion>
> 
>  <groupId>com.mycompany.app</groupId>
>  <artifactId>my-app</artifactId>
>  <version>1.0-SNAPSHOT</version>
>  <packaging>jar</packaging>
> 
>  <name>Maven Quick Start Archetype</name>
>  <url>http://maven.apache.org</url>
> 
>  <dependencies>
>    <dependency>
>      <groupId>junit</groupId>
>      <artifactId>junit</artifactId>
>      <version>4.11</version>
>      <scope>test</scope>
>    </dependency>
>  </dependencies>
> 
>  <build>
>    <resources>
>      <resource>
>        <directory>src/main/resources</directory>
>        <filtering>true</filtering>
>      </resource>
>    </resources>
>  </build>
></project>
>```
>
>You'll notice that we had to add the `build`, `resources`, and `resource` elements which weren't there before. In addition, we had to explicitly state that the resources are located in the src/main/resources directory. All of this information was provided as default values previously, but because the default value for `filtering` is false, we had to add this to our pom.xml in order to override that default value and set `filtering` to true.
>
>To reference a property defined in your pom.xml, the property name uses the names of the XML elements that define the value, with "pom" being allowed as an alias for the project (root) element. So `${project.name}` refers to the name of the project, `${project.version}` refers to the version of the project, `${project.build.finalName}` refers to the final name of the file created when the built project is packaged, etc. Note that some elements of the POM have default values, so don't need to be explicitly defined in your `pom.xml` for the values to be available here. Similarly, values in the user's `settings.xml` can be referenced using property names beginning with "settings" (for example, `${settings.localRepository}` refers to the path of the user's local repository).
>
>To continue our example, let's add a couple of properties to the `application.properties` file (which we put in the `src/main/resources` directory) whose values will be supplied when the resource is filtered:
>
>```
># application.propertiesapplication.name=${project.name}application.version=${project.version}
>```
>
>With that in place, you can execute the following command (process-resources is the build lifecycle phase where the resources are copied and filtered):
>
>```
>mvn process-resources
>```
>
>and the `application.properties` file under `target/classes` (and will eventually go into the jar) looks like this:
>
>```
># application.propertiesapplication.name=Maven Quick Start Archetypeapplication.version=1.0-SNAPSHOT
>```
>
>To reference a property defined in an external file, all you need to do is add a reference to this external file in your pom.xml. First, let's create our external properties file and call it `src/main/filters/filter.properties`:
>
>```
># filter.propertiesmy.filter.value=hello!
>```
>
>Next, we'll add a reference to this new file in the `pom.xml`:
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
>  <modelVersion>4.0.0</modelVersion>
> 
>  <groupId>com.mycompany.app</groupId>
>  <artifactId>my-app</artifactId>
>  <version>1.0-SNAPSHOT</version>
>  <packaging>jar</packaging>
> 
>  <name>Maven Quick Start Archetype</name>
>  <url>http://maven.apache.org</url>
> 
>  <dependencies>
>    <dependency>
>      <groupId>junit</groupId>
>      <artifactId>junit</artifactId>
>      <version>4.11</version>
>      <scope>test</scope>
>    </dependency>
>  </dependencies>
> 
>  <build>
>    <resources>
>      <resource>
>        <directory>src/main/resources</directory>
>        <filtering>true</filtering>
>      </resource>
>    </resources>
>  </build>
> 
>  <properties>
>    <my.filter.value>hello</my.filter.value>
>  </properties>
></project>
>```
>
>Then, if we add a reference to this property in the `application.properties` file:
>
>```
># application.propertiesapplication.name=${project.name}application.version=${project.version}message=${my.filter.value}
>```
>
>the next execution of the `mvn process-resources` command will put our new property value into `application.properties`. As an alternative to defining the my.filter.value property in an external file, you could also have defined it in the `properties` section of your `pom.xml` and you'd get the same effect (notice I don't need the references to `src/main/filters/filter.properties` either):
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
>  <modelVersion>4.0.0</modelVersion>
> 
>  <groupId>com.mycompany.app</groupId>
>  <artifactId>my-app</artifactId>
>  <version>1.0-SNAPSHOT</version>
>  <packaging>jar</packaging>
> 
>  <name>Maven Quick Start Archetype</name>
>  <url>http://maven.apache.org</url>
> 
>  <dependencies>
>    <dependency>
>      <groupId>junit</groupId>
>      <artifactId>junit</artifactId>
>      <version>4.11</version>
>      <scope>test</scope>
>    </dependency>
>  </dependencies>
> 
>  <build>
>    <resources>
>      <resource>
>        <directory>src/main/resources</directory>
>        <filtering>true</filtering>
>      </resource>
>    </resources>
>  </build>
> 
>  <properties>
>    <my.filter.value>hello</my.filter.value>
>  </properties>
></project>
>```
>
>Filtering resources can also get values from system properties; either the system properties built into Java (like `java.version` or `user.home`) or properties defined on the command line using the standard Java -D parameter. To continue the example, let's change our `application.properties` file to look like this:
>
>```
># application.propertiesjava.version=${java.version}command.line.prop=${command.line.prop}
>```
>
>Now, when you execute the following command (note the definition of the command.line.prop property on the command line), the `application.properties` file will contain the values from the system properties.
>
>```
>mvn process-resources "-Dcommand.line.prop=hello again"
>```

### build

>build的种类有俩种，分别是基本的build和profile的build
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
>  ...
>  <!-- "Project Build" contains more elements than just the BaseBuild set -->
>  <build>...</build>
> 
>  <profiles>
>    <profile>
>      <!-- "Profile Build" contains a subset of "Project Build"s elements -->
>      <build>...</build>
>    </profile>
>  </profiles>
></project>
>```
>
>#### The BaseBuild Element Set
>
>```
>BaseBuild is exactly as it sounds: the base set of elements between the two build elements in the POM.
>
><build>
>  <defaultGoal>install</defaultGoal>
>  <directory>${basedir}/target</directory>
>  <finalName>${artifactId}-${version}</finalName>
>  <filters>
>    <filter>filters/filter1.properties</filter>
>  </filters>
>  ...
></build>
>```
>
>- **defaultGoal**: the default goal or phase to execute if none is given. If a goal is given, it should be defined as it is in the command line (such as `jar:jar`). The same goes for if a phase is defined (such as install).
>- **directory**: This is the directory where the build will dump its files or, in Maven parlance, the build's target. It aptly defaults to `${basedir}/target`.
>- **finalName**: This is the name of the bundled project when it is finally built (sans the file extension, for example: `my-project-1.0.jar`). It defaults to `${artifactId}-${version}`. The term "finalName" is kind of a misnomer, however, as plugins that build the bundled project have every right to ignore/modify this name (but they usually do not). For example, if the `maven-jar-plugin` is configured to give a jar a `classifier` of `test`, then the actual jar defined above will be built as `my-project-1.0-test.jar`.
>- **filter**:Defines `*.properties` files that contain a list of properties that apply to resources which accept their settings (covered below). In other words, the "`name=value`" pairs defined within the filter files replace `${name}` strings within resources on build. The example above defines the `filter1.properties` file under the `filter/` directory. Maven's default filter directory is `${basedir}/src/main/filters/`.
>
>### Resources
>
>Another feature of `build` elements is specifying where resources exist within your project. Resources are not (usually) code. They are not compiled, but are items meant to be bundled within your project or used for various other reasons, such as code generation.
>
>For example, a Plexus project requires a `configuration.xml` file (which specifies component configurations to the container) to live within the `META-INF/plexus` directory. Although we could just as easily place this file within `src/main/resources/META-INF/plexus`, we want instead to give Plexus its own directory of `src/main/plexus`. In order for the JAR plugin to bundle the resource correctly, you would specify resources similar to the following:
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
>  <build>
>    ...
>    <resources>
>      <resource>
>        <targetPath>META-INF/plexus</targetPath>
>        <filtering>false</filtering>
>        <directory>${basedir}/src/main/plexus</directory>
>        <includes>
>          <include>configuration.xml</include>
>        </includes>
>        <excludes>
>          <exclude>**/*.properties</exclude>
>        </excludes>
>      </resource>
>    </resources>
>    <testResources>
>      ...
>    </testResources>
>    ...
>  </build>
></project>
>```
>
>- **resources**: is a list of resource elements that each describe what and where to include files associated with this project.
>- **targetPath**: Specifies the directory structure to place the set of resources from a build. Target path defaults to the base directory. A commonly specified target path for resources that will be packaged in a JAR is META-INF.
>- **filtering**: is `true` or `false`, denoting if filtering is to be enabled for this resource. Note, that filter `*.properties` files do not have to be defined for filtering to occur - resources can also use properties that are by default defined in the POM (such as ${project.version}), passed into the command line using the "-D" flag (for example, "`-Dname`=`value`") or are explicitly defined by the properties element. Filter files were covered above.
>- **directory**: This element's value defines where the resources are to be found. The default directory for a build is `${basedir}/src/main/resources`.
>- **includes**: A set of files patterns which specify the files to include as resources under that specified directory, using * as a wildcard.
>- **excludes**: The same structure as `includes`, but specifies which files to ignore. In conflicts between `include` and `exclude`, `exclude` wins.
>- **testResources**: The `testResources` element block contains `testResource` elements. Their definitions are similar to `resource` elements, but are naturally used during test phases. The one difference is that the default (Super POM defined) test resource directory for a project is `${basedir}/src/test/resources`. Test resources are not deployed.
>
>
>
>### plugins
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
>  <build>
>    ...
>    <plugins>
>      <plugin>
>        <groupId>org.apache.maven.plugins</groupId>
>        <artifactId>maven-jar-plugin</artifactId>
>        <version>2.6</version>
>        <extensions>false</extensions>
>        <inherited>true</inherited>
>        <configuration>
>          <classifier>test</classifier>
>        </configuration>
>        <dependencies>...</dependencies>
>        <executions>...</executions>
>      </plugin>
>    </plugins>
>  </build>
></project>
>```
>
>Beyond the standard coordinate of `groupId:artifactId:version`, there are elements which configure the plugin or this builds interaction with it.
>
>- **extensions**: `true` or `false`, whether or not to load extensions of this plugin. It is by default false. Extensions are covered later in this document.
>- **inherited**: `true` or `false`, whether or not this plugin configuration should apply to POMs which inherit from this one. Default value is `true`.
>- **configuration**: This is specific to the individual plugin. Without going too in depth into the mechanics of how plugins work, suffice it to say that whatever properties that the plugin Mojo may expect (these are getters and setters in the Java Mojo bean) can be specified here. In the above example, we are setting the classifier property to test in the `maven-jar-plugin`'s Mojo. It may be good to note that all configuration elements, wherever they are within the POM, are intended to pass values to another underlying system, such as a plugin. In other words: values within a `configuration` element are never explicitly required by the POM schema, but a plugin goal has every right to require configuration values.
>
>If your POM declares a parent, it will inherit plugin configuration from either the **build/plugins** or **pluginManagement** sections of the parent.
>
>To illustrate, consider the following fragment from a parent POM:
>
>```
><plugin>
><groupId>my.group</groupId>
><artifactId>my-plugin</artifactId>
><configuration>
>  <items>
>    <item>parent-1</item>
>    <item>parent-2</item>
>  </items>
>  <properties>
>    <parentKey>parent</parentKey>
>  </properties>
></configuration>
></plugin>
>```
>
>And consider the following plugin configuration from a project that uses that parent as its parent:
>
>```
><plugin>
><groupId>my.group</groupId>
><artifactId>my-plugin</artifactId>
><configuration>
>  <items>
>    <item>child-1</item>
>  </items>
>  <properties>
>    <childKey>child</childKey>
>  </properties>
></configuration>
>```
>
>The default behavior is to merge the content of the **configuration** element according to element name. If the child POM has a particular element, that value becomes the effective value. if the child POM does not have an element, but the parent does, the parent value becomes the effective value. Note that this is purely an operation on XML; no code or configuration of the plugin itself is involved. Only the elements, not their values, are involved.
>
>Applying those rules to the example, Maven comes up with:
>
>```
><plugin>
><groupId>my.group</groupId>
><artifactId>my-plugin</artifactId>
><configuration>
>  <items>
>    <item>child-1</item>
>  </items>
>  <properties>
>    <childKey>child</childKey>
>    <parentKey>parent</parentKey>
>  </properties>
></configuration>
>```
>
>You can control how child POMs inherit configuration from parent POMs by adding attributes to the children of the **configuration** element. The attributes are `combine.children` and `combine.self`. Use these attributes in a child POM to control how Maven combines plugin configuration from the parent with the explicit configuration in the child.
>
>Here is the child configuration with illustrations of the two attributes:
>
>````
><configuration>
>  <items combine.children="append">
>    <!-- combine.children="merge" is the default -->
>    <item>child-1</item>
>  </items>
>  <properties combine.self="override">
>    <!-- combine.self="merge" is the default -->
>    <childKey>child</childKey>
>  </properties>
></configuration>
>````
>
>Now, the effective result is the following:
>
>```
><configuration>
>  <items combine.children="append">
>    <item>parent-1</item>
>    <item>parent-2</item>
>    <item>child-1</item>
>  </items>
>  <properties combine.self="override">
>    <childKey>child</childKey>
>  </properties>
></configuration>
>```
>
>**combine.children="append"** results in the concatenation of parent and child elements, in that order. **combine.self="override"**, on the other hand, completely suppresses parent configuration. You cannot use both both **combine.self="override"** and **combine.children="append"** on an element; if you try, *override* will prevail.
>
>Note that these attributes only apply to the configuration element they are declared on, and are not propagated to nested elements. That is if the content of an *item* element from the child POM was a complex structure instead of text, its sub-elements would still be subject to the default merge strategy unless they were themselves marked with attributes.
>
>The combine.* attributes are inherited from parent to child POMs. Take care when adding those attributes a parent POM as this might affect child or grand-child POMs.
>
>- **dependencies**: Dependencies are seen a lot within the POM, and are an element under all plugins element blocks. The dependencies have the same structure and function as under that base build. The major difference in this case is that instead of applying as dependencies of the project, they now apply as dependencies of the plugin that they are under. The power of this is to alter the dependency list of a plugin, perhaps by removing an unused runtime dependency via `exclusions`, or by altering the version of a required dpendency. See above under **Dependencies** for more information.
>
>- executions
>
>  : It is important to keep in mind that a plugin may have multiple goals. Each goal may have a separate configuration, possibly even binding a plugin's goal to a different phase altogether.
>
>   executions configure the execution of a plugin's goals.
>
>  For example, suppose you wanted to bind the `antrun:run` goal to the `verify` phase. We want the task to echo the build directory, as well as avoid passing on this configuration to its children (assuming it is a parent) by setting `inherited`to `false`. You would get an `execution` like this:
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
>  ...
>  <build>
>    <plugins>
>      <plugin>
>        <artifactId>maven-antrun-plugin</artifactId>
>        <version>1.1</version>
>        <executions>
>          <execution>
>            <id>echodir</id>
>            <goals>
>              <goal>run</goal>
>            </goals>
>            <phase>verify</phase>
>            <inherited>false</inherited>
>            <configuration>
>              <tasks>
>                <echo>Build Dir: ${project.build.directory}</echo>
>              </tasks>
>            </configuration>
>          </execution>
>        </executions>
> 
>      </plugin>
>    </plugins>
>  </build>
></project>
>```
>
>- **id**: Self explanatory. It specifies this execution block between all of the others. When the phase is run, it will be shown in the form: `[plugin:goal execution: id]`. In the case of this example: `[antrun:run execution: echodir]`
>- **goals**: Like all pluralized POM elements, this contains a list of singular elements. In this case, a list of plugin `goals` which are being specified by this `execution` block.
>- **phase**: This is the phase that the list of goals will execute in. This is a very powerful option, allowing one to bind any goal to any phase in the build lifecycle, altering the default behavior of Maven.
>- **inherited**: Like the `inherited` element above, setting this false will supress Maven from passing this execution onto its children. This element is only meaningful to parent POMs.
>- **configuration**: Same as above, but confines the configuration to this specific list of goals, rather than all goals under the plugin.
>
>### plugiment
>
>- **pluginManagement**: is an element that is seen along side plugins. Plugin Management contains plugin elements in much the same way, except that rather than configuring plugin information for this particular project build, it is intended to configure project builds that inherit from this one. However, this only configures plugins that are actually referenced within the plugins element in the children. The children have every right to override `pluginManagement` definitions.
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
>  ...
>  <build>
>    ...
>    <pluginManagement>
>      <plugins>
>        <plugin>
>          <groupId>org.apache.maven.plugins</groupId>
>          <artifactId>maven-jar-plugin</artifactId>
>          <version>2.6</version>
>          <executions>
>            <execution>
>              <id>pre-process-classes</id>
>              <phase>compile</phase>
>              <goals>
>                <goal>jar</goal>
>              </goals>
>              <configuration>
>                <classifier>pre-process</classifier>
>              </configuration>
>            </execution>
>          </executions>
>        </plugin>
>      </plugins>
>    </pluginManagement>
>    ...
>  </build>
></project>
>```
>
>If we added these specifications to the plugins element, they would apply only to a single POM. However, if we apply them under the `pluginManagement` element, then this POM *and all inheriting POMs* that add the `maven-jar-plugin` to the build will get the `pre-process-classes` execution as well. So rather than the above mess included in every child `pom.xml`, only the following is required:
>
>```
><project xmlns="http://maven.apache.org/POM/4.0.0"
>  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
>                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
>  ...
>  <build>
>    ...
>    <plugins>
>      <plugin>
>        <groupId>org.apache.maven.plugins</groupId>
>        <artifactId>maven-jar-plugin</artifactId>
>      </plugin>
>    </plugins>
>    ...
>  </build>
></project>
>```
>
>

