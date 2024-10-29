# Running XCUITest with Go-iOS on iOS 17 and Above

**Go-iOS** is an open-source suite of tools written in **Go** that provides developers and QA teams the ability to control iOS devices, primarily on Linux and potentially on Windows (Windows support is in development). Go-iOS enables direct management of apps and device settings and offers a streamlined solution for running UI tests via **XCUITest**. This documentation will guide you through setting up Go-iOS to execute XCUITests and troubleshoot an issue observed on iOS when running individual test cases.

![GitHub Icon](https://github.githubassets.com/favicons/favicon.svg) [Go-iOS GitHub Repository](https://github.com/danielpaulus/go-ios/)

Go-iOS operates through the **DTX Message Framework**, Apple's Remote Procedure Call (RPC) interface. This framework, although complex, enables critical interactions between a test runner and an iOS device, such as:

1. Starting and stopping applications.
2. Running **XCUITests** on physical iOS devices.
3. Manipulating device settings like orientation, location, and screen recordings.

The Go-iOS tool also makes it feasible to run tests on iOS devices within a CI/CD pipeline, giving developers a cross-platform option for iOS testing without requiring macOS.

---

## Workflow Overview: Running Tests with Go-iOS

### Step 1: Go-iOS Setup and Tunnel Configuration

To start testing on iOS devices with Go-iOS, follow these setup steps:

1. **Download Go-iOS** and set up the binary on your Linux system.
2. **Initiate the Device Tunnel** – for iOS 17 and above, a tunnel must be established to connect to the iOS device.
3. **Run XCUITest** – define the app and test bundle identifiers to launch the tests.

### Flowchart: Go-iOS XCUITest Workflow

```plaintext
┌────────────┐     ┌────────────────┐     ┌──────────────┐     ┌───────────────────┐
│ Start Go-iOS │ → │ Open Device Tunnel │ → │ Run XCUITest │ → │ Capture Test Output │
└────────────┘     └────────────────┘     └──────────────┘     └───────────────────┘
```

### Command Breakdown

#### Starting the Go-iOS Tunnel
The tunnel connects Go-iOS to the iOS device, enabling direct communication through the DTX framework. For iOS 17+, this tunnel is mandatory:
```bash
sudo ./go-ios tunnel start --udid <device-udid> --pair-record-path <path-to-record> --tunnel-info-port <port-number>
```

Replace `<device-udid>` with your device’s UDID, `<path-to-record>` with the path to pairing information, and `<port-number>` with an open port.

#### Running XCUITest
After the tunnel is active, execute the XCUITest using Go-iOS. The following example runs a specific test from the suite:
```bash
./go-ios --udid=<device-udid> runtest --bundle-id=com.example.XCUIQuest --test-runner-bundle-id=com.example.UI-Tests.xctrunner --tunnel-info-port=<port-number> --xctest-config "UI Tests.xctest" --test-to-run="UI Tests.ArticleTests/testExample1" --verbose
```

### Example Flow: Full Test Suite vs. Individual Test Cases

```plaintext
┌────────────────────────┐
│ Execute Entire Test Suite│
└────────────────────────┘
         ↓
 All Tests Execute Successfully on iOS 17+
         ↓
Specific Test Execution with --test-to-run Argument
         ↓
Fails to Recognize Specified Test
```

---

## Troubleshooting Issue: Failure to Run Individual Test Case on iOS 17+

When specifying a single test case (using `--test-to-run`), the command does not execute the intended test case on iOS 17+. Instead, the test suite completes instantly, showing zero tests executed. Here’s the observed behavior:

- **Expected Result**: The specified test case, `UI Tests.ArticleTests/testExample1`, runs individually.
- **Actual Result**: The suite passes without executing any tests, logging a successful outcome without actual execution.

### Error Log
```plaintext
Test Suite 'Selected tests' started at ...
Test Suite 'UI Tests.xctest' passed ...
Executed 0 tests, with 0 failures (0 unexpected)
```

### Possible Causes
1. **DTX Framework Handling** – The underlying reverse-engineered framework may not fully support certain syntax or structure in iOS 17.
2. **Test Suite Nomenclature** – Go-iOS might have unhandled edge cases around naming conventions, particularly with complex test suite names or structures.

### Suggested Next Steps for Troubleshooting
1. **Verify Test Suite Structure** – Ensure the test suite name and test case names match expected XCUITest naming conventions.
2. **Explore Go-iOS Logs** – Check Go-iOS debug logs for any warnings or errors related to the `--test-to-run` argument.


---

# Running a Specific Test Suite with Go-iOS

This guide demonstrates how to clone a sample XCUITest project, configure it in Xcode, and execute a specific test suite on a real device using **Go-iOS**.

---

## Step 1: Clone the XCUITest Project

Start by cloning the test project, `XCUIQuest`, which contains the "UI Tests" test suite, I have included the go-ios binary as well in this:

```bash
git clone https://github.com/amankansal-lt/XCUIQuest
```

Once cloned, open the project in **Xcode** to configure and install it on your target device.

## Step 2: Configure and Install the Test Suite

1. **Open the Project in Xcode**:
   - Navigate to the `XCUIQuest` project directory and open the `.xcodeproj` file in Xcode.

2. **Select the "UI Tests" Test Suite**:
   - In Xcode, go to the "Test Navigator" and select the "UI Tests" test suite.
   
3. **Install on a Real Device**:
   - Connect your real iOS device to your computer.
   - Choose the connected device as the target in Xcode.
   - Install the "UI Tests" suite on the device by selecting the "UI Tests" target and pressing `Cmd + U` to build and install.

---

## Step 3: Start the Go-iOS Tunnel

With the app installed, start the **Go-iOS tunnel** to establish a connection to the device.

Run the following command, replacing `<your-pair-record-path>` with your actual path:

```bash
sudo ./go-ios tunnel start --udid 00008110-001E04C10C8A801E --pair-record-path <your-pair-record-path> --tunnel-info-port 27109
```

- **`--udid`**: The device’s unique identifier (UDID).
- **`--pair-record-path`**: The directory path containing the device pairing record (typically in your home directory).
- **`--tunnel-info-port`**: Specifies the port used for the tunnel connection.

**Note**: The tunnel enables communication with iOS 17+ devices through Go-iOS.

---

## Step 4: Run the Specified Test Suite with Go-iOS

With the tunnel running, use the following command to execute a specific test case within the "UI Tests" suite on your device:

```bash
./go-ios --udid=00008110-001E04C10C8A801E runtest --bundle-id=com.example.XCUIQuest --test-runner-bundle-id=com.example.UI-Tests.xctrunner --tunnel-info-port=27109 --xctest-config "UI Tests.xctest" --test-to-run="UI Tests.ArticleTests/testExample1" --verbose
```

### Explanation of Command Options
- **`--udid`**: Device UDID.
- **`--bundle-id`**: Bundle identifier for the app (replace `com.example.XCUIQuest` with the correct bundle ID if different).
- **`--test-runner-bundle-id`**: The bundle ID of the test runner app.
- **`--tunnel-info-port`**: The port set up for the device tunnel.
- **`--xctest-config`**: The test configuration file for "UI Tests.xctest".
- **`--test-to-run`**: Specifies the exact test case to run (`UI Tests.ArticleTests/testExample1`).

Adding `--verbose` provides additional logging information, which is useful for debugging.

---


### Note :
in the source code provided :

1. UI Tests : test target does not work
``` 
./go-ios --udid=00008110-001E04C10C8A801E runtest --bundle-id=com.example.XCUIQuest --test-runner-bundle-id=com.example.XUI-Tests.xctrunner --tunnel-info-port=27109 --xctest-config "UI Tests.xctest" --test-to-run="UI Tests.ArticleTests/testExample1" --verbose

 ```
2. UITest test target works fine 

``` 
go-ios --udid=00008110-001E04C10C8A801E runtest --bundle-id=com.example.XCUIQuest --test-runner-bundle-id=com.example.UITests.xctrunner --tunnel-info-port=27109 --xctest-config "UITests.xctest" --test-to-run="UITests.NewClassTests/testExample1" --verbose
```

## Go-iOS and XCUITest Documentation Summary

For further details, check the
![GitHub Icon](https://github.githubassets.com/favicons/favicon.svg) [Go-iOS GitHub Repository](https://github.com/danielpaulus/go-ios).

--- 
