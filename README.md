# Gitlab-CI for ios application scripts and files

[![Developed by Mad Devs](https://maddevs.io/badge-light.svg)](https://maddevs.io)

## Technologies

*  [gitlab](https://gitlab.com/) - GitLab is a web-based DevOps lifecycle tool that provides a Git-repository manager providing wiki, issue-tracking and CI/CD pipeline[7] features, using an open-source license, developed by GitLab Inc
*  [fastlane](https://fastlane.tools/) - bunch of tools for handling all tedious tasks, like generating screenshots, dealing with code signing, and releasing your application.

## Prerequisites

1.  Computer (Macintosh)
1.  Mac OS >= Darwin Kernel Version 18.5.0 ( MacOS catalina )
1.  ruby >= 3.6.5
1.  rbenv
1.  fastlane (could be updated to last version by user)
1.  XCode
1.  Apple ID (for both Developer Portal and iTunes connect)
1.  gitlab-runner

## Description
Full description is available in our [blog]("")

## Environment variables

*  APPLE_ID - your apple id
*  APP_ID - your application id
*  FASTLANE_PASSWORD - password for your apple id
*  KEYCHAIN_PASSWORD - password for your mac user
*  MATCH_PASSWORD - passphrase to decrypt your profiles
*  MATCH_URL - git url where your certificates are stored
*  TEAM_ID - your team id
*  XCODE_PROJ - path to your *.xcodeproj file
*  XCWORKSPACE - path to your *.xcworkspace
*  APP_NAME - your application name. for example "Staging"
*  APP_ID_NAME - your application id name. for example "your.cool.app.com"
*  PLIST_PATH - path to plist file.
*  GSP_PATH - path to plist file for google.
*  CONFIG - release|debug


## The roadmap

- [*] Speed up upload to testflight
- [*] Add comments
- [ ] Speed up build process
