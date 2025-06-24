# MBF25 Frame Definition v0.2

## Frame Flags

* Add `MBF25 Bits = X` in the Frame definition.
* The format is the same as the existing `Bits` and `MBF21 Bits` fields.
* Example: `MBF25 Bits = NOPAIN+CANBEUSED`
* The value can also be given as a number (sum of the individual flag values below).

| Name     | Value | Description                                          |
|----------|-------|------------------------------------------------------|
| `NOPAIN` | 0x001 | Cannot be interrupted by the thing receiving damage. |
| `CANUSE` | 0x002 | Thing can receive use presses.                       |
