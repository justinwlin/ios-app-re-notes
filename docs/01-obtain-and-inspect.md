# 1. Obtain and inspect an IPA

## Terms that are easy to mix up

| Term | Meaning here |
| --- | --- |
| IPA | A ZIP-format package containing an iOS app under `Payload/`. |
| APK | An Android app package; it is not the file used for this iPhone workflow. |
| Decrypted IPA | The packaged native code is available to inspect rather than encrypted. It is still compiled code. |
| Disassembly | Machine instructions displayed as assembly. |
| Decompilation | An approximation of those instructions in readable pseudocode, not the original project. |
| Signing / sideloading | Preparing an app's signature and installing it on the device. |

## Get a decrypted IPA

Download decrypted IPAs from [ARMConverter's US store](https://armconverter.com/store/us).

1. Open <https://armconverter.com/store/us>.
2. Create an ARMConverter account.
3. Log in with that account.
4. Search for the app and download the **IPA** for the version you want to inspect.
5. Save the `.ipa` locally, outside Git.

Only use apps whose license and terms permit inspection and modification.
Record the download URL, acquisition date, bundle identifier, version, and
SHA-256. A filename containing `Decrypted` is a label to verify, not proof. Run
the encryption-header check below on every download.

## Keep the input and create an inspection copy

From the repository root, change the input path and case name to your file:

```sh
IPA="$HOME/Downloads/MyApp-1.0.ipa"
CASE="artifacts/myapp-1.0"
mkdir -p "$CASE"
shasum -a 256 "$IPA"
unzip -t "$IPA"
unzip -q "$IPA" -d "$CASE/extracted"
```

No rename to `.zip` is required. Use an empty extraction destination to avoid
mixing versions.

Read the app identity, then inspect the executables:

```sh
APP="$(echo "$CASE"/extracted/Payload/*.app)"
plutil -p "$APP/Info.plist"
EXE="$(plutil -extract CFBundleExecutable raw "$APP/Info.plist")"
file "$APP/$EXE"
xcrun otool -l "$APP/$EXE"
# For a Unity app, also inspect the framework that holds the game code:
file "$APP/Frameworks/UnityFramework.framework/UnityFramework"
xcrun otool -l "$APP/Frameworks/UnityFramework.framework/UnityFramework"
```

The `.app` folder and executable names vary across apps. Read
`CFBundleExecutable` from `Info.plist` instead of assuming they match the display
name. In the Mach-O load commands inspect `LC_ENCRYPTION_INFO` or
`LC_ENCRYPTION_INFO_64`. `cryptid = 0` means the segment is not encrypted. This is
a header observation, not a guarantee that the app runs or that every dependency
is usable. Check the relevant framework as well as the small app launcher.

## Identify where the useful code is

A Unity IL2CPP app usually has this layout:

```text
Payload/MyApp.app/
  Info.plist
  MyApp
  Frameworks/UnityFramework.framework/UnityFramework
  Data/Managed/Metadata/global-metadata.dat
```

`UnityFramework` contains the native game code. `global-metadata.dat` contains
the metadata needed to recover useful class and method names. Record its format
version. Extracting these does not create an editable Unity scene project or
recover the original C# method bodies.
