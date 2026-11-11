# Sideload any IPA with SideStore

Use this when a patched or modified IPA should stay installed on the iPhone after
the computer is put away. The computer-side signing steps are in
[Sign and sideload on iPhone](04-sideload-iphone.md). Copy
[the per-app record](../templates/sideload-app.md) for each app.

These steps were checked against [SideStore's docs](https://docs.sidestore.io/) on
3 November 2026. Button names, iOS version splits, and helper apps change. When a
step below and the live page disagree, follow the live page and update this note.

## What SideStore is

[SideStore](https://docs.sidestore.io/docs/faq) is an iPhone app that sideloads
IPAs with an ordinary Apple Account. It is a community fork of AltStore. After a
one-time computer install, SideStore re-signs apps on the phone. A free account's
signature lasts seven days, and a refresh extends that period.

The refresh path is an on-device VPN. Current docs use
[LocalDevVPN](https://docs.sidestore.io/docs/installation/prerequisites), installed
from the App Store or the AltStore PAL source. SideStore's FAQ describes that VPN
as a connection only to the phone itself: no remote server, used so SideStore can
reach iOS's local install service. The error-code page's LocalDevVPN example
address is `10.7.0.1`. Older write-ups call the same arrangement a loopback VPN
and name StosVPN or a WireGuard profile. Those names are now on SideStore's
[alternative / outdated page](https://docs.sidestore.io/docs/advanced/alternative).
The VPN has to be connected, on Wi-Fi, for every install, update, and refresh.

That is why this guide uses SideStore once the IPA exists:

- AltStore is the project SideStore forked. Its usual refresh path depends on
  AltServer running on a computer.
- Sideloadly runs on the Mac or PC. It can inject a tweak, export a modified IPA,
  install it, and refresh it while that computer can see the phone. Its refresh
  daemon stays on the computer. See [doc 04](04-sideload-iphone.md).
- SideStore keeps the seven-day re-sign on the phone, so the computer is only
  required for the first setup and for rebuilding a pairing file.

## Free Apple Account limits

A free Apple Account, also called an Apple ID, has the limits SideStore's
[FAQ](https://docs.sidestore.io/docs/faq) and
[error codes](https://docs.sidestore.io/docs/troubleshooting/error-codes) spell out:

| Limit | What it means here |
| --- | --- |
| 7-day signature | Each free-account install must be refreshed before the days counter hits zero. |
| 3 active apps | SideStore itself uses one slot. Two other apps can be active beside it. |
| 10 App IDs in 7 days | Registering a different app, or a different bundle ID, spends one. Errors `1009` and `3013` are this cap. |

The three-app cap and the ten-App-ID cap are different. Deleting an app frees an
active slot. App IDs cannot be deleted by hand. SideStore's error text says an
App ID renews when that app is still installed seven days after it was installed
or last renewed, and it expires when the app has been deleted. Read the dates in
**My Apps** before trying another bundle ID.

A spare Apple Account is the right signing identity for this work. The limits
attach to that account, iloader's sign-in is case-sensitive, and the account does
not have to be the one signed in on the phone. SideStore's FAQ also says shared
older Anisette servers have locked Apple Accounts, and that a virtual Mac from
the Anisette server can show up on the account. That Mac is not a real computer.
Use an official Anisette server, which SideStore can switch under its Anisette
setting. Keep the account email out of Git. Passwords, verification codes,
certificates, and pairing files stay off this repository entirely.

## One-time setup

### Check the phone and the computer

[Prerequisites](https://docs.sidestore.io/docs/installation/prerequisites), as
published on the check date:

- iPhone, iPad, or iPod touch with a passcode, on iOS or iPadOS 15.0 or newer.
  Confirm the version in **Settings → General → About** before installing. The
  on-device steps split again by version; see the table below.
- A computer only for this first install: Windows 8 or newer, macOS High Sierra
  or newer, a current Linux distribution, or a recent Chromebook that is not
  enrolled. This repository's machines are Macs. Windows, Linux, and Chromebook
  each have their own tab on that page, including a Windows warning for 32-bit
  and for Windows 10 on Arm.
- An Apple Account. Prefer the spare one from the previous section.
- Wi-Fi. A cellular connection is not enough for install or refresh.

### Install the VPN, then SideStore

1. Install **LocalDevVPN** from the App Store link on the prerequisites page, or
   from the AltStore PAL source. Open it, connect, and if iOS asks to allow a VPN
   configuration, allow it and enter the passcode. Leave it connected whenever
   SideStore installs or refreshes.
2. Download **iloader** from the macOS section of the same page and install it.
   The SideStore install page says iloader problems are handled on the idevice
   Discord, not as SideStore bugs.
3. Plug the phone in with a data cable. Unlock it. Tap **Trust** and enter the
   passcode if asked.
4. Open iloader. Sign in with the spare Apple Account. The email is case-sensitive.
   Enter the password and any verification code in iloader's own prompts.
5. Select the phone. Choose **Install SideStore (Stable)**. Nightly builds are a
   separate channel; SideStore asks people to use stable when reporting bugs.

### Pairing file

A pairing file is how SideStore keeps talking to the phone's install service
after the cable is unplugged. The current install guide lets iloader create and
place that file. You generate one by hand when SideStore asks you to replace it,
or when refresh starts failing.

Current official replacement steps,
[Pairing File](https://docs.sidestore.io/docs/advanced/pairing-file):

1. Connect the phone over USB. The docs call USB the reliable option. A phone
   that only appears over the network can still work.
2. Open iloader and accept an update if it offers one.
3. Click **Delete Stored Pairing**.
4. Select the phone. On the phone, tap **Trust**.
5. In iloader, click **Manage Pairing File**.
6. Next to **SideStore**, and next to any other listed sideloaded app that should
   receive it, click **Place**. The success line is "Pairing file placed
   successfully!"
7. Open SideStore and refresh it.

The same page says a pairing file can expire after an update or erase, and also
at random. Replacing it is the fix. Error `1006` adds a reset inside SideStore
first: **Settings → Reset Pairing File**. Then, in iloader, **Delete Stored
Pairing**, refresh the device list, pair the phone, and **Manage Pairing File →
Place in All Apps** (or **Place** beside SideStore). If it still fails, restart
the phone and the computer and place the file again.

**JitterbugPair** is documented on the
[alternative / outdated page](https://docs.sidestore.io/docs/advanced/alternative),
not in the current install path. Use it when you are deliberately following that
older page. It writes a `.mobiledevicepairing` file. Zip that file before
AirDrop or cloud transfer. Those transfers often rename it to `.txt`, and
SideStore rejects the renamed file. Current SideStore accepts
`.mobiledevicepairing`. The outdated page notes that very old SideStore builds
also accepted `.plist`. The tool is JitterbugPair, not the Jitterbug app.

Do not commit a pairing file, a zip of one, or a renamed copy. This repository
ignores `*.mobiledevicepairing` and `*.zip`.

### Trust the profile and turn on Developer Mode

Unplug only after iloader has finished. Then follow the on-device section that
matches the phone on the
[install page](https://docs.sidestore.io/docs/installation/install). The three
ranges below are that page's split on 3 November 2026. A newer iOS version may
add a fourth range or change a button.

| iOS / iPadOS on the install page | What that section currently says to do |
| --- | --- |
| 15.0–15.8.6 | **Settings → General → VPN & Device Management → Developer App →** the Apple Account **→ Trust**. No Developer Mode step is listed for this range. |
| 16.0–17.7.7 | Trust the Apple Account the same way, then **Settings → Privacy and Security → Developer Mode**. Turn it on and let the phone restart. Confirm the prompt after restart. |
| 18.0 and newer | The trust button is **Allow & Restart**. Enter the passcode. Then turn on **Developer Mode** under **Privacy and Security** and let the phone restart. |

Developer Mode and the trust prompt are separate. Apple's own description is
[Enabling Developer Mode on a device](https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device).
On iOS 16 and newer the Developer Mode switch sometimes stays hidden until a
computer has started a developer pairing. iloader's install is that pairing for
this setup. If the switch is missing, reconnect once and look again.

### Finish setup before installing anything else

1. Open **LocalDevVPN** and connect.
2. Open **SideStore** and sign in with the same Apple Account used in iloader.
3. Open **My Apps**.
4. Tap the **7 DAYS** counter to the right of SideStore. That counter is the
   days left on the signature, and tapping it refreshes that app. Do this before
   sideloading anything else. The FAQ says iloader signs SideStore with its own
   app group, and this first refresh switches SideStore to its own group.
5. If SideStore asks to revoke or create a signing certificate, tap **Yes** or
   **Refresh Now**.

The install page's success check: SideStore sends you to the Home Screen with a
notification, then the SideStore icon is usable again within a few seconds.

## Install one modified IPA

Repeat this section for every app. Record it in
[templates/sideload-app.md](../templates/sideload-app.md).

An app previously installed with Sideloadly needs one install/import through
SideStore before SideStore manages its refreshes. Signing in to SideStore alone
does not import existing apps. Keep the existing app installed during this transfer;
the [official FAQ](https://docs.sidestore.io/docs/faq) says installing the same or
updated IPA through SideStore should preserve its data. Keep the signing account
and bundle identity consistent. If installation reports an identity conflict,
inspect that error before deleting the existing app and its data.

The finished IPA does not need a new tweak, signing helper, or pairing file embedded
inside it. iloader normally handles SideStore's pairing setup during installation.
First refresh SideStore itself with Wi-Fi and LocalDevVPN connected. If that
reports a pairing-file error, follow the pairing-file section above; a sign-in
success by itself does not verify pairing or app installation. The pairing file
belongs to SideStore's device connection, not to the app's binary patch.

### 1. Start from a decrypted IPA you are entitled to use

A decrypted IPA is a normal IPA whose executable can be inspected and modified.
It is still compiled. How this repository treats that file, including hash and
`Info.plist` checks, is [Obtain and inspect an IPA](01-obtain-and-inspect.md).

For an iPhone, use an **IPA**. **APK** is the Android package format. If the IPA
already contains its changes, skip the injection step below and continue to
copying that IPA to the phone.

Keep the original file unchanged. Write the injected copy somewhere outside Git,
such as `~/Desktop` or the ignored `local/` directory. Record the source, the
app version, and the SHA-256 of both files in the per-app note. A filename that
says "decrypted" or "patched" is a label, not proof.

### 2. Inject the tweak into a new IPA

Use one injector, then hand the resulting IPA to SideStore. SideStore is the
installer and the refresher. Injecting again inside SideStore is not required.

**Sideloadly** ([sideloadly.io](https://sideloadly.io/)) runs on the computer. Its
feature list includes tweak injection for `.dylib`, `.deb`, `.framework`, and
`.bundle` files, plus an install mode that exports the tweaked IPA. Drop in the
decrypted IPA, add the tweak with that injection option, and export. Use the
export as SideStore's input. Framework injection in this repository's own patch
flow is [Choose a patch strategy](03-patch-and-package.md). Sideloadly can also
install by itself; if it does, SideStore's FAQ says you can later install the
same IPA in SideStore without deleting the app, and the data stays.

**Azule** ([Al4ise/Azule](https://github.com/Al4ise/Azule)) is a command-line
injector for `.deb`, `.dylib`, and `.framework` files. The output flag is a
directory. Run `azule -h` if the flags differ from this form:

```sh
azule -i "$HOME/Desktop/App-decrypted.ipa" \
  -o "$HOME/Desktop/sideload-out" \
  -f "$HOME/Desktop/Tweak.dylib"
```

A `.deb` path works in place of the `.dylib`. `-b new.bundle.id` sets a new
bundle ID when you need one. Leave Azule's Apple Account examples unused;
SideStore does the signing.

**Feather** ([khcrysalis/Feather](https://github.com/khcrysalis/Feather)) is an
on-device signer. Its README describes `.deb` and `.dylib` injection through
Ellekit. Use it to produce the modified IPA, then install that IPA with
SideStore so SideStore owns the seven-day refresh. An app Feather installs on
its own is Feather's copy. SideStore's error-code page also uses Feather as a
header-repair step for `ldid` errors: import the IPA, sign it with any
certificate pair, pack the signed app back into an IPA, and install that IPA in
SideStore. That repair is separate from everyday tweak injection.

### 3. Copy the IPA to the phone

AirDrop, iCloud Drive, or a cable copy into the Files app all work. Leave the
file in Files until SideStore has finished installing. Do not add it to this
repository. `*.ipa` is already ignored.

### 4. Install it from SideStore

1. Connect to Wi-Fi. Open LocalDevVPN and connect.
2. Open SideStore.
3. Tap the **+** button and choose the IPA in Files. The pages checked on
   3 November 2026 say to import an IPA "like how you would install any other
   IPA" and do not name that button. In this AltStore fork the local-IPA control
   has been the **+** button. If a build moves or renames it, use the control
   SideStore labels for installing a file from the Files app.
4. Wait until the install finishes. Then launch the app once and write down what
   it actually did. A finished install is not a behavior test.

### Bundle ID clashes

iOS allows one installed app per bundle ID. If the App Store copy of the same
ID is already on the phone, remove that App Store copy before installing, or
give the sideloaded IPA a different bundle ID.

Removing the App Store app deletes its local data with it. Changing the bundle
ID leaves the App Store app in place and starts the sideloaded copy with empty
data. Sideloadly's listed features include changing the bundle ID. Azule's flag
is `-b`. A new bundle ID spends one of the ten weekly App IDs. Extensions, app
groups, and servers that expect the original ID can reject the renamed copy.
Record whichever ID actually got installed.

## Refreshing

### What a refresh does

Refresh re-signs an app that is already on the phone. SideStore asks Apple for a
new provisioning profile for that bundle ID and installs the new signature over
the existing app. Wi-Fi and LocalDevVPN both have to be connected. The computer
stays out of it. Refresh SideStore itself as well as each modified app. The
days-left counter moving upward is the sign that it worked.

The FAQ says SideStore also tries to refresh in the background. Treat that as a
convenience. Confirm the counters after the first few days, because a missed
background refresh looks the same as success until an app refuses to open.

### Refresh from My Apps

Open SideStore with LocalDevVPN connected. In **My Apps**, tap the days counter
on SideStore, then the counter on each other app. The install page describes
that counter as the manual refresh for the app beside it.

### Refresh from Shortcuts every few days

SideStore provides a Shortcuts action that refreshes its apps. The docs site did
not publish a Shortcuts tutorial on the check date, so treat the labels below as
the current shape of that action and re-read them after an iOS or SideStore
update. Shortcuts can show an "Unknown Action" row after an upgrade; delete
that row and add the SideStore refresh action again.

1. Open **Shortcuts** and create a shortcut. Name it something like "Refresh SideStore".
2. Add SideStore's **Refresh All Apps** action. Search for SideStore if it is
   not in the action list. Open the action's arrow and turn off **Show When Run**
   if you do not want a prompt in the middle of the automation.
3. Add a **Wait** of about 30 to 60 seconds after the refresh so Shortcuts stays
   open while signing finishes. Lengthen the wait if SideStore's own counter does
   not move. That duration is practical experience, not a number from the docs.
4. If the shortcut should connect the VPN itself, add a **Set VPN** action that
   connects **LocalDevVPN** before the refresh. The older StosVPN / WireGuard
   profile name belongs to the outdated setup.
5. Run the shortcut once by hand. Then open SideStore and check that the
   days-left numbers increased.
6. In Shortcuts, open **Automation → New Automation → Time of Day**. Pick a time
   when the phone is usually on Wi-Fi and charging, and repeat it every few days
   so a missed run still lands inside the seven-day window. Choose the shortcut.
   Turn on **Run Immediately** when that switch exists. Some iOS versions still
   ask the first time, and some refuse to run a personal automation while the
   phone is locked. The confirmation that matters is the days counter the next
   morning.

### When a signature expires

After seven days without a successful refresh, iOS stops launching the app. The
alert is often **Unable to Verify App**. Related wording says the developer must
be verified, or that an internet connection is required before the app will open.
The icon can look dimmed.

The app's documents and other local data stay until you delete the app. Connect
to Wi-Fi, connect LocalDevVPN, open SideStore, and refresh the app. If the app
has disappeared from **My Apps**, the
[troubleshooting guide](https://docs.sidestore.io/docs/troubleshooting) says to
install the same IPA again without removing the home-screen app. The data should
still be there. Deleting the app is the action that throws the data away.

The same **Unable to Verify App** alert can appear while the signature is still
inside seven days, if the phone is offline when iOS rechecks the free developer
certificate. Connect to the internet and open the app again. If **Settings →
General → VPN & Device Management** shows the Apple Account as untrusted, trust
it there.

## Troubleshooting

Official lists: [common issues](https://docs.sidestore.io/docs/troubleshooting/common-issues),
[error codes](https://docs.sidestore.io/docs/troubleshooting/error-codes), and the
[troubleshooting guide](https://docs.sidestore.io/docs/troubleshooting).

| What you see | What fixes it |
| --- | --- |
| SideStore login returns `-80045` while iloader login works. | Follow [the confirmed sign-in workaround](#sign-in-error-80045). This is an authentication setup issue; this error alone does not establish a pairing-file failure. |
| Pairing file invalid, expired, or missing. Refresh fails right away. Error `1006`, or an AFC / minimuxer file error such as `27`. | Replace the pairing file with iloader, in the steps above. Also try SideStore **Settings → Reset Pairing File**. This shows up after an iOS update, an erase, and sometimes with no obvious cause. |
| VPN not connected. Error `1414`, whose title still says "No Wi-Fi/StosVPN". | The body of that error tells you to connect Wi-Fi and LocalDevVPN. The title is leftover wording. Turn off DNS blockers. Restart LocalDevVPN, then SideStore. If it persists, replace the pairing file. Cellular alone does not satisfy the Wi-Fi check. |
| App ID limit. Error `1009` or `3013`: cannot register more than 10 App IDs in 7 days. | Stop changing bundle IDs. In **My Apps**, read when each App ID expires and wait. Uninstalling does not immediately delete the App ID. A second Apple Account has its own cap and its own three-app slot. |
| **Unable to Verify App** when tapping the icon. | Put the phone on the internet. If the days counter has hit zero, refresh from SideStore. If the signature is still valid, iOS may only need a network check. Trust the Apple Account under **VPN & Device Management** when that profile is untrusted. |
| Expired or revoked certificate. SideStore asks to revoke or create a certificate. Errors `3007` and `3008` are certificate failures; the error-code page still says details for several of those codes are unfinished. | On the prompt, choose **Yes** or **Refresh Now**, then refresh SideStore and each app. Revoking a certificate invalidates apps signed with it until they are refreshed. Error `1100` means the Apple session expired: sign in again. Error `1102` means the Apple Account cannot be used for development until the Developer terms are accepted, or until you switch to another account. |
| Login fails, or the Apple Account locks. | Check the password and the email's capitalization. A missing verification code can be generated on the phone under the Apple Account's sign-in settings; the troubleshooting guide splits that path at iOS 18.1. Switch Anisette server if the current one is down or is a shared older server. |
| Install says the app is an invalid format (`1007`) or cannot be found (`1005`). | The file SideStore received is not a standard IPA, or the path disappeared. Rebuild the export and copy it to Files again. |
| `ldid` error while signing. | Follow the Feather repack steps on the error-code page, then install the packed IPA in SideStore. |

### Sign-in error 80045

Symptom: iloader sign-in succeeds, SideStore opens on the phone, and SideStore
sign-in displays `This action cannot be completed at this time. (-80045)`. Sign-in
worked after the following sequence:

1. Close the login sheet and open **SideStore Settings → User Customizations**.
2. Turn **On-Device Anisette** off. Accept the app restart if prompted.
3. Under **Anisette Servers**, select **SideStore (New)** by name and turn
   **Enable Auto Rotation** off. The official list maps this name to
   `https://new.ani.sidestore.io`; list positions can change.
4. Open **User Customizations → Anisette Client Configuration → Remote Server
   Sync**. Tap **Fetch**, then **Save as Overrides**.
5. Return to **User Customizations** and use **Reset adi.pb** to clear cached
   Anisette provisioning data. This is separate from resetting the device pairing.
6. Fully close and reopen SideStore. With Wi-Fi and LocalDevVPN connected, sign in
   with the same Apple Account used in iloader.

Evidence: the [community workaround and success reports](https://github.com/SideStore/SideStore/issues/1675#issuecomment-6363989273),
the [matching issue](https://github.com/SideStore/SideStore/issues/1709), and the
[official server list](https://github.com/SideStore/anisette-servers/blob/main/servers.json).
This worked on 2026-11-02. The exact phone/SideStore build was not recorded, and
the individual contribution of each setting was not isolated. It is a confirmed
workaround, not a guarantee for every version or server.

After login, refresh SideStore itself under **My Apps** using its remaining-days
counter before importing other apps. Sign-in success alone does not confirm the
first refresh, app transfer, or unattended background refresh.

## Record of each app

Fill one copy of [templates/sideload-app.md](../templates/sideload-app.md) per
sideloaded app. The fields are app name, source IPA and version, tweak and
version, bundle ID, Apple Account used, install date, last refresh, and notes.

Keep a filled copy under `local/sideload/`, which is gitignored, whenever it
contains an Apple Account email. A committed copy is fine once the email,
pairing file, and IPA paths that point at credentials are removed.

## Other ways to keep an app installed

**Paid Apple Developer Program.** SideStore's FAQ describes the paid account as
$99 a year. It removes the three-active-app cap and makes the signature last
365 days instead of 7. Confirm the current price on
[Apple's program page](https://developer.apple.com/programs/) before paying. The
ten-App-ID weekly cap described above is the free-account rule; read Apple's
current account terms for what a paid team allows. Refreshing becomes rare, and
the same SideStore or Sideloadly install path still applies.

**TrollStore.** TrollStore can install an IPA permanently, with no seven-day
refresh, only on iOS versions its project still lists. The compatibility list is
the upstream README: <https://github.com/opa334/TrollStore>. Read that page for
the phone's exact version. This guide does not copy the range, because the list
is the source of truth and install methods also depend on the device. If the
phone's version is outside that list, stay on SideStore.

## Safety

Use a throwaway login when you test a modified client of an account-based app
such as Instagram. Those clients can get the account flagged or banned. The
throwaway login is the account inside the app. It is separate from the spare
Apple Account that signs the IPA.

Only install IPAs you are entitled to use. Keep the original publisher's IPA
unchanged, and keep every IPA, pairing file, certificate, and password out of
Git.

## Pages that were checked

- <https://docs.sidestore.io/>
- <https://docs.sidestore.io/docs/faq>
- <https://docs.sidestore.io/docs/installation/prerequisites>
- <https://docs.sidestore.io/docs/installation/install>
- <https://docs.sidestore.io/docs/advanced/pairing-file>
- <https://docs.sidestore.io/docs/advanced/alternative>
- <https://docs.sidestore.io/docs/troubleshooting>
- <https://docs.sidestore.io/docs/troubleshooting/common-issues>
- <https://docs.sidestore.io/docs/troubleshooting/error-codes>
- <https://sideloadly.io/>
- <https://github.com/Al4ise/Azule>
- <https://github.com/khcrysalis/Feather>
- <https://github.com/opa334/TrollStore>
- <https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device>
- <https://developer.apple.com/programs/>
