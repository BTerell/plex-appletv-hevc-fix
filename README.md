# Fix Apple TV 4K Plex Buffering - HEVC/4K Direct Play Not Working

## Problem

When pressing "Play" on 4K HEVC movies in Plex on Apple TV 4K, you get buffering or errors, but manually selecting the version works fine.

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

During playback, check the dashboard:
1. Go to **Plex Web → Settings → Dashboard**
2. Look for your playback session
3. It should show **"Direct Play"** with **HEVC** codec

## What This Fixes

The default tvOS.xml profile in Plex only supports:
- H.264 codec (not HEVC)
- 1080p max resolution
- No HTTP protocol support for MP4/HEVC

This causes Plex's Media Decision Engine (MDE) to reject Direct Play and attempt transcoding.

The custom profile adds:
- ✅ HEVC/H.265 codec support
- ✅ 4K (3840x2160) resolution support
- ✅ HTTP protocol support for MP4/HEVC containers
- ✅ 10-bit color depth support
- ✅ MKV container support
- ✅ Lossless audio (DTS-HD MA, TrueHD, FLAC)

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
- Look for: `"no direct play video profile exists for http/mp4/hevc"`

### Where are Plex files?
- **Docker:** Volume mount location (check your docker-compose.yml)
- **Linux:** `/var/lib/plexmediaserver/`
- **macOS:** `~/Library/Application Support/Plex Media Server/`
- **Windows:** `%LOCALAPPDATA%\Plex Media Server\`

## Technical Details

This fix addresses the outdated A8-based tvOS profile that Plex ships by default, which doesn't support modern Apple TV 4K (A10X/A12/A15) capabilities including:
- Hardware HEVC decoding up to 4K60
- 10-bit HDR/Dolby Vision
- Dolby Atmos/TrueHD audio passthrough

## Contributing

Found this helpful? Please star the repo! Have improvements? PRs are welcome.

## License

MIT License - Feel free to use and modify.

---

**Note:** This profile has been tested on Apple TV 4K (2017, 2021, 2022 models) with Plex Media Server v1.32+
