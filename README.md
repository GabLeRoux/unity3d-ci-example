# unity3d ci example

[![pipeline status](https://gitlab.com/gableroux/unity3d-gitlab-ci-example/badges/master/pipeline.svg)](https://gitlab.com/gableroux/unity3d-gitlab-ci-example/commits/master)
[![Build Status](https://travis-ci.com/GabLeRoux/unity3d-ci-example.svg?branch=master)](https://travis-ci.com/GabLeRoux/unity3d-ci-example)
[![CircleCI](https://circleci.com/gh/GabLeRoux/unity3d-ci-example.svg?style=svg)](https://circleci.com/gh/GabLeRoux/unity3d-ci-example)

This project is a PoC to **run unity3d tests and builds inside a CI** using [gableroux/unity3d docker image](https://hub.docker.com/r/gableroux/unity3d/). It currently creates builds for Windows, Linux, MacOS and webgl. The webgl build is published by the CI to [gitlab-pages](https://about.gitlab.com/features/pages/) and [github-pages]()! This repository is hosted on multiple remotes to provide examples for [Gitlab-CI](), [Travis]() and [CircleCI]():

* [github](https://github.com/gableroux/unity3d-ci-example)
* [gitlab](https://gitlab.com/gableroux/unity3d-gitlab-ci-example)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Getting started](#getting-started)
- [Points of interest](#points-of-interest)
    - [Build script](#build-script)
    - [CI Configuration](#ci-configuration)
        - [gitlab-ci](#gitlab-ci)
        - [CircleCI](#circleci)
        - [Travis](#travis)
    - [Test files](#test-files)
- [How to activate](#how-to-activate)
    - [Gitlab-CI](#gitlab-ci)
    - [Travis](#travis-1)
    - [CircleCI](#circleci-1)
- [How to add build targets](#how-to-add-build-targets)
    - [gitlab-ci](#gitlab-ci-1)
    - [iOS support](#ios-support)
    - [Android support](#android-support)
- [How to run scripts manually](#how-to-run-scripts-manually)
    - [Test](#test)
    - [Build](#build)
- [Shameless plug](#shameless-plug)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Getting started

If you don't have a Unity project yet:

1. Fork this project from github or gitlab
2. Update the readme and remove undesired CI configurations
3. Follow How to activate instructions
4. Configure your CI

If you already have your own project:

1. Copy desired CI file and scripts in `ci` folder
2. Update the Unity version according to your project version in the CI file. All versions are available at [gableroux/unity3d docker image](https://hub.docker.com/r/gableroux/unity3d/)
3. Copy build script (make sure you use the same path as original project, it must be in an `Editor` folder)
4. Follow How to activate instructions
5. Configure your CI

## Points of interest

This is probably what you're looking for.

### Build script

Script passed to the unity3d command line as argument to create builds

* See [`BuildScript.cs`](Assets/Scripts/Editor/BuildCommand.cs)

### CI Configuration

Pick one, if you're on gitlab, use gitlab-ci as Travis and CircleCI don't support Gitlab as of september 2018, if you're on github, Travis is more popular but CircleCI and [gitlab-ci will also work](https://about.gitlab.com/features/github/). If you can't decide, see [CircleCI vs. GitLab CI/CD](https://about.gitlab.com/comparison/gitlab-vs-circleci.html) and [Travis CI vs GitLab](https://about.gitlab.com/comparison/travis-ci-vs-gitlab.html).

#### gitlab-ci

* [`.gitlab-ci.yml`](.gitlab-ci.yml)

#### CircleCI

* [`.circleci/config.yml`](.circleci/config.yml)

#### Travis

* [`.travis.yml`](.travis.yml)

### Test files

* [`editmode` tests in `Assets/Scripts/Editor/EditModeTests`](Assets/Scripts/Editor/EditModeTests)
* [`playmode` tests in `Assets/Tests/`](Assets/Tests/)

## How to activate

You'll first need to run this locally. All you need is [docker](https://www.docker.com/) installed on your machine.

1. Clone this project
2. Pull the docker image and run bash inside, passing unity username and password to env

    _hint: you should write this to a shell script and execute the shell script so you don't have your credentials stored in your bash history_. Also make sure you use your Unity3d _email address_ for `UNITY_USERNAME` env var.

    ```bash
    UNITY_VERSION=2018.2.3f1
    docker run -it --rm \
    -e "UNITY_USERNAME=username@example.com" \
    -e "UNITY_PASSWORD=example_password" \
    -e "TEST_PLATFORM=linux" \
    -e "WORKDIR=/root/project" \
    -v "$(pwd):/root/project" \
    gableroux/unity3d:$UNITY_VERSION \
    bash
    ```
3. In Unity docker container's bash, run once like this, it will try to activate

    ```bash
    xvfb-run --auto-servernum --server-args='-screen 0 640x480x24' \
    /opt/Unity/Editor/Unity \
    -logFile \
    -batchmode \
    -username "$UNITY_USERNAME" -password "$UNITY_PASSWORD"
    ```

4. Wait for output that looks like this:

    ```
    LICENSE SYSTEM [2017723 8:6:38] Posting <?xml version="1.0" encoding="UTF-8"?><root><SystemInfo><IsoCode>en</IsoCode><UserName>[...]
    ```
    If you get the following error:
    
    > Can't activate unity: No sufficient permissions while processing request HTTP error code 401
    
    Make sure your credentials are valid. You may try to disable 2FA in your account and try again. Once done, you should enable 2FA again for security reasons. See [#11](https://gitlab.com/gableroux/unity3d-gitlab-ci-example/issues/11) for more details.

5. Copy xml content and save as `unity3d.alf`
6. Open https://license.unity3d.com/manual and answer questions
7. Upload `unity3d.alf` for manual activation
8. Download `Unity_v2018.x.ulf`
9. Pass the license as a secret to your CI Configuration (see following sections)

### Gitlab-CI

Gitlab-CI Supports using mutli-line environment variables out of the box. :tada:

Copy the content of `Unity_v2018.x.ulf` license file to your CI's environment variable `UNITY_LICENSE_CONTENT`.

_Note: if you are doing this on windows, chances are the [line endings will be wrong as explained here](https://gitlab.com/gableroux/unity3d-gitlab-ci-example/issues/5#note_95831816). Luckily for you, [`.gitlab-ci.yml`](.gitlab-ci.yml) solves this by removing `\r` character from the env variable so you'll be alright_. [`.gitlab-ci.yml`](.gitlab-ci.yml) will then place the `UNITY_LICENSE_CONTENT` to the right place before running tests or creating the builds.

### Travis

Travis doesn't support multiple-lines environment variables. I had troubles with escaping so I recommend encrypting the license file. `.travis.yml` will decrypt the file and add its content to `UNITY_LICENSE_CONTENT` env var itself afterward.

Move your `Unity_v2018.x.ulf` to `ci` folder (so you don't clutter your project's root). Just make sure you don't track it inside git.

```bash
travis encrypt-file --pro -r YOUR_TRAVIS_USERNAME/YOUR_TRAVIS_REPO_NAME ./ci/Unity_v2018.x.ulf
```

For the record, the message I was getting when trying to paste license content directly into  the env var in travis settings:

> The previous command failed, possibly due to a malformed secure environment variable.
>  Please be sure to escape special characters such as ' ' and '$'.
>  For more information, see https://docs.travis-ci.com/user/encryption-keys.

### CircleCI

CircleCI doesn't support multi-lines environment variables. We can use a solution similar to Travis by encrypting the license and decrypting it from the CI using `openssl` [as explained here](https://github.com/circleci/encrypted-files):

Move your `Unity_v2018.x.ulf` to `ci` folder (so you don't clutter your project's root). Just make sure you don't track it inside git.

1. Generate a strong password
2. Store that password in a new `KEY` env var in CircleCI
3. Encrypt the license using the same `KEY`

```bash
export KEY=insert-your-strong-generated-key-here
openssl aes-256-cbc -e -in ci/Unity_v2018.x.ulf -out ci/Unity_v2018.x.ulf-cipher -k $KEY
git add ci/Unity_v2018.x.ulf-cipher
git commit -m "Add encrypted Unity_v2018.x.ulf using openssl aes-256-cbc KEY"
```

## How to add build targets

Supported build targets can be found [here](https://docs.unity3d.com/ScriptReference/BuildTarget.html)

### gitlab-ci

Update [`.gitlab-ci.yml`](.gitlab-ci.yml) by adding a build section like this:

```yaml
build-StandaloneWindows64:
  <<: *build
  variables:
    BUILD_TARGET: StandaloneWindows64
```

### iOS support

**Help wanted!** See [#16](https://gitlab.com/gableroux/unity3d-gitlab-ci-example/issues/16)

### Android support

**Help wanted!** See [#17](https://gitlab.com/gableroux/unity3d-gitlab-ci-example/issues/17)

## How to run scripts manually

You can execute the local scripts and specify the path of your Unity executable using `UNITY_EXECUTABLE`. You may try this in your project before you setup the whole CI so you confirm it works with your current unity version :+1:

### Test

```bash
UNITY_EXECUTABLE="/Applications/Unity/Hub/Editor/2018.2.6f1/Unity.app/Contents/MacOS/Unity" \
  ./local_test.sh
```

### Build

```bash
UNITY_EXECUTABLE="/Applications/Unity/Hub/Editor/2018.2.6f1/Unity.app/Contents/MacOS/Unity" \
  ./local_build.sh
```

## Shameless plug

I made this for free as a gift to the video game community. If this tool helped you, feel free to become a patron for [Totema Studio](https://totemastudio.com) on Patreon: :beers:

[![Totema Studio Logo](./doc/totema-studio-logo-217.png)](https://patreon.com/totemastudio)

[![Become a Patron](./doc/become_a_patron_button.png)](https://www.patreon.com/bePatron?c=1073078)

## License

[MIT](LICENSE.md) © [Gabriel Le Breton](https://gableroux.com)
