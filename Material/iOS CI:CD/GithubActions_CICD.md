# iOS CI CD with Github Actions

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


## CI/CD General Overview

I have 4 workflows

1. PR Workflow - starts as soon as someone creates a PR.
2. Push Workflow - starts whenever a PR is merged
3. Code Freeze - Manual workflow which requires branch and release branch name as input. It should be triggered when we are ready to cut a release.
4. Release - Manual workflow that builds, archives and uploads the build to testflight

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

### PR Worflow

**Prerequisites**

BUILD_CERTIFICATE_BASE64
P12_PASSWORD
KEYCHAIN_PASSWORD

### Push to default branch workflow (PR Merge)

### Code Freeze Workflow

### Release Workflow

### Post Release Workflow

