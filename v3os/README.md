# V3_OS deployment notes

Snapshot of what runs on the two V3_OS nodes. The original alpha/ and bravo/ folders are unchanged.

- Service units (identical on both nodes, md5-verified) go in /etc/systemd/system/. Scripts go in /opt/nucleus/vlora/ and run under /opt/nucleus/venv.
- Python deps: numpy, pycodec2, soxr, opuslib (opuslib and soxr were pip-installed into the venv for this deployment).
- v3os/alpha/ matches the repo's alpha/ folder. v3os/bravo/ matches bravo/ except vlora_rx_bridge.py, which runs SILENCE_TIMEOUT = 1.0 instead of 0.25. That change was made outside this repo and its effect on audio quality has not been tested in isolation.
- Verified: Alpha runs vlora_rx_bridge.py SILENCE_TIMEOUT = 0.25, Bravo runs 1.0, so the two nodes were not symmetric.
- config.yaml: voice.stream_enabled: true, voice.stream_portnum: 256.
- cot_bridge_sender_prefix.patch: V3_OS's nucleusd cot_bridge.py prepends a 4-byte sender ID to the raw Codec2 forwarded to UDP 4244. That shifts every 8-byte Codec2 frame boundary and corrupts decoding. The patch removes it for the stream path only; the voice-text path is untouched. Apply from the V3_OS repo root with `git apply`.
- Measured on SHORT_FAST (Alpha to Bravo): roughly 17-30% packet loss at 72B chunks. 224B chunks lowered measured loss to about 10% but sounded significantly worse, because each lost packet drops 560ms instead of 180ms. Not recommended.
- After stopping cot-bridge for debugging, start vlora-tx-bridge and vlora-rx-bridge by hand.
