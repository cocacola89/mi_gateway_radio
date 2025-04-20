# Mi Gateway Radio

This custom component for HomeAssistant provides support for controlling radio on **Mi Smarthome Gateway 2** *(DGNWG02LM)* via HomeAssistant's `MediaPlayerEntity`.

This component targets a single use case: *playing custom HLS playlists on a gateway*.  

## Configuration 

```yaml
media_player:
  - platform: mi_gateway_radio
    name: <your player name>
    host: <gateway ip>
    token: <gateway token>
```