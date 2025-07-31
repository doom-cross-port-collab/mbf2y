# MBF2y Frames Definition v0.2.3

## Additional Args

* `Args9` through `Args40` fields have been added; however, their use is limited to very specific codepointers that cannot be properly parameterized otherwise.

## Frame Flags

* Add `MBF2y Bits = X` in the Frame definition.
* The format is the same as the existing `MBF21 Bits` and `Bits` fields.
* Example: `MBF2y Bits = ALLOWUSE+NOPAIN`.
* The value can also be given as a number (sum of the individual flag values below).

| Name             | Value | Description                                                                                                                                                                                |
|------------------|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ALLOWUSE`       | 0x001 | Thing will jump to its Use frame if it intercepts a player use frame.                                                                                                                      |
| `NOPAIN`         | 0x002 | Thing will not jump to its pain state when receiving damage in this frame.                                                                                                                 |
| `FASTWEAPON`     | 0x004 | When used on a weapon frame, causes its duration to be divided by 2 (rounded up) when `pw_fastweapon` is enabled.                                                                          |
| `SKIPFASTWEAPON` | 0x008 | When used on a weapon frame, causes its duration to be treated as 0 when `pw_fastweapon` is enabled. Overrides `FASTWEAPON`.                                                               |
| `SEMIFULLBRIGHT` | 0x010 | This frame's render brightness is drawn as `(lightlevel + 256) / 2`, resulting in a brighter appearance in darker sectors, but the effect becomes less noticeable as the sector brightens. |


## Minimum frame brightness

* Add `Min brightness = X` in the Frame definition.
* Sets the minimum brightness level (equivalent to sector brightness) at which to render the frame.
* Example: `Min brightness = 192` will cause the frame to be drawn as though it's in a sector with a brightness level of 192 unless it would be drawn with a higher brightness for any reason.
* The interaction between this property and `SEMIFULLBRIGHT` is that the `SEMIFULLBRIGHT` takes effect after this property takes effect. For example, if `Min brightness = 144` is applied and the thing (or player, in the case of a weapon frame) is within a sector with a brightness level of 128, the frame will be drawn with a brightness level of 200, while in a sector with a brightness level of 144 or higher, the frame would be drawn with a brightness level of 208.