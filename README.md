# Camera Recorder

Hardware-accelerated multi-camera recording using GStreamer with automatic platform detection.

## Features

- **Hardware encoding** - Auto-detects platform and uses the best available encoder
- **Multi-platform** - Jetson (NVENC), Intel/AMD (VA-API), or software fallback
- **Multi-camera support** - Reolink and Axis cameras
- **Automatic segmentation** - 10-minute MKV files with timestamps
- **Secure credentials** - Passwords stored locally, never committed to git

## Supported Platforms

| Platform | Encoder | CPU Usage | Notes |
|----------|---------|-----------|-------|
| NVIDIA Jetson | nvv4l2h264enc | ~7% | Tested on Jetson Nano/Xavier, Seeed J40 |
| Intel x86 | vaapih264enc | ~10-15% | Requires `gstreamer1.0-vaapi` |
| AMD x86 | vaapih264enc | ~10-15% | Requires `gstreamer1.0-vaapi` |
| Any (fallback) | x264enc | ~165% | Software encoding, limits concurrent streams |

## Requirements

- GStreamer 1.0 with appropriate plugins for your platform
- Network-accessible IP cameras (Reolink, Axis, or other RTSP sources)

### Platform-specific dependencies

**Jetson (pre-installed on JetPack):**
```bash
# Usually already available
gst-inspect-1.0 nvv4l2h264enc
```

**Ubuntu/Debian x86 (Intel/AMD VA-API):**
```bash
sudo apt install gstreamer1.0-vaapi intel-media-va-driver vainfo
# Verify: vainfo
```

**Fallback (software):**
```bash
sudo apt install gstreamer1.0-plugins-ugly gstreamer1.0-libav
```

## Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/turnanalysis/camera-recorder.git
   cd camera-recorder
   ```

2. **Create your credentials file:**
   ```bash
   cp credentials.template credentials.local
   chmod 600 credentials.local
   ```

3. **Edit credentials.local with your camera passwords:**
   ```bash
   nano credentials.local
   ```

4. **Edit cameras.conf if your camera IPs differ from defaults**

## Usage

```bash
# Start recording all cameras
./record_cameras.sh start

# Check status
./record_cameras.sh status

# Stop all recordings gracefully
./record_cameras.sh stop

# Restart recordings
./record_cameras.sh restart
```

## Monitoring

```bash
# Watch CPU usage
htop

# Watch GPU usage (platform-specific)
tegrastats       # Jetson
intel_gpu_top    # Intel (requires intel-gpu-tools)
radeontop        # AMD

# Check logs
tail -f /home/paul/data/logs/*.log
```

## Configuration

### cameras.conf

Define cameras with format: `name|rtsp_url|fps|bitrate`

```
reolink_101|rtsp://$REOLINK_USER:$REOLINK_PASS@192.168.0.101:554/h264Preview_01_main|30|8000000
axis|rtsp://$AXIS_USER:$AXIS_PASS@192.168.0.100/axis-media/media.amp|60|12000000
```

### Bitrate Guidelines

| Resolution | FPS | Recommended Bitrate |
|------------|-----|---------------------|
| 1080p | 30 | 6-8 Mbps |
| 2K (2304x1296) | 30 | 8-10 Mbps |
| 1080p | 60 | 10-12 Mbps |

## Output

Recordings are saved to `/home/paul/data/recordings/<camera_name>/`:

```
/home/paul/data/recordings/
├── reolink_101/
│   ├── reolink_101_20260128_140000.mkv
│   └── reolink_101_20260128_141000.mkv
├── reolink_103/
│   └── ...
└── axis/
    └── ...
```

## CPU Comparison

| Encoder | CPU per Stream | Max Streams (4-core) |
|---------|----------------|----------------------|
| GStreamer nvv4l2h264enc (Jetson) | ~7% | 10+ |
| GStreamer vaapih264enc (Intel) | ~10-15% | 6-8 |
| GStreamer x264enc (software) | ~165% | 2 |

Hardware encoding allows recording multiple cameras while leaving CPU headroom for other tasks.

## Check Detected Platform

```bash
./record_cameras.sh info
```

Override auto-detection if needed:
```bash
ENCODER_PLATFORM=vaapi ./record_cameras.sh start
```

## License

MIT
