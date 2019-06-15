---
layout: post
title: Exploring Chirp SDK in Flutter - Data over audio
description: This blog post explains how to integrate chirp sdk in flutter - data over audio 
tags: [iot, chirp, flutter, dart]
featured_image_thumbnail: assets/images/posts/2019/chirp-flutter.jpg
featured_image: assets/images/posts/2019/chirp-flutter.jpg
---

Cover image by: <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@kaip?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Kai Pilger"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Kai Pilger</span></a>

## Overview

I recently started exploring ChirpSDK - SDK for transmitting data through audio when I explore one of the non-internet or less costly hardware options for IoT solutions to transmit the data. 

Chirp enables your apps to send and receive information using sound. A "chirp" encodes an array of bytes as an audio signal, which can be transmitted by any device with a speaker and received by any device with a microphone and Chirp SDK. It is designed to be robust over distances of several meters, and in noisy, everyday environments.

As the transmission takes place entirely via audio signals, no internet connection or prior pairing is required, and any device within hearing range can receive the data.

To keep it simple and fun, this demo project is developed in my [twitch streams](https://twitch/tv/ksivamuthu) with this solution.

## What does this demo app do?

The idea of the project is we are going to send the data from mobile to another device through audio.

I've built this Flutter application that has a list of dev brand main colors. For e.g

* Angular Red
* React Blue
* Vue Green
* Chirp Yellow

When you tap one of the list items, the app is sending the HEX value of the color through the audio. Then the python application running in my machine receives the audio through a microphone, decodes the HEX value. Then it's calling Philips HUE Bridge API to set the color of the light.

If you tap the "Angular Red" list item, you can hear the audio and the python program receives and sets the light bulb color to the red.

* Repository of the project:
  [Chirp Flutter Dev Colors](https://github.com/ksivamuthu/chirp-flutter-dev-colors)

* Screenshot of the app:

<img style="vertical-align:top" src="https://raw.githubusercontent.com/ksivamuthu/chirp-flutter-dev-colors/master/docs/screenshot.jpg" width="200"><img>

* Me pointing how the color of the light changed:

<img style="vertical-align:middle" src="https://img.youtube.com/vi/XvyMYcOwpY8/maxresdefault.jpg" width="500"><img>

## Prerequisites

* [Flutter](https://flutter.dev/docs)
* [Chirp SDK](https://developers.chirp.io)
* [VSCode](https://code.visualstudio.com) & [VSCode Flutter Extension](https://marketplace.visualstudio.com/items?itemName=Dart-Code.flutter)
* [Python](https://www.python.org/downloads/)
* [Philips Hue Light Bulb & Hue Smart Bridge](https://www2.meethue.com/en-us) **
* Mobile device - iOS or Android

 ** *You can use other bulbs or models. You might need to update the code to call the API of the light you are using.*

## Getting Started

### Configure your Chirp Applications.

Signup/Login into [Chirp SDK Developer Portal](https://developers.chirp.io). To configure your chirp app, navigate to the [applications](https://developers.chirp.io/applications) page and select a protocol. Take a note of the key, secret and config string. We need this to configure chirp SDK later in the app.


### Configure your Wireless Light Bulb

I've used Philips Hue Light Bulb and Philips Smart Bridge here.

<p float="left" align="center">
<img src="https://thepracticaldev.s3.amazonaws.com/i/dk5ugts6c3e8hjl56os3.jpg" width="300" height="300" ></img>
<img src="https://thepracticaldev.s3.amazonaws.com/i/txr43txo6jo4wwxcsukl.jpg" width="300" height="300" ></img>
</p>

Please follow the instruction in [meethue developers page](https://developers.meethue.com/develop/get-started-2/) to set up your bridge and connect your light bulb in your local WiFi Network.


### Flutter Project


#### Create a new Flutter app

* Create a flutter project.

```bash
flutter create chirp_fluter_light
```

* Navigate the chirp_flutter_light directory and run "flutter run" or use VSCode to run the flutter app. 
 
```bash
cd chirp_flutter_light
flutter run ios
```

#### Add ChirpSDK in pubspec.yaml

* To get started with Chirp in your own Flutter app, just add the plugin to your pubspec.yaml.

```yaml
dependencies:
  flutter:
    sdk: flutter
  chirpsdk: ^0.1.0
```

#### Configure ChirpSDK

* To configure the SDK you will need to copy your app_key, app_secret, and app_config from the applications page and set this in main.dart file

```dart
String _appKey = '<Chirp_App_Key>';
String _appSecret = '<Chirp_App_Secret>';
String _appConfig = '<Chirp_App_Config>';
```

#### Implement ChirpSDK

* Create some asynchronous functions to make calls to the SDK. ChirpSDK for Flutter uses platform channels to pass messages to the native iOS/Android SDKs. 

```dart
  Future<void> _initChirp() async {
    await ChirpSDK.init(_appKey, _appSecret);
  }

  Future<void> _configureChirp() async {
    await ChirpSDK.setConfig(_appConfig);
  }

  Future<void> _startAudioProcessing() async {
    await ChirpSDK.start();
  }

  Future<void> _stopAudioProcessing() async {
    await ChirpSDK.stop();
  }

```

* To send the data through audio, implement the below logic in ListViewItem Tap. 

```dart
onTap: () async {
        String identifier = '#${info.lightColor.value.toRadixString(16)}';
        var payload = new Uint8List.fromList(identifier.codeUnits);
        await ChirpSDK.send(payload);
  }
```

#### Permissions:

ChirpSDK will require permission to use the device's microphone to receive data. Use the `simple_permissions` plugin to list and request permissions

```yaml
dependencies:
  flutter:
    sdk: flutter
  chirpsdk: ^0.1.0
  simple_permissions: ^0.1.9
```

Then create a function to request for microphone permissions within the State. This should be called before initializing the SDK.

```dart
Future<void> _requestPermissions() async {
  bool permission = await SimplePermissions.checkPermission(Permission.RecordAudio);
  if (!permission) {
    await SimplePermissions.requestPermission(Permission.RecordAudio);
  }
}
```

Please feel free to refer the Github repository on how the UI is built in this application.

### Python program - listen to​ audio

* To configure the python SDK you will need to copy your app_key, app_secret, and app_config from the applications page and paste into your ~/.chirprc file.

```
[default]
app_key = xXxXXxxxXXXxxXXXxXxXXxXxx
app_secret = xxXxXXXxXxxXXXxXXXXxXxXxxxXxxxXXxXxXxxxXXxXxXXxXxX
app_config = XxXXXXxXxXxxxXxxxXXxXxXxxxXXxXxXXxXxXxxXxXXXxXxxXXXxXXXXxXxXxxxXxxxXXxXxXxxxXXxXxXXxXxX
```

* Install the Chirp Python SDK and its dependencies. Please refer this [documentation](https://developers.chirp.io/docs/getting-started/python)

```bash
brew install portaudio libsndfile //macOS
pip3 install sounddevice.whl //Windows

pip3 install chirpsdk
```

* Connect the SDK to start audio processing. When the data is received, the data is received in callbacks. You can call the Hue Bridge API to set the color here. I've used [qhue](https://github.com/quentinsf/qhue) to call the Philips Hue Bridge API.

```python
class Callbacks(CallbackSet):
    def on_received(self, payload, channel):
        if payload is not None:
            hex = payload.decode('utf-8')
            print('Received: ' + hex)
            // Call the Hue Bridge API here to set the color.
        else:
            print('Decode failed')


chirp = ChirpConnect()
chirp.start(send=True, receive=True)
chirp.set_callbacks(Callbacks())
```

* Run the python program to listen the audio
```bash
python3 index.py
```

## Demo

You can see the demo in action in below video.

<iframe width="560" height="315" src="https://www.youtube.com/embed/XvyMYcOwpY8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Action Items for you

* Learn Flutter & Explore Chirp SDK

* Be Kind and Spread your awesomeness

* And comment below or create a [github issue](https://github.com/ksivamuthu/chirp-flutter-dev-colors/issues) to add your favorite dev brand and its primary color (For e.g Android, Go, etc).


Follow & join me when I'm streaming on [Twitch](https://twitch.tv/ksivamuthu) to have fun and learn together. Usually, I live code mobile and IoT solutions in my live streams.