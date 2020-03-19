# ESPRESSOBin


This is the base Nerves System configuration for the [ESPRESSObin](http://espressobin.net/).

| Feature              | Description                     |
| -------------------- | ------------------------------- |
| CPU                  | upto 1.2 GHz dual-core Cortex-A53 (ARM v8) |
| Memory               | 0.5, 1, 2 GB DRAM (version, options) |
| Storage              | MicroSD, (eMMC v7: option)         |
| Linux kernel         | 4.19 w/ Marvell Armada3720 patches |
| IEx terminal         | UART                               |
| GPIO, I2C, SPI       | maybe Yes - [Elixir Circuits](https://github.com/elixir-circuits) |
| ADC                  | No                              |
| PWM                  | Yes, but no Elixir support      |
| UART                 | 2 available - `ttyMV0` `ttyMV1` (optional) |
| Display              | No |
| Ethernet             | Yes - on board L2SW(can enable DSA, but no Elixir support) |
| WiFi                 | No, but has PCIe slot |
| Bluetooth            | No |
| Audio                | No |

Maybe many devices that connect to USB can be used, but we should support them with Elixir.

## Using

The most common way of using this Nerves System is create a project with `mix
nerves.new` and to export `MIX_TARGET=ebin`. See the [Getting started
guide](https://hexdocs.pm/nerves/getting-started.html#creating-a-new-nerves-app)
for more information.

If you need custom modifications to this system for your device, clone this
repository and update as described in [Making custom
systems](https://hexdocs.pm/nerves/systems.html#customizing-your-own-nerves-system)

If you're new to Nerves, check out the
[nerves_init_gadget](https://github.com/nerves-project/nerves_init_gadget)
project for creating a starter project. It will get you started with the basics
like bringing up networking, initializing the writable application data
partition, and enabling ssh-based firmware updates.  It's easiest to begin by
using the wired Ethernet interface 'eth0' and DHCP.

## bootloader

Bootloader for Armada-3700 family is consit of following modules:
- A3700 utils
- ATF
- u-boot

Published in [Marvell official repogitory](https://github.com/MarvellEmbeddedProcessors).
The boot sequence of A53 is generic ATF's. (BL1 -> BL2 -> BL31 -> BL33(u-boot))

A3700 utils is need for generating boot image that includes some header and informations like as DDRC parameters.
More information will be included in the data sheet under the NDA.

ToDo: Information of environment in U-Boot. work flow of build images.


## Provisioning devices

This system supports storing provisioning information in a small key-value store
outside of any filesystem. Provisioning is an optional step and reasonable
defaults are provided if this is missing.

Provisioning information can be queried using the Nerves.Runtime KV store's
[`Nerves.Runtime.KV.get/1`](https://hexdocs.pm/nerves_runtime/Nerves.Runtime.KV.html#get/1)
function.

Keys used by this system are:

Key                    | Example Value     | Description
:--------------------- | :---------------- | :----------
`nerves_serial_number` | `"12345678"`      | By default, this string is used to create unique hostnames and Erlang node names. If unset, it defaults to part of the Raspberry Pi's device ID.

The normal procedure would be to set these keys once in manufacturing or before
deployment and then leave them alone.

For example, to provision a serial number on a running device, run the following
and reboot:

```elixir
iex> cmd("fw_setenv nerves_serial_number 12345678")
```

This system supports setting the serial number offline. To do this, set the
`NERVES_SERIAL_NUMBER` environment variable when burning the firmware. If you're
programming MicroSD cards using `fwup`, the commandline is:

```sh
sudo NERVES_SERIAL_NUMBER=12345678 fwup path_to_firmware.fw
```

Serial numbers are stored on the MicroSD card so if the MicroSD card is
replaced, the serial number will need to be reprogrammed. The numbers are stored
in a U-boot environment block. This is a special region that is separate from
the application partition so reformatting the application partition will not
lose the serial number or any other data stored in this block.

Additional key value pairs can be provisioned by overriding the default
provisioning.conf file location by setting the environment variable
`NERVES_PROVISIONING=/path/to/provisioning.conf`. The default provisioning.conf
will set the `nerves_serial_number`, if you override the location to this file,
you will be responsible for setting this yourself.

## Linux kernel and ESPRESSObin firmware/userland

Marvell published fimwares, bootloader and kernels in [Github](https://github.com/MarvellEmbeddedProcessors/).
But Armada-3720 families seem to be no longer supported. So I use [Armbian
](https://github.com/armbian/) for the same version of `nerves_system_br` .

Firmware and bootloader are not supported in this project yet.
You should make firmware and bootloader by yourself, and modify the U-boot environment by yourself ;) .

## Linux kernel configuration notes

The Linux kernel compiled for Nerves is a stripped down version of the default
Armbian Linux kernel. This is done to remove unnecessary features, select
some Nerves-specific features like F2FS and SquashFS support, and to save space.

