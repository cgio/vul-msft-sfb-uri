# MSFT SFB URI Vulnerability Report

*Report date: October, 2018*

## Summary

A vulnerability exists in the Skype Meetings App executable (Skype Meetings App.exe). Microsoft SmartScreen and potentially other security features are circumvented. The vulnerability is exploited by launching a target website via the Skype for Business URI (sfb://). Skype Meetings App.exe uses Internet Explorer 11 to render the target website using Internet Explorer 7 Compatibility View. In comparison, on the same test machine, Internet Explorer 11's SmartScreen effectively blocks harmful websites.

## Affected software

File name: Skype Meetings App.exe

Version: 16.2.0.242

SHA-1: 6A882202AE92CA36AB64173A765538616F240855

Product name: Skype Meetings App (32-bit)

Product family:  Skype for Business

File path: `%localappdata%\Microsoft\SkypeForBusinessPlugin\16.2.0.242\Skype Meetings App.exe`

## Test machine

* Windows 10 1803 (OS Build 17134.320)

## To reproduce

Ensure Skype Meetings App is installed.

Several ways to reproduce:

* Via the Windows Run command, use sfb://example.com (omit URL protocol).

* Create an HTML file that automatically redirects or contains an enticing link to sfb://dangerouswebsite.example.com

* Use other common methods for launching Windows URIs

* Command line: `"C:\file-path\Skype Meetings App.exe" sfb://example.com`

Optionally, confirm website content has loaded by right-clicking the Skype Meetings App browser area > View source.

## Examples

#### Microsoft's Windows Defender SmartScreen Demo tests

Fails phishing, malware, blocked download, exploit page, malvertising, and malware download tests:

`sfb://demo.smartscreen.msft.net`

*See included screenshot.*

---

#### WICAR.org browser malware tests

Detected by Windows Defender as Exploit:VBS/CVE-2014-6332.gen (severe):

`sfb://malware.wicar.org/data/ms14_064_ole_not_xp.html`

Detected by Windows Defender as Trojan:JS/CoinHive.A (severe, a JavaScript-based cryptocurrency miner):

`sfb://malware.wicar.org/data/js_crypto_miner.html`

---

#### browseraudit.com tests

To test, use `sfb://browseraudit.com`.

Comparison between regular IE11 and Skype Meetings App's IE11 (IE11 Skype) :

| Browser    | Passed | Warnings | Critical | Skipped |
| :--------- | -----: | -------: | -------: | ------: |
| IE11       | 272    | 118      | 0        | 14      |
| IE11 Skype | 269    | 121      | 0        | 14      |

For example, IE11 Skype has a warning for "browser should not send further requests over plain (non-secure) HTTP" whereas regular IE11 passes this check.

---

#### browserspy.dk tests

To test, use `sfb://browserspy.dk`. 

According to tests, the Skype Meetings App's IE11 allows VBScript, ActiveX, Windows Media Player, and several MIME types different from regular IE11 settings. The user agent is detected as Internet Explorer 7.0.

---

#### HTML hyperlink tests

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>Hello, world!</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
    <a href="sfb://demo.smartscreen.msft.net">Dangerous link</a>
  </body>
</html>
```

Additionally, I uploaded a similar test to:

`https://chadg.io/dangerous-link.html`

Clicking the "dangerous link" (points to `sfb://malware.wicar.org/data/ms14_064_ole_not_xp.html`) hyperlink in Microsoft Edge or Internet Explorer 11 will open the malicious website in the Skype Meetings App without a consent prompt. **This appears to be a significant security risk.** See registry keys at `HKLM\SOFTWARE\Microsoft\Internet Explorer\Low Rights\ElevationPolicy`.

In comparison, Skype for Desktop uses a safer approach. Before calling the phone number, Skype for Desktop asks the user for permission. This safer design can be tested with the "Call the Skype Echo / Sound Test Service" link.

---

## For further investigation

* Domain whitelist is not used: `HKCU\Software\Microsoft\SkypeForBusinessPlugin\16.2\AllowedDomains`

* Omission of "sfb" from "Reserved URI scheme names" list at `https://docs.microsoft.com/en-us/windows/uwp/launch-resume/reserved-uri-scheme-names`

* Undocumented use of mshtml.dll and ShellApp

* Skype for Business Mobile URI (ms-sfb://)

## Notes

* For useful debugging information, set a breakpoint on  PluginLogging.dll's `_OutputLogString@24`. Example log string: `Opening window with URL: //example.com`
