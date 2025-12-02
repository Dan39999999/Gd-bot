SuperGD-Bot/
│
├── .gitignore
├── build.gradle
├── settings.gradle
├── app/
│   ├── build.gradle
│   ├── src/
│   │   ├── main/
│   │   │   ├── AndroidManifest.xml
│   │   │   ├── java/com/supergd/bot/
│   │   │   │   ├── MainActivity.java
│   │   │   │   └── BotService.java
│   │   │   ├── res/
│   │   │   │   ├── layout/activity_main.xml
│   │   │   │   └── values/strings.xml
│   │   │   └── xml/accessibility_service_config.xmlrootProject.name = "SuperGD-Bot"
include(":app")# Gd-bot // Top-level build file
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath "com.android.tools.build:gradle:8.1.1"
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}apply plugin: 'com.android.application'

android {
    compileSdk 34

    defaultConfig {
        applicationId "com.supergd.bot"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
}<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.supergd.bot">

    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.BIND_ACCESSIBILITY_SERVICE"/>
    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:label="SuperGD Bot"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>

        <service
            android:name=".BotService"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
            android:label="GD Bot Service"
            android:exported="true">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService"/>
            </intent-filter>

            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_service_config"/>
        </service>
    </application>
</manifest><accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeWindowContentChanged|typeViewClicked"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:notificationTimeout="100"
    android:canRetrieveWindowContent="true"
    android:settingsActivity="com.supergd.bot.MainActivity"/>package com.supergd.bot;

import android.content.Intent;
import android.os.Bundle;
import android.provider.Settings;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button btnEnable = findViewById(R.id.btnEnable);
        btnEnable.setOnClickListener(v -> {
            Intent intent = new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
            startActivity(intent);
        });

        Button btnStartBot = findViewById(R.id.btnStartBot);
        btnStartBot.setOnClickListener(v -> {
            Intent serviceIntent = new Intent(this, BotService.class);
            startService(serviceIntent);
        });
    }
}package com.supergd.bot;

import android.accessibilityservice.AccessibilityService;
import android.accessibilityservice.GestureDescription;
import android.graphics.Path;
import android.os.Handler;
import android.view.accessibility.AccessibilityEvent;

public class BotService extends AccessibilityService {

    private Handler handler = new Handler();
    private boolean running = false;

    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {}

    @Override
    public void onInterrupt() {}

    @Override
    protected void onServiceConnected() {
        super.onServiceConnected();
        startBot();
    }

    private void startBot() {
        running = true;
        handler.post(botRunnable);
    }

    private Runnable botRunnable = new Runnable() {
        @Override
        public void run() {
            if (!running) return;

            // Tap the screen at X=540, Y=1600
            Path path = new Path();
            path.moveTo(540, 1600);
            GestureDescription.StrokeDescription click =
                    new GestureDescription.StrokeDescription(path, 0, 50);
            dispatchGesture(new GestureDescription.Builder().addStroke(click).build(), null, null);

            // Repeat every 100ms
            handler.postDelayed(this, 100);
        }
    };

    public void stopBot() {
        running = false;
        handler.removeCallbacks(botRunnable);
    }
}<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="24dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/btnEnable"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Enable Accessibility"/>

    <Button
        android:id="@+id/btnStartBot"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start Bot"
        android:layout_marginTop="20dp"/>
</LinearLayout><resources>
    <string name="app_name">SuperGD Bot</string>
</resources>