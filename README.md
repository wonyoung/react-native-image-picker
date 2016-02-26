# react-native-image-picker
A React Native module that allows you to use native UI to select a photo/video from the device library or directly from the camera, like so:

### iOS

<img title="iOS" src="https://github.com/marcshilling/react-native-image-picker/blob/master/images/ios-image.png" width="50%">

### Android

<img title="Android" src="https://github.com/marcshilling/react-native-image-picker/blob/master/images/android-image.png" width="50%">

## Install

### iOS
1. `npm install react-native-image-picker@latest --save`
2. In the XCode's "Project navigator", right click on your project's Libraries folder ➜ `Add Files to <...>`
3. Go to `node_modules` ➜ `react-native-image-picker` ➜ `ios` ➜ select `RNImagePicker.xcodeproj`
4. Add `RNImagePicker.a` to `Build Phases -> Link Binary With Libraries`
5. Compile and have fun

### Android
1. `npm install react-native-image-picker@latest --save`

```gradle
// file: android/settings.gradle
...

include ':react-native-image-picker'
project(':react-native-image-picker').projectDir = new File(settingsDir, '../node_modules/react-native-image-picker/android')
```
```gradle
// file: android/app/build.gradle
...

dependencies {
    ...
    compile project(':react-native-image-picker')
}
```
```xml
<!-- file: android/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.myApp">

    <uses-permission android:name="android.permission.INTERNET" />

    <!-- add following permissions -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-feature android:name="android.hardware.camera" android:required="true"/>
    <uses-feature android:name="android.hardware.camera.autofocus" />
    <!-- -->
    ...
```
```java
// file: MainActivity.java
...
import android.content.Intent; // import
import com.imagepicker.ImagePickerPackage; // import

public class MainActivity extends ReactActivity {

    private ImagePickerPackage mImagePicker; // <--- ADD THIS

   /**
   * A list of packages used by the app. If the app uses additional views
   * or modules besides the default ones, add more packages here.
   */
    @Override
    protected List<ReactPackage> getPackages() {
        mImagePicker = new ImagePickerPackage(this); // <--- AND THIS

        return Arrays.<ReactPackage>asList(
            new MainReactPackage(),
            mImagePicker // <--- AND THIS
        );
    }

    ...

    // AND ADD THIS WHOLE METHOD
    @Override
    public void onActivityResult(final int requestCode, final int resultCode, final Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        mImagePicker.handleActivityResult(requestCode, resultCode, data);
    }
...

```
## Usage
1. In your React Native javascript code, bring in the native module:

  ```javascript
var ImagePickerManager = require('NativeModules').ImagePickerManager;
  ```
2. Use it like so:

  When you want to display the picker:
  ```javascript
  var options = {
    title: 'Select Avatar', // specify null or empty string to remove the title
    cancelButtonTitle: 'Cancel',
    takePhotoButtonTitle: 'Take Photo...', // specify null or empty string to remove this button
    chooseFromLibraryButtonTitle: 'Choose from Library...', // specify null or empty string to remove this button
    customButtons: {
      'Choose Photo from Facebook': 'fb', // [Button Text] : [String returned upon selection]
    },
    cameraType: 'back', // 'front' or 'back'
    mediaType: 'photo', // 'photo' or 'video'
    videoQuality: 'high', // 'low', 'medium', or 'high'
    maxWidth: 100, // photos only
    maxHeight: 100, // photos only
    aspectX: 2, // aspectX:aspectY, the cropping image's ratio of width to height
    aspectY: 1, // aspectX:aspectY, the cropping image's ratio of width to height
    quality: 0.2, // photos only
    angle: 0, // photos only
    allowsEditing: false, // Built in functionality to resize/reposition the image
    noData: false, // photos only - disables the base64 `data` field from being generated (greatly improves performance on large photos)
    multiple: false, // photos only - select multiple photos - android only
    storageOptions: { // if this key is provided, the image will get saved in the documents/pictures directory (rather than a temporary directory)
      skipBackup: true, // image will NOT be backed up to icloud
      path: 'images' // will save image at /Documents/images rather than the root
    }
  };

  /**
   * The first arg will be the options object for customization, the second is
   * your callback which sends bool: didCancel, object: response.
   *
   * response.didCancel will inform you if the user cancelled the process
   * response.error will contain an error message, if there is one
   * response.data is the base64 encoded image data (photos only)
   * response.uri is the uri to the local file asset on the device (photo or video)
   * response.isVertical will be true if the image is vertically oriented
   * response.width & response.height give you the image dimensions
   */

  ImagePickerManager.showImagePicker(options, (response) => {
    console.log('Response = ', response);

    if (response.didCancel) {
      console.log('User cancelled image picker');
    }
    else if (response.error) {
      console.log('ImagePickerManager Error: ', response.error);
    }
    else if (response.customButton) {
      console.log('User tapped custom button: ', response.customButton);
    }
    else {
      // You can display the image using either data:
      const source = {uri: 'data:image/jpeg;base64,' + response.data, isStatic: true};

      // uri (on iOS)
      const source = {uri: response.uri.replace('file://', ''), isStatic: true};
      // uri (on android)
      const source = {uri: response.uri, isStatic: true};

      this.setState({
        avatarSource: source
      });
    }
  });
  ```
  Then later, if you want to display this image in your render() method:
  ```javascript
  <Image source={this.state.avatarSource} style={styles.uploadAvatar} />
  ```

### Directly Launching the Camera or Image Library

  To Launch the Camera or Image Library directly (skipping the alert dialog) you can
  do the following:
  ```javascript
  // Launch Camera:
  ImagePickerManager.launchCamera(options, (response)  => {
    // Same code as in above section!
  });

  // Open Image Library:
  ImagePickerManager.launchImageLibrary(options, (response)  => {
    // Same code as in above section!
  });
  ```

### Options

option | iOS  | Android
------ | ---- | -------
title | OK | OK
cancelButtonTitle | OK | OK
takePhotoButtonTitle | OK | OK
chooseFromLibraryButtonTitle | OK | OK
customButtons | OK | -
cameraType | OK | -
mediaType | OK | -
videoQuality | OK | -
angle | - | OK
aspectX | - | OK
aspectY | - | OK
maxWidth | OK | OK
maxHeight | OK | OK
quality | OK | OK
allowsEditing | OK | OK
noData | OK | OK
storageOptions | OK | if this key is provided, the image will get saved in the pictures directory
