# Firefox on Wayland: Layers of Workarounds

As a longtime Windows user, I always take features like hardware-accelerated video playback for granted. So when I first started to use Firefox on GNOME/Wayland, I immediately noticed the lack of hardware acceleration in video playback. The video decoding hardware in my iGPU stays unused, while the CPU is struggling with these complicated modern video codecs. Firefox also doesn't run under Wayland by default.

Web browsing is important. So these issues need to be dealt with. Unfortunately, related issues in the upstream bug tracker have been consistently marked as low priority. And over the past 6 months, it's only getting worse with each update. This blog post is written for the convenience of those who just want a normal web browsing experience comparable to those provided by browsers on Windows.

Generally speaking, we need 5 flags and 3 global environment variables.

## 5 Flags

Open `about:config` in Firefox and set the following flags:

- `media.ffmpeg.vaapi.enabled` to `true` in order to enable the use of VA-API with FFmpeg.
- `media.ffvpx.enabled` to `false` to disable the internal decoders for VP8/VP9. This is necessary despite [this bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1660336) being fixed.
- `media.rdd-vpx.enabled` to `false` to disable the remote data decoder process for VP8/VP9. As of Firefox 85 its [sandbox blocks VA-API access](https://bugzilla.mozilla.org/show_bug.cgi?id=1673184).
- `security.sandbox.content.level` to `0` to avoid page crash when `media.rdd-vpx.enabled` is set to `false`.
- `media.navigator.mediadatadecoder_vpx_enabled` to `true` to [enable hardware VA-API decoding for WebRTC](https://bugzilla.mozilla.org/show_bug.cgi?id=1709009).

Optionally, force enable WebRender by setting:

- `gfx.webrender.all` to `true`.
- `gfx.webrender.compositor.force-enabled` to `true`.

## 3 Global Environment Variables

Open `/etc/environment` and append the following lines:

```
# Firefox sucks.
# https://wiki.archlinux.org/title/Firefox
# https://bugzilla.mozilla.org/show_bug.cgi?id=1619585
# https://bugzilla.mozilla.org/show_bug.cgi?id=1673184
# https://bugzilla.mozilla.org/show_bug.cgi?id=1635830
# https://mastransky.wordpress.com/2020/03/16/wayland-x11-how-to-run-firefox-in-mixed-environment/
MOZ_ENABLE_WAYLAND=1
MOZ_DISABLE_RDD_SANDBOX=1
MOZ_DBUS_REMOTE=1
```
