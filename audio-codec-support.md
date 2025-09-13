---
title: Audio Codec Support
description: 
published: true
date: 2025-09-13T11:28:04.181Z
tags: 
editor: markdown
dateCreated: 2025-09-13T11:28:04.181Z
---

# Audio Codec Support

AVR Core provides comprehensive support for multiple audio codecs commonly used in VoIP telephony systems. The system automatically detects and handles different codec formats to ensure seamless integration with various PBX systems and telephony infrastructure.

## Supported Audio Codecs

### μ-law (ulaw)
- **Standard**: ITU-T G.711 μ-law
- **Usage**: Primary telephony codec in North America and Japan
- **Bit Rate**: 64 kbps
- **Sample Rate**: 8 kHz
- **Compression**: Logarithmic compression algorithm

### A-law (alaw)
- **Standard**: ITU-T G.711 A-law
- **Usage**: Primary telephony codec in Europe and most international systems
- **Bit Rate**: 64 kbps
- **Sample Rate**: 8 kHz
- **Compression**: Logarithmic compression algorithm

### Linear PCM (slin)
- **Standard**: Uncompressed audio format
- **Usage**: High-quality audio transmission, often used in modern VoIP systems
- **Bit Rate**: 128 kbps (16-bit) or 64 kbps (8-bit)
- **Sample Rate**: 8 kHz (standard telephony)
- **Compression**: None (uncompressed)

## Automatic Codec Detection

AVR Core implements intelligent codec detection that automatically identifies the incoming audio format without requiring manual configuration.

### Detection Algorithm

The system uses a statistical analysis approach to determine the codec:

1. **Audio Sample Analysis**: Incoming audio packets are analyzed for characteristic patterns
2. **RMS Calculation**: Root Mean Square values are calculated for both μ-law and A-law decoded samples
3. **Comparison Logic**: The codec with higher RMS value is selected as the detected format
4. **Logging**: The detected codec is logged for monitoring and debugging purposes

## Audio Processing Pipeline

### Incoming Audio Processing

1. **Packet Reception**: Audio packets are received from Asterisk via AudioSocket
2. **Codec Detection**: Automatic detection occurs on the first few audio packets
3. **Decoding**: Audio is decoded from the detected codec to linear PCM
4. **Processing**: All internal processing uses 8kHz, 16-bit PCM format
5. **Forwarding**: Processed audio is sent to ASR or STS services

### Outgoing Audio Processing

1. **Audio Generation**: TTS or STS services generate audio in PCM format
2. **Chunk Normalization**: Audio is normalized into 320-byte chunks
3. **Encoding**: Audio is encoded back to the appropriate codec format
4. **Transmission**: Encoded audio is sent back to Asterisk

## Technical Specifications

### Internal Audio Format
- **Sample Rate**: 8 kHz
- **Bit Depth**: 16-bit
- **Channels**: Mono
- **Format**: Linear PCM
- **Chunk Size**: 320 bytes (20ms at 8kHz)

### Codec Conversion

#### μ-law to PCM
```javascript
case 'ulaw':
    audioData = alawmulaw.mulaw.decode(audioData);
    break;
```

#### A-law to PCM
```javascript
case 'alaw':
    audioData = alawmulaw.alaw.decode(audioData);
    break;
```

## Configuration

### Environment Variables

No specific configuration is required for codec support as it's handled automatically. However, you can monitor codec detection through the logs:

```
Audio codec detected: ALAW
or
Audio codec detected: ULAW
or
Audio codec detected: SLIN
```

## Integration Examples

### Asterisk Configuration

Ensure your Asterisk configuration uses supported codecs:

```ini
; pjsip.conf
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060

[endpoint_template](!)
type=endpoint
context=default
disallow=all
allow=ulaw,alaw,slin16
```

### Codec Priority

For optimal performance, configure codec priority in your PBX:

1. **Primary**: Linear PCM (slin16) - Best quality, no conversion needed
2. **Secondary**: A-law (alaw) - Good quality, efficient compression
3. **Fallback**: μ-law (ulaw) - Universal compatibility

## Performance Considerations

### CPU Usage
- **Linear PCM**: Lowest CPU usage (no conversion)
- **A-law/μ-law**: Moderate CPU usage (decoding required)
- **Mixed Codecs**: Higher CPU usage (frequent conversions)

### Memory Usage
- **Buffer Size**: 320 bytes per audio chunk
- **Queue Management**: Automatic chunk normalization
- **Garbage Collection**: Efficient buffer management

### Network Bandwidth
- **μ-law/A-law**: 64 kbps per call
- **Linear PCM**: 128 kbps per call
- **Compression**: Built-in codec compression reduces bandwidth

## Best Practices

1. **Consistent Codec Usage**: Use the same codec throughout the call session
2. **Quality vs. Bandwidth**: Balance audio quality with network resources
3. **Monitoring**: Regularly check codec detection logs
4. **Testing**: Test with different codec configurations
5. **Fallback**: Configure multiple codec options for compatibility

## Future Enhancements

Planned improvements for audio codec support:

- **Additional Codecs**: Support for G.722, G.729, and other modern codecs
- **Adaptive Quality**: Dynamic codec selection based on network conditions
- **Enhanced Detection**: Machine learning-based codec detection
- **Performance Optimization**: Hardware-accelerated codec conversion

