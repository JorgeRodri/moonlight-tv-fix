# Moonlight: fix for Android 14 TVs (audio, but black or frozen video)

This patch adds one new setting to Moonlight: **Settings → Video → "Decoder low-latency mode (Android 14 TV fix)"**.
You can choose from these options:

| Option | What it does | Try it when |
|---|---|---|
| Default | Original Moonlight behaviour | – |
| Skip Android KEY_LOW_LATENCY | Doesn't send the Android 11+ `low-latency` flag | **Try this first.** It's the fix that worked on Amlogic Android 14 boxes (`c2.amlogic.*`). |
| Skip vendor/MediaTek/Amlogic options | Doesn't send `vdec-lowlatency` or the `vendor.*low-latency*` keys | Try this second, especially on MediaTek decoders (`c2.mtk.*`, `OMX.MTK.*`). |
| Off | Sends no low-latency options at all | If neither of the above works. Expect a few ms of extra latency. |

## Build the APK without installing anything (GitHub Actions)

1. Create a new **empty** repository on GitHub (it can be private).
2. Upload this whole folder to it, keeping the folder structure. The repository needs `android14-tv-lowlatency.patch` and `.github/workflows/build.yml`.
3. Open the **Actions** tab, click **Build patched Moonlight APK**, then click **Run workflow**. The build takes about 10–15 minutes.
4. When the run turns green, download **moonlight-android14-tv-fix** from the run page and unzip it to get the `.apk`.

## Install on the TV

- **Easy way:** copy the APK to a USB stick and open it with a file manager on the TV, such as "File Commander" or "X-plore". You'll need to allow installs from unknown sources.
- **Or with adb:** turn on Developer options (Settings → System → About → click "Build" 7 times), then turn on USB/Network debugging, then run:
  ```
  adb connect <TV-IP>:5555
  adb install app-nonRoot-debug.apk
  ```

The app installs as **"Moonlight (Debug)"** next to the official app, so it doesn't replace it. You'll have to pair it with Sunshine or Apollo again.

## If no option works, send the logs

```
adb logcat -c
# start a stream in Moonlight (Debug), wait 10 s, stop it
adb logcat -d | grep -iE "limelight|mediacodec|c2\.|omx|codec2|hwc|surfaceflinger" > moonlight-log.txt
```
The log contains the line `Decoder low-latency mode: ... (decoder: XXX)`, which names your decoder. That name tells us which fix to try next.
