
# CCP2 L2.18 Exercise: Add Polish

![todo](https://user-images.githubusercontent.com/9452839/42542800-62b21442-845d-11e8-9b74-e7c6d3e84996.PNG)

## app > res > values > strings.xml

<resources>
    <string name="app_name">Github Query</string>
    <string name="search">Search</string>
    <!--COMPLETED (1) Add a string stating that an error has occurred-->
    <string name="error_message">
        Failed to get results. Please try again.
    </string>
</resources>


## app > res > layout > activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<!--
  Copyright (C) 2016 The Android Open Source Project

  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.example.android.datafrominternet.MainActivity">

    <EditText
        android:id="@+id/et_search_box"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter a query, then click Search"
        android:textSize="22sp" />

    <TextView
        android:id="@+id/tv_url_display"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="Click search and your URL will show up here!"
        android:textSize="22sp" />

    <!--COMPLETED (2) Wrap the ScrollView in a FrameLayout-->
    <!--COMPLETED (3) Make the width and height of the FrameLayout match_parent-->
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ScrollView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp">

            <TextView
                android:id="@+id/tv_github_search_results_json"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Make a search!"
                android:textSize="18sp" />
        </ScrollView>

        <!--COMPLETED (4) Add a TextView to display an error message-->
        <!--COMPLETED (5) Set the text size to 22sp-->
        <!--COMPLETED (6) Give the TextView an id of @+id/tv_error_message_display-->
        <!--COMPLETED (7) Set the layout_height and layout_width to wrap_content-->
        <!--COMPLETED (8) Add 16dp of padding to the error display -->
        <!--COMPLETED (9) Use your strings.xml error message to set the text for the error message-->
        <!--COMPLETED (10) Set the visibility of the view to invisible-->
        <!--COMPLETED (11) Make this TextView a child of the FrameLayout the you added above-->
        <TextView
            android:id="@+id/tv_error_message_display"
            android:textSize="22sp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="16dp"
            android:text="@string/error_message"
            android:visibility="invisible" />

        <!--COMPLETED (18) Add a ProgressBar to indicate loading to your users-->
        <!--COMPLETED (19) Give the ProgressBar an id of @+id/pb_loading_indicator-->
        <!--COMPLETED (20) Set the layout_height and layout_width to 42dp-->
        <!--COMPLETED (21) Set the layout_gravity to center-->
        <!--COMPLETED (22) Set the visibility of the ProgressBar to invisible-->
        <!--COMPLETED (23) Make this ProgressBar a child of the FrameLayout that you added above-->
        <ProgressBar
            android:id="@+id/pb_loading_indicator"
            android:layout_height="42dp"
            android:layout_width="42dp"
            android:layout_gravity="center"
            android:visibility="invisible" />
    </FrameLayout>
</LinearLayout>

## app > java > MainActivity.java

package com.example.android.datafrominternet;

import android.os.AsyncTask;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.EditText;
import android.widget.ProgressBar;
import android.widget.TextView;

import com.example.android.datafrominternet.utilities.NetworkUtils;

import java.io.IOException;
import java.net.URL;

public class MainActivity extends AppCompatActivity {

    private EditText mSearchBoxEditText;

    private TextView mUrlDisplayTextView;

    private TextView mSearchResultsTextView;

    // COMPLETED (12) Create a variable to store a reference to the error message TextView
    private TextView mErrorMessageDisplay;

    // COMPLETED (24) Create a ProgressBar variable to store a reference to the ProgressBar
    private ProgressBar mLoadingIndicator;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mSearchBoxEditText = (EditText) findViewById(R.id.et_search_box);

        mUrlDisplayTextView = (TextView) findViewById(R.id.tv_url_display);
        mSearchResultsTextView = (TextView) findViewById(R.id.tv_github_search_results_json);

        // COMPLETED (13) Get a reference to the error TextView using findViewById
        mErrorMessageDisplay = (TextView) findViewById(R.id.tv_error_message_display);

        // COMPLETED (25) Get a reference to the ProgressBar using findViewById
        mLoadingIndicator = (ProgressBar) findViewById(R.id.pb_loading_indicator);
    }

    /**
     * This method retrieves the search text from the EditText, constructs the
     * URL (using {@link NetworkUtils}) for the github repository you'd like to find, displays
     * that URL in a TextView, and finally fires off an AsyncTask to perform the GET request using
     * our {@link GithubQueryTask}
     */
    private void makeGithubSearchQuery() {
        String githubQuery = mSearchBoxEditText.getText().toString();
        URL githubSearchUrl = NetworkUtils.buildUrl(githubQuery);
        mUrlDisplayTextView.setText(githubSearchUrl.toString());
        new GithubQueryTask().execute(githubSearchUrl);
    }

    // COMPLETED (14) Create a method called showJsonDataView to show the data and hide the error
    /**
     * This method will make the View for the JSON data visible and
     * hide the error message.
     * <p>
     * Since it is okay to redundantly set the visibility of a View, we don't
     * need to check whether each view is currently visible or invisible.
     */
    private void showJsonDataView() {
        // First, make sure the error is invisible
        mErrorMessageDisplay.setVisibility(View.INVISIBLE);
        // Then, make sure the JSON data is visible
        mSearchResultsTextView.setVisibility(View.VISIBLE);
    }

    // COMPLETED (15) Create a method called showErrorMessage to show the error and hide the data
    /**
     * This method will make the error message visible and hide the JSON
     * View.
     * <p>
     * Since it is okay to redundantly set the visibility of a View, we don't
     * need to check whether each view is currently visible or invisible.
     */
    private void showErrorMessage() {
        // First, hide the currently visible data
        mSearchResultsTextView.setVisibility(View.INVISIBLE);
        // Then, show the error
        mErrorMessageDisplay.setVisibility(View.VISIBLE);
    }

    public class GithubQueryTask extends AsyncTask<URL, Void, String> {

        // COMPLETED (26) Override onPreExecute to set the loading indicator to visible
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            mLoadingIndicator.setVisibility(View.VISIBLE);
        }

        @Override
        protected String doInBackground(URL... params) {
            URL searchUrl = params[0];
            String githubSearchResults = null;
            try {
                githubSearchResults = NetworkUtils.getResponseFromHttpUrl(searchUrl);
            } catch (IOException e) {
                e.printStackTrace();
            }
            return githubSearchResults;
        }

        @Override
        protected void onPostExecute(String githubSearchResults) {
            // COMPLETED (27) As soon as the loading is complete, hide the loading indicator
            mLoadingIndicator.setVisibility(View.INVISIBLE);
            if (githubSearchResults != null && !githubSearchResults.equals("")) {
                // COMPLETED (17) Call showJsonDataView if we have valid, non-null results
                showJsonDataView();
                mSearchResultsTextView.setText(githubSearchResults);
            } else {
                // COMPLETED (16) Call showErrorMessage if the result is null in onPostExecute
                showErrorMessage();
            }
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
