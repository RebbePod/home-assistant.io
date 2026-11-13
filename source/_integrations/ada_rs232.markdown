---
title: ADA RS-232
description: Instructions on integrating Audio Design Associates PTM-1645 amplifiers over RS-232 with Home Assistant.
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

The **ADA RS-232** {% term integration %} connects Home Assistant to Audio Design Associates PTM-1645 power amplifiers over their RS-232 serial connection. You can turn each stereo zone on or off using local control. Amplifiers in a daisy chain share one serial connection.

## Supported devices

- Audio Design Associates PTM-1645 with the PAC amplifier controller: eight stereo zones

The PTM-1645 zone controls and automatic address scan have been tested with an ESPHome serial proxy.

## Prerequisites

{% include integrations/serial_connected.md %}

1. Set each amplifier to its normal operating mode, with the default one-to-one trigger mapping.
2. If you connect multiple amplifiers in a daisy chain, give each one a different PAC address from 1 to 9 before connecting the chain. The integration reads existing addresses; it does not change the address stored in an amplifier.
3. If you use a raw TCP serial server, configure its serial port for 19200 baud, 8 data bits, no parity, 1 stop bit, and no flow control.

For an ESPHome connection, use an [ESPHome serial proxy](/integrations/serial/#setting-up-an-esphome-serial-proxy). The proxy's UART must be dedicated to this integration; remove any previous amplifier polling scripts and template switches from that UART.

{% include integrations/config_flow.md %}

Select your serial connection. Setup scans for responding amplifiers without changing their settings or zone power states. If one available amplifier responds, it is selected automatically. If several respond, select one from the discovered list. Addresses that already have an integration entry are excluded from the list.

{% configuration_basic %}
Port:
  description: The local serial port or ESPHome serial proxy connected to your amplifier. You can also enter a network serial server URL manually.
Amplifier:
  description: The existing PAC address of the amplifier to add. This field is shown only when several available amplifiers respond to the scan.
{% endconfiguration_basic %}

### Adding daisy-chained amplifiers

You can connect up to nine uniquely addressed amplifiers to one serial connection. Connect the RS-232 output of one amplifier to the RS-232 input of the next.

To add another amplifier in the chain, go to {% my integrations title="**Settings** > **Devices & services**" %}, select **ADA RS-232**, and select **Add hub**. Select the same serial connection. Setup scans the chain again and offers addresses that are not already configured. Each amplifier gets its own device and zone switches, while all entries on that port share one serial connection.

## Supported functionality

### Switches

- **Zone 1** through **Zone 8**
  - **Description**: Controls power to a stereo output pair and reports the output state received from the amplifier.

Each amplifier's switches belong to one device.

You can turn a zone on or off from its switch entity or use the standard [switch actions](/integrations/switch/). Hardware-trigger changes update these same switches when feedback arrives or at the next poll. The PAC controller does not provide serial volume, mute, source selection, or playback controls.

## Data updates

The integration {% term polling polls %} each configured amplifier every 60 seconds and updates its switches when feedback arrives. After a zone command, it reads the output states again before reporting the result. Home Assistant reads the existing state when it starts; it does not restore or send saved zone states.

If communication fails, the affected switches become unavailable. A shared-connection failure can affect all amplifiers on that connection. The next scheduled poll attempts to reconnect.

## Known limitations

- Hardware voltage triggers and the amplifier's bypass control can override serial commands. A zone can remain on after you turn its switch off if a hardware trigger keeps that output active.
- Feedback reports output states without identifying whether a hardware trigger or a serial command enabled them. Preserving an externally enabled output in a later command can also keep it enabled through serial control.
- Independent zone switches require default one-to-one trigger mappings. Custom trigger mappings are not supported.
- Hardware configuration, including stored addresses, trigger mappings, and Quick Turn On, is not changed by the integration.
- Operation with multiple physical amplifiers in a daisy chain has not been verified on hardware.

## Removing the integration

{% include integrations/remove_device_service.md %}

Removing an amplifier entry leaves its zone power states unchanged. The shared serial connection closes when the last entry using it is removed. No additional amplifier cleanup is required.
