**This repository is in the process of being deprecated.  See the [Android TODO app](https://developer.couchbase.com/documentation/mobile/1.3/training/develop/create-database/index.html) in the training documentation for a more up-to-date sample.

## Working with CB Lite Android 1.4.1, CB Sync Gateway 1.5.1, CB Server 5.0.0
Here are my very rough notes for using this project with newer versions of Couchbase software
1. Import project into Android Studio
1. Update dependencies
1. Add Facebook App
    1. Create a Facebook Developer account: https://developers.facebook.com/
    2. Add the To Do Lite app
1. Configure Facebook account
    1. https://www.hull.io/help/facebook-app-not-setup-error/
1. Configure app for Facebook login
    1. https://developers.facebook.com/docs/facebook-login/android/
    1. Generate development hash key. MacOS
        ```
        keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
        ```
    1. Request to access user’s email address
        1. Facebook account must have an email address, otherwise sync gateway 1.5.1 reports an error
           1. This is fixed in newer versions
        1. LoginActivity.java: 
        ```
        facebookLoginButton.setReadPermissions("email")
        ```
1. Open port access or hardcode the IP address for the sync gateway
    1. if using `localhost:4984` as described in the instructions
        ```
        adb reverse tcp:4984 tcp:4984
        ```
    1. Application.java: 
        ```
        private static final String SYNC_URL_HTTP = "http://sync-gateway-ip-address:4984/todolite";
        ```
1. Configure sync gateway to sync with remote DB
    1. Login to CB Admin console
        1. http://example.com:8091/ui/index.html#!/servers/list
    1. Create bucket and user account
        1. bucket: todolite
        1. user: todolite/password
            1. Bucket Full Access, Bucket Admin
    1. Add to sync gateway config
    1. Background
        1. Shared Bucket Access (1.5.1):
            1. https://developer.couchbase.com/documentation/mobile/1.5/guides/sync-gateway/shared-bucket-access.html
        1. Tutorial showing a similar integration using Couchbase Mobile 2.0 DP: http://docs.couchbase.com/tutorials/travel-sample/develop/java

## ToDo Lite for Android

[![Join the chat at https://gitter.im/couchbase/mobile](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/couchbase/mobile?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

A shared todo app that shows how to use the [Couchbase Lite Android](https://github.com/couchbase/couchbase-lite-android) framework to embed a nonrelational ("NoSQL") document-oriented database in an Android app and sync it with [Couchbase Server](http://www.couchbase.com/nosql-databases/couchbase-server) in a public or private cloud.

![screenshot](http://f.cl.ly/items/1K2e200t2D3s1l0i473e/ToDoLite.gif)

## Get the code

```
$ git clone https://github.com/couchbaselabs/ToDoLite-Android.git
$ cd ToDoLite-Android
```

## Build and run the app

* Import the project into Android Studio by selecting `build.gradle` or `settings.gradle` from the root of the project.
* Run the app using the "play" or "debug" button.

## Point to your own Sync Gateway

1. [Download Sync Gateway](http://www.couchbase.com/nosql-databases/downloads#couchbase-mobile).
2. Start Sync Gateway with the configuration file in the root of this project.

    ```bash
    ~/Downloads/couchbase-sync-gateway/bin/sync_gateway sync-gateway-config.json
    ```

3. Open **Application.java** and update the `SYNC_URL_HTTP` constant to point to your Sync Gateway instance.

    ```java
    private static final String SYNC_URL_HTTP = "http://localhost:4984/todolite";
    ```

    You can use the `adb reverse tcp:4984 tcp:4984` command to open the port access from the host to the Android emulator. This command is only available on devices running android 5.0+ (API 21).

4. Log in with your Facebook account.
5. Add lists and tasks and they should be visible on the Sync Gateway Admin UI on [http://localhost:4985/_admin/](http://localhost:4985/_admin/).

## Community

If you have any comments or suggestions, please join [our forum](https://forums.couchbase.com/c/mobile) and let us know.

## License

Released under the Apache license, 2.0.

Copyright 2011-2014, Couchbase, Inc.
