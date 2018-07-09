
see here for a list of all permissions your app may need

Permissions overview
The purpose of a permission is to protect the privacy of an Android user. Android apps must request permission to access sensitive user data (such as contacts and SMS), as well as certain system features (such as camera and internet). Depending on the feature, the system might grant the permission automatically or might prompt the user to approve the request.

This page provides an overview to how Android permissions work, including: how permissions are presented to the user, the difference between install-time and runtime permission requests, how permissions are enforced, and the types of permissions and their groups. If you just want a how-to guide for using app permissions, instead see Request App Permissions.

Permission approval
An app must publicize the permissions it requires by including <uses-permission> tags in the app manifest. For example, an app that needs to send SMS messages would have this line in the manifest:

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.snazzyapp">

    <uses-permission android:name="android.permission.SEND_SMS"/>

    <application ...>
        ...
    </application>
</manifest>
If your app lists normal permissions in its manifest (that is, permissions that don't pose much risk to the user's privacy or the device's operation), the system automatically grants those permissions to your app.

If your app lists dangerous permissions in its manifest (that is, permissions that could potentially affect the user's privacy or the device's normal operation), such as the SEND_SMS permission above, the user must explicitly agree to grant those permissions.

For more information about normal and dangerous permissions, see Protection levels.

Request prompts for dangerous permissions
Only dangerous permissions require user agreement. The way Android asks the user to grant dangerous permissions depends on the version of Android running on the user's device, and the system version targeted by your app.

Runtime requests (Android 6.0 and higher)
If the device is running Android 6.0 (API level 23) or higher, and the app's targetSdkVersion is 23 or higher, the user isn't notified of any app permissions at install time. Your app must ask the user to grant the dangerous permissions at runtime. When your app requests permission, the user sees a system dialog (as shown in figure 1, left) telling the user which permission group your app is trying to access. The dialog includes a Deny and Allow button.

If the user denies the permission request, the next time your app requests the permission, the dialog contains a checkbox that, when checked, indicates the user doesn't want to be prompted for the permission again (see figure 2, right).



Figure 1. Initial permission dialog (left) and secondary permission request with option to turn off further requests (right)

If the user checks the Never ask again box and taps Deny, the system no longer prompts the user if you later attempt to requests the same permission.

Even if the user grants your app the permission it requested you cannot always rely on having it. Users also have the option to enable and disable permissions one-by-one in system settings. You should always check for and request permissions at runtime to guard against runtime errors (SecurityException).

For details about how to handle runtime permission requests, see Request App Permissions.

Install-time requests (Android 5.1.1 and below)
If the device is running Android 5.1.1 (API level 22) or lower, or the app's targetSdkVersion is 22 or lower while running on any version of Android, the system automatically asks the user to grant all dangerous permissions for your app at install-time (see figure 2).



Figure 2. Install-time permission dialog

If the user clicks Accept, all permissions the app requests are granted. If the user denies the permissions request, the system cancels the installation of the app.

If an app update includes the need for additional permissions the user is prompted to accept those new permissions before updating the app.

For an overview of the recommended user experience patterns for requesting permissions, see App Permissions Best Practices.

To learn how to check for and request permissions from the user, see Request App Permissions.

Permissions for optional hardware features
Access to some hardware features (such as Bluetooth or the camera) require an app permission. However, not all Android devices actually have these hardware features. So if your app requests the CAMERA permission, it's important that you also include the <uses-feature> tag in your manifest to declare whether or not this feature is actually required. For example:

<uses-feature android:name="android.hardware.camera" android:required="false" />
If you declare android:required="false" for the feature, then Google Play allows your app to be installed on devices that don't have the feature. You then must check if the current device has the feature at runtime by calling PackageManager.hasSystemFeature(), and gracefully disable that feature if it's not available.

If you don't provide the <uses-feature> tag, then when Google Play sees that your app requests the corresponding permission, it assumes your app requires this feature. So it filters your app from devices without the feature, as if you declared android:required="true" in the <uses-feature> tag.

For more information, see Google Play and feature-based filtering

Permission enforcement
Permissions aren't only for requesting system functionality. Services provided by apps can enforce custom permissions to restrict who can use them. For more information on declaring custom permissions, see Define a Custom App Permission.

Activity permission enforcement
Permissions applied using the android:permission attribute to the <activity> tag in the manifest restrict who can start that Activity. The permission is checked during Context.startActivity() and Activity.startActivityForResult(). If the caller doesn't have the required permission then SecurityException is thrown from the call.

Service permission enforcement
Permissions applied using the android:permission attribute to the <service> tag in the manifest restrict who can start or bind to the associated Service. The permission is checked during Context.startService(), Context.stopService() and Context.bindService(). If the caller doesn't have the required permission then SecurityException is thrown from the call.

Broadcast permission enforcement
Permissions applied using the android:permission attribute to the <receiver> tag restrict who can send broadcasts to the associated BroadcastReceiver. The permission is checked after Context.sendBroadcast() returns, as the system tries to deliver the submitted broadcast to the given receiver. As a result, a permission failure doesn't result in an exception being thrown back to the caller; it just doesn't deliver the Intent.

In the same way, a permission can be supplied to Context.registerReceiver() to control who can broadcast to a programmatically registered receiver. Going the other way, a permission can be supplied when calling Context.sendBroadcast() to restrict which broadcast receivers are allowed to receive the broadcast.

Note that both a receiver and a broadcaster can require a permission. When this happens, both permission checks must pass for the intent to be delivered to the associated target. For more information, see Restricting broadcasts with permissions.

Content Provider permission enforcement
Permissions applied using the android:permission attribute to the <provider> tag restrict who can access the data in a ContentProvider. (Content providers have an important additional security facility available to them called URI permissions which is described next.) Unlike the other components, there are two separate permission attributes you can set: android:readPermission restricts who can read from the provider, and android:writePermission restricts who can write to it. Note that if a provider is protected with both a read and write permission, holding only the write permission doesn't mean you can read from a provider.

The permissions are checked when you first retrieve a provider (if you don't have either permission, a SecurityException is thrown), and as you perform operations on the provider. Using ContentResolver.query() requires holding the read permission; using ContentResolver.insert(), ContentResolver.update(), ContentResolver.delete() requires the write permission. In all of these cases, not holding the required permission results in a SecurityException being thrown from the call.

URI permissions
The standard permission system described so far is often not sufficient when used with content providers. A content provider may want to protect itself with read and write permissions, while its direct clients also need to hand specific URIs to other apps for them to operate on.

A typical example is attachments in a email app. Access to the emails should be protected by permissions, since this is sensitive user data. However, if a URI to an image attachment is given to an image viewer, that image viewer no longer has permission to open the attachment since it has no reason to hold a permission to access all email.

The solution to this problem is per-URI permissions: when starting an activity or returning a result to an activity, the caller can set Intent.FLAG_GRANT_READ_URI_PERMISSION and/or Intent.FLAG_GRANT_WRITE_URI_PERMISSION. This grants the receiving activity permission access the specific data URI in the intent, regardless of whether it has any permission to access data in the content provider corresponding to the intent.

This mechanism allows a common capability-style model where user interaction (such as opening an attachment or selecting a contact from a list) drives ad-hoc granting of fine-grained permission. This can be a key facility for reducing the permissions needed by apps to only those directly related to their behavior.

To build the most secure implementation that makes other apps accountable for their actions within yor app, you should use fine-grained permissions in this manner and declare your app's support for it with the android:grantUriPermissions attribute or <grant-uri-permissions> tag.

More information can be found in the Context.grantUriPermission(), Context.revokeUriPermission(), and Context.checkUriPermission() methods.

Other permission enforcement
Arbitrarily fine-grained permissions can be enforced at any call into a service. This is accomplished with the Context.checkCallingPermission() method. Call with a desired permission string and it returns an integer indicating whether that permission has been granted to the current calling process. Note that this can only be used when you are executing a call coming in from another process, usually through an IDL interface published from a service or in some other way given to another process.

There are a number of other useful ways to check permissions. If you have the process ID (PID) of another process, you can use the Context.checkPermission() method to check a permission against that PID. If you have the package name of another app, you can use the PackageManager.checkPermission() method to find out whether that particular package has been granted a specific permission.

Automatic permission adjustments
Over time, new restrictions may be added to the platform such that, in order to use certain APIs, your app must request a permission that it previously did not need. Because existing apps assume access to those APIs is freely available, Android may apply the new permission request to the app's manifest to avoid breaking the app on the new platform version (thereby, "grandfathering" your app for the permission). Android makes the decision as to whether an app might need the permission based on the value provided for the targetSdkVersion attribute. If the value is lower than the version in which the permission was added, then Android adds the permission.

For example, the READ_EXTERNAL_STORAGE permission is enforced beginning with API level 19 to restrict access to the shared storage space. If your targetSdkVersion is 18 or lower, this permission is added to your app on newer versions of Android.

Caution: If a permission is automatically added to your app, your app listing on Google Play lists these additional permissions even though your app might not actually require them. To avoid this and remove the default permissions you don't need, always update your targetSdkVersion to be as high as possible. You can see which permissions were added with each release in the Build.VERSION_CODES documentation.

Protection levels
Permissions are divided into several protection levels. The protection level affects whether runtime permission requests are required.

There are three protection levels that affect third-party apps: normal, signature, and dangerous permissions.

Normal permissions
Normal permissions cover areas where your app needs to access data or resources outside the app's sandbox, but where there's very little risk to the user's privacy or the operation of other apps. For example, permission to set the time zone is a normal permission.

If an app declares in its manifest that it needs a normal permission, the system automatically grants the app that permission at install time. The system doesn't prompt the user to grant normal permissions, and users cannot revoke these permissions.

As of Android 8.1 (API level 27), the following permissions are classified as PROTECTION_NORMAL:

ACCESS_LOCATION_EXTRA_COMMANDS
ACCESS_NETWORK_STATE
ACCESS_NOTIFICATION_POLICY
ACCESS_WIFI_STATE
BLUETOOTH
BLUETOOTH_ADMIN
BROADCAST_STICKY
CHANGE_NETWORK_STATE
CHANGE_WIFI_MULTICAST_STATE
CHANGE_WIFI_STATE
DISABLE_KEYGUARD
EXPAND_STATUS_BAR
GET_PACKAGE_SIZE
INSTALL_SHORTCUT
INTERNET
KILL_BACKGROUND_PROCESSES
MANAGE_OWN_CALLS
MODIFY_AUDIO_SETTINGS
NFC
READ_SYNC_SETTINGS
READ_SYNC_STATS
RECEIVE_BOOT_COMPLETED
REORDER_TASKS
REQUEST_COMPANION_RUN_IN_BACKGROUND
REQUEST_COMPANION_USE_DATA_IN_BACKGROUND
REQUEST_DELETE_PACKAGES
REQUEST_IGNORE_BATTERY_OPTIMIZATIONS
SET_ALARM
SET_WALLPAPER
SET_WALLPAPER_HINTS
TRANSMIT_IR
USE_FINGERPRINT
VIBRATE
WAKE_LOCK
WRITE_SYNC_SETTINGS
Signature permissions
The system grants these app permissions at install time, but only when the app that attempts to use a permission is signed by the same certificate as the app that defines the permission.

Note: Some signature permissions aren't for use by third-party apps.
As of Android 8.1 (API level 27), the following permissions that third-party apps can use are classified as PROTECTION_SIGNATURE:

BIND_ACCESSIBILITY_SERVICE
BIND_AUTOFILL_SERVICE
BIND_CARRIER_SERVICES
BIND_CHOOSER_TARGET_SERVICE
BIND_CONDITION_PROVIDER_SERVICE
BIND_DEVICE_ADMIN
BIND_DREAM_SERVICE
BIND_INCALL_SERVICE
BIND_INPUT_METHOD
BIND_MIDI_DEVICE_SERVICE
BIND_NFC_SERVICE
BIND_NOTIFICATION_LISTENER_SERVICE
BIND_PRINT_SERVICE
BIND_SCREENING_SERVICE
BIND_TELECOM_CONNECTION_SERVICE
BIND_TEXT_SERVICE
BIND_TV_INPUT
BIND_VISUAL_VOICEMAIL_SERVICE
BIND_VOICE_INTERACTION
BIND_VPN_SERVICE
BIND_VR_LISTENER_SERVICE
BIND_WALLPAPER
CLEAR_APP_CACHE
MANAGE_DOCUMENTS
READ_VOICEMAIL
REQUEST_INSTALL_PACKAGES
SYSTEM_ALERT_WINDOW
WRITE_SETTINGS
WRITE_VOICEMAIL
Dangerous permissions
Dangerous permissions cover areas where the app wants data or resources that involve the user's private information, or could potentially affect the user's stored data or the operation of other apps. For example, the ability to read the user's contacts is a dangerous permission. If an app declares that it needs a dangerous permission, the user has to explicitly grant the permission to the app. Until the user approves the permission, your app cannot provide functionality that depends on that permission.

To use a dangerous permission, your app must prompt the user to grant permission at runtime. For more details about how the user is prompted, see Request prompt for dangerous permission.

For a list of dangerous permissions, see table 1 below.

Special permissions
There are a couple of permissions that don't behave like normal and dangerous permissions. SYSTEM_ALERT_WINDOW and WRITE_SETTINGS are particularly sensitive, so most apps should not use them. If an app needs one of these permissions, it must declare the permission in the manifest, and send an intent requesting the user's authorization. The system responds to the intent by showing a detailed management screen to the user.

For details on how to request these permissions, see the SYSTEM_ALERT_WINDOW and WRITE_SETTINGS reference entries.

All permissions provided by the Android system can be found at Manifest.permission.

Permission groups
Permissions are organized into groups related to a device's capabilities or features. Under this system, permission requests are handled at the group level and a single permission group corresponds to several permission declarations in the app manifest. For example, the SMS group includes both the READ_SMS and the RECEIVE_SMS declarations. Grouping permissions in this way enables the user to make more meaningful and informed choices, without being overwhelmed by complex and technical permission requests.



All dangerous Android permissions belong to permission groups. Any permission can belong to a permission group regardless of protection level. However, a permission's group only affects the user experience if the permission is dangerous.

If the device is running Android 6.0 (API level 23) and the app's targetSdkVersion is 23 or higher, the following system behavior applies when your app requests a dangerous permission:

If the app doesn't currently have any permissions in the permission group, the system shows the permission request dialog to the user describing the permission group that the app wants access to. The dialog doesn't describe the specific permission within that group. For example, if an app requests the READ_CONTACTS permission, the system dialog just says the app needs access to the device's contacts. If the user grants approval, the system gives the app just the permission it requested.
If the app has already been granted another dangerous permission in the same permission group, the system immediately grants the permission without any interaction with the user. For example, if an app had previously requested and been granted the READ_CONTACTS permission, and it then requests WRITE_CONTACTS, the system immediately grants that permission without showing the permissions dialog to the user.
Caution: Future versions of the Android SDK might move a particular permission from one group to another. Therefore, don't base your app's logic on the structure of these permission groups.

For example, READ_CONTACTS is in the same permission group as WRITE_CONTACTS as of Android 8.1 (API level 27). If your app requests the READ_CONTACTS permission, and then requests the WRITE_CONTACTS permission, don't assume that the system can automatically grant the WRITE_CONTACTS permission.

If the device is running Android 5.1 (API level 22) or lower, or the app's targetSdkVersion is 22 or lower, the system asks the user to grant the permissions at install time. Once again, the system just tells the user what permission groups the app needs, not the individual permissions. For example, when an app requests READ_CONTACTS the install dialog lists the Contacts group. When the user accepts, only the READ_CONTACTS permission is granted to the app.

Note: Your app still needs to explicitly request every permission it needs, even if the user has already granted another permission in the same group. In addition, the grouping of permissions into groups may change in future Android releases. Your code shouldn't have logic that depends on a set of particular permissions being in the same group.

Table 1. Dangerous permissions and permission groups.

Permission Group	Permissions
CALENDAR	
READ_CALENDAR
WRITE_CALENDAR
CAMERA	
CAMERA
CONTACTS	
READ_CONTACTS
WRITE_CONTACTS
GET_ACCOUNTS
LOCATION	
ACCESS_FINE_LOCATION
ACCESS_COARSE_LOCATION
MICROPHONE	
RECORD_AUDIO
PHONE	
READ_PHONE_STATE
READ_PHONE_NUMBERS
CALL_PHONE
ANSWER_PHONE_CALLS
READ_CALL_LOG
WRITE_CALL_LOG
ADD_VOICEMAIL
USE_SIP
PROCESS_OUTGOING_CALLS
SENSORS	
BODY_SENSORS
SMS	
SEND_SMS
RECEIVE_SMS
READ_SMS
RECEIVE_WAP_PUSH
RECEIVE_MMS
STORAGE	
READ_EXTERNAL_STORAGE
WRITE_EXTERNAL_STORAGE
Viewing an app's permissions
You can view all the permissions currently defined in the system using the Settings app and the shell command adb shell pm list permissions. To use the Settings app, go to Settings > Apps. Pick an app and scroll down to see the permissions that the app uses. For developers, the adb '-s' option displays the permissions in a form similar to how the user sees them:

$ adb shell pm list permissions -s
All Permissions:

Network communication: view Wi-Fi state, create Bluetooth connections, full
internet access, view network state

Your location: access extra location provider commands, fine (GPS) location,
mock location sources for testing, coarse (network-based) location

Services that cost you money: send SMS messages, directly call phone numbers

...
You can also use the adb -g option to grant all permissions automatically when installing an app on an emulator or test device:

$ adb shell install -g MyApp.apk
Additional resources
Request app permissions: The how-to guide for requesting permissions in your app.
Permissions that imply feature requirements: Information about how requesting some permissions implicitly restricts your app to devices that include the corresponding hardware or software feature.
<uses-permission>: API reference for the manifest tag that declares your app's required permissions.
Device compatibility: Information about how Android works on different types of devices and an introduction to how you can optimize your app for each device or restrict your app's availability to different devices.
Android Security Overview: A detailed discussion about the Android platform's security model.
"Mother, May I?" Asking for Permissions: This video from Android Dev Summit 2015 describes best practices for requesting permissions.
Android M Permissions: This video from Google I/O 2015 explains changes made to the permissions model in Android 6.0.
Next
arrow_forward Request app permissions
Content and code samples on this page are subject to the licenses described in the Content License. Java is a registered trademark of Oracle and/or its affiliates.

Last updated June 15, 2018
