# Mi Gateway Radio

This custom component for HomeAssistant provides support for controlling radio on **Mi Smarthome Gateway 2** *(DGNWG02LM)* via HomeAssistant's `MediaPlayerEntity`.

This component targets a single use case: *playing custom HLS playlists on a gateway*.  

> **⚠️ ATTENTION ⚠️** \
You **MUST** provide URL(s) for compatible HLS radio stream(s) for this custom component. See [HLS radio streams](#hls-radio-streams) section to learn more.  

## Background

Even though **Mi Smarthome Gateway 2** is long out-of-production, there are still some devices out in the wild. Some people (myself included) still keep this old devices exclusively for its radio function. I moved all of my zigbee child devices on more recent bridges and added it to HomeAssistant via supported integrations. 

This component provides support exclusively for the radio function.

All of the custom components [out there](#references) are either outdated and no longer work in modern HomeAssistant versions, or target other use-cases (e.g. playing built-in radio stations). This custom component aims to bring life to this old devices in modern HomeAssistant installations.

## Configuration 

```yaml
media_player:
  - platform: mi_gateway_radio
    name: <your player name>
    host: <gateway ip>
    token: <gateway token>
```

## HLS radio streams

**Mi Smarthome Gateway 2** *(DGNWG02LM)* has a built-in radio function. It does work in recent **Mi Home** app but is limited to the predefined list of radio stations (mostly Chinese). This radio stations are no more than a standard [HLS streams](https://en.wikipedia.org/wiki/HTTP_Live_Streaming).

Luckily the gateway has a somewhat *secret* command for playing a custom HLS stream - `play_specify_fm`.

> The only catch is that the gateway expects quite a particular HLS-stream characteristics: `ADTS`-stream with `7sec` `AAC`-chunks, `44100` bitrate, with or without `SBR` and **NO** `IDv3` metadata.   

The HLS-stream of your choice (e.g. you favourite internet radio station) **MAY** turn out to be compatible, but, frankly, the chances are quite odd.

In the recent past there was a publicly available [internet resource](https://ximiraga.ru) with compatible HLS-streams, but it is also long-dead.

So nowadays there are 2 alternatives for acquiring compatible HLS-stream URLs for this custom component: 
 - either using built-in radio stations via it's public internet URLs;
 - or self-hosting your own radio stream.

### Built-in station URLs

As far as I know all of the built-in radio stations are hosted at [http://live.xmcdn.com/live/](http://live.xmcdn.com/live/). You can [search for this prefix on Github](https://github.com/search?q=http%3A%2F%2Flive.xmcdn.com%2Flive%2F&type=code) and get some URLs to try.

As of myself I used [http://live.xmcdn.com/live/764/64.m3u8](http://live.xmcdn.com/live/764/64.m3u8) for testing and can confirm that it is live and working.

### Self-hosting 

Self-hosting a radio station may appear intimidating at first glance, but in fact it is not a big challenge. All it requires is serving a `.m3u8` file and `.aac` chunk files referenced from it. 

It can even be done statically (but mostly for testing) - all you need is a good old `Apache` or other HTTP server of your choice. The main downside of static serving is that every time you would start playing - it will start from the first chunk.

Dynamic serving is a way better alternative, but requires some work to be done. The most common solution is to use `ffmpeg` to perform chunking and some HTTP-server to serve it. 

There is a number of solutions for dynamic serving of compatible HLS streams specifically targeted on Mi Smarthome Gateway 2:

- [https://github.com/LennyLip/xiaomi-gateway-radio-stream-home-assistant](https://github.com/LennyLip/xiaomi-gateway-radio-stream-home-assistant)

- [https://habr.com/ru/articles/411003/](https://habr.com/ru/articles/411003/)

- [https://github.com/ashdkv/miwifiradio](https://github.com/ashdkv/miwifiradio)

All of it seemed too complex for me to manage audio tracks after the setup, so I ended up using [AzuraCast](https://www.azuracast.com/) in Docker on my Synology. 

This may seem to to be an overkill, but I quite like the simplicity of managing tracks via the Web-interface and I also use *AzuraCast* streams for my HomePods.

If you decide to walk down this road with *AzuraCast* - keep in mind that the last version without `IDv3` metadata is `0.18.5` which is quite outdated. Recent versions of *AzuraCast* embed `IDv3` metadata into `.aac` chunks which breaks compatibility with Mi Smarthome Gateway 2.

You would have to use a custom **Liquidsoap** config in *AzuraCast* to make the HLS-stream compatible:

```
jazz = %ffmpeg(format="adts",
    %audio(
        codec="aac",
        channels=2,
        ar=44100,
        b="64k" 
    )
) 

hls_streams = [("jazz", jazz)]
hls_streams_info = [("jazz",{ bandwidth=40000, codecs="mp4a.40.60", extname="aac", video_size = null() })]
                                                                                                  
def hls_segment_name(~position,~extname,stream_name) = 
    timestamp = int_of_float(time()) 
    duration = 4 
    "#{stream_name}_#{duration}_#{timestamp}_#{position}.#{extname}" 
end

output.file.hls(playlist="live.m3u8", 
    segment_duration=7.0, 
    segments=5,
    segments_overhead=2, 
    segment_name=hls_segment_name, 
    streams_info=hls_streams_info,
    prefix="http://192.168.1.150:7080/hls/classic_jazz/",
    persist_at="/var/azuracast/stations/jazz/config/hls.config", 
    "/var/azuracast/stations/classic_jazz/hls",
    hls_streams,  
    radio 
)
```

> **⚠️ ATTENTION ⚠️** \
Don't forget to enable **advanced** configuration otions in your AzuraCast installation. Otherwise your custom **Liquidsoap** config will not be applied.  


## References

Special thanks to all who contributed in making this custom component possible:

[https://github.com/h4v1nfun/xiaomi_miio_gateway/](https://github.com/h4v1nfun/xiaomi_miio_gateway)
[https://github.com/fanthos/xiaomigateway](https://github.com/fanthos/xiaomigateway)
[https://github.com/igzero/xiaomigateway](https://github.com/igzero/xiaomigateway)
[https://github.com/yunsean/xiaomi_miio_radio](https://github.com/yunsean/xiaomi_miio_radio)
[https://github.com/gheesung/xiaomigateway](https://github.com/gheesung/xiaomigateway)
[https://github.com/shaonianzhentan/xiaomi_radio](https://github.com/shaonianzhentan/xiaomi_radio)

> **💡 TIP 💡** \
If you are interested in modifying this component to suit your needs you can find other contributions by [simply searching](https://github.com/search?q=play_specify_fm+language%3APython+&type=code) for `play_specify_fm` on Github. This search term refers to a *MIIO* command name and provides decent search results.  
