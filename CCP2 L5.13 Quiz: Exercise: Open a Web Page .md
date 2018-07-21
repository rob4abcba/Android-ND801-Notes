
QUIZ QUESTION
Now it's your turn! Complete the TODOs in T04b.01-Exercise-OpenWebpage.

Create a function called openWebPage that accepts a String as a parameter.

Replace Toast message.

Call your function with https://www.udacity.com as the parameter

Showing  with 36 additions and 8 deletions.
   
44  app/src/main/java/com/example/android/implicitintents/MainActivity.java
@@ -15,6 +15,8 @@
  */	  */
 package com.example.android.implicitintents;	 package com.example.android.implicitintents;
 	 
+import android.content.Intent;
+import android.net.Uri;
 import android.os.Bundle;	 import android.os.Bundle;
 import android.support.v7.app.AppCompatActivity;	 import android.support.v7.app.AppCompatActivity;
 import android.view.View;	 import android.view.View;
@@ -35,10 +37,11 @@ protected void onCreate(Bundle savedInstanceState) {
      * @param v Button that was clicked.	      * @param v Button that was clicked.
      */	      */
     public void onClickOpenWebpageButton(View v) {	     public void onClickOpenWebpageButton(View v) {
-        // TODO (5) Create a String that contains a URL ( make sure it starts with http:// or https:// )	+        // COMPLETED (5) Create a String that contains a URL ( make sure it starts with http:// or https:// )
+        String urlAsString = "http://www.udacity.com";
 	 
-        // TODO (6) Replace the Toast with a call to openWebPage, passing in the URL String from the previous step	+        // COMPLETED (6) Replace the Toast with a call to openWebPage, passing in the URL String from the previous step
-        Toast.makeText(this, "TODO: Open a web page when this button is clicked", Toast.LENGTH_SHORT).show();	+        openWebPage(urlAsString);
     }	     }
 	 
     /**	     /**
@@ -77,12 +80,37 @@ public void createYourOwn(View v) {
                 .show();	                 .show();
     }	     }
 	 
-    // TODO (1) Create a method called openWebPage that accepts a String as a parameter	+    // COMPLETED (1) Create a method called openWebPage that accepts a String as a parameter
-    // Do steps 2 - 4 within openWebPage	+    /**
+     * This method fires off an implicit Intent to open a webpage.
+     *
+     * @param url Url of webpage to open. Should start with http:// or https:// as that is the
+     *            scheme of the URI expected with this Intent according to the Common Intents page
+     */
+    private void openWebPage(String url) {
+        // COMPLETED (2) Use Uri.parse to parse the String into a Uri
+        /*
+         * We wanted to demonstrate the Uri.parse method because its usage occurs frequently. You
+         * could have just as easily passed in a Uri as the parameter of this method.
+         */
+        Uri webpage = Uri.parse(url);
 	 
-        // TODO (2) Use Uri.parse to parse the String into a Uri	+        // COMPLETED (3) Create an Intent with Intent.ACTION_VIEW and the webpage Uri as parameters
+        /*
+         * Here, we create the Intent with the action of ACTION_VIEW. This action allows the user
+         * to view particular content. In this case, our webpage URL.
+         */
+        Intent intent = new Intent(Intent.ACTION_VIEW, webpage);
 	 
-        // TODO (3) Create an Intent with Intent.ACTION_VIEW and the webpage Uri as parameters	+        // COMPLETED (4) Verify that this Intent can be launched and then call startActivity
+        /*
+         * This is a check we perform with every implicit Intent that we launch. In some cases,
+         * the device where this code is running might not have an Activity to perform the action
+         * with the data we've specified. Without this check, in those cases your app would crash.
+         */
+        if (intent.resolveActivity(getPackageManager()) != null) {
+            startActivity(intent);
+        }
+    }
 	 
-        // TODO (4) Verify that this Intent can be launched and then call startActivity	
 } 	 } 
No commit comments
