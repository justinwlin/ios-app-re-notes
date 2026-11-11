# 4. Sign and sideload on iPhone

Installation and device behavior must be recorded as results, rather than assumed.

## Set up the phone

1. Install [Sideloadly from its official site](https://sideloadly.io/) on the Mac.
2. Connect the iPhone with a data-capable cable, unlock it, and accept **Trust This
   Computer**. Allow the accessory connection on the Mac if prompted.
3. On iOS 16 or newer, open **Settings → Privacy & Security → Developer Mode**.
   Enable it, restart, unlock, and confirm the enable prompt. If the setting is
   absent, begin pairing the device in Xcode and check again. See
   [Apple's Developer Mode instructions](https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device)
   and [computer-trust instructions](https://support.apple.com/109054).

## Sign and install

1. In Sideloadly's device dropdown, choose the **iPhone**. The **Apple Silicon**
   entry selects the Mac instead.
2. Drag the **modified** IPA onto the IPA icon.

3. Enter the ordinary Apple Account email you use for signing. The field does not
   ask for a special developer number or Team ID. A free account works. Enter
   account credentials and any verification code in the signing tool's prompts.
4. Click **Start** and wait for successful installation. If the IPA already
   contains the changes, additional framework injection is unnecessary.
5. If the iPhone reports **Untrusted Developer**, open **Settings → General → VPN &
   Device Management**. Under **Developer App**, select the email used for signing,
   tap **Trust**, and follow confirmation/restart prompts. The entry may not appear
   until an app has been installed successfully. These steps are covered by
   [Sideloadly's FAQ](https://sideloadly.io/faq).

Free-account signatures last seven days. Sideloadly's refresh daemon can renew them
when it can reach the device through USB or configured Wi-Fi pairing. For Wi-Fi on
macOS, enable **Show this iPhone when on Wi-Fi** in Finder and keep both devices on
the same network. See the [refresh and Wi-Fi FAQ](https://sideloadly.io/faq).

Sideloadly is the computer-side injector and signer. To
re-sign a finished IPA on the phone afterward, without leaving Sideloadly's
daemon running, follow [Sideload any IPA with SideStore](sidestore.md). Export
the tweaked IPA from Sideloadly and install that file in SideStore.

## Record the device test

Write a short test list for the intended change before installing:

- Confirm that launching reaches the normal app screen.
- Exercise the changed behavior and check the observable outcome.
- Check neighboring flows that use the same code, such as navigation and completion.
- Background/resume and relaunch the app, then check the behavior again.

Record the device model, iOS version, signing tool version, signing date, installed
bundle ID, and observed results in your [investigation record](../templates/investigation.md).
Do not commit Apple Account passwords, signing certificates, or provisioning files.

