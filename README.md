[![npm][npm]][npm-url]
[![deps][deps]][deps-url]

# React Native Voice

A speech-to-text library for [React Native](https://facebook.github.io/react-native/).

```sh
npm i react-native-voice --save
```

- [React Native Voice](#react-native-voice)
  - [Linking](#linking)
    - [Manually Link Android](#manually-link-android)
    - [Manually Link iOS](#manually-link-ios)
  - [Usage](#usage)
    - [Example](#example)
  - [API](#api)
  - [Speech recognition options](#speech-recognition-options)
  - [Events](#events)
    - [Handling errors](#handling-errors)
    - [Android](#android)
    - [iOS](#ios)

## Linking

Manually or automatically link the NativeModule

```sh
react-native link react-native-voice
```

### Manually Link Android

- In `android/setting.gradle`

```gradle
...
include ':react-native-voice', ':app'
project(':react-native-voice').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-voice/android')
```

- In `android/app/build.gradle`

```gradle
...
dependencies {
    ...
    compile project(':react-native-voice')
}
```

- In `MainApplication.java`

```java
import android.app.Application;
import com.facebook.react.ReactApplication;
import com.facebook.react.ReactPackage;
...
import com.wenkesj.voice.VoicePackage; // <------ Add this!
...

public class MainActivity extends Activity implements ReactApplication {
...
    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new VoicePackage() // <------ Add this!
        );
    }
}
```

### Manually Link iOS

- Drag the Voice.xcodeproj from the react-native-voice/ios folder to the Libraries group on Xcode in your poject. [Manual linking](https://facebook.github.io/react-native/docs/linking-libraries-ios.html)

- Click on your main project file (the one that represents the .xcodeproj) select Build Phases and drag the static library, lib.Voice.a, from the Libraries/Voice.xcodeproj/Products folder to Link Binary With Libraries

## Usage

[(Outdated) Full example for Android and iOS.](https://github.com/wenkesj/react-native-voice/tree/master/VoiceTest)

### Example

```javascript
import React, { Component } from "react";
import { TouchableOpacity, View, Text } from "react-native";
import Voice from "react-native-voice";

class VoiceTest extends Component {
  constructor(props) {
    Voice.onSpeechStart = this.onSpeechStartHandler.bind(this);
    Voice.onSpeechEnd = this.onSpeechEndHandler.bind(this);
    Voice.onSpeechPartialResults = this.onSpeechPartialResultsHandler.bind(this);
    Voice.onSpeechResults = this.onSpeechResultsHandler.bind(this);
    Voice.onSpeechError = this.onSpeechErrorHandler.bind(this);
  }

  componentWillUnmount() {
    // Remove all listeners
    Voice.removeAllListeners();
  }

  onSpeechStartHandler() {
    console.log("Speech started");
    // Update state to notify user that speech recognition has started
  }

   onSpeechPartialResultsHandler(e) {
    // e = { value: string[] }
    // Loop through e.value for speech transcription results
    console.log("Partial results", e);
  }

  onSpeechResultsHandler(e) {
    // e = { value: string[] }
    // Loop through e.value for speech transcription results
    console.log("Speech results", e);
  }

  onSpeechEndHandler(e) {
    // e = { error?: boolean }
    console.log("Speech ended", e);
  }

  onSpeechErrorHandler(e) {
    // e = { code?: string, message?: string }
    switch (e.code) { ... }
  }

  onStartButtonPress = async () => {
    try {
      await Voice.start("en_US");
    } catch (exception) {
      // exception = Error | { code: string, message?: string }
      onSpeechErrorHandler(exception);
    }
  };

  render() {
    return (
      <TouchableOpacity onPress={this.onStartButtonPress}>
        <View>
          <Text>Start</Text>
        </View>
      </TouchableOpacity>
    );
  }
}
```

## API

Static access to the Voice API.

**All methods _now_ return a `new Promise` for `async/await` compatibility.**

| Method Name                                                         | Description                                                                                           | Platform     |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------ |
| Voice.isAvailable()                                                 | Checks whether a speech recognition service is available on the system.                               | Android, iOS |
| Voice.isReady()                                                     | Checks whether speech recognition is ready to be used.                                                | iOS          |
| Voice.start(locale: string, options: {})                            | Starts speech recognition. See [Speech recognition options](#speech-recognition-options) for options. | Android, iOS |
| Voice.stop()                                                        | Stops listening for speech. Returns null if no error occurs.                                          | Android, iOS |
| Voice.cancel()                                                      | Cancels the speech recognition. Returns null if no error occurs.                                      | Android, iOS |
| Voice.destroy()                                                     | Destroys the current SpeechRecognizer instance. Returns null if no error occurs.                      | Android, iOS |
| Voice.removeAllListeners()                                          | Cleans/nullifies overridden `Voice` static methods.                                                   | Android, iOS |
| Voice.isRecognizing()                                               | Return if the SpeechRecognizer is recognizing.                                                        | Android, iOS |
| Voice.setCategory(category: string)                                 | Sets the iOS audio category.                                                                          | iOS          |
| Voice.setPermissionRationaleAndroid(title: string, message: string) | Sets the permission rationale when requesting microphone permissions                                  | Android      |
| Voice.requestPermissionsAndroid()                                   | Requests permissions to use the microphone. Note: already checked in `Voice.start()`                  | Android      |

## Speech recognition options

Usage: `Voice.start(locale: string, options: {})`

| Option Key                                                        | Description                                                                                                                                                                                               | Default                      | Platform |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- | -------- |
| `EXTRA_LANGUAGE_MODEL`: string                                    | Informs the recognizer which speech model to prefer when performing. See [link](https://developer.android.com/reference/android/speech/RecognizerIntent.html#EXTRA_LANGUAGE_MODEL) for additional options | `'LANGUAGE_MODEL_FREE_FORM'` | Android  |
| `EXTRA_MAX_RESULTS`: number                                       | Optional limit on the maximum number of results to return. If omitted the recognizer will choose how many results to return                                                                               | `5`                          | Android  |
| `EXTRA_PARTIAL_RESULTS`: boolean                                  | Optional boolean to indicate whether partial results should be returned by the recognizer as the user speaks                                                                                              | `true`                       | Android  |
| `EXTRA_SPEECH_INPUT_COMPLETE_`<br>`SILENCE_LENGTH_MILLIS`: number | The amount of time that it should take after we stop hearing speech to consider the input complete.                                                                                                       | not set                      | Android  |
| `EXTRA_SPEECH_INPUT_MINIMUM_`<br>`LENGTH_MILLIS`: number | The minimum length of an utterance                                                           | not set                      | Android  |
| `continous`: boolean                                              | Enables continuous speech recognition                                                                                                                                                                     | `false`                      | iOS      |

## Events

Callbacks that are invoked when a native event emitted.

| Event Name                          | Description                                            | Event                                | Platform     |
| ----------------------------------- | ------------------------------------------------------ | ------------------------------------ | ------------ |
| Voice.onSpeechStart()               | Invoked when `.start()` is called without error.       | `null`                               | Android, iOS |
| Voice.onSpeechRecognized(event)     | Invoked when speech is recognized.                     | `{ isFinal: boolean }`               | Android, iOS |
| Voice.onSpeechEnd(event)            | Invoked when SpeechRecognizer stops recognition.       | `{ error?: boolean }`                | Android, iOS |
| Voice.onSpeechError(event)          | Invoked when an error occurs.                          | `{ code: string, message?: string }` | Android, iOS |
| Voice.onSpeechResults(event)        | Invoked when SpeechRecognizer is finished recognizing. | `{ value: Array<string> }`           | Android, iOS |
| Voice.onSpeechPartialResults(event) | Invoked when any results are computed.                 | `{ value: Array<string> }`           | Android, iOS |
| Voice.onSpeechVolumeChanged(event)  | Invoked when pitch that is recognized changed.         | `{ value: pitch in dB }`             | Android      |

### Handling errors

This applies to `Voice.onSpeechError(e)` and when `await Voice.start()` throws an exception.

```javascript
  onSpeechErrorHandler(e) {
    // e: { code: string, message?: string }
    // switch (e.code) { ... }
  }

  ...
  try {
    await Voice.start();
  } catch (e) {
    // Note: on Android this will *likely* return an Error object.
    // e: Error | { code: string, message?: string }
    // switch (e.code) { ... }
  }
```

| Code               | Description                                                     | Platform     |
| ------------------ | --------------------------------------------------------------- | ------------ |
| `permissions`      | User denied microphone/speech recognition permissions           | Android, iOS |
| `recognizer_busy`  | Speech recognition has already started                          | Android, iOS |
| `not_available`    | Speech recognition is not available on the device               | Android, iOS |
| `audio`            | Audio engine / Audio session error                              | Android, iOS |
| `network`          | Network error                                                   | Android      |
| `network_timeout`  | Network timeout error                                           | Android      |
| `speech_timeout`   | Speech recognition timeout                                      | Android      |
| `no_match`         | No recognition matches                                          | Android      |
| `server`           | Server error                                                    | Android      |
| `restricted`       | Speech recognition is restricted                                | iOS          |
| `not_authorized`   | Speech recognition is not authorized                            | iOS          |
| `not_ready`        | Speech recognition is not ready to start                        | iOS          |
| `recognition_init` | Speech recognition initialization failed                        | iOS          |
| `start_recording`  | `[inputNode installTapOnBus:0...]` call failed                  | iOS          |
| `input`            | Audio engine has no input node                                  | iOS          |
| `recognition_fail` | General failure while using recognition. Has a `"message"` prop | iOS          |

<h2 align="center">Permissions</h2>

<p align="center">Arguably the most important part.</p>

### Android

While the included `VoiceTest` app works without explicit permissions checks and requests, it may be necessary to add a permission request for `RECORD_AUDIO` for some configurations.
Since Android M (6.0), [user need to grant permission at runtime (and not during app installation)](https://developer.android.com/training/permissions/requesting.html).
By default, calling the `startSpeech` method will invoke `RECORD AUDIO` permission popup to the user. This can be disabled by passing `REQUEST_PERMISSIONS_AUTO: true` in the options argument.

### iOS

Need to include permissions for `NSMicrophoneUsageDescription` and `NSSpeechRecognitionUsageDescription` inside Info.plist for iOS. See the included `VoiceTest` for how to handle these cases.

```xml
<dict>
  ...
  <key>NSMicrophoneUsageDescription</key>
  <string>Description of why you require the use of the microphone</string>
  <key>NSSpeechRecognitionUsageDescription</key>
  <string>Description of why you require the use of the speech recognition</string>
  ...
</dict>
```

Please see the documentation provided by ReactNative for this: [PermissionsAndroid](http://facebook.github.io/react-native/docs/permissionsandroid.html)

[npm]: https://img.shields.io/npm/v/react-native-voice.svg
[npm-url]: https://npmjs.com/package/react-native-voice
[deps]: https://david-dm.org/wenkesj/react-native-voice.svg
[deps-url]: https://david-dm.org/wenkesj/react-native-voice.svg

<h2 align="center">Contibutors</h2>

* @asafron
* @BrendanFDMoore
* @brudny
* @chitezh
* @ifsnow
* @jamsch
* @misino
* @Noitidart
* @ohtangza & @hayanmind
* @rudiedev6
* @tdonia
* @wenkesj
