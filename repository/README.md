Local repository containing the published by artifacts of `libary-project` and `platform-project` used by this sample repository. Built by performing the following bash commands:

```bash
cd platform-project
./gradlew clean build publish
cd ..
cd library-project
./gradlew clean build publish
```