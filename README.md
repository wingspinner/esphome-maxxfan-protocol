# esphome-maxxfan-protocol

Maxxfan infrared remote control protocol component for ESPHome.

This component was originally developed for [Minuet](https://github.com/j9brown/minuet), a smart brushless DC motor controller for the [MAXXAIR Maxxfan](https://www.maxxair.com/products/fans/maxxfan-deluxe/).  You can also use it for other projects.

## Configuration

### Including the component

Add this stanza to your ESPHome configuration to pull the `maxxfan_protocol` component from github.

```yaml
# Import the maxxfan protocol component from Github.
external_components:
  - source: github://j9brown/esphome-maxxfan-protocol@main
    components: [ maxxfan_protocol ]
```

### Receiving Maxxfan remote control messages

Here's what you need to add to your ESPHome configuration to receive Maxxfan messages, assuming you have connected a 38 KHz infrared receiver to pin 2.

The `on_maxxfan` handler receives the [MaxxfanData](components/maxxfan_protocol/maxxfan_protocol.h) object that was parsed from the received message.

```yaml
# Enable the Maxxfan protocol.
maxxfan_protocol:

# Configure the IR receiver.
remote_receiver:
  - id: minuet_ir
    pin:
      number: 2  # Change this to the pin your IR receiver is attached to
      mode: input
      inverted: true
    rmt_channel: 2
    memory_blocks: 2
    dump: maxxfan
    on_maxxfan:
      then:
        lambda: |-
          // The variable 'x' holds a MaxxfanData object parsed from the received message
          ESP_LOGD("maxxfan-example", "Fan state: %s", x.fan_on ? "on" : "off");
```

### Transmitting Maxxfan remote control messages

*Not implemented yet.  Please send the author a message or a pull request if you would like this feature.  It's pretty straightforward.*

### Example code

See also [maxxfan-example.yaml](maxxfan-example.yaml) as a starting point for your configuration.

## Protocol details

The Maxxfan IR remote protocol looks like RS232 with 1 start bit (zero), 8 data bits encoded least-significant-bit first, no parity bits, and 2 stop bits (ones). A mark symbol encodes a zero bit and a space symbol encodes a one bit.  Each bit period is 800 us.  The transmission ends with a series of space symbols.

The packet is 16 bytes long and consists of a fixed preamble followed by some control fields and a checksum.

Here's an example of a packet and its decoding:

| Field      | Encoded bits    | Decoded bits | Meaning                                             |
| ---------- | --------------- | ------------ | -------------------------------------------------   |
| preamble   | `0 01011010 11` | `01011010`   | 0x5A bit pattern                                    |
| preamble   | `0 10100101 11` | `10100101`   | 0xA5 bit pattern (0x5A inverted)                    |
| preamble   | `0 00000001 11` | `10000000`   | 0x80 bit pattern                                    |
| preamble   | `0 11111110 11` | `01111111`   | 0x7F bit pattern (0x80 inverted)                    |
| preamble   | `0 00000010 11` | `01000000`   | 0x40 bit pattern                                    |
| preamble   | `0 11111101 11` | `10111111`   | 0xBF bit pattern (0x40 inverted)                    |
| preamble   | `0 00000100 11` | `00100000`   | 0x20 bit pattern                                    |
| preamble   | `0 11111011 11` | `11011111`   | 0xDF bit pattern (0x20 inverted)                    |
| preamble   | `0 00001000 11` | `00010000`   | 0x10 bit pattern                                    |
| preamble   | `0 00110011 11` | `11001100`   | 0xCC bit pattern                                    |
| state      | `0 00100100 11` | `00100100`   | state: fan off, fan exhaust, cover close, warn      |
| speed      | `0 00100110 11` | `01100100`   | speed percent: 100%                                 |
| auto temp  | `0 00100010 11` | `01000100`   | thermostat temperature setpoint in Fahrenheit: 68 F |
| ???        | `0 11111111 11` | `11111111`   | unknown purpose, always 0xff                        |
| ???        | `0 11000100 11` | `00100011`   | unknown purpose, always 0x23                        |
| checksum   | `0 00011011 11` | `11011000`   | XOR of previous 5 bytes                             |
| end        | `11111111`      |              | end of transmission                                 |

The *speed* field sets fan speed percentage as a multiple of 10 between 0 and 100 percent inclusively.

The *auto temp* field sets the auto mode thermostat temperature setpoint in the range from 29 to 99 Fahrenheit inclusively.

The *state* field holds a combination of flags as follows (numbered from LSB):

| Bit | Flag            | State | Meaning                                                                                                          |
| --- | --------------- | ----- | ---------------------------------------------------------------------------------------------------------------- |
| 0   | fan state       | `0`   | Fan off                                                                                                          |
|     |                 | `1`   | Fan on                                                                                                           |
| 1   | cover override? | `0`   | Cover is controlled by the fan state as usual, open when fan on, closed otherwise                                |
|     |                 | `1`   | Either auto mode (thermostat controlled) or ceiling fan mode (fan on with cover closed) is active                |
| 2   | fan direction   | `0`   | Intake                                                                                                           |
|     |                 | `1`   | Exhaust                                                                                                          |
| 3   | cover state     | `0`   | Cover closed                                                                                                     |
|     |                 | `1`   | Cover open                                                                                                       |
| 4   | mode            | `0`   | Manual mode                                                                                                      |
|     |                 | `1`   | Auto mode (thermostat controlled)                                                                                |
| 5   | warn            | `0`   | No warning                                                                                                       |
|     |                 | `1`   | Warn (beep twice) because the user attempted to raise or lower the speed or temperature beyond the allowed range |
| 6   | ???             | `0`   | Unknown purpose, always 0                                                                                        |
| 7   | ???             | `0`   | Unknown purpose, always 0                                                                                        |

Note: The *cover override* flag could perhaps be defined differently than presumed above.  Perhaps it just means "a special mode is active"?

## Acknowledgements

Thanks to [skypeachblue](https://github.com/skypeachblue) and [wingspinner](https://github.com/wingspinner) for publishing information about their [reverse engineering](https://github.com/skypeachblue/maxxfan-reversing) of the Maxxfan IR remote control protocol.  It helped me create this component.
