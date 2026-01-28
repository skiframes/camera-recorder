# Camera Recorder

GPU-accelerated multi-camera recording for Jetson devices using GStreamer and NVENC hardware encoding.

## Features

- **Hardware encoding** via Jetson NVENC - uses ~7% CPU per camera vs ~165% with software encoding
- **Multi-camera support** - Reolink and Axis cameras
- **Automatic segmentation** - 10-minute MKV files with timestamps
- **Secure credentials** - Passwords stored locally, never committed to git

## Requirements

- NVIDIA Jetson (tested on Jetson Nano/Xavier)
- GStreamer with NVIDIA plugins (`nvv4l2decoder`, `nvv4l2h264enc`)
- Network-accessible IP cameras (Reolink, Axis, or other RTSP sources)

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

# Watch GPU/NVENC usage
tegrastats

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

| Encoder | CPU per Stream |
|---------|----------------|
| ffmpeg libx264 | ~165% |
| GStreamer nvv4l2h264enc | ~7% |

This allows recording 4+ cameras simultaneously on a Jetson while leaving CPU headroom for other tasks like live streaming.

## License

MIT
