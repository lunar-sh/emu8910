# emu8910

```
   ______  _____  _____ ___ ______ 
  / __/  |/  / / / ( _ ) _ <  / _ \
 / _// /|_/ / /_/ / _  \_, / / // /
/___/_/  /_/\____/\___/___/_/\___/ 
                                   
```
This repository contains a modular, single file `Typescript` implementation of General Instrument's [`AY-3-8910`](https://en.wikipedia.org/wiki/General_Instrument_AY-3-8910) `PSG` (programmable sound generator) in
less than `1K` lines and without any magic/wierd constants!

The emulator allows for fine tuning of the `DAC` look-up table though modification of the following constants:
```
const DAC_DECAY = 1.3;
const DAC_SHIFT = 40;
```

It implements most of the `PSG's` original registers. <br>

The offical datasheet for the `PSG` can be found [`here`](http://map.grauw.nl/resources/sound/generalinstrument_ay-3-8910.pdf).

> **Online Player/Emulator by DrSnuggles** <br>
> [`player link`](https://files.lunar.sh/apps/AYSir/?engine=lunar) 

Sound output is achieved in the browser through an `AudioContext()` hook. <br>
This `emulator` also adds interrupt support (with a user defined frequency) for updating the `PSG's` registers.

> Note: `AY8910` emulators naturally generate high frequency content which must be suppressed. This is achieved with a `FIR` `low-pass` filter using a procedurally generated `Blackman-Harris` window.

The `FIR` response can be changed through modification of the following constants:

```
const FIR_CUTOFF = 2500; // Hz
const FIR_TAPS = 100; // N taps
```

> Note: `FIR` LPF (Low-Pass Filter) data is generated procedurally.

Compile with `tsc emu8910.ts`.

To use simply create a `PSG49` object as follows:
```
var emu8910 = new PSG49(YM_CLOCK_ZX, 50);
```
Which sets the default clock speed (`1.75 MHz`) and interrupt frequency (`50 Hz`). 

This exposes a `PSG` `register` file in the `emu8910.register` object:
```
emu8910.register.A_FINE
emu8910.register.A_COARSE
emu8910.register.B_FINE
emu8910.register.B_COARSE
emu8910.register.C_FINE
emu8910.register.C_COARSE
emu8910.register.NOISE_PERIOD
emu8910.register.MIXER
emu8910.register.A_VOL
emu8910.register.B_VOL
emu8910.register.C_VOL
emu8910.register.ENV_FINE
emu8910.register.ENV_COARSE
emu8910.register.ENV_SHAPE
```

The `register` file is then used to control the `PSG` or extract state information.

To play a `FYM` module:
```
song = new FYMReader(<BUFFER>);
emu8910.interrupt.routine = <ISR_FUNC>
emu8910.clock.frequency = song.getClockRate()
emu8910.interrupt.frequency = song.getFrameRate()
```

> Note: You can download modules from [https://ftp.modland.com/pub/modules/](https://ftp.modland.com/pub/modules/)

This snippet sets the `ISR` (Interrupt Service Routine) function which read/writes to the `PSG` register file and sets 
the `clock` and `interrupt` frequency.

To stop playback:
```
emu8910.driver.device.suspend()
emu8910.interrupt.frequency = 0
```
To resume:
```
emu8910.driver.device.resume()
emu8910.interrupt.frequency = song.getFrameRate()
```

> Note: You can access the emulator's internal `register` file with `emu8910.register`.

These registers need to be updated at the frequency of the `ISR`.

Files:

* `emu8910.js` - precompiled js emulator core
* `src/emu8910.ts` - `core` emulator implementation
* `fym.js` - `FYM` `(Fast YM)` format parser
* `parser.js` - `PSG` register parser
* `index.html` - `HTML` boilerplate

To run demo clone and start web server from the root directory: `python3 -m http.server 8000` and navigate to [http://127.0.0.1:8000](http://127.0.0.1:8000).

> Note: Click anywhere on the page to start audio output.

# Signature

```
+---------------------------------------+
|     .-.       .-.       .-.           |
|    /   \     /   \     /   \     +    |
|         \   /     \   /     \   /     |
|          "_"       "_"       "_"      |
|                                       |
|  _   _   _ _  _   _   ___   ___ _  _  |
| | | | | | | \| | /_\ | _ \ / __| || | |
| | |_| |_| | .` |/ _ \|   /_\__ \ __ | |
| |____\___/|_|\_/_/ \_\_|_(_)___/_||_| |
|                                       |
|                                       |
| Lunar RF Labs                         |
| Email: root@lunar.sh                  |
|                                       |
| Research Laboratories                 |
| OpenAlias (BTC, XMR): lunar.sh        |
| Copyright (C) 2022-2024               |
+---------------------------------------+
```
