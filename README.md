# KMP Android App Publish Action

This GitHub Action automates the process of building and deploying Android applications to Firebase App Distribution using Fastlane.

## Prerequisites

Before using this action, make sure you have:

1. A valid Firebase project set up with App Distribution enabled
2. Google Services JSON file for your Android project
3. Firebase Service Account credentials JSON file
4. Release keystore file for signing your Android app

## Setup

### Fastlane Setup

Create a `Gemfile` in your project root with the following content:

```ruby
source "https://rubygems.org"

gem "fastlane"
gem "firebase_app_distribution"
```

Create a `fastlane/Fastfile` with the following content:

```ruby
default_platform(:android)

platform :android do
  desc "Publish Release Artifacts to Firebase App Distribution"
  lane :deployReleaseApkOnFirebase do |options|
    # Generate version
    generateVersion = generateVersion(
        platform: "firebase",
        appId: options[:appId],
        serviceCredsFile: options[:serviceCredsFile]
    )

    # Generate Release Note
    releaseNotes = generateFullReleaseNote()

    buildAndSignApp(
      taskName: "assembleProd",
      buildType: "Release",
      storeFile: options[:storeFile],
      storePassword: options[:storePassword],
      keyAlias: options[:keyAlias],
      keyPassword: options[:keyPassword],
    )

    firebase_app_distribution(
      app: options[:appId],
      android_artifact_type: "APK",
      android_artifact_path: options[:apkFile],
      service_credentials_file: options[:serviceCredsFile],
      groups: options[:groups],
      release_notes: "#{releaseNotes}",
    )
  end
end
```

## Usage

Add the following workflow to your GitHub Actions:

```yaml
name: Deploy to Firebase

on:
  push:
    branches: [ main ]
  # Or trigger on release
  release:
    types: [created]

jobs:
  deploy:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Firebase
        uses: openMF/kmp-android-firebase-publish-action@v1.0.0
        with:
          android_package_name: 'app'  # Your Android module name
          release_type: 'prod'  # Release type (prod/demo)
          google_services: ${{ secrets.GOOGLE_SERVICES }}
          firebase_creds: ${{ secrets.FIREBASE_CREDS }}
          keystore_file: ${{ secrets.RELEASE_KEYSTORE }}
          keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
          keystore_alias: ${{ secrets.KEYSTORE_ALIAS }}
          keystore_alias_password: ${{ secrets.KEYSTORE_ALIAS_PASSWORD }}
          tester_groups: 'qa-team,beta-testers'
```

## Inputs

| Input                     | Description                                       | Required |
|---------------------------|---------------------------------------------------|----------|
| `android_package_name`    | Name of your Android project module (e.g., 'app') | Yes      |
| `release_type`            | Type of Release eg. `prod` or `demo`              | No       |
| `keystore_file`           | Base64 encoded release keystore file              | Yes      |
| `keystore_password`       | Password for the keystore file                    | Yes      |
| `keystore_alias`          | Alias for the keystore file                       | Yes      |
| `keystore_alias_password` | Password for the keystore alias                   | Yes      |
| `google_services`         | Base64 encoded Google services JSON file          | Yes      |
| `firebase_creds`          | Base64 encoded Firebase credentials JSON file     | Yes      |
| `tester_groups`           | Comma-separated list of Firebase tester groups    | Yes      |

## Outputs
`firebase-app` - The path to the generated APK files.

## Setting up Secrets

1. Encode your files to base64:
```bash
base64 -i path/to/release.keystore -o keystore.txt
base64 -i path/to/google-services.json -o google-services.txt
base64 -i path/to/firebase-credentials.json -o firebase-creds.txt
```

2. Add the following secrets to your GitHub repository:
- `GOOGLE_SERVICES`: Content of google-services.txt
- `FIREBASE_CREDS`: Content of firebase-creds.txt
- `RELEASE_KEYSTORE`: Content of keystore.txt
- `KEYSTORE_PASSWORD`: Your keystore password
- `KEYSTORE_ALIAS`: Your keystore alias
- `KEYSTORE_ALIAS_PASSWORD`: Your keystore alias password
