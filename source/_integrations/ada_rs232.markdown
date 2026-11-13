---
title: ADA RS-232
description: Instructions on integrating the Audio Design Associates PTM-1645 amplifier over RS-232 with Home Assistant.
ha_release: "2026.12"
ha_categories:
  - media_entertainment.speakers_audio
ha_iot_class: Local Polling
ha_config_flow: true
ha_codeowners:
  - "@RebbePod"
ha_domain: ada_rs232
ha_integration_type: hub
ha_platforms:
  - switch
ha_quality_scale: bronze
---

The **ADA RS-232** {% term integration %} connects Home Assistant to the Audio Design Associates PTM-1645 power amplifier over its RS-232 serial port. You can turn each of its eight stereo zones on or off using local control.

## Supported devices

- Audio Design Associates PTM-1645 with the PAC amplifier controller

## Prerequisites

{% include integrations/serial_connected.md %}

1. Set the amplifier to its normal operating mode, with the default one-to-one trigger mapping.
2. Find the amplifier's existing PAC address, from 1 to 9.
3. If you use a raw TCP serial server, configure its serial port for 19200 baud, 8 data bits, no parity, 1 stop bit, and no flow control.

For an ESPHome connection, use an [ESPHome serial proxy](/integrations/serial/#setting-up-an-esphome-serial-proxy). The proxy's UART must be dedicated to this integration; remove any previous amplifier polling scripts and template switches from that UART.

{% include integrations/config_flow.md %}

{% configuration_basic %}
Port:
  description: The local serial port or ESPHome serial proxy connected to your amplifier. You can also enter a network serial server URL manually.
Amplifier address:
  description: The amplifier's existing PAC address, from 1 to 9. The default is 1. This selects the amplifier to control; it does not change the address stored in the amplifier.
{% endconfiguration_basic %}

## Supported functionality

### Switches

- **Zone 1** through **Zone 8**
  - **Description**: Controls power to a stereo output pair and reports the output state received from the amplifier. All eight switches belong to one amplifier device.

You can turn a zone on or off from its switch entity or use the standard [switch actions](/integrations/switch/). The PTM-1645 does not provide serial volume, mute, source selection, or playback controls through its PAC controller.

## Data updates

The integration {% term polling polls %} the amplifier every 60 seconds and updates the switches when feedback arrives. After a zone command, it reads the output states again before reporting the result. Home Assistant reads the existing state when it starts; it does not restore or send saved zone states.

If communication fails, the switches become unavailable. The next scheduled poll attempts to reconnect.

## Known limitations

- Hardware voltage triggers and the amplifier's bypass control can override serial commands. A zone can remain on after you turn its switch off if a hardware trigger keeps that output active.
- Feedback reports output states without identifying whether a hardware trigger or a serial command enabled them. Preserving an externally enabled output in a later command can also keep it enabled through serial control.
- Custom trigger mappings and daisy-chained amplifiers are not supported. Use one amplifier per serial connection.

## Removing the integration

{% include integrations/remove_device_service.md %}

Removing the integration closes the serial connection and leaves the amplifier's zone power states unchanged. No additional amplifier cleanup is required.
