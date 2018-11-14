
Log.x (String TAG, String message);

TAG can be anyname you want, but a common strategy is to use the CLASS NAME as the TAG.

Log.e(String TAG, String message); // error
Log.w(String TAG, String message); // warning
Log.i(String TAG, String message); // info
Log.d(String TAG, String message); // debug
Log.v(String TAG, String message); // verbose

Log.wtf(String TAG, String message); // What a Terrible Failure (WTF :-)
