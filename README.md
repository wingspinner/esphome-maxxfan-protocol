# esphome-maxxfan-protocol

Maxxfan infrared remote control protocol component for ESPHome.

This component was originally developed for [Minuet](https://github.com/j9brown/minuet), a smart brushless DC motor controller for the [MAXXAIR Maxxfan](https://www.maxxair.com/products/fans/maxxfan-deluxe/).  You can also use it for other projects.

## Configuration

### Including the component

Add this stanza to your ESPHome configuration to pull the `maxxfan_protocol` component from github.

```yaml
external_components:
  - source: github://j9brown/esphome-maxxfan-protocol@main
    components: [ maxxfan_protocol ]
```

### Receiving Maxxfan remote control messages

Here's what you need to add to your ESPHome configuration to receive Maxxfan messages, assuming you have connected a 38 KHz infrared receiver to pin 2.

The `on_maxxfan` handler receives the [MaxxfanData](components/maxxfan_protocol/maxxfan_protocol.h) object that was parsed from the received message.

```yaml
remote_receiver:
  - id: minuet_ir
    pin:
      number: 2
      mode: input
      inverted: true
    rmt_channel: 2
    memory_blocks: 2
    dump: maxxfan
    on_maxxfan:
      then:
        lambda: |-
          // The variable 'x' holds a MaxxfanData object parsed from the received message
```

### Transmitting Maxxfan remote control messages

*Not implemented yet.  Please send the author a message or a pull request if you would like this feature.  It's pretty straightforward.*

## Acknowledgements

Thanks to [skypeachblue](https://github.com/skypeachblue/maxxfan-reversing) and [wingspinner](https://github.com/wingspinner) for all their work in successfully reverse engineering the Maxxfan IR remote control protocol which I leveraged and expanded apon to develop this code.
