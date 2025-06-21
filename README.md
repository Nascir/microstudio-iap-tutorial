# How to Add Google Play In-App Purchases to Your microStudio Game

## Introduction

This tutorial demonstrates how to integrate Google Play In-App Purchases (IAP) into your microStudio game and generate an AAB file for release on the Google Play Store.

> [!NOTE]
> **Android App Bundle (AAB)** is the publishing format for Android apps on Google Play. It includes all your app's compiled code and resources but defers APK generation and signing to Google Play. This allows Google to deliver optimized APKs tailored to each user's device configuration.

## Prerequisites

Here's what you need before we dive in:

- **OS environment:** A macOS environment was used for this tutorial. While untested, Windows and Linux environments should also work.
- **microStudio:** Learn more about microStudio [here](https://microstudio.dev/).
- **microScript 2.0:** Good working knowledge of the programming language used in microStudio.
- **JavaScript:** Basic working knowledge is helpful, but not required.
- **Node.js & npm:** Essential for managing Capacitor and Cordova plugins. Learn more about Node.js & npm [here](https://nodejs.org/).
- **Capacitor:** Version 6.x.x is required, as later versions (7.x.x) have known compatibility issues. Learn more about Capacitor [here](https://capacitorjs.com/).
- **Cordova Purchase Plugin:** Version 13 by Jean-Christophe Hoelt. Learn more about the plugin [here](https://github.com/j3k0/cordova-plugin-purchase).
- **Android Studio:** Learn more about Android Studio [here](https://developer.android.com/studio).
- **Google Play Console:** For setting up your in-app purchase products and publishing your game on Google Play. Learn more about Google Play Console [here](https://play.google.com/console).

We will cover each of these prerequisites in detail throughout the tutorial.

## Developing a microStudio App Featuring IAP Calls

Let's create a simple microStudio app with a large button. Clicking the button will initiate the in-app purchase.

### Project structure

We will divide the project into five parts:
1. main code,
2. Button class,
3. PurchaseButton class,
4. global IAP function,
5. ClearButton class.

### Main code

Below is the main microScript code:

```
init = function()
  global.isPremium = false
  
  // purchaseButton
  purchaseButton = new PurchaseButton(
    0,    // x
    0,    // y
    160,  // width
    50,   // height
    4,    // radius
    
    color = object
      untouched = "rgb(147,16,16)"
      touched = "rgb(105,12,12)"
      callback = "rgb(23,105,23)"
    end,
    
    text = object
      content = "Upgrade to Premium"
      size = 16
      color = "rgb(255,255,255)"
    end,
    
    function() purchase() end // callback
    )

    // clearButton
    local height = 20
    local y = screen.height / 2 - height / 2
    clearButton = new ClearButton(
      0,            // x
      y,            // y
      screen.width, // width
      height,       // height
      0,            // radius
      
      color = object
      untouched ="rgb(48,48,48)"
      touched ="rgb(62,62,62)"
      end,
      
      text = object
      content = "Clear Premium status (dev)"
      size = 10
      color ="rgb(193,193,193)"
      end,
      
      function() clearPremium() end // callback
    )
    
  if storage.get("is_premium") then
    purchaseButton.setPremium()
  end
    
end

update = function()
  purchaseButton.update()
  clearButton.update()
end

draw = function()
  screen.clear()
  purchaseButton.draw()
  clearButton.draw()
  
  screen.drawText(
    "IAP in microStudio",
    0,
    -screen.height / 2 + 24,
    8,
    "rgb(79,79,79)")
end
```

### Button class

Below is the Button class:

```
Button = class
  constructor = function(x, y, width, height, radius, color, text, callback)
    this.x = x
    this.y = y
    this.width = width
    this.height = height
    this.radius = radius
    
    this.color = color // object:
      // color = object
      //   untouched = "rgb(0,0,0)"
      //   touched = "rgb(0,0,0)"
      //   callback = "rgb(0,0,0)" // option
      // end
    
    this.color.current = this.color.untouched

    this.text = text // object:
      // text = object
      //   content = ""
      //   size = 0
      //   color = "rgb(0,0,0)"
      // end
    
    this.callback = callback

    this.touchOver = false
    this.isTouched = false
  end

  handleButton = function()
    this.touchOver =
    touch.x >= this.x - this.width / 2 and
    touch.x <= this.x + this.width / 2 and
    touch.y <= this.y + this.height / 2 and
    touch.y >= this.y - this.height / 2
    
    this.isTouched = this.color.current == this.color.touched
    
    if touch.press and this.touchOver then
      this.color.current = this.color.touched
      elsif touch.release and this.touchOver and this.isTouched then
        this.color.current = this.color.untouched
        callback()
      elsif touch.touching and not this.touchOver and not touch.release then
        this.color.current = this.color.untouched
      elsif not touch.touching and this.color.current == this.color.touched then
      this.color.current = this.color.untouched
    end
  end

  update = function()
    handleButton()
  end

  draw = function()
    screen.fillRoundRect(
      this.x,
      this.y,
      this.width,
      this.height,
      this.radius,
      this.color.current)

    screen.drawText(
      this.text.content,
      this.x,
      this.y,
      this.text.size,
      this.text.color)
  end
end
```

### PurchaseButton class (extended from the Button class)

Below is the PurchaseButton class:

```
PurchaseButton = class extends Button
  constructor = function(x, y, width, height, radius, color, text, callback)
    super(x, y, width, height, radius, color, text, callback)
  end

  setPremium = function()
    global.isPremium = true
    this.text.content = "You are Premium"
    this.color.current = this.color.callback
  end

  purchase = function()
    if global.inAppPurchasePlugin then
      global.inAppPurchasePlugin()
    else
      // Fallback for testing in microStudio environment
      unlockPremium()
      print("Simulated purchase succeeded in microStudio")
    end
  end

  update = function()
    if global.isPremium then return end
    super()
  end

  draw = function()
    super()
  end
end
```

In the `purchase()` method, we call the globally-scoped `inAppPurchasePlugin()` function. This function is not part of the microScript code; it will be defined later as JavaScript code in the `index.html` file.

### Global IAP function

Below is the global IAP function:

```
unlockPremium = function()
  if global.isPremium then return end

  storage.set("is_premium", true)
  purchaseButton.setPremium()

  // Add any other logic to unlock premium features in your app
end
```

The globally-scoped `unlockPremium()` function will be called by the JavaScript code in the `index.html` file provided later in this tutorial.

### ClearButton class (extended from the Button class)

Below is the ClearButton class:

```
ClearButton = class extends Button
  constructor = function(x, y, width, height, radius, color, text, callback)
    super(x, y, width, height, radius, color, text, callback)
  end

  clearPremium = function()
    storage.set("is_premium", false)
    global.isPremium = false
    purchaseButton.text.content = "Upgrade to Premium"
    purchaseButton.color.current = purchaseButton.color.untouched
    print("Premium status cleared")
  end

  update = function()
    if not global.isPremium then return end
    super()
  end

  draw = function()
    if not global.isPremium then return end
    super()
  end
end
```

The `clearPremium()` method is designed to reset the premium status for testing purposes within the microStudio environment.

## Node.js & npm

If you don't have Node.js, download and install it from https://nodejs.org/. This will also install npm (Node Package Manager).

Verify installation by opening your terminal and running:

```
node --version
npm --version
```

## Android Studio and SDK

Ensure Android Studio is installed and its Android SDK are configured. Follow the official Capacitor guide for [Android environment setup](https://capacitorjs.com/docs/getting-started/environment-setup#android-requirements "null").

## Capacitor App

Now, let's create a new Capacitor app.

Open your terminal and navigate to a directory where you want to create your new Capacitor app (e.g., `cd Documents/Projects`). Capacitor will create a new project directory within this location.

Run the following command to start the creator, which will scaffold a new Capacitor application:

```
npm init @capacitor/app
```

The interactive creator will ask for your app's name, its project directory, and a unique ID (typically in reverse domain name style, e.g., `com.yourcompany.appname`).

For this tutorial, we'll use `my-awesome-app` as the name for your Capacitor application's directory.    

Navigate to the newly created Capacitor app directory:

```
cd my-awesome-app
```

In your `my-awesome-app` directory, create a new `dist` directory to hold the HTML5 files you will export from microStudio:

```
mkdir dist
```

It is essential that this folder's name matches the `webDir` value in your `capacitor.config.json` file. By default, this is `"webDir": "dist"`.

Open microStudio and export your project to HTML5:
1. navigate to the *Publish* tab,
2. Click *Export to HTML5*,
3. download the zip file.

Unzip the downloaded zip file and copy all its contents (index.html, .js files, assets, etc.) to the `dist` directory located within your Capacitor app directory.

In your `my-awesome-app` directory, install the following core libraries, plugins, and the Android platform, ensuring all are at version 6:

```
npm install @capacitor/core@6 @capacitor/camera@6 @capacitor/splash-screen@6 @capacitor/android@6
```

In your `my-awesome-app` directory, install the Capacitor Command Line Interface (CLI) at version 6:

```
npm install -D @capacitor/cli@6
```

In your `my-awesome-app` directory, install Cordova Plugin Purchase:

```
npm install cordova-plugin-purchase
```

In your `my-awesome-app` directory, add the Android platform:

```
npx cap add android
```

Open index.html in `dist` directory in your favorite text editor or IDE.

In the `index.html` file, find the comment `// signal that the game is started` and insert the following JavaScript code directly below it:

```javascript
const { store, ProductType, Platform } = CdvPurchase;

store.register([{
	id: 'test-non-consumable',
	type: ProductType.NON_CONSUMABLE,
	platform: Platform.TEST,
}]);

store.when()
	.approved(transaction => {
	 unlockFeature();
	 transaction.finish();
	});
	
store.initialize([Platform.TEST]);

function unlockFeature() {
	window.player.call("unlockPremium");
}

function buy() {
	store.get('test-non-consumable').getOffer().order();
}

window.player.setGlobal("inAppPurchasePlugin",function() { buy() });
```

This code allows you to test a purchase on your device using a native testing platform provided by Google. This enables you to simulate (test) purchases even without a Google Play Console account.

> [!IMPORTANT]
> For a production release, replace the `test-non-consumable` ID with your actual product ID from the Google Play Console and change `Platform.TEST` to `Platform.GOOGLE_PLAY`.

After saving `index.html`, sync your project by running the following command in your `my-awesome-app` directory:

```
npx cap sync
```

> [!TIP]
> Each time you create a new version of your app in microStudio, you must repeat the following steps:
>- export your new project to HTML5,
>- copy the unzipped files to the `dist` directory,
>- inject the JavaScript code for the Cordova Purchase Plugin into `index.html`,
>- sync your project (run `npx cap sync` in your `my-awesome-app` directory).

## Android Studio

Now you are ready to open the project in Android Studio using this command in your `my-awesome-app` directory:

```
npx cap open android
```

After Android Studio is open, wait for the Android Gradle project import to complete. If Android Studio recommends updating the Android Gradle Plugin (AGP), use the AGP Upgrade Assistant to update the project.

In Android Studio, run the app on an installed virtual device (e.g., a Pixel 9 emulator). Alternatively, you can install and run the app on a real Android device connected via a USB cable.

> [!IMPORTANT]
>  To test in-app purchases, you must be signed in to the Google Play Store on your testing device, whether it is virtual or real.

This is not a comprehensive Android Studio tutorial. Generally, I'll provide only some tips you'll need:

- Create and configure icons for your app. You have two options for this: raster icons (PNGs) via `File` → `New` → `Image Asset`, or (my personal preference) a vector icon via `File` → `New` → `Vector Asset`.
- Set up a new `versionCode` for every updated app version by opening the `build.gradle (Module :app)` file, located in the `Android/Gradle scripts` path within your Android project. The version code is crucial for Google Play Console; you are only allowed to use integer increments (e.g., 1, 2, 3). You will also find `versionName "1.0"`, which is the public-facing version number that you can set as you prefer.
- Open `AndroidManifest.xml` in the `Android/app/manifests` path within your Android project and review the code for icon setup. If you wish to lock the screen orientation, you can do so by adding, for example, `android:screenOrientation="portrait"` for portrait mode within the `<activity>` section of that file.
- Finally, build the AAB by navigating to `Build` → `Generate Signed Bundle / APK`, selecting "Android App Bundle," and following the prompts to create or use an upload key.

You are now ready to set up your app in the Google Play Console and upload the AAB file.

Good luck!

Nascir

Last revised on June 21, 2025.
