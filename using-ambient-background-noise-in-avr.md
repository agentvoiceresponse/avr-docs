---
title: Using Ambient Background Noise in AVR
description: AVR Core now supports the ability to add ambient background sounds to calls.   This feature allows you to simulate real-world environments (e.g., office, café, nature) or introduce controlled background noise during testing.
published: true
date: 2025-08-25T17:25:31.161Z
tags: avr, background, noice, ambient
editor: markdown
dateCreated: 2025-08-25T17:25:26.461Z
---

# Using Ambient Background Noise in AVR

AVR Core now supports the ability to add **ambient background sounds** to calls.  
This feature allows you to simulate real-world environments (e.g., office, café, nature) or introduce controlled background noise during testing.

## Ambient Sounds Directory

AVR includes a default set of pre-recorded sound files inside the `ambient_sounds` directory.  
These files are already present if you clone [avr-infra](https://github.com/agentvoiceresponse/avr-infra) locally.

You can also provide your own files, as long as they follow the format:

- **Sample Rate**: 8kHz  
- **Bit Depth**: 16-bit  
- **Channels**: Mono  
- **Format**: Raw PCM (no headers)

## Example Ambient Sounds

- `office_background.raw` – Subtle office environment sounds  

👉 You can find example files here: [ambient_sounds/](https://github.com/agentvoiceresponse/avr-infra/tree/main/ambient_sounds)

## Creating Custom Ambient Sounds

You can generate your own files using [SoX](http://sox.sourceforge.net/):

```bash
sox input_file.wav -r 8000 -c 1 -b 16 -e signed output_file.raw
```
## Docker Compose Example

Below is an example configuration with ambient noise enabled:

```yaml
avr-core:
  image: agentvoiceresponse/avr-core
  platform: linux/x86_64
  container_name: avr-core
  restart: always
  environment:
    - PORT=5001 
    ...
    - AMBIENT_NOISE_FILE=ambient_sounds/office_background.raw
    - AMBIENT_NOISE_LEVEL=0.90
  volumes:
    - ./ambient_sounds:/usr/src/app/ambient_sounds
  ports:
    - 5001:5001
  networks:
    - avr
```

## Configuration Options

| Variable             | Description                                                         | Example Value                           |
|----------------------|---------------------------------------------------------------------|-----------------------------------------|
| AMBIENT_NOISE_FILE   | Path to the ambient sound file (inside container)                   | `ambient_sounds/office_background.raw`  |
| AMBIENT_NOISE_LEVEL  | Volume level for the background noise (0.0 = mute, 1.0 = 100% volume) | `0.10` (10%), `0.50` (50%), `0.90` (default 90%) |


## Notes

- Ensure you mount the directory containing your ambient sounds into the container using the volumes directive:

```yaml
volumes:
  - ./ambient_sounds:/usr/src/app/ambient_sounds
```

- Update AMBIENT_NOISE_FILE to point to the correct file inside the container.
- Adjust AMBIENT_NOISE_LEVEL to suit your preference.

With this feature, AVR can simulate more realistic environments and make your tests or demos closer to real-world conditions.