---
layout: post
---
## Custom Launcher App
We can create an app to replace the default Google launcher. We can add all sorts of cool things like search apps, sort the apps according to priority etc.  
To demonstrate this I have created a [sample](https://github.com/shubhamdhabhai/Launcher-Demo) which acts as a launcher and shows the list of apps. You can also search an app by its name.

![Alt text](/custom_launcher_for_home/launcher_demo_1.png)       ![Alt text](/custom_launcher_for_home/launcher_demo_2.png)

### Show me the code...
This is a very simple app with just one activity.  
We need to add **HOME** category in the intent filter of our main activity so that whenever home button is pressed we get options between our app and default google launcher. We can set out app as default launcher.

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.demo.shubhamdhabhai.launcherdemo">

    <application
      ...
        <activity android:name=".MainActivity"
            android:theme="@style/Fullscreen">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.HOME" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

The layout file looks like this  
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    tools:context="com.demo.shubhamdhabhai.launcherdemo.MainActivity"
    android:id="@+id/ll_main_layout"
    android:orientation="vertical">

    <android.support.v7.widget.SearchView
        android:id="@+id/sv_app_list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <android.support.v7.widget.RecyclerView
        android:id="@+id/rv_app_list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layoutManager="android.support.v7.widget.LinearLayoutManager"/>
</LinearLayout>
```
I am using a search view for searching apps and a recycler view to show list of apps.  
**MainActivity** is where all the magic happens. It contains some basic things like adapter with filter and search view implementation.

The user should see the same wallpaper as he sees in the default Google launcher for a seamless experience. For this we will take help of WallpaperManager.

```
final WallpaperManager wallpaperManager = WallpaperManager.getInstance(this);
final Drawable wallpaperDrawable = wallpaperManager.getDrawable();

LinearLayout mainLayout = findViewById(R.id.ll_main_layout);
mainLayout.setBackground(wallpaperDrawable);
```  
We get the wallpaper as drawable and set it as a background of our layout.  



To get the list of app we need **PackageManager**
```
 PackageManager manager = getPackageManager();
 List<App> apps = new ArrayList<>();

 Intent mainIntent = new Intent(Intent.ACTION_MAIN, null);
 mainIntent.addCategory(Intent.CATEGORY_LAUNCHER);

 List<ResolveInfo> availableActivities = manager.queryIntentActivities(mainIntent, 0);
```

We want all the activities which have **CATEGORY_LAUNCHER** added in Intent filter because we just want to get the apps which have a **LAUNCHER**. If we don't use this we will get all the activities, even the apps used internally by android os.

The variable **availableActivities** contains information about all the launcher activities. We can extract the information that we want like the icon, name and package name.

```

for (ResolveInfo resolveInfo : availableActivities) {
    App app = new App();
    app.setAppName(resolveInfo.loadLabel(manager).toString());
    app.setAppPackageName(resolveInfo.activityInfo.packageName);
    app.setIcon(resolveInfo.activityInfo.loadIcon(manager));
    apps.add(app);
}
```
Using these informations we can show the list of apps. We do not need package name for showing the apps but we need it to open the app when the user clicks on it.  
This can be done very easily. We just need to get the launch intent for a package name and then we can start an activity using that intent.
```
@Override
   public void onAppCLicked(App app) {
       Intent launchIntentForPackage = getPackageManager().getLaunchIntentForPackage(app.getAppPackageName());
       startActivity(launchIntentForPackage);
   }
```

That's it, our own launcher app is ready. We can do some more cool stuffs like opening a deeplink inside the app for example opening a particular item in Flipkart or opening feed of a user in Instagram.

May the force be with you!!
