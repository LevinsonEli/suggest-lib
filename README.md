# suggest-lib

## Overview

`suggest-lib` is a Maven-based Java library project. This repository provides a multi-branch pipeline setup to manage different stages of development and release processes using Jenkins and Maven. It also serves as a dependency for the project [toxictypo-app](https://github.com/LevinsonEli/toxictypo-app)

#MVN #Jenkins #JFrog #Artifactory #AWS #CI #DependencyManagement

## Goals of suggest-lib Multi-Branch Pipeline

The pipeline is designed to handle different branches with specific tasks:

### 1. For branch `feature/*`:
- Compile
- Test

### 2. For branch `master`:
- Accept parameters `MAJOR`, `MINOR` (For manual creation)
- Calculate and set a 3-number version (`MAJOR.MINOR.PATCH`) in `version.txt`
- Compile
- Test
- Publish to JFrog Artifactory

### 3. For branch `release/*`:
- Accept parameters `MAJOR`, `MINOR` (For manual creation)
- Calculate and set a 3-number version (`MAJOR.MINOR.PATCH`) in `version.txt`
- Set version using MVN
- Compile
- Test
- Publish to JFrog Artifactory
- Clean Up: `git tag` with the 3-number version (`MAJOR.MINOR.PATCH`) and push the tag to the remote repository
