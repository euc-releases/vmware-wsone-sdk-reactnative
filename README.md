# ws1-sdk-react-native

## Initial Setup
<medium>Please find the [Prerequisites](https://github.com/vmwareairwatchsdk/vmware-wsone-sdk-reactnative/blob/master/GettingStarted.md) for using the React Native SDK </medium>

## Getting started

`$ npm install ws1-sdk-react-native --save`

### Mostly automatic installation

`$ react-native link ws1-sdk-react-native`

## Additional Setup
### iOS
Add following code in AppDelegate
```objective-c
-(BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
  //Add following code for posting Notification for URL
  NSNotification *info = [[NSNotification alloc]initWithName:@"UIApplicationOpenURLOptionsSourceApplicationKey" object:url userInfo:options];
  [[NSNotificationCenter defaultCenter] postNotification:info];
  
  return YES;
}
```

### Android

1. Add the library files location to the application build configuration
```java
    repositories {
        flatDir {
            dirs "$rootDir/../node_modules/ws1-sdk-react-native/android/libs"
        }
    }
```

2. Modify AndroidManifest.xml for Main Launcher
```java
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:configChanges="keyboard|keyboardHidden|orientation|screenSize|uiMode"
            android:launchMode="singleTask"
            android:windowSoftInputMode="adjustResize">

        </activity>
        <activity
            android:name="com.airwatch.login.ui.activity.SDKSplashActivity" android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" /> 
            </intent-filter>
        </activity>
```
2. Update your Main Activity 
```java
import com.workspaceonesdk.WorkspaceOneSdkActivity;
public class MainActivity extends WorkspaceOneSdkActivity {

  /**
   * Returns the name of the main component registered from JavaScript. This is used to schedule
   * rendering of the component.
   */
  @Override
  protected String getMainComponentName() {
    return "example";
  }
}
```
3. . Update your Android Application subclass as follows 
    -  Declare that the class implements the WorkspaceOneSDKApplication interface.
    -  Move the code from the body of your onCreate method, if any, to an override of the AWSDKApplication onPostCreate method.
    -  Override the AWSDKApplication getMainActivityIntent() method to return an Intent for the application’s main Activity.
    -  Override the following Android Application methods: 
        - attachBaseContext

```java
import com.workspaceonesdk.WorkspaceOneSdkApplication;
public class MainApplication extends WorkspaceOneSdkApplication implements ReactApplication {

    // Application-specific overrides : Comment onCreate() out and move the code to onPostCreate()

    //  @Override
    //  public void onCreate() {
    //    super.onCreate();
    //  }

    // Application-specific overrides : Copy all the code from onCreate() to onPostCreate()
    @Override
    public void onPostCreate() {
        super.onPostCreate();
        SharedPreferences preferences = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());
        preferences.edit().putString("debug_http_host", "localhost:8088").apply();
        SoLoader.init(this, /* native exopackage */ false);
        initializeFlipper(this, getReactNativeHost().getReactInstanceManager());
        
        // Code from the application's original onCreate() would go here
    }
    
    
    public void attachBaseContext(@NotNull Context base) {
        super.attachBaseContext(base);
        attachBaseContext(this);
    }

    
    @NotNull
    @Override
    public Intent getMainActivityIntent() {
        // Replace MainActivity with application's original main activity
        return new Intent(getApplicationContext(), MainActivity.class);
    }
}
```
## Usage

```javascript to initialize the SDK
import WorkspaceOneSdk from 'react-native-workspace-one-sdk';
import { NativeModules} from 'react-native';
const {WorkspaceOneSdk } = NativeModules;

export default class App extends Component {
componentDidMount() {

     // Start SDK.
      WorkspaceOneSdk.startSDK()
      const eventEmitter = new NativeEventEmitter(NativeModules.WorkspaceOneSdk);
      this.eventListner = eventEmitter.addListener('initSuccess',(event) => {
        console.log("SDK Init Success",event);
      });
      this.eventListner = eventEmitter.addListener('initFailure',(event) => {
        console.log("SDK Init Failed",event);
      });
      
  }
}
```

```javascript to access Environment info
import React, { Component } from 'react';
import { NativeModules,StyleSheet, View, Button,Platform, Text} from 'react-native';
const {WorkspaceOneSdk } = NativeModules;
export default class Information extends Component {

    constructor(){
 
        super();
   
        this.state = {
   
            UserName:'User Not Found',
            GroupId: 'GroupId Not Found',
            ServerName: 'Server Name Not Found'
        }
   
    }


    componentDidMount = async() => {

          try {
           const  useName = await WorkspaceOneSdk.userName();
            this.setState({
                      UserName: 'User Name : ' + useName
                  });
          } catch (error) {
            console.error(error);
          }

          try {
            var groupId = await WorkspaceOneSdk.groupId();
            this.setState({
                      GroupId:  'Group Id : ' + groupId 
                  });
          } catch (error) {
            console.error(error);
          }

          try {
            const serverName = await WorkspaceOneSdk.serverName();
            this.setState({
              ServerName: 'Server Name : ' + serverName 
                  });
          } catch (error) {
            console.error(error);
          }

    }

}

