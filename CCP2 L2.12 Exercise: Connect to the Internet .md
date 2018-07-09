# CCP2 L2.12 Exercise: Connect to the Internet

Now it's your turn! Follow the TODOS. But it will crash when you run it.

![screen shot 2018-07-09 at 1 39 04 am](https://user-images.githubusercontent.com/9452839/42461369-3341b338-8355-11e8-9e81-644a7a2c559a.png)


## Exercise Code
Exercise: T02.04-Exercise-ConnectingToTheInternet


## Solution Code

### app > manifests > AndroidManifest.xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.android.datafrominternet">

    <!--COMPLETED (1) Add the INTERNET permission-->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name="com.example.android.datafrominternet.MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>

</manifest>

**

### app > java > MainActivity.java

package com.example.android.datafrominternet;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.EditText;
import android.widget.TextView;

import com.example.android.datafrominternet.utilities.NetworkUtils;

import java.io.IOException;
import java.net.URL;

public class MainActivity extends AppCompatActivity {

    private EditText mSearchBoxEditText;

    private TextView mUrlDisplayTextView;

    private TextView mSearchResultsTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mSearchBoxEditText = (EditText) findViewById(R.id.et_search_box);

        mUrlDisplayTextView = (TextView) findViewById(R.id.tv_url_display);
        mSearchResultsTextView = (TextView) findViewById(R.id.tv_github_search_results_json);
    }

    /**
     * This method retrieves the search text from the EditText, constructs
     * the URL (using {@link NetworkUtils}) for the github repository you'd like to find, displays
     * that URL in a TextView, and finally fires off an AsyncTask to perform the GET request using
     * our (not yet created) {@link GithubQueryTask}
     */
    private void makeGithubSearchQuery() {
        String githubQuery = mSearchBoxEditText.getText().toString();
        URL githubSearchUrl = NetworkUtils.buildUrl(githubQuery);
        mUrlDisplayTextView.setText(githubSearchUrl.toString());
        // COMPLETED (2) Call getResponseFromHttpUrl and display the results in mSearchResultsTextView
        // COMPLETED (3) Surround the call to getResponseFromHttpUrl with a try / catch block to catch an IO Exception
        String githubSearchResults = null;
        try {
            githubSearchResults = NetworkUtils.getResponseFromHttpUrl(githubSearchUrl);
            mSearchResultsTextView.setText(githubSearchResults);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int itemThatWasClickedId = item.getItemId();
        if (itemThatWasClickedId == R.id.action_search) {
            makeGithubSearchQuery();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
}


## QUESTION 2 OF 2
Look in the Android Monitor logcat. What error do you see when the app tries to connect to the Internet?

Bottom Pane > Logcat

07-09 08:54:58.572 3117-3117/com.example.android.datafrominternet D/NetworkSecurityConfig: No Network Security Config specified, using platform default
07-09 08:54:58.679 3117-3117/com.example.android.datafrominternet D/AndroidRuntime: Shutting down VM
07-09 08:54:58.804 3117-3117/com.example.android.datafrominternet E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.example.android.datafrominternet, PID: 3117
    android.os.NetworkOnMainThreadException

Great work! If you're running on Android Honeycomb or higher, the answer is C. In Gingerbread you could technically still do this, but it was bad practice.
