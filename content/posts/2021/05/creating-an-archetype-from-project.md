---
title: "Creating an archetype from an existing project"
date: 2021-05-29T15:13:31+02:00
draft: false
tags: ["maven", "archetypes"]
categories: ["development"]
ShowToc: true
TocOpen: false
weight: 2
---

If you are creating a cloud of microservices or are in an environment where you have to generate multiple maven projects or modules, it is very useful to create archetypes to handle these new modules/projects creation.

## What is an archetype?
[Maven archetype](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html) is a templated project. Maven offers several archetypes that allow you to create new maven modules ready to run in few seconds.

## How to use a generic archetype?
[Archetype usage](https://maven.apache.org/archetype/maven-archetype-plugin/usage.html) is quite simple. If you want to generate an archetype based on the default list, just run:

```
mvn archetype:generate
```
And then choose one of the options, provide the details and confirm. The new project will be already created.

## How to use a custom archetype?

In this case, we need to specify the groupId and artifactId of the archetype to be able to use a custom one. As example, let's see the [archetype provided by Adobe for AEM](https://github.com/adobe/aem-project-archetype):

```
mvn archetype:generate  \
     -DarchetypeGroupId=com.adobe.aem \
     -D archetypeArtifactId=aem-project-archetype \
     -D archetypeVersion=27
```

If you run the command, you will be asked about some properties of your project. Some properties are standard (`groupId`, `version`, `artifactId` and `package`), but others were specific of this artifact, like `appTitle` or `sdkVersion`. These properties will help the archetype to generate a new project for you and in few seconds and knowing nothing about Adobe AEM you have your new project ready to run.

## How the archetype works?

Internally, an archetype has all the files (pom file, classes, readme, etc...) with variables that will be replaced during project generation. As example, you can see these variables in several parts of the pom file below.

```xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>${groupId}</groupId>
        <artifactId>${rootArtifactId}</artifactId>
        <version>${version}</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <artifactId>${artifactId}</artifactId>
    <name>${appTitle} - Core</name>
    <description>Core bundle for ${appTitle}</description>
...
```
Example extracted from [Github adobe/aem-project-archetype project](https://github.com/adobe/aem-project-archetype/blob/71d3bbcba7ec37897910ca8042b61004f0963c34/src/main/archetype/core/pom.xml#L17-L28)

Maven has a [good documentation how to organize and create your archetype](https://maven.apache.org/guides/mini/guide-creating-archetypes.html) from scratch.

## Creating an archetype from a project

In most of the cases, it is hard to maintain an archetype because:
1. It is not a project itself, so you need to use your archetype to see if the generated project is working.
2. Templating is good, but it is hard to read and you need to "imagine" how the code will look like after.
3. We are lazy, and if it is hard to understand, it will be hard to maintain. People will go back to copy-paste projects and replace the text.

A simple way to bypass all these constraints is to create an archetype from an example project. Our example project is a perfect compilable project with all desired configurations we want to keep together in the clones. 

### Create your archetype example project

Some characteristics are desired in the archetype example project:
1. It should be simple, but complete. Remember to include all dependencies and one example of each element of your project. For example, if the project is a microservice, it makes sense to include a controller, a service and a repository.
2. Name it in a coherent manner (you  will see it in the next steps). I would recommend, for example, having a controller called `ArchetypeExampleController` which talks to `ArchetypeExampleService`.
3. In each file, place the same prefix in the variables, so it will be easy to replace them. For example, if you need a variable for your `ArchetypeExampleService`, a good name is `archetypeVariableService`.
4. Include the dot files as part of this project and a good readme.

### archetype:create-from-project

The maven goal [archetype:create-from-project](https://maven.apache.org/archetype/maven-archetype-plugin/create-from-project-mojo.html) is the key to generate an archetype from a project.

As part of the configuration, you can pass a archetype.properties file as parameter. This property file should like that:

```properties
# these are standard properties
package=com.almeida.tomas
groupId=archetype.it
artifactId=basic
version=1.0.0-SNAPSHOT

# here we add our personalized properties
defaultClassPrefix=ArchetypeExample
defaultVariablePrefix=archetypeVariable
```

As you see above, I only included the changing part of the files or variables and maven will take care of replacing them for me.

### Step-by-step

Let's go through all the steps from our archetype example project until our archetype.

#### 1. Rename all dot files 

It was a known issue that dot files are not included in the archetype. So we need to rename them to include it as part of our archetype.

```shell
mv .gitignore dot.gitignore
mv .file dot.file
```

#### 2. Call the maven goal with the desired parameters. 

```shell
mvn -U clean archetype:create-from-project \
      -Dinteractive=false \
      -DkeepParent=true \
      -DpropertyFile=archetype.properties \
      -DpackageName=com.almeida.tomas \
      -Darchetype.filteredExtensions=java,xml,md
```
In the [documentation](https://maven.apache.org/archetype/maven-archetype-plugin/create-from-project-mojo.html) you can see more parameters and why and when they are used, but in summary:
- `-Dinteractive=false`: interactive mode is disabled.
- `-DkeepParent=true`: keep the parent. 
- `-DpropertyFile=archetype.properties`: use our archetype.properties file to check for the variables. 
- `-DpackageName=com.almeida.tomas`: the package name for java sources that should be included in the archetype.
- `-Darchetype.filteredExtensions=java,xml,md`: files with the selected extensions will be checked and the content and the name of the files will be changed by the variables.

#### 3. Clean up the generated archetype metadata

In some cases the generation includes a default value of a property and we want to be sure that the all values will be requested to the user.

```shel
cd target/generated-sources/archetype/
sed -i 's/.*defaultValue.*//g' src/main/resources/META-INF/maven/archetype-metadata.xml 
```

#### 4. Install or deploy the archetype
If you are running these commands locally install, the archetype in your local repo:
```shell
# be sure to be in the target/generated-sources/archetype folder
mvn -B -U clean install
```

My recommendation is to configure a jenkins job, so everytime a change is pushed to the project, jenkins runs the commands and push a new version to your repository. So Jenkins should run a 
`mvn -B -U clean deploy`

## Use your created archetype

The created archetype will have the same artifactId with a `-archetype` suffix. So based in our example:

```shell
mvn archetype:generate \
     -DarchetypeGroupId=tomas.examples \
     -DarchetypeArtifactId=archetypeProject-archetype \
     -DarchetypeVersion=1.0.0-SNAPSHOT
```

Now, we will be asked to provide a value to the default and personalized variables:

```shell
Define value for property 'groupId': com.tomas.almeida
Define value for property 'artifactId': demo-project
Define value for property 'version' 1.0-SNAPSHOT: 1.0.0-SNAPSHOT
Define value for property 'package' com.tomas.almeida: com.tomas.almeida.domain 
Define value for property 'defaultClassPrefix': Demo 
Define value for property 'defaultVariablePrefix': demo

```

Confirm the values you have entered are correct:

```shell
Confirm properties configuration:
groupId: com.tomas.almeida
artifactId: demo-project
version: 1.0.0-SNAPSHOT
package: com.tomas.almeida.domain 
defaultClassPrefix: Demo
defaultVariablePrefix: demo
 Y: : Y
```

A new folder will be created and the name is the artifacId name. In our example, `demo-project`.

```shell
cd demo-project
```

The dot files need to be renamed:

```shell
mv dot.gitignore .gitignore
mv dot.file .file
```

So, you can now create several clones of your archetype example project in minutes!

---
If you found any error or want to help me to improve this article, please [comment the pull request](https://github.com/tomasalmeida/tomasalmeida.github.io/pull/5).