---
title: Jokes app
weight: 2
---

# Jokes app

## Introduction

This will be a fun one (hopefully).
Today we are going to kill the boredom by writing an app that tells jokes.

First you need to create a new Flutter project.

```sh
flutter create make_me_laugh
```

You can use a different project name if you want.

## Data

You could write a long list of jokes yourself for the app.
However, that sounds like a lot of work for something supposed to be fun.

Instead, let's use jokes other people have written.
Luckily there is a nice free API we can use to fetch random jokes.

Head over to [jokeapi](https://jokeapi.dev/#try-it).

I suggest that you select _Programming_ as the category.
It is also recommended that you select everything under **Select flags to
blacklist** as some of the jokes will be really offensive otherwise.

![Recommended jokeapi settings](../images/jokeapi.png)

The URL should look something like this:

```
https://v2.jokeapi.dev/joke/Programming?blacklistFlags=nsfw,religious,political,racist,sexist,explicit
```

Feel free to hit the _Send Request_ button a couple of times to see how the API
responds.

Remember the URL because you will need it later.

There are two types of jokes `single` and `twopart`.
You can tell which type you get from the `type` field in the response.

For the next step you need to merge a response of both types.
So, you get something like:

```json
{
  "error": false,
  "category": "Programming",
  "type": "twopart",
  "setup": "Why do programmers confuse Halloween and Christmas?",
  "delivery": "Because Oct 31 = Dec 25",
  "joke": "Java is like Alzheimer's, it starts off slow, but eventually, your memory is gone.",
  "flags": {
    "nsfw": false,
    "religious": false,
    "political": false,
    "racist": false,
    "sexist": false,
    "explicit": false
  },
  "id": 11,
  "safe": true,
  "lang": "en"
}
```

It should have both `setup`, `delivery` and `joke` fields.

Copy the merged JSON.
Head over to [JSON to Dart](https://jsontodart.zariman.dev/) and paste
it in.
In the "Class Name" input, type `JokeDto` and hit the _Generate_ button.

Copy all the generated Dart code and paste it into a new file named
`joke_dto.dart` inside your project.

It generated a DTO class for you, from the json, with convince methods to help
convert to and from JSON.

Now that you have a class for the data, you need some code to fetch it from the
API.
For that you need a package.

```sh
flutter pub add http
```

_There is a HTTP client build into Dart but the
[http package](https://pub.dev/packages/http) is a lot nicer to work with._

You also need to add the [provider package](https://pub.dev/packages/provider).

```sh
flutter pub add provider
```

In `android/app/src/main/AndroidManifest.xml`, you need add a couple of lines:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- These two lines -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    ...
```

Adding permissions to AndroidManifest.xml is required to access the internet.

Add a new file called `data_source.dart` to your project with the following content:

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

import 'joke_dto.dart';

class DataSource {
  Future<JokeDto> getJoke() async {
    // Your URL from goes here...
    const url = "https://v2.jokeapi.dev/joke/Programming?blacklistFlags=nsfw,religious,political,racist,sexist,explicit";
    final response = await http.get(Uri.parse(url));
    final map = json.decode(response.body);
    return JokeDto.fromJson(map);
  }
}
```

## Make a UI

In `main.dart`, wrap `MaterialApp` with a provider for `DataSource`:

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Provider(
      create: (context) => DataSource(),
      child: MaterialApp(
        // ...
      ),
    );
  }
}
```

Now, replace `MyHomePage` with:

```dart
class JokePage extends StatefulWidget {
  const JokePage({super.key});

  @override
  State<JokePage> createState() => _JokePageState();
}

class _JokePageState extends State<JokePage> {
  JokeDto? joke;

  @override
  void initState() {
    super.initState();
    _loadJoke();
  }

  _loadJoke() async {
    setState(() {
      joke = null;
    });
    final newJoke = await context.read<DataSource>().getJoke();
    setState(() {
      joke = newJoke;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("Jokes")),
      body: Column(
        children: [
          if (joke == null) const CircularProgressIndicator(),
          if (joke?.joke != null) Text(joke!.joke!),
          if (joke?.setup != null) Text(joke!.setup!),
          if (joke?.delivery != null) Text(joke!.delivery!),
          TextButton(onPressed: _loadJoke, child: const Text("Show another")),
        ],
      ),
    );
  }
}
```

You should have a working app by now.
Try it out!

It looks really boring, doesn't it?
Spend some time to make it pretty, before you head to the challenges.

## Challenges

The following challenges can be completed independent of each other.

You are not required to complete them all, but you should at least read the
text.
However, completing the challenges will make your app a lot more awesome 😎.

### Challenge 1 - Add some graphics

Without graphics the app looks pretty boring.
So, let's fix it!

You are not required to follow the steps in this section.
You can get creative and add some other graphics instead.
With the [Image
widget](https://api.flutter.dev/flutter/widgets/Image-class.html) you can easily
add images.

Anyway, I thought it would be cool if it looks like there are different cartoon
characters telling jokes.

I've found an avatar library/service called
[DiceBear](https://www.dicebear.com/how-to-use/http-api/), that have a HTTP API.

Find a [avatar style](https://www.dicebear.com/styles/) you like.

You can get a new avatar for each joke with:

```dart
"https://api.dicebear.com/7.x/pixel-art/svg?seed=${joke.id}"
```

Where `joke.id` is the id field from the DTO.

Replace `pixel-art` with your preferred style.

SVG is a nice format since it looks crisp no matter the size.
However Flutter out-of-the-box doesn't easily allow you to draw SVGs.
But that can easily be fixed just by adding another package.

```sh
flutter pub add flutter_svg
```

Now you can add an avatar to the UI with following widget:

```dart
SvgPicture.network("https://api.dicebear.com/7.x/adventurer/svg?seed=${joke?.id}")
```

Mine ended up looking like this:

![My version of the app](images/my_app.jpg)

### Challenge 2 - Settings

Remember, there were a lot of settings you could change on the [jokeapi
website](https://jokeapi.dev/)?

Wouldn't it be cool if your users could change the settings themselves?

For that you need a couple things.

1. an object to hold the settings
2. (optionally) persistent storage of the settings
3. another page to change the settings
4. update `DataSource` to use the settings

#### 1. Object to hold settings

You can solve the first, by simply adding a `Settings` class with fields for the
settings you want the user to be able to change.

You can make the `Settings` class accessible across your application with a
provider (just like DataSource).
I suggest using a
[MultiProvider](https://pub.dev/packages/provider#multiprovider) to cleanly
provide both `DataSource` and your new settings object.

#### 2. Persistence (optional)

You can use the [localstorage](https://pub.dev/packages/localstorage) package to
persist simple data in your app.

One caveat, is that whatever data you try to persist, needs to be JSON
serializable.
Meaning, you can only use types like: int, double, bool, String List & Map.
Also, you will have to cast the value you get back to the appropriate type.

Add `load` and `save` methods to `Settings` and use `LocalStorage` to persist
settings.

**Important:** You need to wait for `LocalStorage` to be ready before you can use it:

```dart
final LocalStorage storage = LocalStorage('settings');
await storage.ready
```

#### 3. Settings page

Create a new `StatefulWidget` for the settings page.

You can navigate to it with:

```dart
Navigator.of(context).push(MaterialPageRoute(builder: (context) => const SettingsPage()));
```

Which can be invoked from `onPressed` on a button.

You can use the [settings_ui](https://pub.dev/packages/settings_ui) package to
easily create a UI with toggles for various settings such as categories.

Remember to call `setState(() {})` when you want the UI to update.

You can use [PopScope](https://api.flutter.dev/flutter/widgets/PopScope-class.html)
to save the settings before the widget is popped from the navigation stack.

```dart
Widget build(BuildContext context) {
  return PopScope(
    canPop: false,
    onPopInvoked: (didPop) async {
      final navigator = Navigator.of(context);
      await settings.save(settings);
      if (navigator.canPop()) navigator.pop();
    },
    child: Scaffold(
      appBar: AppBar(title: const Text("Settings")),
      body: SettingsList(sections: [
        // your settings here
      ]),
    ),
  );
}
```

#### 4. Use settings in DataSource

Update `DataSource` to use the settings:

```dart
class DataSource {
  Future<JokeDto> getJoke(Settings settings) async {
    final categories = settings.categories.isEmpty ? "Any" : settings.categories.map((e) => e.name).join(",");
    final url = "https://v2.jokeapi.dev/joke/$categories";
    final response = await http.get(Uri.parse(url));
    final map = json.decode(response.body);
    return JokeDto.fromJson(map);
  }
}
```

You can get the settings from within your widget using
`context.read<Settings>()`.

### Challenge 3 - Read it out loud

Wouldn't it be cool if your app could read the jokes out loud?

What you need is some text-to-speech (aka speech synthesis) functionality.

The [Cloud Text-To-Speech](https://pub.dev/packages/cloud_text_to_speech)
package allows you to easily use cloud services from major providers to convert
text to sound.

You will also need another package such as
[audioplayers](https://pub.dev/packages/audioplayers) to play the sound.

Third, your app will need secrets to access the speech cloud service.
As you know, one should never commit secrets to source repository.
You can store the secrets in a `.env` file and read them using
[flutter_dotenv](https://pub.dev/packages/flutter_dotenv).

Add the packages:

```sh
flutter pub add cloud_text_to_speech
flutter pub add audioplayers
flutter pub add flutter_dotenv
```

1. Add `.env` to `.gitignore`
2. Go to [Azure Portal](https://portal.azure.com/).
3. Create a new **Speech Services** resource in the region of North Europe.
4. Copy one of the two keys on the resource overview page under **Keys and endpoint** section.

Create a `.env` file with your key at the root of your Flutter project:

```sh
TTS_SUBSCRIPTION_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TTS_REGION=northeurope
```

**IMPORTANT** verify that `.env` file isn't included before you commit and push.

Change main method in `main.dart`:

```dart
void main() async {
  await dotenv.load(fileName: ".env");
  TtsMicrosoft.init(
    subscriptionKey: dotenv.env["TTS_SUBSCRIPTION_KEY"]!,
    region: dotenv.env["TTS_REGION"]!,
    withLogs: true,
  );
  runApp(const MyApp());
}
```

Add the following to one of your widget state objects:

```dart
final player = AudioPlayer();

@override
void dispose() {
  player.stop().then((value) => player.dispose());
  super.dispose();
}
```

You can now have the text converted to sound and play it with the following
code.

```dart
final voicesResponse = await TtsMicrosoft.getVoices();
final voices = voicesResponse.voices;

TtsParamsMicrosoft ttsParams = TtsParamsMicrosoft(
    voice:
        voices.firstWhere((element) => element.locale.code.startsWith("en-")),
    audioFormat: AudioOutputFormatMicrosoft.audio48Khz192kBitrateMonoMp3,
    text: textYouWantToBeSpoken,
);

final ttsResponse = await TtsMicrosoft.convertTts(ttsParams);

player.play(BytesSource(ttsResponse.audio.buffer.asUint8List()));
```

Be mindful not to make excessive amounts of request to the service, as you will
likely hit a limit at some point.
