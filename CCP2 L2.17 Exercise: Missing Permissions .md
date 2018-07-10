
# CP2 L2.17 Exercise: Missing Permissions

## Missing Permission
Comment out the <uses-permission android:name="android.permission.INTERNET" /> statement in the AndroidManifest.xml and then run the app. Make a search in the app, and then look in the Android Monitor logcat.

## QUIZ QUESTION
In the Android Monitor logcat, what error do you see when the app tries to connect to the Internet?

ANSWER: That's right! You see a SecurityException. Now you'll know to look to make sure that you've added the appropriate permissions when you see a SecurityException. You can uncomment that uses-permission statement now if you like --- although you're going to be opening up a new project for the next exercise.



