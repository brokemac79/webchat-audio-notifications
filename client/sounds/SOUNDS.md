# Sound Files Attribution

All sound files in this directory are royalty-free and free for commercial use.

## Files

### notification.mp3 (13KB)
- **Source:** Mixkit.co
- **License:** Mixkit Free License (royalty-free, commercial use allowed)
- **Description:** Subtle notification chime
- **URL:** https://mixkit.co/free-sound-effects/notification/

### alert.mp3 (63KB)
- **Source:** Mixkit.co
- **License:** Mixkit Free License (royalty-free, commercial use allowed)
- **Description:** More prominent alert sound
- **URL:** https://mixkit.co/free-sound-effects/notification/

## License

These sounds are used under the Mixkit Free License which allows:
- ✅ Commercial use
- ✅ Personal projects
- ✅ No attribution required (but appreciated)
- ✅ Modification allowed

## Notes

- For WebM format support, convert these MP3s using FFmpeg:
  ```bash
  ffmpeg -i notification.mp3 -c:a libopus notification.webm
  ffmpeg -i alert.mp3 -c:a libopus alert.webm
  ```

- Current implementation uses MP3 files which are supported by all modern browsers
- WebM files would provide smaller file sizes and better quality

## Alternatives

If you want to use different sounds:
1. Visit https://mixkit.co/free-sound-effects/notification/
2. Download your preferred sound
3. Replace notification.mp3 with your file
4. Update soundPath in WebchatNotifications constructor if using different name
