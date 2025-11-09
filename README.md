# Fix Apple TV 4K Plex Buffering - HEVC/4K Direct Play Not Working

![Plex Version](https://img.shields.io/badge/Plex-v1.40%2B-e5a00d)
![Apple TV](https://img.shields.io/badge/Apple%20TV-4K%20(all%20models)-000000)
![Platform](https://img.shields.io/badge/Platform-Docker%20%7C%20Linux%20%7C%20macOS%20%7C%20Windows-blue)

> **Tested on:** Plex Media Server v1.42.2, Apple TV 4K (2nd/3rd gen), tvOS 17-18

**Requirements:**
- Plex Media Server v1.40 or newer
- Apple TV 4K (any generation)
- tvOS 11 or newer

## Problem

When pressing "Play" on 4K HEVC movies in Plex on Apple TV 4K, you get buffering or errors, but manually selecting the version works fine.

**Error Message:**
```
App cannot direct play this item. No direct play video profile exists for protocol http,
with container mkv, and video codec hevc
```
(May also appear with container `mp4` instead of `mkv`)

**Root Cause:** Plex's default tvOS profile doesn't support Direct Play for modern HEVC/H.265 content over HTTP protocol, causing it to attempt Direct Stream or transcoding which fails.

## Solution: Custom tvOS Profile

### Step 1: Check Apple TV Settings First

On your Apple TV, go to **Settings → Apps → Plex**

Ensure these settings:
- **Video Quality → Remote Streaming:** Set to **Maximum** or **Original Quality**
- **Auto Adjust Quality:** Turn this **OFF**
- **Play Smaller Videos:** Set to **Original Quality**

### Step 2: Create Custom tvOS Profile on Plex Server

#### On your Plex server (via SSH or terminal):

1. **Stop Plex** (important to prevent file corruption):
```bash
# Docker:
docker stop plex

# Linux (systemd):
sudo systemctl stop plexmediaserver

# macOS:
# Quit Plex from menu bar or use:
killall "Plex Media Server"

# Windows:
# Stop via Services (services.msc) or PowerShell:
Stop-Service -Name "PlexService"
```

2. **Create the Profiles directory** (if it doesn't exist):
```bash
# Docker volume path (adjust to your config location):
mkdir -p "/path/to/plex/config/Library/Application Support/Plex Media Server/Profiles"

# Standard Linux install:
sudo mkdir -p "/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Profiles"

# macOS:
mkdir -p "$HOME/Library/Application Support/Plex Media Server/Profiles"

# Windows (PowerShell):
New-Item -ItemType Directory -Force -Path "$env:LOCALAPPDATA\Plex Media Server\Profiles"
```

3. **Download the custom tvOS.xml file:**
```bash
# Docker (adjust path):
curl -o "/path/to/plex/config/Library/Application Support/Plex Media Server/Profiles/tvOS.xml" \
  https://raw.githubusercontent.com/BTerell/plex-appletv-hevc-fix/main/tvOS.xml

# Standard Linux:
sudo curl -o "/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Profiles/tvOS.xml" \
  https://raw.githubusercontent.com/BTerell/plex-appletv-hevc-fix/main/tvOS.xml

# macOS:
curl -o "$HOME/Library/Application Support/Plex Media Server/Profiles/tvOS.xml" \
  https://raw.githubusercontent.com/BTerell/plex-appletv-hevc-fix/main/tvOS.xml

# Windows (PowerShell):
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/BTerell/plex-appletv-hevc-fix/main/tvOS.xml" `
  -OutFile "$env:LOCALAPPDATA\Plex Media Server\Profiles\tvOS.xml"
```

Or manually create the file (see [tvOS.xml](tvOS.xml) in this repo).

4. **Fix file ownership** (Linux/macOS only):
```bash
# Docker (check your Plex UID - usually 1000):
chown -R 1000:1000 "/path/to/plex/config/Library/Application Support/Plex Media Server/Profiles/"

# Linux:
sudo chown -R plex:plex "/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Profiles/"

# macOS:
chown -R $(whoami) "$HOME/Library/Application Support/Plex Media Server/Profiles/"

# Windows: No ownership changes needed
```

5. **Start Plex:**
```bash
# Docker:
docker start plex

# Linux (systemd):
sudo systemctl start plexmediaserver

# macOS:
# Open Plex from Applications folder

# Windows:
# Start via Services (services.msc) or PowerShell:
Start-Service -Name "PlexService"
```

### Step 3: Test on Apple TV

1. Force quit the Plex app on Apple TV (double-click TV button, swipe up on Plex)
2. Reopen Plex
3. Try playing a 4K HEVC movie with the "Play" button (not manually selecting version)
4. Check if it Direct Plays without buffering

### Step 4: Verify Direct Play is Working

**During Playback:**
1. Open **Plex Web UI → Settings → Dashboard**
2. Find your active playback session
3. Check the playback info shows:
   - ✅ **Direct Play** (not "Direct Stream" or "Transcode")
   - ✅ Codec: **HEVC** or **H265**
   - ✅ Container: **mkv** or **mp4**

**Or check "Now Playing" tray:**
- Click on the session → Shows `(playing)` with no transcoding icon

## What This Fixes

✅ **Enables 4K HEVC Direct Play** - No more server-side video transcoding
✅ **Reduces server CPU/bandwidth requirements** - Direct Play uses minimal resources
✅ **Fixes "Play" button errors** - Auto-play now works without manual version selection
✅ **Supports 10-bit HDR content** - Full color depth for modern 4K files

### Before vs After Comparison

| Feature | Default tvOS Profile | Custom Profile |
|---------|---------------------|----------------|
| Video Codecs | H.264 only | H.264, HEVC/H.265 |
| Max Resolution | 1080p | 4K (3840×2160) |
| Color Depth | 8-bit | 10-bit HDR |
| Containers | MP4, MOV (limited) | MP4, MOV, MKV |
| HTTP Protocol | Not supported | Full support |
| Audio Codecs | AAC, AC3 (limited) | AAA, AC3, E-AC3 (Dolby Digital Plus) |

### Common Error Messages This Fixes

If you see these in Plex logs or Apple TV, this profile will fix them:
- `"no direct play video profile exists for http/mp4/hevc"`
- `"no direct play video profile exists for protocol http, with container mkv, and video codec hevc"`
- `"Direct Play is disabled"`
- `"media must be transcoded in order to use the hls protocol"`
- `"MDE: no direct play video profile exists for protocol http"`

## Troubleshooting

### Profile not loading?
- Check file ownership matches Plex user
- Check file is named exactly `tvOS.xml`
- Check XML syntax is valid (no typos)
- Restart Plex server

### Still buffering?
- Verify Apple TV quality settings are set to Maximum
- Check network bandwidth (4K needs 15-40 Mbps)
- Check Plex logs: `Plex Media Server/Logs/Plex Media Server.log`
- Look for errors like:
  - `"no direct play video profile exists for protocol http, with container mkv, and video codec hevc"`
  - `"no direct play video profile exists for protocol http, with container mp4, and video codec hevc"`

### Where are Plex files?
- **Docker:** Volume mount location (check your docker-compose.yml)
- **Linux:** `/var/lib/plexmediaserver/`
- **macOS:** `~/Library/Application Support/Plex Media Server/`
- **Windows:** `%LOCALAPPDATA%\Plex Media Server\`

## Known Limitations

**What this DOES NOT fix:**
- ❌ Network bandwidth issues (you still need 15-40 Mbps for 4K)
- ❌ Subtitle compatibility (PGS/image-based subtitles may force video transcoding)
- ⚠️ Audio passthrough for DTS/TrueHD (requires eARC-capable receiver - will work but not guaranteed for all setups)

**Apple TV Audio Notes:**
- ✅ **Works natively:** AAC, AC3 (Dolby Digital 5.1), E-AC3 (Dolby Digital Plus/Atmos)
- ⚠️ **Requires eARC passthrough:** DTS, DTS-HD MA, TrueHD, FLAC
- 📝 Without eARC: Plex will transcode these to AC3 (video still Direct Plays!)

**Apple TV Requirements:**
- ✅ Apple TV 4K (1st gen or later) - older Apple TV HD doesn't support HEVC
- ✅ tvOS 11+ (recommended: tvOS 15+)
- ✅ For lossless audio passthrough: eARC-capable receiver (DTS-HD MA, TrueHD)

## Technical Details

This fix addresses the outdated A8-based tvOS profile that Plex ships by default, which doesn't support modern Apple TV 4K (A10X/A12/A15) capabilities including:
- Hardware HEVC decoding up to 4K60
- 10-bit HDR/Dolby Vision
- E-AC3 (Dolby Digital Plus) with Atmos metadata

## Quick Validation

After installing, verify the profile is loaded:

```bash
# Docker - check logs during startup
docker logs plex 2>&1 | grep -i "profile"

# Linux - monitor logs
tail -f "/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Logs/Plex Media Server.log" | grep -i "profile"

# Look for: "Loading client profile: tvOS"
```

The profile should appear as a custom client profile in Plex's Media Decision Engine (MDE).

## Contributing

⭐ **Found this helpful? Please star the repo!** ⭐

Have improvements or found issues? PRs and bug reports are welcome.

### Discussion
- Having issues? [Open an issue](https://github.com/BTerell/plex-appletv-hevc-fix/issues)
- Questions? [Start a discussion](https://github.com/BTerell/plex-appletv-hevc-fix/discussions)

## License

MIT License - Feel free to use, modify, and distribute.

---

**Note:** This profile has been tested on Apple TV 4K (2nd/3rd gen) with Plex Media Server v1.42.2+

**Credits:** Profile originally created to fix Apple TV 4K HEVC Direct Play issues. Shared freely to help the Plex community.
