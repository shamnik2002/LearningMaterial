# iOS CI CD with Github Actions

## Sample iOS repo with workflows 
https://github.com/shamnik2002/SampleCICDApp/actions 

<hr>

## App Set Up
1. I usually create repo on git and then create the iOS project to make the set up easier https://github.com/shamnik2002/SampleCICDApp
2. Once iOS App is created, I include SDWebImage package so we can handle SPM as well in the CI/CD process
3. Next we will create xcconfig files for debug and release environments
  -  For this material we really need to make it easier for our Code Freeze workflow to allow editing App version and build number. More details later on why and how it is used
  -  Go to Xcode menu File -> New -> File from template
    
<img width="490" height="350" alt="Screenshot 2026-01-21 at 12 22 37 PM" src="https://github.com/user-attachments/assets/f1df9706-0582-4224-8117-f36a10e3a091" />

  - I leave at the same level as project file, make sure to not include it in any targets since we don't need it to compile.
  - Some projects create a base config and include it in environment specific configs if needed.
  - Add the below info to each config as needed
<img width="1403" height="461" alt="Screenshot 2026-01-21 at 12 28 01 PM" src="https://github.com/user-attachments/assets/14b66da1-9003-47c0-a525-f207a3a667e7" />
  - Now lets edit the project file to make sure we read the values from the config files
    
      -  Select the project (not target) -> Info tab
        
      -  Set the config for debug and release
        
<img width="691" height="326" alt="Screenshot 2026-01-21 at 12 32 01 PM" src="https://github.com/user-attachments/assets/489e7e59-77c2-465a-a3f0-26900c1b52eb" />
      
      -  Make  sure the schemes are updated so for run we pick debug and archive is set to release config

      - Next got to target -> build Settings and set the Current Project Version (build number) and Marketing Version (App version)
<img width="1131" height="291" alt="Screenshot 2026-01-21 at 12 39 44 PM" src="https://github.com/user-attachments/assets/0171e0df-1700-4f03-875a-d2a8e06ef397" />

4. Finally create app icon as that is a requirement to upload builds. I used Icon Composer (https://developer.apple.com/icon-composer/) to create an icon. make sure you icon doesn't have any transparent background. Add the app icon to you.
   
  - Credit/Resources
    
      - https://medium.com/@hdmdhr/xcode-scheme-environment-project-configuration-setup-recipe-22c940585984
        
      - https://developer.apple.com/documentation/xcode/configuring-the-build-settings-of-a-target

<hr>
## Code Signing Set Up

For this material we will do manual code signing

**Provisioning Profile**

A .mobileprovision file is like a permission slip issued by Apple. It tells iOS devices:

  - Which devices are allowed to run your app (by device UDID).
  
  - Which app it’s for (identified by Bundle ID).
  
  - Who signed it (your Apple Developer identity or the dev identity you’re purchasing).
    
  - Whether it’s for development, ad hoc, or distribution (App Store or enterprise).

**Ceritifcate**

a digital credential from Apple that verifies a developer's identity, allowing them to securely sign and distribute apps, ensuring the code comes from a trusted source and hasn't been tampered with

  - Authentication: Proves to Apple that you are who you say you are.
    
  - Code Signing: Digitally signs your app's code, linking it to your developer identity.
    
  - Security: Ensures apps haven't been altered since they were signed, protecting users from malicious code. 

### Steps

 1. Go to developer.apple.com -> Certificates, Identifiers & Profiles and create a new identifier for your app
 2. Select App ID -> App 
<img width="1206" height="428" alt="Screenshot 2026-01-21 at 1 58 33 PM" src="https://github.com/user-attachments/assets/7be94542-bb69-4ea0-b14e-2efa1833d93e" />

  3. Now create the profiles for development and distribution using the same bundle ID. If you never created certificates before then you need to create one for development and distribution before creating the profiles. Select the appropriate certificate when creating the provisioning profile.
  4. Once done download both profiles profiles and the certs. Double click them to make sure they are installed.
  5. Now go to appstoreconnect to create your app where you will upload it and distribute it to testflight or ap store.
  6. Go to apps -> create a new app
<img width="603" height="694" alt="Screenshot 2026-01-21 at 2 07 32 PM" src="https://github.com/user-attachments/assets/0a2cc463-31c0-409b-a739-cfd256930fcc" />

  7. Once app is created, now go to Users and Access -> App Store Connect API -> Integrations
  8. We need to create AppID that we can use to upload the build within our git workflow
  9. Generate the key. You need the below info for uploading the build
      
        - Key ID you just created
          
        - Issuer ID you should see just above the keys
          
        - You should see a download button next to the key created download and save the file in a secure place. It should be a .p8 extension. Note that once you download it you can't download it again.
          
  10. Now let's get the certs required for CICD
  11. Go to Keychain -> certificates -> right click on the development and distribution and export. They should be .p12 files, you need to set a password for you certificates. You need this password later so keep it in passwords app.

<hr>

## General Git Set up

I try to follow the general branch guidelines

**main**
- protected branch with necessary rules like PR with approval required to merge
- has the latest stable release with release tags

**develop**
- protected branch with necessary rules like PR with approval required to merge
- has the ongoing development
- This is where we cut release branch from when we code freeze

**release/<App Version>**
- Not a protected branch
- Used between code freeze to actual release incase we need to cherry pick fixes
- Created via git workflow

Make sure you give permissions to git to allow to commit to branch within workflow. Go to Github -> Your repo -> settings -> Actions -> general (in the left pane)

You need GITHUB_TOKEN to push to a branch. it is generated automatically and the var is available to use in the workflow.

<img width="860" height="390" alt="Screenshot 2026-01-21 at 2 55 15 PM" src="https://github.com/user-attachments/assets/280d10f4-5625-49b7-bb74-c17548f7cfee" />

<hr>

## CI/CD General Overview

I have 3 workflows

1. PR Workflow - starts as soon as someone creates a PR against develop or merges the PR to develop.
2. Code Freeze - Manual workflow which requires branch and release branch name as input. It should be triggered when we are ready to cut a release.
3. Release - Manual workflow that builds, archives and uploads the build to testflight

Before we get to these workflows. We need to set up environment and secrets that are required for the workflows.

**Environment**

1. Go to github -> your repo -> Settings -> Environments (left pane)
2. Create Dev and Prod environment. We will use Dev for CI worflow and Prod for CD.
3. Now lets add the secrets we need to access within the worflow for each environment
     - GitHub secrets are encrypted variables used to securely store sensitive information, such as API keys, access tokens, and credentials, within your GitHub environment. They prevent sensitive data from being exposed in your code or workflow logs.
4. Dev environment requires below
     - BUILD_CERTIFICATE_BASE64 - this is Base 64 encoded value of the dev certificate, the .p12 file
     - PROVISIONING_PROFILE_BASE64 - this is base 64 encoded value of your dev provisioning profile
     - P12_PASSWORD - this is the password you set up for your certificate
     - KEYCHAIN_PASSWORD - password you need to create for the keychain on the github runner. I use the passwords app to create the password, should be unique for each project.
5. Prod environment requires all of the above + the below secrets
     - EXPORT_OPTIONS_PLIST_BASE64 this is base64 encoded value of the exportOptions.plist file that is generated when you archive your project. We will modify as needed, more details below.
     - ISSUER_ID - app store connect issure ID
     - APP_KEY_ID - app store connect app Key ID that we generated
     - APP_STORE_CONNECT_KEY - this is the value within the .p8 file that we downloaded when we created the key. Just open the file in a text editor and copy it.
6. Getting Export Options plist
   
     - In Xcode Archive your project
       
<img width="1381" height="166" alt="Screenshot 2026-01-21 at 6 03 16 PM" src="https://github.com/user-attachments/assets/be4e62f9-8e6d-4388-863a-de4cf88b00fd" />
 
     - Select distribute App
     - Select Testflight option (make sure you are logged in Xcode)
     - Once done select export option at the bottom. Make sure to export it to a folder outside your repo since we do not want to check in any of these files
     - Now open the ExportOptions.plist file that is in the folder we exported
     - NOTE: I'm not 100% if this is the right approach, but this is the only way it worked for me.
     - Edit the plist 
         - Remove the keys destination, generateAppStoreInformation, manageAppVersionAndBuildNumber, testflightInternalTestingOnly
         - Change signingStyle to manual
         - Add provisioningProfiles and signingCertificate. we use the name for these as values. For profile you can copy the name from profile on developer account. For certificate you can copy the name from keychain.

<img width="801" height="267" alt="Screenshot 2026-01-21 at 6 16 41 PM" src="https://github.com/user-attachments/assets/873501cc-565e-461a-a373-e40762cbe49b" />


To base64 encode use the below command. make sure you run it in the same directory where the file is. pbcopy will make sure the value is copied to clipboard.
```
  base64 -i <filename> | pbcopy
```

Now let's add all these to github secrets under the environments

1. Go to github -> your repo -> Settings -> Environments (left pane)
2. Select the environment and add secret
<img width="711" height="445" alt="Screenshot 2026-01-21 at 5 54 27 PM" src="https://github.com/user-attachments/assets/045c59db-ddb8-45a8-b63d-7b78b9570ecc" />

3. make sure to remove any extra new lines/spaces when you paste the value
4. Here is how dev environment secrets should look
<img width="866" height="368" alt="Screenshot 2026-01-21 at 6 00 03 PM" src="https://github.com/user-attachments/assets/def81ab2-0e6c-471c-98ef-878af99e2d31" />

5. and here is how prod environment secrets should look

<img width="876" height="581" alt="Screenshot 2026-01-21 at 6 29 32 PM" src="https://github.com/user-attachments/assets/f6e7fcef-a977-4cf7-8317-8603af2366e0" />


Below we will go in detail how each workflow is set up.

............................................................................................................................................................

### Notes:

1. Double check indentations if work flow fails
2. If a single command spans multiple lines then make sure `\` is the last character on the line
3. We are the mercy of what git has to offer in terms of environment, macOS, iOS, XCode, iOS simulators and devices
4. Simulators / device need to match exactly or builds fail, even then randomly some combinations are not available at times.

............................................................................................................................................................

### PR Worflow

**Prerequisites**

BUILD_CERTIFICATE_BASE64

P12_PASSWORD

KEYCHAIN_PASSWORD

1. To create a worflow, go to Github -> your repo -> Actions and pick any template we will just edit it as necessary. This should create the directory structure for you as well. All workflows need to be in <yourApp> / .github / workflows / <workflow>.yml
2. Name the workflow to reflect this will be triggered on PR submission.
3. For this follow our trigger is Pull request creation
```
      # This is a basic workflow to help you get started with Actions
      name: PR Submission

      # Controls when the workflow will run
      on:
        # Triggers the workflow on push or pull request events but only for the "main" branch 
        pull_request:
          branches: [ "develop" ]
        push:
          branches: [ "develop" ]
```
4. Next we specify the Jobs and for each job we specify the steps that need to run

```
# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build_and_test:
    # The type of runner that the job will run on
    runs-on: macos-26
    environment: Dev

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
```
5. build_and_test is the name of the job.
6. Set the runner to use a specific macOS to allow predictable CI flow
7. Set our environment to Dev so the secrets used will be from this environment we set up previously
8. Now let's work on Steps

```
      - name: Checkout repository
        uses: actions/checkout@v2
```
9. We use a predefine git action to checkout the repo on runner

```
      - name: Select Xcode
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: '26'
```
10. Select a specific Xcode version
```
      - name: Install Certificates and Provisioning Profile
        env:
          BUILD_CERTIFICATE_BASE64: ${{ secrets.BUILD_CERTIFICATE_BASE64 }}
          P12_PASSWORD: ${{ secrets.P12_PASSWORD }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}          
          
        run: |
```
11. Now we install our certificates and provisioning profiles.
12. Create all the varriables we need. Here we are just reading the values from the github secrets we created
```
        run: |
            # create variables
            CERTIFICATE_PATH=$RUNNER_TEMP/build_certificate.p12
            PP_PATH=$RUNNER_TEMP/build_pp.mobileprovision
            KEYCHAIN_PATH=$RUNNER_TEMP/app-signing.keychain-db

            echo "create variables done"
```
13. Create variables to hold the files into which we will copy the values from github secrets
```
            # import certificate and provisioning profile from secrets
            echo -n "$BUILD_CERTIFICATE_BASE64" | base64 --decode > "$CERTIFICATE_PATH"
            ls -lh "$CERTIFICATE_PATH"
            echo "certificate exists"
            file "$CERTIFICATE_PATH"
            echo "check file type"
            echo -n "${{ secrets.PROVISIONING_PROFILE_BASE64 }}" | base64 --decode > "$PP_PATH"

            echo "import cert and provisioning profiles done"
```
14. Now import certificate and provisioning profile to the specific file paths we defined
15. For both we need to decode since our github secrets were encoded in base64
```
            # create keychain
            security create-keychain -p "$KEYCHAIN_PASSWORD" "$KEYCHAIN_PATH"
            echo "create keychain done"
            security set-keychain-settings -lut 21600 "$KEYCHAIN_PATH"
            echo "set keychain done"
            security unlock-keychain -p "$KEYCHAIN_PASSWORD" "$KEYCHAIN_PATH"
            echo "unlock keychain done"
```
16. Now we need to set up keychain so we can import the certificate and provisioning profiles there
17. Create keychain at the given path and set a password for it. This is the keychain password we created and added to the github secrets
18. Next we need to configure the key chain settings
19. -l stands for lock when systems times out, -u stands for lock after timeout, and -t is the timeout in seconds. So we are locking the keychain after 6 hours of inactivity
20. Next unlock the keychain using our password
```
            # import certificates into keychain            
            security import "$CERTIFICATE_PATH" -P "$P12_PASSWORD" -A -t cert -f pkcs12 -k "$KEYCHAIN_PATH"
            echo "import cert and p12 password done"
            security set-key-partition-list -S apple-tool:,apple: -k "$KEYCHAIN_PASSWORD" "$KEYCHAIN_PATH"
            echo "set-key-partition-list done"
            security list-keychain -d user -s "$KEYCHAIN_PATH"
            echo "list-keychain done"

            echo "import certificates into keychain done"
```
21. Now import the certificate in the keychain
22. we specify the certificate at the path we created, provide the password for the certificate to unlock it
23. -A allows all applications access
24. -t cert specifies the entity that is being imported, here it is a certificate
25. -f pkcs12 specifies the file type
26. -k "$KEYCHAIN_PATH" tells that it needs to be imported here
```
            # import provisioning profile
            mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
            cp "$PP_PATH" ~/Library/MobileDevice/Provisioning\ Profiles
            security find-identity -p codesigning -v
            security cms -D -i ~/Library/MobileDevice/Provisioning\ Profiles/build_pp.mobileprovision
            echo "import provisioning profile done"
```
27. Now import provisioning profile in the keychain
28. provisioning profiles are stored at a specific path and that's where xcode will read from
29. create the directory and copy the provisioning profile there
30. Now verify whether the certificates are correctly installed and usable. You should see in the logs of your workflow.
31. Now verify provisioning profile, we log this as well. these are more for debugging so can be removed once the workflow is set up

```
      # save derived data to a predictable path
      - name: Cache SPM Dependencies
        uses: actions/cache@v4
        id: spm-cache
        with:
          path: |
            
            ${{ github.workspace }}/DerivedData/**/SourcePackages/checkouts            
          key: ${{ runner.os }}-spm-${{ hashFiles('**/Package.resolved') }}
          restore-keys: |
            ${{ runner.os }}-spm-
```
32. Next if you have packages, it is a good idea to cache them
33. NOTE: for every first run of this workflow on your PR, there will be a cache miss. The Cache is unique to each branch, so subsequent runs will benefit when you address feedback on your PR.
34. We are using a predefined git action.
35. We need to specify where the packages are and then use the package.resolved as the key since it will change if there are any changes to the package.swift file or packages pulled.
```
    - name: Build and Test
        run: |
          set -euo pipefail 
          xcodebuild clean test \
            -project SampleCICDApp/SampleCICDApp.xcodeproj \
            -scheme "SampleCICDApp" \
            -configuration Debug \
            -destination 'platform=iOS Simulator,name=iPhone 17'  \
            -derivedDataPath $GITHUB_WORKSPACE/DerivedData \
          | xcbeautify
```
36. finally we build and test our code
37. set -euo pipefail basically tells github to bail early in case of any errors
38. Next we need to specify the project, scheme to use, configuration
39. destination is the simulator/device we want to run the tests on. You need to make sure they match exactly to what git has to offer
40. if not sure you are use the below step as part of you workflow to check available simulators
```
     - name: List available simulators (optional, for debugging)
        run: xcrun simctl list devices available
        'platform=iOS Simulator,name=iPhone 17,OS=18.0' \
         -derivedDataPath $GITHUB_WORKSPACE/DerivedData
```
41. You can skip testing specific targets if needed using the command `-skip-testing <test target> \`
42. You should specify the derived data path, makes it easy to cache SPM, we have a predictable path
43. xcbeautify to make logs readable. But when debugging also use -verbose
```
     - name: Upload xcresult
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: xcresult
          path: |
            ${{ github.workspace }}/DerivedData/Logs/Test/*.xcresult
```
44. Finally upload results in case of failures

That was a lot, but we will reuse some of this in next flows so major chunk is done!

............................................................................................................................................................

### Code Freeze Workflow

This is the simplest workflow all we do here is accept a release version, cut a release branch with the version number off develop branch and commit it.

1. This ia manual workflow for now so no triggers set up for this
```
on:
  workflow_dispatch:
    # Inputs the workflow accepts.
    inputs:
      release_version:
        # Friendly description to be shown in the UI instead of 'name'
        description: "Release version (e.g. 1.2.0)"
        # Input has to be provided for the workflow to run
        required: true
        # The data type of the input
        type: string
```
2. This specifies that we require inputs like release version. No validation yet but that's something we need to add in future.
```
jobs:
  release:
    runs-on: macos-latest
    permissions:
      contents: write
```
3. This specifies th runner OS, since we are not actually doing a build macos-latest is ok.
4. we also make sure the job has write permission
```
steps:
      - name: Checkout develop
        uses: actions/checkout@v4

      - name: Create release branch
        run: |
          RELEASE_BRANCH="release/${{ inputs.release_version }}"
          git checkout -b "$RELEASE_BRANCH"
```
5. Checkout develop which is our default branch
6. now we use the release version to create release branch
```
     - name: Update iOS App version and build number
        run: |
          pwd
          ls
          cd SpaceNews
          echo "$(<"release.xcconfig")"

          BUILD_NUMBER=$GITHUB_RUN_NUMBER
          MARKETING_VERSION=${{ inputs.release_version }}          
          
          sed -i '' \
            -e "s/^MARKETING_VERSION = .*/MARKETING_VERSION = $MARKETING_VERSION/" \
            -e "s/^CURRENT_PROJECT_VERSION = .*/CURRENT_PROJECT_VERSION = $BUILD_NUMBER/" \
            release.xcconfig

          echo "Build number and version updated"
          echo "$(<"release.xcconfig")"
```
7. Next step is to update the build number and app version
8. You can skip checking for release.xcconfig this is just for debugging where I print the contents to confirm update
9. For build number we rely on GITHUB_RUN_NUMBER which is provided by git and the number increment everytime we execute the workflow. It does not update if we re-run the workflow due to failed jobs. This is considered a better approach since it avoids concurrency issues in case we have parallel builds running. But you can increment it yourself by reading the last build number if needed.
10. Next we use the input taht we got for release version and update it. Marketing version in xconfig and build settings is the app version.
11. sed is stream editor command, -i means edit file in place
12. s stands for substitue, /^MARKETING_VERSION = .*/ is the pattern to match. ^ is start of line then string to match and .* is to match any char after =

```
     - name: Commit version bump
        run: |
          git config user.name "github-actions"
          git config user.email "github-actions@github.com"
          BUILD_NUMBER=$GITHUB_RUN_NUMBER
          
          git add .
          git commit -m "chore: cut release ${{ inputs.release_version }} ($BUILD_NUMBER)"

     - name: Push release branch
        run: |
          git push origin HEAD
```
13. Here is commit the changes to git and push to remote
14. You should see the branch on your repo with the changes

**NOTES**
- If you already tried archiving and uploading to testflight via xcode before then make sure you build number is set correctly in your xcconfig files
- App store won't let you upload if the build number is smaller than last build there.

............................................................................................................................................................

### Release Workflow

1. This is also a manual workflow so we get to select the branch we need to run this workflow on. here the branch will be the release branch we created in the code freeze workflow. Input accepts branch name but it's more a ack and we don't really use it in the workflow.
2. Release workflow reuses the steps checkout repo, set xcode, install certs/profiles, cache SPM from the PR workflow the only difference is that the env = Prod here. This means git will read the secrets from prod environment to download the distribution certs/profile and everything needed.
3. The only missing piece here is that we also need the export options plist as part of the install certs/ profiles step
```
            # import export options plist
            echo -n "$EXPORT_OPTS" | base64 --decode > "$EXPORT_OPTS_PATH"

            echo "import export options plist done"
```
5. So we will directly jump to the step where we create the archive
```
      - name: Build and Archive
        run: |
          set -euo pipefail 
         xcodebuild clean archive \
            -project SampleCICDApp/SampleCICDApp.xcodeproj \
            -scheme "SampleCICDApp" \
            -configuration Release \
            -destination generic/platform=iOS  \
            -derivedDataPath $GITHUB_WORKSPACE/DerivedData \
            -archivePath $GITHUB_WORKSPACE/SampleCICDApp.xcarchive \
            -verbose \
          | xcbeautify
```
6. Instead of test we archive here. Also make sure to set configuration to release, destination can be generic.
7. Always set dereivedDataPath and archivePath so it's easy to grab stuff when needed from a predictable path.
8. -verbose is optional in case you need detailed logs
```
    - name: Upload xcresult
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: xcresult
          path: |
            ${{ github.workspace }}/DerivedData/Logs/Test/*.xcresult
```
9. Next upload xcresult in case of failure. You could also on success if you want to monitor the health of the system
```
     - name: Export ipa
        run: |          
          EXPORT_OPTS_PATH=$RUNNER_TEMP/ExportOptions.plist
          echo "$(<"$EXPORT_OPTS_PATH")"
          xcodebuild -exportArchive -archivePath $GITHUB_WORKSPACE/SampleCICDApp.xcarchive -exportOptionsPlist $EXPORT_OPTS_PATH -exportPath $GITHUB_WORKSPACE/build -verbose

```
10. Here is export the ipa. This is where exportOptions.plist if needed and based on the settings there the ipa is created
```
     - name: Upload application
        id: upload_step
        env:
          API_KEY_BASE64: ${{ secrets.APP_STORE_CONNECT_KEY }}
          APPSTORE_KEY_ID: ${{ secrets.APP_KEY_ID }}
          APPSTORE_ISSUER_ID: ${{ secrets.ISSUER_ID }}
        run: |
          mkdir -p ~/.appstoreconnect/private_keys
          echo -n "$API_KEY_BASE64" > ~/.appstoreconnect/private_keys/AuthKey_$APPSTORE_KEY_ID.p8
          xcrun altool --validate-app -f $GITHUB_WORKSPACE/build/SampleCICDApp.ipa -t ios --apiKey "$APPSTORE_KEY_ID" --apiIssuer "$APPSTORE_ISSUER_ID"
          xcrun altool --upload-app -f $GITHUB_WORKSPACE/build/SampleCICDApp.ipa -t ios --apiKey "$APPSTORE_KEY_ID" --apiIssuer "$APPSTORE_ISSUER_ID"
```
11. Now we upload the ipa using the app store connect key, issureID and app key ID
12. First read the secrets from git
13. Now we need to create the .p8 file with the app store connect key. The file has a specific name format where it uses the app Key ID.
14. We use the altool to first validate the app and then uplaod the build

**NOTE**
Usually works ok but if you see any failure like failed to connect to account means it could be one of the below issues
- ExportOptions.plist was not configured properly. Make sure you follow the steps above where we edited the exportOptions.plist
- App store connect key value is incorrect
............................................................................................................................................................

### Post Release Workflow

<hr>
