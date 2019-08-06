# UnityGame-as-an-AAR
Project demonstrating how to implement a Unity game in an android app as an AAR.

**1. Unity**

The below instructions work with Unity 2018.4, the settings, syntax, editor layout etc. might be different in future versions.

**1.1. (optional) Set up a communication channel between Unity and Android**

If the Unity game and Android app have to exchange data implement proper methods for communication: 

**1.1.1. Send data from Android to Unity**

Android can send data or rather call functions in Unity through the UnityPlayer.UnitySendMessage() method. 
The method accepts 3 arguments:
- name of the objects containing the function we want to call
- name of the function we want to call
- (string) parameter we want to pass to the function

```JAVA
// Android (Java):
UnityPlayer.UnitySendMessage("Main Camera", "switchObject", "cube");
```
Details: https://docs.unity3d.com/Manual/PluginsForIOS.html (see "Calling C# back from native code" (works the same in case of andorid and java).

In the above case, the active unity scene contains a Main Camera object with a switchObject() function displayed below. 

```C#
// Unity (C#):
public void switchObject(string displayed_object){
	if (displayed_object == "cube") {
		// do something
	}
}
```
**1.1.2. Send data from Unity to Android**

Unity can call Android functions by creating an AndroidJavaClass object of type UnityPlayer and using it to access the current activity. Once an AndroidJavaObject of the current Activity is obtained any method within the Activity can be called by specifying the name of the method and the arguments that should be passed to it.
```C#
// Unity player (C#):
AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer");

AndroidJavaObject activity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");

activity.Call("gameFinished", gameIsFinished);
```
Details: https://docs.unity3d.com/ScriptReference/AndroidJavaObject.Call.html

The code above calls the gameFinished() method displayed below that is located in the currenlty open Activity.
```JAVA
public void gameFinished(String params){
	// do something
}
```
**1.2. Adjust Unity build and project settings**
The Unity project build settings and player settings have to be adjusted so everything works once exported as na Android project. 

**1.2.1. Build settings**

Navigate to build settings (File > Build settings)
- Swtich the target platform to Android (download the module if you don't have it installed).
- Set Texture Compression to ETC2
- Make sure the Build System is set to Gradle
- Export Project should be ticked

**1.2.2. Player settings**

Navigate to player settings (bottom left corner of Build settings). Under "Other settings":
- Change Package Name (e.g. com.Blizzard.Hearthstone)
- Adjust API level (e.g. Minimum above Android 7.0 if you're using ARCore)
- Set Scripting Backend to IL2CPP 
- Tick ARM64 after IL2CPP has been selected

**1.3. Export the project**

Navigate to Build Settings (File > Build settings) and press **export** at the bottom of the window.
If prompted download the Android NDK and/or locate it for the Unity editor.

**2. Android**

Open Android Studio and open the project that Unity exported.

**2.1. Adjuts project files**
Certain Android project files should be adjusted in order to make the project into a module and to prevent possible errors while importing the module into our main android app:
- In AndroidManifest.xml comment or delete the <intent-filter> within the <activity> tag 
- In build.gradle:
  - locate and change "com.android.application" to "com.android.library"
  - locate and set compileSdkVersion to match the main project
  - locate defaultConfig{...} delete ApplicationId and adjust minSdkVersion and targetSdkVersion to match the main project
  - locate and comment the entire bundle{...}
- Finnaly sync the build.gradle
  
**2.2. Build the project**

Simply build the android project. 
The unity app will appear in build/outputs/aar in the form of a .aar file.

**2.3. Import the .aar file into the main project**

Open the (main) android project in which you want to use you unity game.
- Import the new module:
  - Open the Create New Module menu under File > New > New Module...
  - Select Import JAR/.AAR Package
  - Locate the .aar file (where it was built earlier)
  - Press Finish
- In build.gradle(app) in dependencies{...} add the line: 
  ```
  implementation project(":<modulename>") 
  ```
  where ```<modulename>``` matches the name of the imported module.
- Finnaly some ManifestMerge errors might appear which can be resolved by opening the ManifestMerge page in the lower left corner of the project Manifest file. 

**TODO**

Add refferencing and opening module in code.
  





