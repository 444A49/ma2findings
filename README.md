# MA2findings

## Preamble

Most of the techniques and tips discussed on [minifindings](https://github.com/444A49/minifindings) are applicable to the Mavic Air 2. Tools, commands, files. If something here doesn't make sense or assumes things you're not familiar with, check the other guide.

## Purpose

My aim on this project is not to disable NFZ or do hardware changes on the drone, as this is well above my paygrade. DJI is not an improvised company. Really capable people can find exploits and hacks, but it is no easy feat. Pay close attention to [Drone Hacks](https://drone-hacks.com/) and [No Limit Dronez](https://nolimitdronez.com/) for advanced stuff.

In this file I'll collect things that I've found about the drone, useful parameters and other stuff that might help other people. 

## Tools

* [OG's DJI Firmware Tools](https://github.com/o-gs/dji-firmware-tools/)
* Any linux distribution
* Shell files

## AC

### Serial mode

Just like the Mini, the MA2 has a serial mode that can be enabled with the following command:

```./comm_serialtalk.py /dev/ttyACM0 --cmd_set=0 --cmd_id=1 --sender=1001 --receiver_type=31```

There's a certain window of time during boot up where you can issue this command (too soon and the port won't be even open, too late and you won't get any reponse back). Just make sure you issue it as soon as `ttyACM0` becomes available.

I don't know why, but the MA2 doesn't accept commands as soon as serial mode is enabled. I added a 10 second delay on all my scripts to give it enough time to finalize the boot process.

### Parameters

To set a param, issue the following command:

`./comm_og_service_tool.py /dev/ttyACM0 WM231 FlycParam set #parameter_name #new_value`

It might be easier to use the Drone Hacks or NLD client, but using the raw commands will allow you to set parameters with any device, even your phone. This is useful to test combinations of parameters without a laptop while you are away from home.


#### Disable autolanding & controller beeping on low battery

`battery_type_0 0`

This one is a must if you want to prevent the drone from autolanding. Keep in mind that there will be no visual/audio about battery level. Fly app will show 0 minutes to RTH, 0 minutes to forced landing. It will still show the time you have until the battery is depleted.

This parameter is only available **on firmware 0113 and 0130**. On 0250 onwards it was locked to value `1`.

#### "Gentle/Beginner mode"

Descent speed in the MA2 is quite slow, even in Sport mode. At most you can get 10km/h pulling down the left stick, 18km/h if you also make it go forward. Unlike the Mini, the `vert_vel_down` parameter is already maxed out at -3; Mini allows you to set values up to -10.

Luckily there's a Gente/Beginner mode that seems to be leftovers from the N3 flight controller that allows you to set this mode which is quite slow on every aspect, but it **doesn't** have it's parameters locked to low values! Meaning that you can enable this mode and tweak it to make it more powerful than Sport mode.

This was taken **from the [MAVIC Club Channel](https://t.me/mavic_club_channel) Telegram**.

First enable the Gentle/Beginner by changing the control mode from `8` to `6`:

`fswitch_selection_1|g_config.control.control_mode[1] 6`

This parameter has pipes and square brackets, so you'll have to escape them before running the command. It also needs to be issued with `--alt` for some reason. In summary, just run this `./comm_og_service_tool.py /dev/ttyACM0 WM231 FlycParam set --alt fswitch_selection_1\|g_config.control.control_mode\[1\] 6`.

This has the side effect of not showing up the selected mode in the Fly app. If you flip the RC switch to Sport mode, the app will tell you are in Normal mode. This is normal. If you use Sport mode to disable sensors (like to catch it with your hands on landing) keep in mind **this won't happen anymore**. "Sport" mode and Normal mode will not change the behaviour of the sensors. If you have them enabled, they'll remain enabled, and vice versa.

Then tweak the following parameters:

`mode_gentle_cfg_vert_vel_up_0 6`

`mode_gentle_cfg_vert_vel_down_0 -8`

`vert_vel_down_adding_max_0 -10`

`mode_gentle_cfg_rc_scale_0 0.95`

`mode_gentle_cfg_tilt_atti_range_0 35`

`mode_gentle_cfg_tors_gyro_range_0 100`

This will give you more or less the same speed performance as Sport mode, but with faster descent speeds. Pulling down just the left stick will get you -30km/h.

#### Disable leds

You can disable the christmas LED lights on the arms with the parameter `forearm_led_ctrl|g_config.misc_cfg.forearm_lamp_ctrl`. It's a bitmask. Right most digit controls the front leds, 3rd digit from the right the back leds. Which means:

* 5 = Both LEDs on
* 4 = Only back LEDs on
* 1 = Only front LEDs on
* 0 = All LEDs off

Parameter name has a pipe on it so you need to issue it withe `--alt` flag.

`./comm_og_service_tool.py /dev/ttyACM0 WM231 FlycParam set --alt forearm_led_ctrl\|g_config.misc_cfg.forearm_lamp_ctrl 0`

Credit to StanLee, M4xw, and LazyPilot on dji-rev channel.

## RC

### FCC mode

FCC mode can be forced in the AC via the RC. The process is a bit more convoluted than usual and it's not permanent, so you'll need to do it on every flight. Follow these steps:

1) Connect the RC to a computer/phone/anything that can run `comm_serialtalk.py`.
2) Let the RC and AC connect to each other
3) Run the following command: `./comm_serialtalk.py /dev/ttyACM0 --sender_type 2 --sender_index 0 --receiver_type 9 --receiver_index 0 --encrypt_type 0 --ack_type 2 --pack_type 0 --cmd_set 9 --cmd_id 39 --payload_hex 00024800FFFF0200000000`
4) On the FlyApp, open the transmission tab and verify that the string `1km` is above the grey line.
