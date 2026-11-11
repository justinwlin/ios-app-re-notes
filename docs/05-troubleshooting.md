# 5. Troubleshooting and lessons learned

| Symptom | Next useful check |
| --- | --- |
| An IPA is mistaken for source code | Extract it and identify the engine. A decrypted executable is still compiled. |
| Hopper takes a long time on the whole framework | Save/reuse the database and inspect selected metadata-mapped addresses. Record incomplete analysis explicitly. |
| A Hopper export says `procedure_not_defined` | Use the actual Procedure control at the entry point, then check that a procedure object exists. |
| A nice pseudocode view disagrees with behavior | Recheck the raw instructions, types, offsets, callers, and branches. |
| Xcode tools fail after an update | Finish Xcode's first-launch component setup. Check `xcode-select -p`, `xcodebuild -version`, and `xcrun --find llvm-objdump`. |
| An address from another version looks wrong | Verify the framework SHA-256 and remap the method. Never transplant offsets between builds. |
| The patch builder rejects the original bytes/hash | Stop and verify the input version. Its check is intentional, not a reason to remove the guard. |
| Sideloadly shows only Apple Silicon | That entry is the Mac. Connect/unlock/trust the phone and select it in the dropdown. |
| Sideloadly's Start button is disabled | Check that an IPA, device, and Apple Account email are selected. |
| Developer Mode is missing | Begin pairing the iPhone with Xcode, then revisit Privacy & Security. |
| The app reports Untrusted Developer | Trust the signing email under General → VPN & Device Management. |
| No Developer App entry appears | Check that the installation actually finished successfully. |
| The app installs but crashes | Capture the exact error/device log and verify the signed package. Static instruction tests do not prove launch compatibility. |
| A skipped operation leaves an overlay spinning | Inspect the completion/callback path, not just the display function. |
| A previously installed app expires | Refresh/re-sign it through Sideloadly; free-account signatures last seven days. |
| SideStore pairing, VPN, App ID, or refresh errors | Use the [SideStore guide](sidestore.md). Its troubleshooting table matches the current official error codes. |
| SideStore sign-in fails with `-80045` while iloader sign-in works | Use the [Anisette configuration workaround](sidestore.md#sign-in-error-80045). First refresh and app installation remain separate checks. |

The useful pattern is: establish the exact artifact, recover names where possible,
confirm the behavior in native code, make a small reproducible change, preserve
completion paths, validate the package, and finally test on the actual phone.

Tool documentation: [Hopper](https://www.hopperapp.com/tutorial.html),
[Sideloadly](https://sideloadly.io/faq),
[Apple Developer Mode](https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device).

