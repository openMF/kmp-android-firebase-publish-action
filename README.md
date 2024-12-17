# Firebase App Distribution Action Using Fastlane

## Overview

This GitHub Actions workflow automates the process of building and deploying an Android application to Firebase App Distribution. It handles version generation, release notes creation, secret inflation, app building, and deployment.

## Workflow Inputs

The workflow requires the following inputs:

### Project Configuration
- `android_package_name`: Name of the Android project module (Required)

### Signing Credentials
- `keystore_file`: Base64 encoded keystore file (Required)
- `keystore_password`: Password for the keystore file (Required)
- `key_alias`: Key alias for the keystore file (Required)
- `key_password`: Password for the key alias (Required)

### Firebase and Google Services
- `google_services`: Google services JSON file (Required)
- `firebase_creds`: Firebase credentials JSON file (Required)

### GitHub Configuration
- `github_token`: GitHub token for repository interactions (Required)
- `target_branch`: Target branch for deployment (Required)

## Workflow Steps

### 1. Environment Setup
- Sets up Java 17 development environment using Zulu OpenJDK distribution
- Configures Gradle build system
- Installs Ruby and Fastlane with Firebase App Distribution plugin

### 2. Version Generation
- Generates a unique version number based on commit count and existing tags
- Creates version files for tracking

### 3. Release Notes Generation
- Retrieves the latest release tag
- Generates release notes using GitHub's release notes API
- Creates changelog files for GitHub and beta releases

### 4. Secret Inflation
- Decodes and sets up required credentials:
    - Creates mock debug google-services.json
    - Inflates keystore file
    - Inflates Google services JSON
    - Inflates Firebase credentials

### 5. Android App Build
- Builds the Android app in release configuration
- Uses provided keystore and signing credentials
- Applies generated version number

### 6. Firebase Deployment
- Deploys the built app to Firebase App Distribution using Fastlane

## Prerequisites

To use this workflow, ensure you have:
- A GitHub repository with an Android project
- Firebase App Distribution setup
- Required credentials and tokens
- Fastlane and Ruby installed in your development environment

## Security Considerations
- Use GitHub Secrets to manage sensitive information like tokens and credentials
- The workflow uses base64 encoding for secure credential handling
- Mock files are used to prevent exposure of sensitive configuration

## Permissions
```yaml
permissions:
  contents: write
```

## Example Usage

```yaml
- uses: openMF/KMP-android-firebase-publish-action@v1.0.0
  with:
    android_package_name: 'app'
    keystore_file: ${{ secrets.KEYSTORE_FILE }}
    keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
    key_alias: 'release'
    key_password: ${{ secrets.KEY_PASSWORD }}
    google_services: ${{ secrets.GOOGLE_SERVICES_JSON }}
    firebase_creds: ${{ secrets.FIREBASE_CREDENTIALS }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    target_branch: 'main'
```
