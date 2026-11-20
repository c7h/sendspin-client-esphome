# ESPHome Sendspin caster

ESPHome configuration for the development ESP32 Sendspin player with an ES8388 codec.

The repository does not identify a commercial board model. The physical board is therefore **unverified/not named**. The active ESP-IDF target is the original `esp32` chip with quad PSRAM; this project deliberately does not select a named PlatformIO board or an ESP32-S3.

## Extracted hardware definition

The values below come from the active generated `sendspin_client/sdkconfig`. The source defaults are called out where they differ.

|Function|GPIO / value|Evidence and meaning|
|---|---:|---|
|I²C controller|port 0|ESP-IDF I²C master bus used for codec control|
|ES8388 I²C SDA|23|Codec control bus|
|ES8388 I²C SCL|22|Codec control bus|
|ES8388 I²C frequency|100 kHz|Codec control bus speed|
|ES8388 7-bit address|`0x10`|Active configuration; ES8388 CE low; driver probes `0x11` only as fallback|
|I²S controller|port 0|ESP-IDF playback channel|
|I²S MCLK|0|ESP32-generated master clock to ES8388|
|I²S BCLK|27|Bit clock to ES8388|
|I²S WS/LRCK|25|Word-select/frame clock to ES8388|
|I²S DOUT → ES8388 DIN|2|Playback sample data|
|I²S DIN|unused|Playback-only; no ADC/microphone path is configured|
|I²S clock source|default PLL|`SENDSPIN_I2S_USE_APLL` is disabled in active `sdkconfig`|
|WS2812 status LED data|15|Active `sdkconfig` value; do not use Kconfig’s unused default of GPIO21|
|IR transmitter|4|Existing ESP-IDF stereo power-control output; not part of the initial ESPHome audio path|

The active build target is `esp32`, PSRAM is enabled in quad mode, and the codec is an ES8388. No SD-card GPIO assignment is active: SD recording and benchmark options are disabled in the active build, so their conditional GPIO defaults do not represent connected hardware.

## Source evidence

All paths below are relative to this project directory and point into the source repository:

- [`../sendspin_client/sdkconfig`](../sendspin_client/sdkconfig): active target, quad-PSRAM settings, Wi-Fi-independent board values, status LED GPIO15, and IR GPIO4. Wi-Fi values in this file are intentionally not copied here.
- [`../sendspin_client/main/Kconfig.projbuild`](../sendspin_client/main/Kconfig.projbuild): board declarations and defaults, including the source status LED default GPIO21, ES8388 address alternatives, and conditional SD-card settings.
- [`../sendspin_client/main/es8388.c`](../sendspin_client/main/es8388.c): ES8388 address fallback, I²C bus setup, and playback-only I²S GPIO roles.
- [`../sendspin_client/main/app_config.h`](../sendspin_client/main/app_config.h): application mapping of Kconfig values to ES8388/I²S semantics, codec capabilities, 48 kHz stereo output, and the 512 KiB FLAC ring buffer.
- [`../sendspin_client/README.md`](../sendspin_client/README.md): typical ES8388 wiring table and the documented source LED default. The active generated configuration overrides that LED default with GPIO15.

## ESPHome deployment

Copy `secrets.yaml.example` to `secrets.yaml` and replace its placeholders with deployment-specific credentials. Never commit `secrets.yaml`.

Validate with:

```sh
esphome config esphome-sendspin-caster.yaml
esphome compile esphome-sendspin-caster.yaml
```

The configuration intentionally documents and validates the static ESPHome mapping only. Unless a physical board is connected and boot logs are inspected, no playback or LED output claim is made.
