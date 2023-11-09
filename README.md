# project guide iOS

1. Create a git repo

   [GitHub](https://github.com/)

2. Create a project

   use Xcode

3. Use cocoapods

   - create `Profile` file: `pod init` 
   - input any frameworks you need and run `pod install`
   - after `pod install` successful, will auto create `xcworkspace` file
   - modify `.gitignore` file, add `Pod/` to ignore `Pod` floder
   - ignore pod framewroks warnings, add `inhibit_all_warnings!` to `Podfile` 

4. Multiple project

   you can use a single project to develop all features, but it's better to split your project, like main project and a common component project, workspace can help to to manager multiple projects

   - under the main project floder, create a `Dynamic framworks` or `Static library`
   - open the main project's workspace and add the subproject
   - link the subproject to main project, open the main project target's `Build Phases`, open `Link Binary Witth Librarues` and add the subproject's framework
   - pay attention to that the subproject's class or functions need mark `Public`
   - multiple projects use the same `Podfile` to manage the frameworks, need pay attention to the project's path, so you may meet `Unable to find a target named` check the error project's path and set `project 'xx/xx'`in `Podflie`
   - when you create a `framwork` and link it to the main project, you can run successfully on the iOS simulator, but when use the real device it may crash, need go to the `Build Phases` and add a new `Copy File Scprit` and set the `framework` you need

5. Multiple configuration and scheme

   sometime you need different environment's build, like `dev` or `prod`, 

   - create multiple configuration, go to main project's `Project`'s `Info`, as default project contain `Debug` and `Release` configuration, you can reuse the exist configuations or create new, for example, create a new `Debug(Dev)` configuration, and select `Duplicate Debug Configuation` , xcode will auto create new configuations for everywhere
   - create new scheme, also can duplicate exist one and select the corresponding configuration
   - if your main project is linked with some other subproject, subporject also create the same configurations, otherwise the main project will complie fail
   - you may meet `unable to load contents file list` error, maybe because the `cocoapods`, you can re-run `pod install`
   - you may meet `dyld: Library not loaded … Reason: Image not found`, try to re-run `pod install`

6. Manage certificates

   use `fastlane` tool manage certificates

   - create a new repo to store certificates
   - cd to certificate's git repo root floder
   - run `fastlane match init` to create a `Matchfile` file
     - select your storage modes (like `git`)
     - input your certificate repo's url (like https://xxxxx)
   -  after last step successful, open `Matchfile` input `app_identifier` and `username`
        - `app_identifier` is your app's `bundle id`, so you need login you apple developer to create the app id manually
        - `username` is your apple account email
        -  need to make sure that your apple developer's device list is not empty, because `fastlane` create the `profile` need device, otherwise will fail
   - run `fastlane match development` and it will help to create the dev profile
   - if it's the first time to run `fastlane match`, will popup some questions
     - Passphrase for Match storage: remember it, if you change the develop machine need input it
     - Digit code:  apple will send to you
     - If you have multiple apple account will let you choose
     - When you meet `Password for login keychain` please input your computer's unlock password
   - run `fastlane match appstore` will create the distribution profile
   - after `fastlane match` successful, fastlane will auto save the certification to your machine's keychain
   - now you can open your main project and choose the profile
   - if meet unmatched error just clear the cache and restart Xcode

7. Manage test devices

   - If your main project hasn't create fastlane script, run `fastlane init` and then will let you choose the mode, select manual and write the script later

   - after `fastlane init` there will create a fastlane floder under your main project path

   - open the `Appfile` under the `fastlane` floder, iuput infos, `bundle id` 、`apple account`、`team id`、`itc team id` (you can find the `itc team id` when you upload your app to testflight at the first time)

   - run `fastlane match init` under the `fastlane` floder to create a match file and input the certificatees's repo url 、on need set  `apple account`、`bundle id` in the `Matchfile` you can manage them in the `Appfile`

   - now you can write your fastlane script

   - open the `Fastfile`

   - ```js
       lane :devices do
         register_devices(
           devices: {
             "device's name" => "device's UDID"
           }
         )
         match(type: "development", force_for_new_devices: true)
       end
     ```

8. Build and upload to TestFlight

   - first of all, you need create your app in the `Apple Store Connect`

   - continue open the `Fastfile` and write the script

   - can use the format date string as the build number

     ```js
       def buildNumberUpdate
       # use the date format string as build number
       increment_build_number(
         build_number: "#{Time.new.strftime("%Y.%m.%d.%H.%M")}"
       )
       end
     ```

   - build and upload to testflight script

     ```js
       lane :build_dev do
         match(type: "appstore", readonly: true)
         buildNumberUpdate
         build_app( 
           # for more details => https://docs.fastlane.tools/actions/build_app
           workspace: "XXX",  # workspace's name
           scheme: "XXX", # scheme's name
           export_method: "app-store", # export mode
           output_directory: "./builds/testflight",  # the path to save the ipa
           configuration: "XXX",  # compile environment
           clean: true   # if need clean before build
         )
         upload_to_testflight
       end
     ```
   - generate upload access
   ```
   https://stackoverflow.com/questions/74210927/fastlane-altool-error-unable-to-upload-archive-failed-to-get-authorization
   ```

9. Third frameworks

   - [R.swift](https://github.com/mac-cain13/R.swift)

     if you main project links some subproject, the R.swift script should be created in the subproject, and the R.swift script should add `--accessLevel public`
     if you init R.swift in sub-project, you need create a assets and link it to `Development Assets`

   - [Sourcery](https://github.com/krzysztofzablocki/Sourcery)

     Auto protocols help you reduce repeative code

   - [SwiftLint](https://github.com/realm/SwiftLint)

     if your workspace contain multiple projects, you should config swfit lint for each project

     suggest you create a `.swiftlint.yml` under the project's floder, that you can edit youself's config file

   - [multi-language-mobile](https://github.com/codeLockers/multi-language-mobile)

     you can manage your copy file on the google sheet and share it, because it's a node project,

     - you need to enable your [google drive](https://developers.google.com/drive/api/v3/quickstart/nodejs), go to the website and sign in

     - go to [Google Cloud Platform](https://console.cloud.google.com/home/dashboard?project=fishpond-translation) to create a project and get the `Credentials`

     - create project and select the project

     - enter the project's homepage and scroll down the page to find the `Google Drive API` click it and enable it

     - now you should jump to the `Google Drive API` page automaticlly, select the credentials menu and click `CREATE CREDENTIALs` 

     - before that you may need to `CONFIGURE CONSENT SCREEN` , just go and select the `Intrenal` type then need you input some infos

     - after that back to `Credentials` menu, and click `CREATE CREDENTIALs` choose `OAuth client ID` 

     - create OAuth client ID, select `Desktop app` and input name, then download the json file

     - remame the json file as `google-credential.json` and move it to the main project root path

     - create google translation file

       | Key      | en    | Zh   |
       | -------- | ----- | ---- |
       | test key | hello | 你好 |

     - run `yarn add multi-language-mobile --dev` add the tool

     - edit the `package.json`

     - add `"localize": "multi-language"` to the `script` block so you can run `yarn localize` to update your languages copy

     - get the google file's id and paste it to `package.jsons`

       ```js
         "multi-language": {
           "googleCredential": "./google-credential.json",
           "googleFileId": "google file id",
           "outputDir": "file output path",
           "platforms": [
             "ios"
           ]
         }
       ```

     - run `yarn localize`, at the first time you will get a url, you can get a code from the website

     - you can download the translation file now, but still need to add the  `localizable.string` to project manually

     - `multi-language-mobile` only support Chinese and English,  if you want support more, you can find `multi-language-mobile-extension` tool
     
     10. can use [String+Extension](https://gist.github.com/jhonny-me/dd2edb43fb167d1f55f79021718507b7) combine with `R.swift`
     11. Check the latest [solution](https://github.com/jhonny-me/mirrorrim#How-to-setup-Google-Account) 
     
   - [husky](https://typicode.github.io/husky/#/)

     help you to standard your git flow

     - first go to the website and jump to the `Automic(recommended)`

     - follow the guide an install husky `npm install husky -D`([github](https://github.com/typicode/husky))

     - `package.json` 、`.git`  should in same path when you install `husky`

     - after install successful, `.husky` floder will be added to your project

     - run `npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'` will add the` commit-msg` script 

     - install `commitlint` tool, `npm install @commitlint/cli @commitlint/config-conventional`

     - exprort linit file, `echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js`

     - edit the `commitlint.config.js` file 

       ```js
       "module.exports = {extends: ['@commitlint/config-conventional']}"
       
       module.exports = {
           extends: ['@commitlint/config-conventional'],
           rules: {
               'type-enum': [2, 'always', [
                   "feat", "fix", "docs", "style", "refactor", "perf", "test", "build", "ci", "chore", "revert"
               ]],
               'subject-full-stop': [0, 'never'],
               'subject-case': [0, 'never']
       
           }
       };
       ```

10. Manager git branch

    Nothing to say, different team and people have different rules

