# Atreus Firmware

This is my own firmware for the [Atreus keyboard](https://github.com/technomancy/atreus) I designed and built.

## Usage

Install `gcc-avr` and the
[teensy loader](http://www.pjrc.com/teensy/loader_cli.html).

Run `make upload` or `make upload LAYOUT=softdvorak` to use a different layout.

Designed to run on a Teensy 2 but could probably be adapted to run on
other USB-capable atmega boards.

TODO: document double-duty fn behaviour.

## Pinout

Outputs:

    |------------+----+----+----+----|
    | row number |  0 |  1 |  2 |  3 |
    |------------+----+----+----+----|
    | pin number | D0 | D1 | D2 | D3 |
    |------------+----+----+----+----|

Inputs:

    |---------------+----+----+----+----+----+----+----+----+----+----+----|
    | column number |  0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 |  9 | 10 |
    |---------------+----+----+----+----+----+----+----+----+----+----+----|
    | pin number    | B0 | B1 | B2 | B3 | B4 | B5 | B6 | B7 | F4 | F5 | F6 |
    |---------------+----+----+----+----+----+----+----+----+----+----+----|

## Layouts in JSON

See `qwerty.json` for an example.

The JSON layouts are parsed by the `atreus.el` code in Emacs. `M-x
atreus-make` will compile a given JSON file into a `.hex` file for
uploading, and `M-x atreus-view` will compile JSON into an HTML table
and open it in the browser. The makefile will invoke Emacs to
precompile the `layout.h` file, but if you just want the default you
can copy `layout_qwerty.h` to `layout.h` and edit the makefile to skip
the Emacs invocation.

The layout JSON should simply be a three-dimensional array. At the top
level, every array element is a layer. You can have up to 64
layers. Each layer is an array of rows, and each row is an array of
keycodes. A keycode can either be a keypress (described in
`usb_keyboard.h`), a keypress with a modifier (like `["shift", "7"]`
to insert a `&` character), `"fn"` to switch to layer 1 while held, or
`["layer", 0]` to switch to a given layer beyond while the key is
being held. It's also advised to include a `["reset"]` key as this is
used to program updates to the firmware without opening the case to
hit the hardware reset button. Finally, an empty string can be used to
indicate a keypress which does nothing.

## Layouts in C

Layouts in C are arrays of integers. For basic key presses, use the
keycodes defined in `usb_keyboard.h`. For modified key presses, add
256 to the code listed there for ctrl, 512 for shift, 1024 for alt,
and 2048 for super/gui. Layouts can also include references to
functions to execute. Place a `void` function pointer in the
`layout_functions` array and add 110 to the index of the function, and
that keycode will cause the function to be invoked.

The `layouts` pointer should be set to an array of layouts. Every scan
through the keyboard matrix will set the current layout to the
`current_layout_number`th element of the `layouts` array. To make a
keybinding that changes the current layout, simply set this number.

However, most functions will be called the final pass where each
keypress is looked up in the currently active layout. That means if
you have a function that changes the current layout, it needs to run
sooner so it can affect regular keycode lookups. Keycodes between 200
and 255 will trigger functions that run on a separate pre-invoke pass
before the rest of the keycodes are looked up, so this is how
layer-changing functions should be defined.

## How it works

Since the microcontroller has a limited number of pins, the switches
are wired in a matrix where each has its positive contact connected to
those in the same row and its negative contact wired in with those
above and below it. Reading the state of the switches can only happen
a row at a time: a single row (output) pin is brought low, and all the
column inputs are read. Any of them that read low are recorded as a
keypress. Low voltage is used to for pressed keys because each input
pin has a built-in pullup resistor.

However, because of the electrical properties of switches, it's
necessary to go through a
[debouncing](https://en.wikipedia.org/wiki/Switch#Contact_bounce)
process as the switches settle. This means taking a few scans over the
matrix and waiting until you get N successive reads of the same state
before counting any single keypress or release as legitimate.

## TODO

* Improve debouncing
* Support layer toggle bindings
* SVG output of layouts?

## License

Copyright © 2014 Phil Hagelberg

Released under the [GNU GPL version 3](https://www.gnu.org/licenses/gpl.html).
