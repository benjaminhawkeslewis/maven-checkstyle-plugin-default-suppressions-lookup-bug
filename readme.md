# Maven Checkstyle Plugin Default Suppressions Lookup Bug

This codebase defines a parent Maven module and two submodules:

- shared-checks defines a shared Checkstyle configuration XML file referencing
  a default suppressions configuration (another XML file). Both files are
  included in the packaged jar.
- checked-module applies the Maven Checkstyle plugin and pulls in the
  shared-checks as a dependency for the plugin's execution. checkstyle.xml is
  successfully loaded out of the classpath, but suppressions.xml referenced by
  checkstyle.xml is not.

Reproduce by running:

    mvn -U clean checkstyle:check package

Yielding the error:

    [ERROR] Failed to execute goal org.apache.maven.plugins:maven-checkstyle-plugin:2.12.1:check (default-cli) on project checked-module: Failed during checkstyle configuration: cannot initialize module SuppressionFilter - Cannot set property 'file' in module SuppressionFilter to 'suppressions.xml': unable to find suppressions.xml: InvocationTargetException -> [Help 1]

The stack trace shows this occurs when attempting to load the file from the
classpath:

    Caused by: java.io.FileNotFoundException: suppressions.xml
        at com.puppycrawl.tools.checkstyle.filters.SuppressionsLoader.loadSuppressions(SuppressionsLoader.java:165)

Checkstyle maintainers consider this to be a problem with Maven Checkstyle
plugin's classloader manipulation: https://github.com/checkstyle/checkstyle/issues/62
