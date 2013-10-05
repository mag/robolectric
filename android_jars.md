---
layout: default
title: Android Jars
---

## How Robolectric Loads Android Jars

To use the real Android code (as opposed to the SDK), the Robolectric maintainers have checked out the full AOSP and built all the code needed by Robolectric.
These built artifacts are stored in Maven Central under org.robolectric. Robolectric is compiled against these JARs.

We have not (yet) produced source JARs for these compiled class JARs. If you are debugging Robolectric, it is helpful to attach the sources directly from where they reside in the AOSP build. We don't have a good mapping from the various JARs to the correct source roots. Here's our best guess, assuming your IDE can traverse down and find all the sources under a higher-level directory:

```
android-base:             AOSP/frameworks/base
android-ext:     		  AOSP/external
android-libcore:          AOSP/libcore
android-telephony-common: AOSP/frameworks/opt
android-policy:           AOSP/frameworks/base/policy
```

## Building the Android Artifacts

See [Downloading and Building](http://source.android.com/source/building.html) in the AOSP documentation. When you use the 'repo init' command, make sure to give it the appropriate tag name. For example for the latest build, we used this repo command:

	repo init -u https://android.googlesource.com/platform/manifest -b android-4.3_r2

This takes a long, long time to download the entire AOSP source tree.

## Packaging and Uploading the Android Artifacts

Assuming you have checked out AOSP, you can now produce the 6 jars that robolectric needs. We have a script to do this in `scripts/construct-real-android-artifacts.sh`.

There are comments at the top of that script that will tell you which commands to run to actually build the JARs. Once the JARs are built, the next step is to
run the `construct-real-android-artifacts.sh` to prepare the maven artifacts.

To do so, you'll need `gpg` for signing each JAR file. See [This HOWTO](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-7a.DeploySnapshotsandStageReleaseswithMaven) for some background on how to get `gpg` installed.

In addition to `gpg`, it is helpful to have `gpg-agent` installed. This is so that you don't have to type your key's passphrase more than once. There are comments in construct-real-android-artifacts.sh that can help you get `gpg-agent` set up.

You are ready to run `construct-real-android-artifacts.sh` once you have `gpg` setup and the android JARs built in the AOSP. This will take care of:

- Renaming the jars
- Copies them into a temporary folder
- Generates a pom for each artifact
- Generates empty JARs to satisfy Sonatype (sources, javadoc, etc are currently an empty, stand-in jar)
- Generates a bundle.jar for uploading to Sonatype

To upload artifacts to Sonatype you will need to push your public `gpg` key to a public keyserver (see the Sonatype docs, above). Even once you've done this, the Sonatype upload may fail in the signing "rule". Wait 20 minutes or so, it takes a while before the keys are really available.

### Install Real Android Artifacts in Local Maven Repository

For developers iterating on building Android JARs and debugging Robolectric, it is possible to install the built JARs into the local Maven repository. This can be accomplished with `./scripts/mavenize-android-locally.sh`. Make sure to pass it the temporary directory where `construct-real-android-artifacts.sh` put the prepared artifacts.

### Publishing Real Android Artifacts to Sonatype and Maven Central

Publish each bundle.jar located in the output directory indicated by `construct-real-android-artifacts.sh` by following the instructions [here](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-7b.StageExistingArtifacts) using the "Staging Upload" directions (scroll down some). Upload the Artifact Bundle in the Nexus UI and then continue on the guide to [Release It](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-8a.ReleaseIt) to promote the artifacts to the release repository from the staging area. Once the artifacts are released on Sonatype, they will appear magically on Maven Central within 24 hours (in our experience it took just a few hours).