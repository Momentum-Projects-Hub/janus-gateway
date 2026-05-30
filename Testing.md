Do these in order, each takes 2 minutes:

1. Video Call — tests peer-to-peer routing through Janus

http://localhost:8000/demos/videocall.html
Open in 2 tabs. Register as alice in one, bob in the other. Call from alice to bob.

2. Video Room — tests the SFU (multi-party)

http://localhost:8000/demos/videoroom.html
Open in 3 tabs. Join room 1234 in all three. You should see all three video feeds simultaneously.

3. Audio Bridge — tests server-side audio mixing

http://localhost:8000/demos/audiobridge.html
Open in 2 tabs. Join room 1234 in both. Speak in one tab — you should hear it in the other.

4. Screen Sharing — tests the screenshare flow

http://localhost:8000/demos/screensharing.html
Open in 2 tabs. Share your screen in one, watch it in the other.

5. Record & Play — tests recording pipeline

http://localhost:8000/demos/recordplay.html
Record a short clip, then play it back. This confirms MJR recording works.

Once those all pass, your Janus install is fully functional. At that point the only thing left untested is the SIP plugin — for that, the fastest path is a free account at linphone.org/freesip, no server setup needed, just credentials you plug into the SIP demo.