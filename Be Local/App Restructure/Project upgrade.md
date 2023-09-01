## iOS missing steps
- WatermelonDb:
	- **Add Swift support to your Xcode project**:
    
    You only need to do this if you don't have Swift already set up in your project.
    
    - Open `ios/YourAppName.xcodeproj` in Xcode
    - Right-click on **(your app name)** in the Project Navigator on the left, and click **New File…**
    - Create a single empty Swift file (`wmelon.swift`) to the project (make sure that **Your App Name** target is selected when adding), and when Xcode asks, press **Create Bridging Header** and **do not remove** the Swift file afterwards
	- **Link WatermelonDB's native library (using CocoaPods)**
    Open your `Podfile` and add this:
    ```
    # Uncomment this line if you're not using auto-linking
    # pod 'WatermelonDB', path: '../node_modules/@nozbe/watermelondb'

	# WatermelonDB dependency, should not be needed on modern React Native
	# (please file an issue if this causes issues for you)
	# pod 'React-jsi', path: '../node_modules/react-native/ReactCommon/jsi',
	# modular_headers: true

	# WatermelonDB dependency pod 'simdjson', path: '../node_modules/@nozbe/simdjson',
	# modular_headers: true
	```

- 