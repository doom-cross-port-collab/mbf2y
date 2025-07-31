# MBF2y Thing Specification v0.2.3
MBF2y expands upon the work done in MBF21 and ID24, vastly expanding modding potential primarily by leveraging existing code.

## SPECIAL Item Enhancements
Building upon the custom `SPECIAL` thing behaviors from `ID24HACKED`, the following changes have been made:

### New pickup types
The following enum values have been added to the `Pickup item type` field:

| Value | Description           | Notes                                                                                                                                                                                                                                                                                                                                                    |
|-------|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 22    | Armor repair          | Cannot be picked up if a player's current `armortype` is 0. Otherwise, adds the `Pickup armor amount` (default of 1) to the player's current armor, up to the `Max Armor` value in the `[MISC]` block (default 200) if the player's current `armortype` is 2 (blue armor) and half of that value if the player's current `armortype` is 1 (green armor). |
| 23    | Double damage powerup | Double damage. See (.\powerups.md). Unless a `Pickup powerup duration` is set, defaults to giving the powerup for `(30*TICRATE)` tics.                                                                                                                                                                                                                   |
| 24    | Quad damage powerup   | Quad damage. See (.\powerups.md). Unless a `Pickup powerup duration` is set, defaults to giving the powerup for `(30*TICRATE)` tics.                                                                                                                                                                                                                     |
| 25    | Leech health powerup  | Leech health. See (.\powerups.md). Unless a `Pickup powerup duration` is set, defaults to giving the powerup for `(30*TICRATE)` tics.                                                                                                                                                                                                                    |
| 26    | Infinite ammo powerup | Infinite ammo. See (.\powerups.md). Unless a `Pickup powerup duration` is set, defaults to giving the powerup for `(30*TICRATE)` tics.                                                                                                                                                                                                                   |
| 27    | Fast weapon powerup   | Fast weapon. See (.\powerups.md). Unless a `Pickup powerup duration` is set, defaults to giving the powerup for `(30*TICRATE)` tics.                                                                                                                                                                                                                     |

### Pickup health amount
* Add `Pickup health amount = H` in Thing definition.
* `H` is a nonnegative integer. Values of 0 have no effect.
* When the player picks up a `SPECIAL` item this property defined, behavior depends upon the `Pickup item type`:
    * `8 (health bonus)` or `11 (soulsphere)`: Changes the amount of bonus healing to `H`
    * `9 (stimpack)` or `10 (medikit)`: Changes the amount of healing up to the player's maximum to `H`
    * `12 (megasphere)` or `18 (berserk)`: If the player's current health is lower than `M`, sets the health to `M`
    * This field has no effect with other `Pickup item type`s.

### Pickup armor amount
* Add `Pickup armor amount = A` or `Pickup armour amount = A` in Thing definition.
* `A` is a nonnegative integer. Values of 0 have no effect.
* When the player picks up a `SPECIAL` item with this property defined, behavior depends upon the `Pickup item type`:
    * `13 (armor bonus)`: Changes the amount of bonus armor to gain to `A`.
    * `12 (megasphere)`: Changes the player's armor to blue armor (`armortype` of 2); if the player's current amor value is less than `A`, sets it to `A`.
    * `14 (green armor)`: If the player's current armor is equal to or more than `A`, do not pick up the item. If the player's current armor is less than `A`, pick up the item, and set the player's `armortype` to `1`.
    * `15 (blue armor)`: If the player's current armor is equal to or more than `A`, do not pick up the item unless the player's current `armortype` is `0` or `1`. Otherwise, pick up the item and set `armortype` to 2 and the player's current armor amount to `A`.
    * `22 (armor repair)`: If the player's current `armortype` is `0`, do not pick up. If the player's current `armortype` is `1` and the player's current armor amount is less than 100 (or `Max Green Armor`, if defined in the `[MISC]` block), increase it by `A` up to a maximum of 100 or the `Max Green Armor` value. IF the player's current `armortype` is `2` and the player's current armor amount is less than `Max Armor` in the `[MISC]` block (default of 200), increase it by `A` up to a maximum of 200 or the `Max Armor` value. Do not pick up if the player is already at or over the maximum armor value for their `armortype`.

### Pickup Powerup Duration
* Add `Pickup powerup duration = D` in Thing definition.
* `D` is a nonnegative integer and is multipled by `TICRATE` after parsing.
* If `D` is nonzero, when a player picks up a `SPECIAL` item with a `Pickup item type` that grants a non-computer area map powerup (values `17` through `21` and `23` through `27`), it overrides the duration of the powerup. If the player already has the same powerup item type and its current remaining tic count is greater than this value, no change is made to the powerup's remaining tic value, though the pickup still occurs; if its current remaining tic count is lower than this value, it sets it to this value instead.

## Additional Dehacked Thing Groups

### Projectile Collision Groups
* Add `Projectile collision group = C` in Thing definition.
* `C` is a nonnegative integer.
* Projectiles with the same value of `C` will skip all collision detections with Things with the same value of `C`, completely ignoring them. Ripper projectiles will not play their rip sound, spawn blood objects, or trigger pain states when this happens.

## New Thing Flags

* Add `MBF2y Bits = X` in the Thing definition.
* The format is the same as the existing `Bits`, `MBF21 Bits`, and `ID24 Bits` fields.
* Example: `MBF2y Bits = NODAMAGE+ANTITELEFRAG`.
* The value can also be given as a number (sum of the individual flag values below).

| Flag              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                              | Value             | A_AddFlags / A_RemoveFlags Effects                                                                                                                                               |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NODAMAGE          | Thing does not lose health when taking damage.                                                                                                                                                                                                                                                                                                                                                                                                           | 0x00000000000001  | Can be changed without issue.                                                                                                                                                    |
| NOCRUSH           | If Thing is a `CORPSE`, it does not turn into gibs when crushed. If Thing is `DROPPED`, it does not disappear when crushed. (Thing will still take damage from crushers.)                                                                                                                                                                                                                                                                                | 0x00000000000002  | Can be changed without issue.                                                                                                                                                    |
| NOTAUTOAIMED      | Player hitscan autoaim (but not BFG-style spray attacks) ignore this Thing completely.                                                                                                                                                                                                                                                                                                                                                                   | 0x00000000000004  | Can be changed without issue.                                                                                                                                                    |
| NOSPRAY           | `A_BFGSpray` and `A_Spray` ignore this Thing completely.                                                                                                                                                                                                                                                                                                                                                                                                 | 0x00000000000008  | Can be changed without issue.                                                                                                                                                    |
| ANTITELEFRAG      | Things that would telefrag this thing are telefragged instead.                                                                                                                                                                                                                                                                                                                                                                                           | 0x00000000000010  | Can be changed without issue.                                                                                                                                                    |
| PUSHABLE          | `SOLID` Thing can be pushed by other `SOLID` Things. This uses an identical method to Hexen.                                                                                                                                                                                                                                                                                                                                                             | 0x00000000000020  | Can be changed without issue.                                                                                                                                                    |
| CANNOTPUSH        | `SOLID` Thing cannot push other `PUSHABLE` objects.                                                                                                                                                                                                                                                                                                                                                                                                      | 0x00000000000040  | Can be changed without issue.                                                                                                                                                    |
| FLOATBOB          | GZDoom-style renderer-only floatbobbing.                                                                                                                                                                                                                                                                                                                                                                                                                 | 0x00000000000080  | Can be changed without issue. Removing the flag will not reset the internal float amount, but will pause it from incrementing.                                                   |
| DEADFLOAT         | Thing does not lose the `NOGRAVITY` flag upon death. **This is turned on for Lost Souls by default.**                                                                                                                                                                                                                                                                                                                                                    | 0x00000000000100  | Can be changed without issue.                                                                                                                                                    |
| ONLYSLAMSOLID     | When charging, Thing does not lose momentum when colliding with a non-`SOLID` Thing.                                                                                                                                                                                                                                                                                                                                                                     | 0x00000000000200  | Can be changed without issue.                                                                                                                                                    |
| KEEPCHARGETARGET  | Thing remembers its target after colliding after a charge.                                                                                                                                                                                                                                                                                                                                                                                               | 0x00000000000400  | Can be changed without issue.                                                                                                                                                    |
| NOSTOPCHARGE      | When charging, Thing does not lose momentum or stop charging when it receives non-lethal damage.                                                                                                                                                                                                                                                                                                                                                         | 0x00000000000800  | Can be changed without issue.                                                                                                                                                    |
| TUNNEL            | Ripper projectile remembers the last Thing it damaged and will not damage the same Thing again until it damages a different Thing.                                                                                                                                                                                                                                                                                                                       | 0x00000000001000  | Can be changed without issue. Removing it will not cause a ripper projectile to forget its last damaged Thing, however.                                                          |
| NORIPBLOOD        | Ripper projectile does not produce blood or puffs when damaging a `SHOOTABLE` Thing.                                                                                                                                                                                                                                                                                                                                                                     | 0x00000000002000  | Can be changed without issue.                                                                                                                                                    |
| ACTIVATOR         | Thing can activate most player-activated linedefs as though it were a voodoo doll.                                                                                                                                                                                                                                                                                                                                                                       | 0x00000000004000  | Can be changed without issue.                                                                                                                                                    |
| FASTTHING         | Thing uses fast projectile-style movement calculation even if it isn't a `MISSILE` or doesn't have `SKULLFLY` activated.                                                                                                                                                                                                                                                                                                                                 | 0x00000000008000  | Can be changed without issue.                                                                                                                                                    |
| NOFASTMISSILE     | Bypasses the fast movement checks on `MISSILE` or `SKULLFLY` objects. Note that using this in conjunction with very high speed values will result in bad behavior.                                                                                                                                                                                                                                                                                       | 0x00000000010000  | Can be changed without issue.                                                                                                                                                    |
| SOLIDMISSILE      | `MISSILE`, `BOUNCES`, or `BOUNCER` thing collides with all impassible linedefs, including lower/upper sky sidedefs.                                                                                                                                                                                                                                                                                                                                      | 0x00000000020000  | Can be changed without issue.                                                                                                                                                    |
| LOYAL             | Thing will never infight with other Things of the same "faction." For example, a non-`FRIENDLY` monster with this flag may take damage from other non-`FRIENDLY` monsters, but it will never target them deliberately. A `FRIENDLY` monster with this flag will never deliberately target a Player or another `FRIENDLY` monster after taking damage.                                                                                                    | 0x00000000040000  | Can be changed without issue.                                                                                                                                                    |
| KEEPTARGET        | Once Thing obtains a target, it will not change its target until the original target is dead.                                                                                                                                                                                                                                                                                                                                                            | 0x00000000080000  | Can be changed without issue.                                                                                                                                                    |
| ONLYTARGETDMG     | If this Thing has a living target, then only damage sourced from its target can damage it. This implies `KEEPTARGET`.                                                                                                                                                                                                                                                                                                                                    | 0x00000000100000  | Can be changed without issue.                                                                                                                                                    |
| ISMONSTER         | Thing is considered a monster. It does not spawn when using the `-nomonsters` command-line parameter, and any cheat codes or source port-specific commands that affect all monsters, such as DSDA-Doom's `TNTEM` or GZDoom's `kill monsters` console command, should also affect this Thing.                                                                                                                                                             | 0x00000000200000  | Can be changed without issue, but this will have little effect in the game, as things have already spawned by then. Could ostensibly affect `kill monsters` or `TNTEM` behavior. |
| BOUNCER           | Alternative to MBF's janky `BOUNCE` flag. Designed to be combined with the `MISSILE`, `BOUNCEONWALLS`, `BOUNCEONSOLID`,<br>`BOUNCEONSHOOTABLE`, `BOUNCEONFLOOR`,  `BOUNCEONCEILING`, and/or `BOUNCEONSKY` flags.                                                                                                                                                                                                                                         | 0x00000000400000  | Can be changed without issue, though this will not affect a thing's current momentum.                                                                                            |
| BOUNCEONWALLS     | `BOUNCER` Thing bounces off of vertical surfaces, such as one-sided linedefs or upper and lower sidedefs. It will still pass through sky sidedefs if it is a `MISSILE`, the linedef does not have the `Block projectiles` flag, and the Thing does not have the `SOLIDMISSILE` and/or `BOUNCEONSKY` flags.                                                                                                                                               | 0x00000000800000  | Can be changed without issue, though this will not affect a thing's current momentum.                                                                                            |
| BOUNCEONSOLID     | `BOUNCER` Thing bounces off of `SOLID` Things. If it is a `MISSILE`, it will still explode on collision with a `SHOOTABLE` Thing unless it also has the `BOUNCEONSHOOTABLE` flag.                                                                                                                                                                                                                                                                        | 0x00000001000000  | Can be changed without issue, though this will not affect a thing's current momentum.                                                                                            |
| BOUNCEONSHOOTABLE | `MISSILE` and `BOUNCER` Thing bounces off of `SHOOTABLE` Things. It will still inflict damage upon impact.                                                                                                                                                                                                                                                                                                                                               | 0x00000002000000  | Can be changed without issue, though this will not affect a thing's current momentum.                                                                                            |
| BOUNCEONFLOOR     | `BOUNCER` Thing bounces off of non-sky floors.                                                                                                                                                                                                                                                                                                                                                                                                           | 0x00000004000000  | Can be changed without issue, though this will not affect a thing's current momentum.                                                                                            |
| BOUNCEONCEILING   | `BOUNCER` Thing bounces off of non-sky ceilings.                                                                                                                                                                                                                                                                                                                                                                                                         | 0x00000008000000  | Can be changed without issue, though this will not affect a thing's current momentum.                                                                                            |
| BOUNCEONSKY       | If `BOUNCER` Thing has `BOUNCEONFLOOR`, will bounce on sky floors. If `BOUNCER` Thing has `BOUNCEONCEILING`, will bounce on sky ceilings.                                                                                                                                                                                                                                                                                                                | 0x00000010000000  | Can be changed without issue, though this will not affect a thing's current momentum.                                                                                            |
| ANTIGRAV          | Thing will "fall" toward the ceiling and come at rest when it hits a the ceiling instead of the floor. This can be combined with `LOGRAV`.                                                                                                                                                                                                                                                                                                               | 0x000000020000000 | Can be changed without issue.                                                                                                                                                    |
| CANTLEAVEFLOORPIC | Thing cannot cross into a sector with a different floor texture.                                                                                                                                                                                                                                                                                                                                                                                         | 0x00000040000000  | Can be changed without issue.                                                                                                                                                    |
| FLOORHUGGER       | Projectile moves along the floor and will pass over all obstacles until it hits a solid wall (or, if not a ripper projectile, a `SOLID` and `SHOOTABLE` thing). Thing will have its Z momentum set to 0 while this flag is active.                                                                                                                                                                                                                       | 0x00000080000000  | Can be changed without issue.                                                                                                                                                    |
| FORCEBLOOD        | `MISSILE` Thing will cause a `SOLID` `SHOOTABLE` Thing that it hits to produce blood or a puff as if it were a hitscan attack.                                                                                                                                                                                                                                                                                                                           | 0x00000100000000  | Can be changed without issue.                                                                                                                                                    |
| PAINLESS          | Things that receive radius or missile damage from this Thing will never enter their pain state.                                                                                                                                                                                                                                                                                                                                                          | 0x00000200000000  | Can be changed without issue.                                                                                                                                                    |
| TOUCHYTARGET      | Thing will set its target to the last living `SOLID` and `SHOOTABLE` thing to touch it.                                                                                                                                                                                                                                                                                                                                                                  | 0x00000400000000  | Can be changed without issue.                                                                                                                                                    |
| TOUCHYUSE         | Thing will set its target to the last living `SOLID` and `SHOOTABLE` thing to activate its Use state.                                                                                                                                                                                                                                                                                                                                                    | 0x00000800000000  | Can be changed without issue.                                                                                                                                                    |
| NOPAIN            | Thing will never enter its pain state while taking damage.                                                                                                                                                                                                                                                                                                                                                                                               | 0x00001000000000  | Can be changed without issue.                                                                                                                                                    |
| EXTREMEDEATH      | If Thing is a `MISSILE` or directly causes radius damage and a Thing is killed by it, it will always trigger the XDeath state.                                                                                                                                                                                                                                                                                                                           | 0x00002000000000  | Can be changed without issue.                                                                                                                                                    |
| NOEXTREMEDEATH    | If Thing is a `MISSILE` or directly causes radius damage and a Thing is killed by it, it will never trigger the XDeath state.                                                                                                                                                                                                                                                                                                                            | 0x00004000000000  | Can be changed without issue.                                                                                                                                                    |
| RESETONDEATH      | Thing's `Counter1`, `Counter2`, `Counter3`, and `Counter4` values are reset to their initial values upon death.                                                                                                                                                                                                                                                                                                                                          | 0x00008000000000  | Can be changed without issue.                                                                                                                                                    |
| COLLIDEONZMOVE    | Collision with other Things is checked even when a `MISSILE` object only has vertical momentum. Has no effect on non-`MISSILE` objects.                                                                                                                                                                                                                                                                                                                  | 0x00010000000000  | Can be changed without issue.                                                                                                                                                    |
| PASSMOBJ          | Even if it's `SOLID`, this Thing performs Z-collision checks with all other Things, and other Things perform Z-collision checks with it. If another Thing is standing on this Thing when its Z position moves, the other Thing will change its Z position as well. (To be determined: how this interacts with `SPAWNONCEILING` + `NOGRAVITY` objects on ceilings that move down and potentially cause collisions, or when anchored to a moving floor...) | 0x00020000000000  | Can be changed without issue, but this may result in actors getting stuck.                                                                                                       |
| HITOWNER          | `MISSILE` can collide with its owner. For best results, this should be added a couple of tics after the projectile spawns.                                                                                                                                                                                                                                                                                                                               | 0x00040000000000  | Can be changed without issue.                                                                                                                                                    |
| NODAMAGETHRUST    | Other Things that take damage directly from this Thing (such as if it's a `MISSILE`, a hitscan puff, an object calling `A_RadiusDamage`, or a monster calling a melee attack codepointer) are not pushed by the damage.                                                                                                                                                                                                                                  | 0x00080000000000  | Can be changed without issue.                                                                                                                                                    |
| NOTHRUST          | Sources of damage do not move this Thing at all.                                                                                                                                                                                                                                                                                                                                                                                                         | 0x00100000000000  | Can be changed without issue.                                                                                                                                                    |
| NOSCROLL          | Thing is not moved horizontally by any sector-based movement, but will still move up and down with the floor as normal.                                                                                                                                                                                                                                                                                                                                  | 0x00200000000000  | Can be changed without issue.                                                                                                                                                    |
| HURTONTOUCH       | Solid, non-`MISSILE` object inflicts damage on collision with `SHOOTABLE` objects as though it were a projectile, but doesn't enter its Death state.                                                                                                                                                                                                                                                                                                     | 0x00400000000000  | Can be changed without issue.                                                                                                                                                    |
| DONTHITTHINGS     | This Thing never collides with other Things. Useful for particle effects that might have `MISSILE` set or for when things with `DIEONSOLID` shouldn't collide with Things.                                                                                                                                                                                                                                                                               | 0x00800000000000  | Can be changed without issue.                                                                                                                                                    |
| NOSECTORDMG       | This Thing does not take damage from damaging floors even if the sector is enabled to do so.                                                                                                                                                                                                                                                                                                                                                             | 0x01000000000000  | Can be changed without issue.                                                                                                                                                    |
| DIEONSOLID        | This Thing dies on collision with any `SOLID` Thing or any linedef it cannot cross (such as a high step, one-sided linedef, or impassible linedef).                                                                                                                                                                                                                                                                                                      | 0x02000000000000  | Can be changed without issue.                                                                                                                                                    |
| GENERIC1          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x04000000000000  | Can be changed without issue.                                                                                                                                                    |
| GENERIC2          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x08000000000000  | Can be changed without issue.                                                                                                                                                    |
| GENERIC3          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x10000000000000  | Can be changed without issue.                                                                                                                                                    |
| GENERIC4          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x20000000000000  | Can be changed without issue.                                                                                                                                                    |


## Map Object Flags

`GENERIC1`, `GENERIC2`, `GENERIC3`, and `GENERIC4` should be exposed to the map editor like the `AMBUSH` and `FRIENDLY` flags.

| Bit | Value  | Description |
|-----|--------|-------------|
| 8   | 0x0100 | `GENERIC1`  |
| 9   | 0x0200 | `GENERIC2`  |
| 10  | 0x0400 | `GENERIC3`  |
| 11  | 0x0800 | `GENERIC4`  |


## New Thing Properties

### Counters
Things now have instanced global counters that can be manipulated and checked via codepointers. The following mobj properties have been added to facilitate this:

| Property Name | Description                                                              | Default Value                         | Minimum Value | Maximum Value                   |
|---------------|--------------------------------------------------------------------------|---------------------------------------|---------------|---------------------------------|
| `counter1`    | The current value of the Thing's first internal counter.                 | 0 / Thing's `Counter 1 initial` value | 0             | The Thing's `counter1max` value |
| `counter1max` | The maximum value of that the Thing's first internal counter can reach.  | 255 / Thing's `Counter 1 max` value.  | 1             | 255                             |
| `counter2`    | The current value of the Thing's second internal counter.                | 0 / Thing's `Counter 2 initial` value | 0             | The Thing's `counter2max` value |
| `counter2max` | The maximum value of that the Thing's second internal counter can reach. | 255 / Thing's `Counter 2 max` value.  | 1             | 255                             |
| `counter3`    | The current value of the Thing's third internal counter.                 | 0 / Thing's `Counter 3 initial` value | 0             | The Thing's `counter3max` value |
| `counter3max` | The maximum value of that the Thing's third internal counter can reach.  | 255 / Thing's `Counter 3 max` value.  | 1             | 255                             |
| `counter4`    | The current value of the Thing's fourth internal counter.                | 0 / Thing's `Counter 4 initial` value | 0             | The Thing's `counter4max` value |
| `counter4max` | The maximum value of that the Thing's fourth internal counter can reach. | 255 / Thing's `Counter 4 max` value.  | 1             | 255                             |

These counters are instanced per Thing. While `counter1` ... `counter4` can be manipulated at runtime via codepointers, `counter1max` ... `counter4max` are static values inherited entirely from the Thing's mobjinfo. Unless the Thing has the `RESETONDEATH` MBF2y flag set, the values of `counter1` ... `counter4` will persist even when a Thing dies and has been resurrected. On the other hand, it is expected that if a Thing dies and respawns via `-respawn` or Nightmare! difficulty, it will reinitialize its counter values, as this is technically an entirely new Thing.

When a codepointer would result in a counter being below 0 or above its maximum value, it will be set to 0 or the max value appropriately.

For more information on these, check the Thing Properties and Thing Codepointers subsections.

### Use frame
* Sets the state for the thing to jump to when it intercepts a use press.
* Things will only intercept use presses when their current frame has the `ALLOWUSE` flag set.
* Use presses require line of sight between the thing pressing use and the thing being used.
* Things cannot use themselves, but may use other things of the same type as well as dead or inanimate things.
* Precedence is given to the first linedef or thing closest to the player when use is pressed.
* Can be used to create thing-based switches, interactive actors, and many other effects.
* For multi-state presses, consider using the `GENERIC1`, `GENERIC2`, ... flags or counters.
* Add `Use frame = X` in the Thing definition.
* `X` represents a frame number.

### Damage dice
* Used on projectiles or charging enemies like Lost Souls. Can be used to parameterize the damage dealt by projectiles.
* Defaults to 8 if not set.
* Add `Damage dice = X` in the Thing definition.
* If set to 0, no RNG call should be made when damage is inflicted.

### Flat damage
* Used on projectiles or charging enemies like Lost Souls. Adds a flat damage amount to the randomized damge to the attack when it hits.
* Add `Flat damage = X` in the Thing definition.

### Melee threshold
* Taken from Crispy Doom. Allows generalizing the distance in map units at which an enemy will switch from attempting ranged attacks to melee attacks.
* This is ignored if the Thing has the `LONGMELEE` MBF21 Flag set.
* Defaults to 0 if not set.
* Add `Melee threshold = x` in the Thing defintiion.

### Max target range
* Taken from Crispy Doom. Allows generalizing the maximum distance in map units at which an enemy will initialize ranged attacks.
* This is ignored if the Thing has the `SHORTMRANGE` MBF21 Flag set.
* Defaults to 0 if not set, whcih means unlimited range.
* Add `Max target range = X` in the Thing definition.

### Min missile chance
* Taken from Crispy Doom. Allows generalizing the minimum likelihood of a missile attack.
* This is ignored if the Thing has the `HIGHERMPROB` MBF21 Bit set.
* Defaults to 200 if not set.
* Add `Min missile chance = x` in the Thing definition.

### Missile chance multiplier
* Taken from Crispy Doom. Allows generalizing a multiplier in fixed notation for missile firing chance based on distance.
* This is ignored if the Thing has the `RANGEHALF` MBF21 Flag set.
* Defaults to 65536 if not set, which is equivalent to 1.0.
* Add `Missile chance multiplier = X` in the Thing definition.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Crush frame
* Sets the state for the Thing to jump to if it's a corpse and has been crushed.
* Defaults to `S_GIBS`, which has a value of 895, if not set.
* Add `Crush frame = X` in the Thing definition.
* `X` represents a frame number.

### Crash frame
* Sets the state for the Thing to jump to if it's currently `FRAGILE` and takes damage.
* If not set, the `Dying frame` is used.
* Add `Crash frame = X` in the Thing definition.
* `X` represents a frame number.

### Damage type (UNDER CONSIDERATION)
- When used on a puff or projectile, determines the damage type for the purposes of damage type factors and damage type Pain/Death/XDeath states.
- Defaults to 0 if not set, which means untyped damage.
- Add `Damage type = X` in the Thing definition.
- `X` is a nonnegative integer from 0 to 9.

### Damage type pain frames (UNDER CONSIDERATION)
* If the source of damage that triggers this Thing's pain state is typed, then the Thing will jump to this state instead of its default pain state, if available.
* Add `Injury frame Y = X` in the Thing definition.
* `Y` is a nonnegative integer from 0 to 9 and corresponds to a damage type.
* `X` represents a frame number.
* If not set, the thing will use its default pain state (if one exists).

### Damage type death frames (UNDER CONSIDERATION)
* If the source of damage that triggers this Thing's death state is typed, then the Thing will jump to this state instead of its default death state, if available.
* Add `Death frame Y = X` in the Thing definition.
* `Y` is a nonnegative integer from 0 to 9 and corresponds to a damage type.
* `X` represents a frame number.
* If not set, the thing will use its default death state.

### Damage type exploding frames (UNDER CONSIDERATION)
* If the source of damage that triggers this Thing's exploding state is typed, then the Thing will jump to this state instead of its default exploding state, if available.
* Add `Exploding frame Y = X` in the Thing definition.
* `Y` is a nonnegative integer from 0 to 9 and corresponds to a damage type.
* `X` represents a frame number.
* If not set, the thing will default to the death state of the same damage type if one exists. If not, it will default to its default exploding state if one exists. If not, it will default to its default death state.

### Damage factor (UNDER CONSIDERATION)
- Multiplies damage that the Thing receives from damage sources with the corresponding typed damage.
* Add `Damage factor Y = X` in the Thing definition.
* `X` Defaults to 65536 if not set, which is equivalent to 1.0.
* A value of 0 or 0.0 means that a Thing is completely immune to damage from the specified type.
* Multiple damage factors can be added with different values of `Y`, and if not provided, `Y` defaults to 0; `Damage factor = X` is equivalent to `Damage factor 0 = X`.
* `Y` is a nonnegative integer from 0 to 9.
* `X` has a minimum value of 0.0.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Wall horizontal bounce (UNDER CONSIDERATION)
- When a Thing is a `BOUNCER` and bounces off of a wall or solid obstacle, its X/Y momentum after bouncing will be multiplied by this value.
- Add `Wall horizontal bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 32768 (0.5) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Wall vertical bounce (UNDER CONSIDERATION)
- When a Thing is a `BOUNCER` and bounces off of a wall or solid obstacle, its Z momentum after bouncing will be multiplied by this value.
- Add `Wall vertical bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 65536 (1.0) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Floor horizontal bounce (UNDER CONSIDERATION)
- When a Thing is a `BOUNCER` and bounces off of a floor or ceiling, its X/Y momentum after bouncing will be multiplied by this value.
- Add `Floor horizontal bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 49152 (0.75) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Floor vertical bounce (UNDER CONSIDERATION)
- When a Thing is a `BOUNCER` and bounces off of a floor (unless the Thing has `ANTIGRAV`, in which case, if it bounces off of a ceiling) its Z momentum after bouncing will be multiplied by this value.
- Add `Floor vertical bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 32768 (0.5) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Max bounces (UNDER CONSIDERATION)
- When a Thing is a `BOUNCER` and a `MISSILE`, it will enter its death state when the number of times it has bounced is greater than this number.
- Add `Max bounces = X` in the Thing definition.
- `X` is an integer greater than 0.

### Counter initial values
To be determined.

### Counter max values
To be determined.

## New and Updated Thing Codepointers
Refer to the MBF21 spec for format information.

It is now possible for frames to have up to ten args by adding `Args9 = X` and `Args10 = X` in the State definition.

### Updated MBF21 Codepointers

* **A_AddFlags(flags, flags2, flags3, flags4)**
    * Adds the specified thing flags to the caller. This command has been updated to support ID24 and MBF2y flags.
    * Args:
        * `flags (int)`: Standard actor flag(s) to add
        * `flags2 (int)`: MBF21 actor flag(s) to add
        * `flags3 (int)`: ID24 actor flag(s) to add
        * `flags4 (int)`: MBF2y actor flag(s) to add
    * Notes:
        * `flags3` and `flags4` will only be processed if the relevant compatibility levels are in use.

* **A_RemoveFlags(flags, flags2, flags3, flags4)**
    * Removes the specified thing flags from the caller. This command has been updated to support ID24 and MBF2y flags.
    * Args:
        * `flags (int)`: Standard actor flag(s) to remove
        * `flags2 (int)`: MBF21 actor flag(s) to remove
        * `flags3 (int)`: ID24 actor flag(s) to remove
        * `flags4 (int)`: MBF2y actor flag(s) to remove
    * Notes:
        * `flags3` and `flags4` will only be processed if the relevant compatibility levels are in use.

* **A_JumpIfFlagsSet(state, flags, flags2, flags3, flags4)**
    * Jumps to a state if the caller has the specified thing flags set. This command has been updated to support ID24 and MBF2y flags.
    * Args:
        * `state (uint)`: State to jump to.
        * `flags (int)`: Standard actor flag(s) to check
        * `flags2 (int)`: MBF21 actor flag(s) to check
        * `flags3 (int)`: ID24 actor flag(s) to check
        * `flags4 (int)`: MBF2y actor flag(s) to check
    * Notes:
        * `flags3` and `flags4` will only be processed if the relevant compatibility levels are in use.

### Actor Movement and Look Codepointers

* **A_Step2(distx, disty, distz, flags)**
    * A generic movement codepointer. Attempts to move the calling thing by the given distance.
    * Args:
        * `distx (int)`: X distance to attempt to move (forward/backward).
        * `disty (int)`: Y distance to attempt to move (left/right).
        * `distz (int)`: Z distance to attempt to move (up/down).
        * `flags (int)`: Allows further customization:
            * `SE_NOUSE (0x001)`: If not set, things that call this codepointer will attempt to interact with doors or switches that they run into. This prevents that behavior.
            * `SE_ALLOWDROPOFF (0x002)`: Allows the thing to move off of ledges even if it lacks `DROPOFF`.
            * `SE_NOSTUCK (0x004)`: Causes the codepointer to no-op if the resulting movement would cause the caller to be stuck on a ledge.
            * `SE_CLIPTHROUGH (0x008)`: Allows movement to continue even if there isn't a direct path, allowing for pseudo-teleportation.
            * `SE_ACTIVESOUND (0x010)`: Performs a check for the caller to make its `Active sound`, provided it has one.
    * Notes:
        * Regardless of whether or not `NOSTUCK` is used, this codepointer will not do anything if it would result in a `SOLID` object moving into an impassible obstacle or other `SOLID` object.
        * Leverages the same movement checks as the fast projectile movement.

* **A_Look2(fov, range, flags)**
    * Similar to `A_Look`, but a maximum distance and field of view can be applied. Optionally, the Thing's `Active sound` can be played as though it were moving.
    * Note that Things with `AMBUSH` enabled will always look in all directions for a target when their sector has a valid `soundtarget` (or `oldsoundtarget`, when `comp_nostickysoundtarget` is enabled).
    * Args:
        * `fov (fixed)`: Field-of-view, relative to calling actor's angle, to search for targets in. If zero, the search will occur in all directions. The default value is `180.0`.
        * `range (uint)`: Maximum distance at which an actor will be able to spot an enemy, in map units. The default value is 0, which means unlimited range.
        * `flags (int)`: Allows further customization:
            * `LO2_ACTIVEOSUND (0x01)`: Perform a random check using the same logic as though the actor were calling `A_Chase` to play the Thing's `Active sound`.
    * Notes:
        * This function will behave exactly the same as `A_Look` aside from the parameterized `fov` and `range`.

* **A_Wander(maxturn, flags)**
    * Makes the calling actor move aimlessly without performing attack checks or `A_Look`.
    * Args:
       * `maxturn (fixed)`: Maximum angle in which the actor will turn in either direction. A value of `0.0` (or, practically, a value equal to or greater than `180.0`) will mean that the actor can turn in any direction. The default value is `0.0`.
        * `flags (int)`: Allows further customization.
            * `WAN_NORANDOMTURN (0x001)`: The actor will not attempt to turn at random, but may still turn when it encounters an obstacle. This can be used to force a thing to move in its current facing direction.
            * `WAN_NOFACEDIR (0x002)`: The actor will not change its angle to face the direction of travel.
            * `WAN_STOPIFBLOCKED (0x004)`: The actor will not automatically turn when colliding with an obstacle and will simply stop moving. It may turn randomly depending on the value of `norandomturn`, however.
            * `WAN_ACTIVESOUND (0x008)`: Perform a random check using the same logic as though the actor were calling `A_Chase` to play the Thing's `Active sound`.
    * Notes:
        * This can be called whether or not the actor as a target.
        * Despite its name, this codepointer is designed to be used both within and outside of an actor's `Wander state` sequence.

* **A_Chase2(meleestate, missilestate, strafechance, straferange, strafespeed, maxturn, flags)**
    * A parameterized version of `A_Chase`.
    * Args:
        * `meleestate (int)`: State to jump to when the actor performs a melee attack. A value of 0 means to use the actor's default `Melee state`, while a negative value means that no melee attack checks should be performed, and the random table should not move.
        * `missilestate (int)`: State to jump to when the actor performs a ranged attack. A value of 0 means to use the actor's default `Missile state`, while a negative value means that no ranged attack checks should be performed, and the random table should not move.
        * `strafechance (uint)`: Probability out of 255 that the actor will start strafing if it's within `straferange` of its target.
        * `strafespeed (uint)`: Speed at which the actor will move while it's strafing.
        * `straferange (fixed)`: Distance in map units from its target at which the actor will perform strafe checks.
        * `maxturn (fixed)`: Maximum angle in which the actor will turn in either direction. A value of `0.0` (or, practically, a value equal to or greater than `180.0`) will mean that the actor can turn in any direction.
        * `flags (int)`: Allows further customization:
            * `CH2_NORANDOMTURN (0x001)`: The actor will not attempt to turn at random, but may still turn when it encounters an obstacle. This can be used to force a thing to move in its current facing direction.
            * `CH2_NOFACEDIR (0x002)`: The actor will not change its angle to face the direction of travel.
            * `CH2_STOPBLOCKED (0x004)`: The actor will not automatically turn when colliding with an obstacle and will simply stop moving. It may turn randomly depending on the state of `Ch2_NORANDOMTURN`, however.
            * `makesound (int)`: Perform a random check using the same logic as though the actor were calling `A_Chase` to play the Thing's `Active sound`.
    * Notes:
        * The strafing behavior is a parameterized version of that found in multiple Heretic/Hexen enemies.

* **A_Patrol(flags)**
    * Causes the calling actor to move its speed in its current direction. It will attempt to activate any linedef specials that it comes into contact with that it can activate and will cause the actor to round its movement to the nearest 45 degree angle and face its movement direction so long as its current frame is calling this codepointer. The actor will also snap to the X and Y position of any `MT_PATROL_POINT` objects that it comes within 16 map units of and change its direction to the `MT_PATROL_POINT` object's direction, though it will not perform the snap and turn if it is already facing in this direction.
    * Args:
        * `flags (int)`: Allows further customization:
            * `PAT_ATTEMPTFLOAT (0x001)`: If the calling actor has NOGRAVITY and FLOAT enabled, if it collides with an upper sidedef or lower sidedef that is not impassible to it and there is room for it to fit vertically into the sector on the other side, it will attempt to lower or raise its Z position until it is able to enter the sector by a maximum value of its Speed (and will use a smaller increment if using the full Speed value would cause it to overshoot).
            * `PAT_DONTUSE (0x002)`: Prevents the calling actor from attempting to use linedefs that it bumps into.
            * `PAT_LOOK (0x004)`: Calls `A_Look` while moving.
            * `PAT_ATTACKCHECK (0x008)`: If the thing has a valid shootable target, it will perform attack checks (including melee attacks) as though `A_Chase` were being called.
            * `PAT_ACTIVESOUND (0x010)`: Perform a random check using the same logic as though the actor were calling `A_Chase` to play the Thing's `Active sound`.
    * Notes:
        * If using patrols as part of a actor's "Look" sequence and then jumping to `A_Chase` when it spots a target, it is recommended to use `A_AddFlags` or `A_RemoveFlags` and `A_JumpIfFlagsSet` to prevent the actor from returning to patrolling as soon as its target is dead. Otherwise, it will simply continue moving in its current cardinal direction as soon as the target is killed.

* **A_ChangeVelocityEx(velx, vely, velz, flags)**
    * Sets caller's own velocity, either to absolute values or relative values.
    * Args:
        * `velx (fixed)`: Amount of X (forward/back) momentum to apply
        * `vely (fixed)`: Amount of Y (left/right) momentum to apply
        * `velz (fixed)`: Amount of Z (up/down) momentum to apply
        * `flags (int)`: Allows further customization:
            * `CVE_DONTIGNOREZERO (0x001)`: Typically, `velx`, `vely`, and `velz` values of 0 are ignored, but this will cause them to be parsed when setting absolute momentum or multiplying momentum.
            * `CVE_ABSOLUTEANGLE (0x002)`: Does not use caller's current angle for `velx` and `vely`.
            * `CVE_ADDXVELOCITY (0x004)`: Adds `velx` to current X velocity.
            * `CVE_ADDYVELOCITY (0x008)`: Adds `vely` to current Y velocity.
            * `CVE_ADDZVELOCITY (0x010)`: Adds `velz` to current Z velocity.
            * `CVE_MULTXVELOCITY (0x020)`: Multiplies current X velocity by `velx`. Supercedes `ADDXVELOCITY`.
            * `CVE_MULTYVELOCITY (0x040)`: Multiplies current Y velocity by `vely`. Supercedes `ADDYVELOCITY`.
            * `CVE_MULTZVELOCITY (0x080)`: Multiplies current Z velocity by `velz`. Supercedes `ADDZVELOCITY`.
        * Notes:
            * Subject to change (and seeking feedback).

### Actor Logic Codepointers

* **A_JumpIfTargetInSightEx(state, fov, flags)**
    * Jumps to a state if caller's target is in line-of-sight. Optionally checks for a direct path between the the caller and its target.
    * Args:
        * `state (uint)`: State to jump to
        * `fov (fixed)`: Field-of-view, relative to calling actor's angle, to check for target in. If zero, the check will occur in all directions.
        * `flags (int)`: Allows further customization:
            * `JIS_CHECKPATH (0x001)`: Checks for `SOLID` obstructions in the path between caller and its target.
            * `JIS_CHECKHITSCAN (0x002)`: Checks for a clear path for a hitscan. Will also check for hitscan-blocking linedefs (but not any self-referencing sectors).
            * `JIS_CHECKLINES (0x004)`: Checks for any impassible linedefs, including "Block monsters" linedefs or "Block land monsters" linedefs as appropriate given the caller's current flags.
        * Notes:
            * Additional flags may be added as needed in future spec revisions.

* **A_JumpIfTracerInSightEx(state, fov, flags)**
    * Jumps to a state if caller's tracer target is in line-of-sight. Optionally checks for a direct path between the the caller and its target.
    * Args:
        * `state (uint)`: State to jump to
        * `fov (fixed)`: Field-of-view, relative to calling actor's angle, to check for tracer target in. If zero, the check will occur in all directions.
        * `flags (int)`: Allows further customization:
            * `JIS_CHECKPATH (0x001)`: Checks for `SOLID` obstructions in the path between caller and its tracer target.
            * `JIS_CHECKHITSCAN (0x002)`: Checks for a clear path for a hitscan. Will also check for hitscan-blocking linedefs (but not any self-referencing sectors).
            * `JIS_CHECKLINES (0x004)`: Checks for any impassible linedefs, including "Block monsters" linedefs or "Block land monsters" linedefs as appropriate given the caller's current flags.
        * Notes:
            * Additional flags may be added as needed in future spec revisions.

* **A_JumpIfAnyPlayerCloser(state, distance, flags)**
    * Jumps to a state if any player mobj is closer than the specified distance.
    * Args:
        * `state (uint)`: State to jump to
        * `distance (fixed)`: Distance threshold, in map units
        * `flats (int)`: Allows further customization:
            * `JIC_SETTARGET (0x001)`: If conditions are met, sets its target to the closest player mobj.
            * `JIC_SETTRACER (0x002)`: If conditions are met, sets its tracer target to the closest player mobj.
            * `JIC_NEEDSLOS (0x004)`: Players are only considered to be in range if line of sight can be drawn between the caller and them (but direction is not important).
    * Notes:
        * Combines MBF21's `A_JumpIfTargetCloser` with a blockmap iterator. As such, the jump will only occur if distance is below the given value.

* **A_JumpIfTargetHealthBelow(state, health)**
    * Jumps to a state if caller's target's health is below the specified threshold.
    * Args:
        * `state (uint)`: State to jump to
        * `health (int)`: Health to check
    * Notes:
        * The jump will only occur if health is BELOW the given value — e.g. `A_JumpIfTargetHealthBelow(420, 69)` will jump to state 420 if the target's health is 68 or lower.

* **A_JumpIfTracerHealthBelow(state, health)**
    * Jumps to a state if caller's tracer target's health is below the specified threshold.
    * Args:
        * `state (uint)`: State to jump to
        * `health (int)`: Health to check
    * Notes:
        * The jump will only occur if health is BELOW the given value — e.g. `A_JumpIfTracerHealthBelow(420, 69)` will jump to state 420 if the tracer target's health is 68 or lower.

* **A_JumpIfTargetZCloser(state, distance, hilo)**
    * Jumps to a state if a target's Z position is closer than the specified vertical distance. The smallest of the following distances is used:
        * From the top of the caller (based on its current height) to the top of the target (based on its current height)
        * From the top of the caller (based on its current height) to the Z position of the target
        * From the Z position of the caller to the top of the target (based on its current height)
        * From the Z position of the caller to the Z position of the target
    * `state(uint):` State to jump to.
    * `distance (fixed)`: Vertical distance threshold, in map units.
    * `hilo (uint)`: 0 = check distance in both directions; 1 = return true even if the target is below the caller past the given distance as long as they aren't above the caller past the given distance; 2 = return true even if the target is above the caller past the given distance as long as they aren't below the caller past the given distance
* Notes:
    * `hilo` may be reconfigured into a bitwise field with aliases similar to flags.

* **A_JumpIfTracerZCloser(state, distance, hilo)**
    * Jumps to a state if a tracer target's Z position is closer than the specified vertical distance. The smallest of the following distances is used:
        * From the top of the caller (based on its current height) to the top of the tracer target (based on its current height)
        * From the top of the caller (based on its current height) to the Z position of the tracer target
        * From the Z position of the caller to the top of the tracer target (based on its current height)
        * From the Z position of the caller to the Z position of the tracer target
    * `state(uint):` State to jump to.
    * `distance (fixed)`: Vertical distance threshold, in map units.
    * `hilo (uint)`: 0 = check distance in both directions; 1 = return true even if the tracer target is below the caller past the given distance as long as they aren't above the caller past the given distance; 2 = return true even if the tracer target is above the caller past the given distance as long as they aren't below the caller past the given distance
* Notes:
    * `hilo` may be reconfigured into a bitwise field with aliases similar to flags.

* **A_JumpIfHasTarget(state, checkhealth, thingtype)**
    * Jumps to a state if the caller has a target set, optionally checking to see if the target is alive or has a specific thing type/
    * Args:
        * `state (uint)`: State to jump to.
        * `checkhealth (uint)`: If nonzero, only jump if the target has more than 0 hit points remaining.
        * `thingtype (uint)`: Thing type to check for.
    * Notes:
        * `checkhealth` should not care if the target is `SHOOTABLE` or not.

* **A_JumpIfHasTracer(state, checkhealth, thingtype)**
    * Jumps to a state if the caller has a tracer target set, optionally checking to see if the tracer target is alive or has a specific thing type/
    * Args:
        * `state (uint)`: State to jump to.
        * `checkhealth (uint)`: If nonzero, only jump if the tracer target has more than 0 hit points remaining.
        * `thingtype (uint)`: Thing type to check for.
    * Notes:
        * `checkhealth` should not care if the tracer target is `SHOOTABLE` or not.

* **A_JumpIfTargetFlagsSet(state, flags, flags2, flags3, flags4)**
    * Jumps to a state if the caller's target has the specified thing flags set. This command has been updated to support ID24 and MBF2y flags.
    * Args:
        * `state (uint)`: State to jump to.
        * `flags (int)`: Standard actor flag(s) to check
        * `flags2 (int)`: MBF21 actor flag(s) to check
        * `flags3 (int)`: ID24 actor flag(s) to check
        * `flags4 (int)`: MBF2y actor flag(s) to check
    * Notes:
        * Will no-op if the caller has no target.

* **A_JumpIfTracerFlagsSet(state, flags, flags2, flags3, flags4)**
    * Jumps to a state if the caller's tracer target has the specified thing flags set. This command has been updated to support ID24 and MBF2y flags.
    * Args:
        * `state (uint)`: State to jump to.
        * `flags (int)`: Standard actor flag(s) to check
        * `flags2 (int)`: MBF21 actor flag(s) to check
        * `flags3 (int)`: ID24 actor flag(s) to check
        * `flags4 (int)`: MBF2y actor flag(s) to check
    * Notes:
        * Will no-op if the caller has no tracer target.

* **A_JumpIfBlocked(state, angle, ofs_x, ofs_y)**
    * Jumps to state if caller's position, or optionally a relative position, is blocked by a `SOLID` Thing sidedefs that the Thing would not be able to cross.
    * Args:
        * `state (int)`: State to jump to.
        * `angle (fixed)`: Angle (degrees), relative to calling actor's angle.
        * `x_ofs (fixed)`: X (forward/back) position offset to check.
        * `y_ofs (fixed)`: Y (left/right) position offset to check.
    * Notes:
        * Does not check for a clear path.
        * When using the `ofs_x` and `ofs_y` values to check a relative position, the logic is essentially that the actor is "moved" to the destination, the collision check is performed, and then it is immediately "moved" back to its original position.

* **A_JumpIfSkill(skillnum, state, orhigher)**
    * Jumps to state based on the current skill selection.
    * Args:
        * `skillnum (uint)`: ID of the skill to check for. By default, 0 = ITYTD, 1 = HNTR, 2 = HMP, 3 = UV, 4 = Nightmare.
        * `state (uint)`: State to jump to if conditions are met.
        * `orhigher (uint)`: If nonzero, will also jump if the current skill is higher than the given value.
    * Notes:
        * If custom skills are defined, this is based off of the ID of the skill as defined in `UMAPINFO`.
        * In the case of `ZMAPINFO`-defined skills, as it defines skills by name and not by ID, IDs will correlate to the skills in the order in which they're defined.

* **A_MultiJump(state1, value1, state2, value2 ... state20, value20)**
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

### Target and Tracer Manipulation Codepointers

* **A_AddTargetFlags(flags, flags2, flags3, flags4)**
    * Adds the specified thing flags to the caller's target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to add
        * `flags2 (int)`: MBF21 actor flag(s) to add
        * `flags3 (int)`: ID24 actor flag(s) to add
        * `flags4 (int)`: MBF2y actor flag(s) to add
    * Notes:
        * Will no-op if the caller has no target.

* **A_AddTracerFlags(flags, flags2, flags3, flags4)**
    * Adds the specified thing flags to the caller's tracer target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to add
        * `flags2 (int)`: MBF21 actor flag(s) to add
        * `flags3 (int)`: ID24 actor flag(s) to add
        * `flags4 (int)`: MBF2y actor flag(s) to add
    * Notes:
        * Will no-op if the caller has no tracer target.

* **A_RemoveTargetFlags(flags, flags2, flags3, flags4)**
    * Removes the specified thing flags from the caller's target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to remove
        * `flags2 (int)`: MBF21 actor flag(s) to remove
        * `flags3 (int)`: ID24 actor flag(s) to remove
        * `flags4 (int)`: MBF2y actor flag(s) to remove
    * Notes:
        * Will no-op if the caller has no target.

* **A_RemoveTracerFlags(flags, flags2, flags3, flags4)**
    * Removes the specified thing flags from the caller's tracer target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to remove
        * `flags2 (int)`: MBF21 actor flag(s) to remove
        * `flags3 (int)`: ID24 actor flag(s) to remove
        * `flags4 (int)`: MBF2y actor flag(s) to remove
    * Notes:
        * Will no-op if the caller has no tracer target.

* **A_ClearTarget**
    * Clears the caller's target field.
    * No args.

### Parameterized Monster Attack Codepointers

* **A_MonsterMeleeAttackEx(damagebase, damagedice, sound, range, flatdamage, damagetype, flags)**
    * Generic monster melee attack. This command has been updated to support an optional flat damage parameter and damage type.
    * Args:
        * `damagebase (uint)`: Base damage of attack; if not set, defaults to 3
        * `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to 8
        * `sound (uint)`: Sound to play if attack hits
        * `range (fixed)`: Attack range; if not set, defaults to calling actor's melee range property
        * `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to 0
        * `damagetype (uint)`: Damage type of attack; if not set, defaults to 0
    * Notes:
        * Damage formula is: `damage = (damagebase * random(1, damagedice) + flatdamage)` unless either `damagebase` or `damagedice` is 0, in which case, `damage = flatdamage`.

* **A_MonsterProjectileEx(type, angle, pitch, hoffset, voffset, meleestate, meleedistance, flags)**
    * * Generic monster projectile attack that includes support for combination attacks by diverting to another frame.
    * Args:
        * `type (uint)`: Type (dehnum) of actor to spawn
        * `angle (fixed)`: Angle (degrees), relative to calling actor's angle
        * `pitch (fixed)`: Pitch (degrees), relative to calling actor's pitch
        * `hoffset (fixed)`: Horizontal spawn offset, relative to calling actor's angle
        * `voffset (fixed)`: Vertical spawn offset, relative to actor's default projectile fire height
        * `meleestate (int)`: State to jump to if a melee attack is made instead
        * `meleedistance (fixed)`: Distance, in map units, at which the melee attack should be made instead
        * `flags (int)`: Allows further customzation:
            * `MPE_NOFACETARGET (0x001)`: Skips the `A_FaceTarget` check.
            * `MPE_TARGETLESS (0x002)`: Performs the attack without any expectation of a target in the given direction. Implies `MPE_NOFACETARGET`.
    * Notes:
        * The spawned projectile's `tracer` pointer is always set to the spawner's `target`, for generic seeker missile support.
        * The `meleestate` is jumped to immediately if conditions are met.

* **A_MonsterZapTarget(type)**
    * Parameterized version of `A_VileTarget` that allows choosing what thing gets spawned in place of the "fire".
    * Args:
        * `type (uint)`: Thing ID to spawn over the monster's target if in line of sight.
    * Notes:
        * Otherwise identical to `A_VileTarget`.

* **A_MonsterZapAttack(damagebase, damagedice, flatdamage, blastdamage, blastradius, thrust, sound, damagetype, flags)**
    * Parameterized version of `A_VileAttack` that allows customization.
    * Args:
        * `damagebase (uint)`: Base damage of the hitscan attack; if not set, defaults to 0
        * `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to 0
        * `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to 20
        * `blastdamage (uint)`: Maximum blast damage to do if a tracer is present; if not set, defaults to 70
        * `thrust (int)`: Amount of vertical momentum to apply; if not set, defaults to 1000
        * `sound (uint)`: Sound to play if attack succeeds; if not set, defaults to 0 (no sound)
        * `flags (int)`: Allows further customization:
            * `MZA_ALWAYSPLAYSOUND (0x01)`: Sound will play even if attack fails.
            * `MZA_ASSUMEPLAYERMASS (0x02)`: Regardless of the target's mass, it will receive the same amount of momentum as though it had the player's default mass.
            * `MZA_NOLINEOFSIGHT (0x04)`: The attack will succeed no matter what, even if the caller cannot see its target.

* **A_MonsterChargeAttack(angle, hspread, vspread, speed)**
    * Parameterized version of `A_SkullAttack`.
    * Args:
        * `angle (fixed)`: Angle (degrees), relative to calling actor's angle
        * `hspread (fixed)`: Horizontal spread (degrees), in fixed point.
        * `vspread (fixed)`: Vertical spread (degrees), in fixed point.
        * `speed (int)`: Speed at which the caller will launch itself; defaults to 20 if not set
        * `nofacetarget`: If nonzero, do not call `A_FaceTarget` before attacking.
    * Notes:
        * As with projectiles, damage calculations and damage type are handled by the the thing's properties.

* **A_MonsterChargeSeek(threshold, maxturnangle, pitchthreshold, maxpitchchange)**
    * Version of `A_SeekTracer` that is intended for charging monsters and doesn't rely on the tracer field.
    * Args:
        * `threshold (fixed)`: If angle to target is lower than this, monster will 'snap' directly to face its target
        * `maxturnangle (fixed)`: Maximum angle a monster will turn towards its target if angle is above the threshold
        * `pitchthreshold (fixed)`: If pitch to target is lower than this, monster will adjust its pitch to target its target
        * `maxpitchchange (fixed)`: Maximum change in angle a monster will change its pitch toward its target if pitch is above the threshold
    * Notes:
        * Implementation requires calculation of the angle in pitch between two things.

* **A_MonsterSpawnAttack(type, state, angle, hspread, vspread)**
    * Parameterized version of `A_PainAttack`.
    * Args:
        * `type (uint)`: Thing type to fire
        * `state (uint)`: If nonzero, state at which to set the thing; otherwise, use its Missile frame if it has one; if it has no Missile frame, then use its Spawn frame as a fallback 
        * `angle (fixed)`: Angle (degrees), relative to calling actor's angle
        * `hspread (fixed)`: Horizontal spread (degrees), in fixed point.
        * `vspread (fixed)`: Vertical spread (degrees), in fixed point.

* **A_MonsterSpawnDie(type, state, number, flags)**
    * Parameterized version of `A_PainDie` that kills the caller and then fires a number of the specified actor.
    * Args:
        * `type (uint)`: Thing type to spawn.
        * `state (uint)`: If nonzero, state at which to set the spawned thing; otherwise, use its Missile frame if it has one; if it has no Missile frame, the nuse its Spawn frame as a fallback
        * `number (uint)`: Number of the thing to attempt to spawn; defaults to 1 if not set and has a maximum value of 5
        * `flags (int)`: Allows further customization:
            * `MSD_SPAWNRANDOMAMOUNT (0x01)`: If `number` is more than 1, will randomly spawn between 1 and that value based on RNG. The RNG should be similar to generating "Damage dice" values and advance the RNG table identically if this flag is set.
            * `MSD_INHERITTARGET (0x02)`: Automatically sets the spawned thing's target to the caller's target if it has one.
    * Notes:
        * A hard-coded array of angles at which to spawn things are provided for each valid value of `number`.

### Miscellaneous Actor Codepointers

* **A_ProjectileSpray(range, damagebase, damagedice, flatdamage, damagetype, splashthing, flags)**
    * Parameterized version of `A_BFGSpray` that allows customizing damage and visual indicators.
    * Args:
        * `range (uint)`: Maximum range (in map units) from the caller's target at which to deal damage; defaults to 1024 if not set
        * `damagebase (uint)`: Base damage of each ray; if not set, defaults to 7
        * `damagedice (uint)`: Attack damage random multiplier per ray; if not set, defaults to 7
        * `flatdamge (uint)`: Flat damage to add to attack; if not set, defaults to 40
        * `damagetype (uint)`: Damage type to use for the attack; if not set, defaults to 0
        * `splashthing (int)`: Thing type to spawn over a thing struck by an attack; if not set, defaults to no thing
        * `flags (int)`: Allows further customization:
            * `PS_ONLYSPRAYONCE (0x01)`: Each thing found will only be hit by one tracer
            * `PS_USEPROJECTILELOS (0x02)`: Instead of using the caller's target, will look in all 360 degrees from itself; best used with `ONLYSPRAYONCE`.
            * `PS_DONTIGNORETARGET (0x04)`: Usually, the spray check will ignore the caller's target; if this is set, then the caller's target can be targeted as well
    * Notes:
        * As with `A_BFGSpray`, will ignore things with `NOSPRAY` set.

* **A_SetHealth(health)**
    * Sets caller's health to the specified value.
    * Args:
        * `sethealth (int)`: Amount to set health to.
    * Notes:
        * To prevent bad logic, has no effect if the caller's health is currently 0 or lower.

* **A_RegainHealth(healamount, max)**
    * Heals caller by a specified amount, up to a specified maximum.
    * Args:
        * `healamount (int)`: Amount of health to regain. Note that this can be negative, though this skips the usual damage checks (no pain state, no blood, no targeting changes, etc).
        * `max (uint)`: The maximum amount of health to heal up to; if 0, uses the thing's `spawnhealth` instead; if not set, defaults to 0
    * Notes:
        * To prevent bad logic, has no effect if the caller's health is currently 0 or lower.

* **A_DropItem(thingtype, nodropped)**
    * Can be used to make a thing drop another thing and/or a random spawner, even outside of the death state.
    * Args:
        * `thingtype (int)`: Thing to drop. If negative, no thing is dropped (presumably to use a random spawner instead); if not set, defaults to -1
        * `nodropped (uint)`: If nonzero, the dropped thing/random spawner spawn thing will not have the `DROPPED` flag set.
    * Notes:
        * Functionally, just re-uses the existing drop item code.

* **A_SelfRaise**
    * Self-resurrecting codepointer. Similar in many ways to the `A_HealChase` or `A_VileChase` codepointers' heal functions, but can only be called on self and can be used on any duration of a frame.
    * No args.
    * Notes:
        * Does not check for frame length; simply jumps to the Raise state and resurrects the caller if it is currently a `CORPSE` as though it had been resurrected via `A_HealChase` or `A_VileChase`.



### Map Manipulation Codepointers

* **A_LineEffectEx(special, tag)**
    * Remotely executes a Doom or parameterized Boom linedef special.
    * Args:
        * `special (uint)`: Linedef special to execute.
        * `tag (uint)`: Sector tag to execute the special against.
    * Notes:
        * Unlike `A_LineEffect`, linedef specials that require a front sector will execute, using the information based on the sector the caller is currently within.