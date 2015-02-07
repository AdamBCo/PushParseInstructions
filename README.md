## 1) Generate the SSL Certificate

1. Launch the Keychain Access application on your Mac.
2. Select the menu item Keychain Access > Certificate Assistant > Request a Certificate From a Certificate Authority…
3. Enter your email address, name and leave the CAEmail as "Required"
4. In the "Request is:" change the option from "Emailed to the CA" to "Saved to disk"
5. Click "Continue" to download the .certSigningRequest file to your desktop.


## 2) Create an App ID

1. Go to the [Apple Developer Member Center](https://developer.apple.com/membercenter/ "Title").
2. Select Certificates, Identifiers & Profiles.
3. Select Identifiers from the iOS Apps section.
4. Click the + button in the top right to register a new App Id.
5. Enter a name for your new App ID, then make sure to select the checkbox next to Push Notifications under App Services.
6. Choose an App ID Prefix. The default selection should be correct in most cases.
7. Under App ID Suffix, select Explicit App ID. Enter your iOS app's Bundle ID.
8. Select "Continue" and make sure that all the values were entered correctly.

### 3) Creating a Development SSL Certificate

1. Select your newly created App ID from the list of iOS App IDs, then select "Settings".
2. Scroll down to the Push Notifications section and select "Create Certificate..." under the "Development SSL Certificate" section.
3. Click "Continue"
4. Click "Choose File" and upload the .certSigningRequest you created in the beginning.
4. Select "Generate" and then click "Done" once the SLL certificate has been created.
5. Double click on the downloaded SSL certificate to install it in your Keychain.
6. In Keychain Access, under "My Certificates", find the certificate you just added.
7. Right-click on it, select "Export Apple Development IOS Push Services:...", and save it as a .p12 file.
8. When prompted for a password, ignore it and click "Enter".


### 3) Creating a Production SSL certificate.

It's going to get a little repetitive here, but we're going to be creating a Distribution Provisioning Profile above.

1. Select your newly created App ID from the list of iOS App IDs, then select "Settings".
2. Scroll down to the Push Notifications section and select "Create Certificate..." under the "Production SSL Certificate" section.
3. Click "Continue"
4. Click "Choose File" and upload the .certSigningRequest you created in the beginning.
4. Select "Generate" and then click "Done" once the SLL certificate has been created.
5. Double click on the downloaded SSL certificate to install it in your Keychain.
6. In Keychain Access, under "My Certificates", find the certificate you just added.
7. Right-click on it, select "Export Apple Development IOS Push Services:...", and save it as a .p12 file.
8. When prompted for a password, ignore it and click "Enter".


### 5) Configure Parse

1. To use Push Notifications with Parse, you will need to enable this feature in your Parse app and upload the Push SSL certificate you created above.
2. Navigate to your Parse app on the Parse Dashboard, and select the "Settings" tab.
3. Select "Push" from the left-hand side menu and select "Select your certificate" under the "Apple Push Certificates" header.
4. Upload the "Development .p12 Certificate" you exported from your Keychain earlier.
5.  Upload the "Distribution .p12 Certificate" you exported from your Keychain earlier.



### 3) Create an iOS App Development Provisioning Profile

1. Navigate to the Apple Developer Member Center website, and select Certificates, Identifiers & Profiles.
2. Select Provisioning Profiles from the iOS Apps section.
3.  Select the + button in the top right to create a new iOS Provisioning Profile.
4. Choose "iOS App Development" as your provisioning profile type then select "Continue". We will create Ad Hoc and App Store profiles later.
5. Choose the App ID you created earlier from the drop down then select "Continue".
6. Make sure to select your iOS Development certificate in the next screen, then select "Continue".
7. You will be asked to select which devices will be included in the provisioning profile. Select "Continue" after selecting the devices you will be testing with.
8. Choose a name for this provisioning profile, such as "My Parse Push App Development Profile", then select "Generate".
9. Download the generated provisioning profile from the next screen by selecting the "Download" button.
10. Install the profile by double-clicking on the downloaded file.

### 6) Double check that things were set up properly



### 4) Configuring a Push Enabled iOS Application

1. In the Info.plist file under the Supporting Files folder, modify the Bundle Identifier field to match your App ID's Bundle Identifier (ex. com.example.MyParsePushApp).
2. Select your project file in the left-hand menu. Under Project, navigate to "Build Settings", and find (or search for) the "Code Signing Identity" field.
3. Set all values under this heading to match the provisioning profile installed earlier (ex. "My Parse Push App Development Profile").
4. On the left-hand side, select your project's name under "Targets". Again, navigate to "Build Settings" and find the "Code Signing Identity" field. Ensure that all values here also match the new provisioning profile.

### 4) Adding Parse Code to our application.

5. Adding Code for a Push Enabled iOS Application
6. We are now ready to start programming. We need to make a few modification to the app delegate in order to receive push notifications.

* To register the current device for push, call the method [application registerForRemoteNotifications] in the app delegate's -application:didFinishLaunchingWithOptions: method.

``` 
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ...
  UIUserNotificationType userNotificationTypes = (UIUserNotificationTypeAlert |
                                                  UIUserNotificationTypeBadge |
                                                  UIUserNotificationTypeSound);
  UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:userNotificationTypes
                                                                           categories:nil];
  [application registerUserNotificationSettings:settings];
  [application registerForRemoteNotifications];
  ...
}

```

* If your application doesn't already contains the method [didRegisterForRemoteNotificationWithDeviceToken:] copy and paste the code below in your App Delegate, other wise copy and past the code iinside the method.

```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  // Store the deviceToken in the current installation and save it to Parse.
  PFInstallation *currentInstallation = [PFInstallation currentInstallation];
  [currentInstallation setDeviceTokenFromData:deviceToken];
  currentInstallation.channels = @[ @"global" ];
  [currentInstallation saveInBackground];
}

```

When a push notification is received while the application is not in the foreground, it is displayed in the iOS Notification Center. However, if the notification is received while the app is active, it is up to the app to handle it. To do so, we can implement the [application:didReceiveRemoteNotification] method in the app delegate. In our case, we will simply ask Parse to handle it for us. Parse will create a modal alert and display the push notification's content.

```
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  [PFPush handlePush:userInfo];
}
```

You should now run your application (on your iOS device) to make sure everything is set up correctly. If it is, the first time you run this app you should see a modal alert requesting permission from the user to send push notifications.


## Sending Push Notifications

Parse allows you to send push notifications from the Parse website, the web API as well as the Parse SDK.

6.1. Parse website

Let's start with the Parse website. By navigating to your Parse app and selecting the "Push Notifications" tab, you can use the text box to broadcast a message to "Everyone". Simply enter a message and click send! If you've installed the app on a device, you should see the notification appear within a few seconds.

6.2. REST API

You can use the Parse REST API to send push notifications to all iOS devices by sending a POST request. Here is an example of a broadcast notification containing the message "Hello World!" sent using curl. Detailed information about the required format can be found in the REST documentation.

curl -X POST \
  -H "X-Parse-Application-Id: ${APPLICATION_ID}" \
  -H "X-Parse-REST-API-Key: ${REST_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
        "where": {
          "deviceType": "ios"
        },
        "data": {
          "alert": "Hello World!"
        }
      }' \
  https://api.parse.com/1/push
6.3. From the App

You can also send push notifications directly from a mobile application. Remember that you need to have enabled this feature in the Parse app's settings tab by selecting "Yes" under the heading "Client push enabled?". There are several methods that can be called to send push notifications. You can consult the full list in the iOS API documentation. Here is an example:

// Create our Installation query
PFQuery *pushQuery = [PFInstallation query];
[pushQuery whereKey:@"deviceType" equalTo:@"ios"];
 
// Send push notification to query
[PFPush sendPushMessageToQueryInBackground:pushQuery 
                               withMessage:@"Hello World!"];
6.4. From Cloud Code

Finally, you can send pushes automatically from Cloud Code. The Parse.Cloud.afterSave method lets us execute arbitrary code after an object is saved successfully. For example, if you wanted to send a push whenever a "Comment" object is saved:

Parse.Cloud.afterSave("Comment", function(request) {
  // Our "Comment" class has a "text" key with the body of the comment itself
  var commentText = request.object.get('text');
 
  var pushQuery = new Parse.Query(Parse.Installation);
  pushQuery.equalTo('deviceType', 'ios');
    
  Parse.Push.send({
    where: pushQuery, // Set our Installation query
    data: {
      alert: "New comment: " + commentText
    }
  }, {
    success: function() {
      // Push was successful
    },
    error: function(error) {
      throw "Got an error " + error.code + " : " + error.message;
    }
  });
});
In this tutorial, we've learned how to enable and use push notifications in an iOS application. We began by creating an App ID, generating an associated SSL certificate, and then linking this App ID to a new provisioning profile. Next, we configured the Parse app by uploading the SSL certificate. Afterwards, we created an iOS application and configured it to use the new App ID and provisioning profile. We then implemented a set of delegation methods used to register the app to use push. Finally, we looked at four ways to send push notifications to users: the Parse website, the REST API, the Parse iOS SDK, and Cloud Code.

If you run into any problems, take a look at the "Troubleshooting Tips" section below.

Read on to learn the necessary steps to get your app ready for the App Store.

7. Preparing for the App Store

You've configured your app to receive push notifications during development. Prior to submitting your app to the App Store, you will need to configure push notifications for distribution.

There are two types of distribution profiles: Ad Hoc, and App Store. You will need the latter to submit your app to the App Store, however it is good practice to test push notifications using an Ad Hoc profile prior to submitting your app.

7.1. Configuring your App for Distribution Push Notifications

In Section 1.3., you configured your App ID for Push Notifications in Development. Retrace steps 1 through 7, but select "Production Push SSL Certificate" in step 2 instead.

Your App ID should now be configured for both Development and Distribution push notifications. Make sure to download the new Production SSL Certificate from the App ID Settings screen.

Configured for Development and Distribution
Double click on the downloaded SSL certificate to install it in your keychain. Right-click on it and export it as a .p12 file. Again, don't enter an export password when prompted.

Go back to Section 2 and retrace steps 1 through 10, making sure to select "Ad Hoc" under Distribution in Step 4. You should also use a different name in Step 8, such as "My Parse Push App Ad Hoc Profile". This should help you distinguish between development and distribution profiles.

Retrace the steps from Section 3, and upload your exported Production .p12 certificate to Parse instead.

In Section 4, you configured your app to use a Development provisioning profile. Retrace your steps, but this time choose your new Distribution Ad Hoc provisioning profile instead.

Build and run your app on an iOS device. Verify that push notifications are delivered successfully.

Note that once you have uploaded a production push certificate to Parse, you will only be able to target devices using a distribution provisioning profile. Devices running an app signed with a development provisioning profile will need to install the newly provisioned build again.

7.2. Configuring your App for App Store Distribution

You have now confirmed that your app is configured correctly to receive distribution push notifications using an Ad Hoc provisioning profile. Now it's time to submit your app to the App Store.

Follow steps 1 through 10 from Section 2, making sure to select "App Store" under Distribution for Step 4. Note that this time around, since you will be submitting your app to the App Store, you can skip Step 7 (selecting test devices).

Go through Section 4 again, this time selecting your new App Store Distribution provisioning profile.

Build and archive your iOS app, then submit to the App Store.

