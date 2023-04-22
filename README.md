# openrewrite-gradle-adddependency
Testing [Open Rewrite](https://docs.openrewrite.org/) using [Gradle](https://gradle.org/) AddDependency recipe on [Groovy](https://groovy-lang.org/) code

The code manipulates the build.gradle file
 * Adds a new plugin
 * Removes a dependency
 * Adds a new dependency (working from version 7.40.0...) 

---

## TL;DR

This code was used to report this bug [Gradle AddDependency onlyIfUsing fails to detect usage in Groovy #3053 ](https://github.com/openrewrite/rewrite/issues/3053)

Fixed through [Correctly and completely manage source set assignment to all compilation units #191](https://github.com/openrewrite/rewrite-gradle-plugin/pull/191)

Which is fixed from version 7.40.0 Release (2023-04-21)

_Below is the original error report..._

---

## Description
I am trying to add a dependency `implementation "sag:sagslager-klient:1.0.0"` to the dependencies part of my `build.gradle` file with OpenRewrite and the `org.openrewrite.gradle.AddDependency` recipe.

The project is written in Groovy and uses Gradle for building.

I can remove dependencies and add plugins to the `build.gradle` file, but I can't add dependencies...

## Issue
I'm pretty sure the issues is because the `onlyIfUsing` rule isn't triggered by `*.groovy` files but only by `*.java` files. \
If I add a `*.java` file (or rename the `*.groovy` file) it works...

## How to reproduce

Running the rewriteDryRun results in the following output (on the provided code sample):

    > Task :rewriteDryRun
    Validating active recipes
    Parsing sources from project openrewrite
    All sources parsed, running active recipes: com.yourorg.myexample
    These recipes would make results to build.gradle:
    com.yourorg.myexample
    org.openrewrite.gradle.plugins.AddBuildPlugin: {pluginId=com.energizedwork.webdriver-binaries, version=1.4}
    org.openrewrite.gradle.RemoveGradleDependency: {configuration=implementation, groupId=sag, artifactId=sagslager-grails4-plugin*}
    org.openrewrite.Recipe$AdHocRecipe
    Report available:
    /home/thomas/workspace/openrewrite-gradle-adddependency/build/reports/rewrite/rewrite.patch
    Run 'gradle rewriteRun' to apply the recipes.

And generates the following patch file missing the `+    implementation "sag:sagslager-klient:1.0.0"` dependency:

    diff --git a/build.gradle b/build.gradle
    index e1ededf..64e07c1 100644
    --- a/build.gradle
    +++ b/build.gradle
    @@ -5,6 +5,7 @@ org.openrewrite.Recipe$AdHocRecipe, org.openrewrite.gradle.RemoveGradleDependency, org.openrewrite.gradle.plugins.AddBuildPlugin

    plugins {
    id "org.openrewrite.rewrite" version "5.38.0"
    +    id "com.energizedwork.webdriver-binaries" version "1.4"
         }

    apply plugin: "groovy"
    @@ -15,7 +16,7 @@
    }

    dependencies {
    -    implementation 'sag:sagslager-grails4-plugin:6.30.0'
    +
    }

    rewrite {

If I rename the Groovy file from:
    `src/main/groovy/MyClass.groovy`
To:
    `src/main/groovy/MyClass.java` \
And run the rewriteDryRun it results in the following and expected output:

    > Task :rewriteDryRun
    Validating active recipes
    Parsing sources from project openrewrite
    All sources parsed, running active recipes: com.yourorg.myexample
    These recipes would make results to build.gradle:
    com.yourorg.myexample
    org.openrewrite.gradle.AddDependency: {groupId=sag, artifactId=sagslager-klient, version=1.0.0, configuration=implementation, onlyIfUsing=java.util.Date}
    org.openrewrite.gradle.plugins.AddBuildPlugin: {pluginId=com.energizedwork.webdriver-binaries, version=1.4}
    org.openrewrite.Recipe$AdHocRecipe
    org.openrewrite.gradle.RemoveGradleDependency: {configuration=implementation, groupId=sag, artifactId=sagslager-grails4-plugin*}
    org.openrewrite.Recipe$AdHocRecipe
    Report available:
    /home/thomas/workspace/openrewrite-gradle-adddependency/build/reports/rewrite/rewrite.patch
    Run 'gradle rewriteRun' to apply the recipes.

And the dependency is added to the patch file:

    diff --git a/build.gradle b/build.gradle
    index e1ededf..5c3e9ba 100644
    --- a/build.gradle
    +++ b/build.gradle
    @@ -5,6 +5,7 @@ org.openrewrite.gradle.AddDependency, org.openrewrite.gradle.RemoveGradleDependency, org.openrewrite.Recipe$AdHocRecipe, org.openrewrite.gradle.plugins.AddBuildPlugin

    plugins {
    id "org.openrewrite.rewrite" version "5.38.0"
    +    id "com.energizedwork.webdriver-binaries" version "1.4"
         }

    apply plugin: "groovy"
    @@ -15,7 +16,7 @@
    }

    dependencies {
    -    implementation 'sag:sagslager-grails4-plugin:6.30.0'
    +    implementation "sag:sagslager-klient:1.0.0"
         }

    rewrite {

