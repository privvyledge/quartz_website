
## RTSP Server
1. Install and run the [RTSP server](https://github.com/bluenviron/mediamtx):
	1.  Docker: `docker run --rm -it --network=host bluenviron/mediamtx:latest`
2. Start a stream (video file, camera feed) using gstreamer or FFMPeg and publish to the RTSP server
### From video
#### Using FFMPEG
1. Install FFMPEG: `sudo apt update && sudo apt install ffmpeg`
2. Play the video and republish it to the server: `ffmpeg -re -stream_loop -1 -i file.ts -c copy -f rtsp rtsp://localhost:8554/mystream`
#### Using Gstreamer
1. Install GStreamer and plugins:
2. Play the video and republish it to the server: `gst-launch-1.0 rtspclientsink name=s location=rtsp://localhost:8554/mystream filesrc location=file.mp4 ! qtdemux name=d d.video_0 ! queue ! s.sink_0 d.audio_0 ! queue ! s.sink_1`

## RTSP Client:
### Using Gstreamer
1. Create a client/subscriber node: `gst-launch-1.0 -v rtspsrc location=rtsp://127.0.0.1:8554/test latency=100 buffer-mode=auto ! decodebin ! videoconvert ! autovideosink`
	1. or with ROS2 GScam `rtspsrc location=rtsp://127.0.0.1:8554/test do-retransmission=false latency=100 buffer-mode=auto ! decodebin ! videoconvert ! videoscale ! videorate ! video/x-raw,width=640,height=480  ! videoconvert ! video/x-raw,format=RGB8`
