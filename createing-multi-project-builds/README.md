# Gradle Multi-Project Build Demo

create a gradle for multi project build(https://guides.gradle.org/creating-multi-project-builds/)

### Run gradle init
```
$ gradle init  
```
This command will create build.gradle, settings.gradle, gradlew

### Check settings.gradle

You can only one line except comments.  
rootProject.name = 'creating-multi-project-builds'

### Setting build.gradle in root project

Open build.gradle file and add following codes.
```
allprojects {
    repositories {
        jcenter()
    }
}
subprojects {
    version = '1.0'
}
```

"allprojects" block is used to add configuration items will apply to all sub-projects including root project.  
"subprojects" block is used to add configuration items only apply to all sub-projects.

### Create Groovy library project

Create greeting-library folder in root project.
Open build.gradle in greeing-library folder.  
Add following codes.
```
apply plugin: 'groovy'

dependencies {
    compile 'org.codehaus.groovy:grvooy:2.4.10'

    testCompile 'org.spockframework:spock-core:1.0-groovy-2.4', {
        exclude module: 'groovy-all'
    }
}
```

Create src/main/groovy/greeter, src/test/groovy/greeter folder and create following source code in each folder.

greeting-libarary/src/main/groovy/greeter/GreetingFormatter.groovy
```
package greeter

import groovy.transform.CompileStatic

@CompileStatic
class GreetingFormatter {
    static String greeting(final String name) {
        "Hello, ${name.capitalize()}"
    }
}
```
  
greeting-library/src/test/groovy/greeter/GreetingFormatterSpec.groovy
```
package greeter

import spock.lang.Specification

class GreetingFormatterSpec extends Specification {
    def 'Creating a greeting'() {
        
        expect: 'The greeting to be correctly capitalized'
        GreetingFormatter.greeting('gradlephant') == 'Hello, Gradlephant'
    }
}
```

open settings.gradle in root directory and add following code
  
settings.gradle
```
include 'greeting-library'
```

### Create Java Application Project

Create greeter folder and open build.gradle

greeter/build.gradle
```
apply plugin: 'java'
apply plugin: 'application'
```

In greeter, create directory for source code.
```
$ mkdir -p greeter/src/main/java/greeter
```

greeter/src/main/java/greeter/Greeter.java
```
package greeter;

public class Greeter {
    public static void main(String[] args) {
        final String output = GreetingFormatter.greeting(args[0]);
        System.out.println(output);
    }
}
```

You have to tell Gradle the name of the class which is the entry point.  
Open build.gradle in greeter project.  

greeter/build.gradle
```
mainClassName = 'greeter.Greeter'
```

This java application depends on groovy-library created by us.  
Adding dependencies in build.gradle.  

greeter/build.gradle
```
dependencies {
    compile project(':greeting-library')
}
```

To test your application, add test code.  

greeter/src/test/groovy/greeter/GreeterSpec.groovy
```
package greeter
import spock.lang.Specification

class GreeterSpec extends Specification {
    def 'Calling the entry point'() {
        setup: 'Re-route standard out'
        def buf = new ByteArrayOutputStream(1024)
        System.out = new PrintStream(buf)
        
        when: 'The entrypoint is executed'
        Greeter.main('gradlephant')
        
        then: 'The correct greeting is output'
        buf.toString() == "Hello, Gradlephant\n"
    }
}
```

add dependency in greeter/build.gradle
```
dependencies {
    testCompile 'org.spockframework:spock-core:1.0-groovy-2.4', {
        exclude module: 'groovy-all'
    }
}
```

### Add documentation

Open build.gradle in root

build.gradle
```
plugins {
    id 'org.asciidoctor.convert' versions '1.5.3' apply false
}
```

Create docs folder and add build.gradle file in docs.
```
$ mkdir docs
$ touch docs/build.gradle
```

Add following contents to build.gradle

docs/build.gradle
```
apply plugin: 'org.asciidoctor.convert'

asciidoctor {
    sources { 
        include 'greeter.adoc'
    }
}
build.dependsOn 'asciidoctor'
```

To build, add following sentences settings.gradle in root

settings.gradle
```
include 'docs'
```

Create documents in docs.

docs/src/docs/asciidoc/greeter.adoc
```
= Greeter Command-line Application

A simple application demonstrating the flexibility of a Gradle multi-project.

== installation

Unpack the ZIP or TAR file in a suitable location

== Usage
[listing]
----
$ cd greeting-1.0
$ ./bin/greeter gradlephant

Hello, Gradlephant
----
```

run gradlew asciidoctor
```
$ ./gradlew asciidoctor
```

### Include document in distribution archive
greeter/build.gradle
```
distZip {
    from project(':docs').asciidoctor, {
        into "${project.name}-${version}"
    }
}
distTar {
    from project(':docs').asciidoctor, {
        into "${project.name}-${version}"
    }
}
```
Use project(:NAME).TASKNAME foramt to reference a task instance in another project.

### Refactor build.gradle
in greeter/build.gradle and greeting-library/build.gradle, Some contents are duplicated.
Gradle provides a ability to place such common build script code in the root project

in build.gradle
```
configure(subproject.findAll {it.name == 'greeter' || it.name == 'greeting-libraray'} ) {
    apply plugin: 'groovy'
    dependencies {
        testCompile 'org.spockframework:spock-core:1.0-groovy-2.4', {
            exclude module: 'groovy-all'
        }
    }
}
```

duplicated content in greeting-library/build.gradle and greeter/build.gradle can be removed.


