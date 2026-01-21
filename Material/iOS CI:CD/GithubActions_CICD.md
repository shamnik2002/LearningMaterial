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
        
## Code Signing set up

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

