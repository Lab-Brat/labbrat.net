---
title: "Automating Flutter/Firebase APK build with Github Actions"
date: 2023-09-02T23:42:12+03:00
tags:
    - actions
    - cicd
categories:
    - DevOps
---

This blog post will cover a way to publish APK files 
with GitHub Actions for Flutter mobile app with Firebase backend.  

Workflow [example](https://github.com/Lab-Brat/gentoo_update_flutter/blob/main/.github/workflows/main.yml).  

### GitHub Repository Secrets
Flutter/Firebase apps have 3 files that hold sensitive information:
* `services.json` - Google Cloud credentials in JSON format, this is used by Firebase
* `keystore.jks` - Key used to sign the app
* `key.properties` - Key properties, such as key password, alias and it's location

All of the above will be encoded (not encrypted!) in BASE64 and save in GitHub 
Actions secrets. It must be encrypted for consistency, for example `services.json` 
contains a log of tabs and whitespaces that might be incorrectly stored in secrets.   

To encode in BASE64:
```bash
cat <filename> | base64 -w 0
```
`-w 0` parameter is being used to make sure the BASE64 encoded string is 1 string, 
and not multiple strings separated by newline.  

After that copy the output and save it at 
`GitHub Project` -> `Settings` -> `Secrets and Variables` -> `Actions` -> `New Repository Secret`  


### Workflow

Now, let's create a workflow step by step.  

First, it will be fun every time a tag is created. 
```yaml
name: 🤖 📦 Build and release packages for Android
on:
  push:
    tags:        
      - '*'
```

Import actions. In this workflow 4 actions are used:
* `checkout` - to pull and checkout the code
* `setup-java` - java is needed to build Flutter apps. It should be the same version as used in local development.
* `flutter-action` - and, of course, Flutter actions will be needed.
* `upload-artifact` - used to upload created apk file to artifacts sections of the workflow. Use in the end of the workflow.
```yaml
jobs:
  build-n-release:
    name: 🤖 📦 Build and release
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v2
        with:
          distribution: 'zulu'
          java-version: '11'
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.10.6'
```

Decode `services.json` and other sensitive information, 
and write it to files in the build environment:
```yaml
      - name: 🔑 Create Google Services JSON File
        env:
          GOOGLE_SERVICES_JSON: ${{ secrets.GOOGLE_SERVICES_JSON }}
        run: echo $GOOGLE_SERVICES_JSON | base64 -di > ./android/app/google-services.json

      - name: 🔑 Create the Keystore
        env:
          KEYSTORE_BASE64: ${{ secrets.KEYSTORE }}
        run: echo $KEYSTORE_BASE64 | base64 -di > android/upload-keystore.jks 

      - name: 🔑 Create Key Properties
        env:
          KEY_PROPERTIES_BASE64: ${{ secrets.KEY_PROPERTIES }}
        run: echo $KEY_PROPERTIES_BASE64 | base64 -di > android/key.properties 
```

Update the dependency list and build APK file and an App Bundle:
```yaml
      - name: 🪺 Update Flutter dependency list
        run: flutter pub get

      - name: ⚙️ Build apk
        run: flutter build apk

      - name: ⚙️ Build bundle (aab)
        run: flutter build appbundle
```

Finally, upload APK and App Bundle to artifacts:
```yaml
      - name: 📦 Save APK as artifact
        uses: actions/upload-artifact@v1
        with:
          name: "gentoo_update-${{  github.ref_name }}.apk"
          path: build/app/outputs/apk/release/app-release.apk

      - name: 📦 Save Bundle as artifact
        uses: actions/upload-artifact@v1
        with:
          name: "gentoo_update-${{  github.ref_name }}.aab"
          path: build/app/outputs/bundle/release/app-release.aab
```

Both APK and App Bundle will be uploaded to 
`Actions` -> `<Job Name>` -> `Articats` (bottom of the screen).  

From here, I like to test the app on a device or emulator first, 
before adding it to release.


### Links
* [[Link](https://damienaicheh.github.io/flutter/github/actions/2021/04/29/build-sign-flutter-android-github-actions-en.html)] - Blog post on building and signing Flutter apps with Github Actions
* [[Link](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)] - Github Secrets

