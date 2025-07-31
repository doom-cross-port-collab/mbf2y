# MBF2y Weapon Definition v0.2.3

## Weapon Flags

* Add `MBF2y Bits = X` in the Weapon definition.
* The format is the same as the existing `Bits` and `MBF21 Bits` fields.
* Example: `MBF2y Bits = NODESELECT+NOAUTOAIM`
* The value can also be given as a number (sum of the individual flag values below).

| Name             | Value | Description                                                                                                         |
|------------------|-------|---------------------------------------------------------------------------------------------------------------------|
| `NODESELECT`     | 0x001 | Weapon cannot be switched away from until `A_Lower` is called or the weapon is removed from the player's inventory. |
| `NOVERTAUTOAIM`  | 0x002 | Weapon never uses vertical autoaim, regardless of codepointers.                                                     |
| `NOHORIZAUTOAIM` | 0x004 | Weapon never uses horizontal autoaim, regardless of codepointers.                                                   |


## New Weapon Codepointers

* **A_WeaponJumpIfPowerup(state, powerup)**
    * Jumps to a frame if the player has the specified powerup.
    * Args:
        * `state (uint)`: Frame to jump to
        * `powerup (uint)`: Powerup to check for:
            * 0: Berserk
            * 1: Lite-amp
            * 2: Radsuit
            * 3: Partial invisibility
            * 4: Invulnerability
            * 5: All map
            * 6: Double damage
            * 7: Quad damage
            * 8: Health leech
            * 9: Infinite ammo
            * 10: Fast weapons

* **A_WeaponJumpIfAmmoDivisible(state, amount)**
    * Jumps to a frame if a weapon's ammo count is evenly divisible by a given number, similar to the pistol from Duke Nukem 3-D or the Uzis in Shadow Warrior.
    * Args:
        * `state (uint)`: Frame to jump to
        * `amount (uint)`: Divisor to check.
    * Notes:
        * `amount` must be greater than zero, or else this codepointer will no-op.

* **A_WeaponMultiJump(state1, value1, state2, value2 ... state20, value20)**
    * Jumps to one of up to four possible states based on the result of a `P_Random` call. If the result is greater than the last defined of `value1` ... `value4`, then it will jump to the next frame instead as normal.
    * Args:
        * `state1` (uint): The frame to jump to if the random value is equal to or less than `value1`.
        * `value1` (uint): If the random number is equal to or less than this value, then this codepointer will jump to `state1`.
        * ...
        * `state20` (uint): The frame to jump to if the random value is equal to or less than `value20`.
        * `value20` (uint): If the random number is equal to or less than this value, then this codepointer will jump to `state20`.
    * Notes:
        * An ascending order for `value1` through `value20` should be used, as only one `P_Random` call is made, and it is compared against `value1` through `value20` in that order.
        * A value of 0 or 256 or greater for `value2` through `value20` will result in their respective state arguments being ignored, as there is no possible way to meet those values.

* **A_WeaponRemove(flags)**
    * Removes the calling weapon from the player's inventory and changes the player's current weapon to a different weapon.
    * Args:
        * `flags (int)`: Allows further customization:
            * `WR_REMOVEAMMO (0x001)`: Also removes all ammunition associated with the weapon.
            * `WR_SKIPLOWER (0x002)`: Skips the lowering animation for the weapon and instantly changes weapons.
    * Notes:
        * Has no effect on `wp_fist`.

* **A_WeaponBulletAttack2(hspread, vspread, numbullets, damagebase, damagedice, flatdamage, hitscantype, flags)**
    * Generic weapon hitscan attack.
    * Args:
        * `hspread (fixed)`: Horizontal spread (degrees, in fixed point)
        * `vspread (fixed)`: Vertical spread (degrees, in fixed point)
        * `numbullets (uint)`: Number of hitscans to fire; if not set, defaults to 1
        * `damagebase (uint)`: Base damage of attack; if not set, defaults to 5
        * `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to 3
        * `flatdamage (uint)`: Flat damage to add to damage amount; if not set, defaults to 0
        * `hitscantype (uint)`: Hitscan type to use
        * `flags (int)`: Allows further customization:
            * `WB2_NOHORIZONTALAUTOAIM (0x001)`: No horizontal aim adjustments are applied to the projectile.
            * `WB2_NOAUTOAIM (0x002)`: No autoaim whatsoever is applied to the projectile.
            * `WB2_SPREADPROJECTILE (0x004)`: When enabled, the `angle` and `pitch` will be parsed as a horizontal and vertical spread respectively.
            * `WP2_CHECKAMMO (0x008)`: Will call `A_Lower` if the weapon's ammo type is less than the weapon's `Ammo per shot` value.
            * `WB2_USEAMMO (0x010)`: Will subtract the weapon's `Ammo per shot` value from the weapon's ammo type, to a minimum of 0.

* **A_WeaponProjectileAttack2(type, angle, pitch, hoffset, voffset, flags)**
  * Generic weapon projectile attack.
  * Args:
    * `type (uint)`: Type (dehnum) of actor to spawn
    * `angle (fixed)`: Angle (degrees), relative to player's angle
    * `pitch (fixed)`: Pitch (degrees), relative to player's pitch
    * `hoffset (fixed)`: Horizontal spawn offset, relative to player's angle
    * `voffset (fixed)`: Vertical spawn offset, relative to player's default projectile fire height
    * `flags (int)`: Allows further customization:
        * `WP2_NOHORIZONTALAUTOAIM (0x001)`: No horizontal aim adjustments are applied to the projectile.
        * `WP2_NOAUTOAIM (0x002)`: No autoaim whatsoever is applied to the projectile.
        * `WP2_SINGLEAUTOAIM (0x004)`: Useful for weapons that shoot multiple projectiles via 0-tic `A_WeaponProjectileEx` calls in a spread. Will ignore the angle, pitch, and offset for the purposes of finding an autoaim target, which should cause all `A_WeaponProjectileEx` attacks from the same weapon occurring on the same tic to have the same autoaim target, then apply the angle, pitch, and offset at the same time.
        * `WP2_AUTOAIMWITHVOFFSET (0x008)`: The player projectile will account for vertical offset before determining its pitch when determining autoaim pitch.
        * `WP2_SPREADPROJECTILE (0x010)`: When enabled, the `angle` and `pitch` will be parsed as a horizontal and vertical spread respectively.
        * `WP2_CHECKAMMO (0x020)`: Will call `A_Lower` if the weapon's ammo type is less than the weapon's `Ammo per shot` value.
        * `WP2_USEAMMO (0x040)`: Will subtract the weapon's `Ammo per shot` value from the weapon's ammo type, to a minimum of 0.
  * Notes:
    * The spawned projectile's `tracer` pointer is set to the player's autoaim target, if available.