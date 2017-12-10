# Gradle Basic Demo

create a simple gradle project(https://guides.gradle.org/creating-new-gradle-builds/)

### Run task command

gradle tasks

### Generate gradle wrapper

gradle wrapper

### Run the properties task

./gradlew properties

### Configure a Gradle core task

in a build.gradle file

task <taskname>( options ) {   
	...code...  
}  
  
exampels)  
  
tasks copy(type: Copy) {  
	from 'src'  
	into 'dest'  
}  

### Configure a task and use a plugins

ina build.gradle file

plugins {  
	id '<plugins names>'  
}  
  
"plugins" should be written top of the file.



