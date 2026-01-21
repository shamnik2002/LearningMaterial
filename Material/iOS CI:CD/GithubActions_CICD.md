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
          
### Provisioning profiles, certificates

## App Store Connect Set up

## Git Set up

## CI/CD General Overview
### workflows

### PR Worflow

### Push to default branch workflow (PR Merge)

### Code Freeze Workflow

### Release Workflow

### Post Release Workflow

