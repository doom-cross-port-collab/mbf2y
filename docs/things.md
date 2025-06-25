# MBF25 Thing Specification v0.2.1
MBF25 expands upon the work done in MBF21 and ID24, vastly expanding modding potential primarily by leveraging existing code.

## Additional Dehacked Thing Groups

Expanding and building upon the functionaluty of MBF21's `Infighting group`, `Projectile group`, and `Splash group`, additional groups have beeen created.

### Multiple Group Support
Up to 3 additional Infighting, Projectile, and Splash Groups can be defined on a single Thing, as well as the new Groups outlined below.
* Example: `Infighting group = M` and `Infighting group 2 = N` in Thing definition.
    * In this example, the Thing would not engage in infighting with a target that has any Infighting group that matches `M` or `N`.
* DEHACKED parsers should treat `{grouptype} group 1` as equivalent to `{grouptype} group` for user friendliness.
* Things can have `{grouptype} group 1`, `{grouptype} group 2`, `{grouptype} group 3`, and `{grouptype} group 4` defined.
* Note: Things with a Projectile Group of -1 will still take projectile damage from other Things in the same species despite any other Projectile Groups they may share. 

### Projectile Collision Groups
* Add `Projectile collision group = C` in Thing definition.
* `C` is a nonnegative integer.
* Projectiles with the same value of `C` will skip all collision detections with Things with the same value of `C`, completely ignoring them. Ripper projectiles will not play their rip sound, spawn blood objects, or trigger pain states when this happens.

### Hitscan Groups
- Add `Hitscan group = H` in Thing definition.
- `H` is a nonnegative integer.
- Hitscan attacks from Things with a value of `H` will produce puff things instead of blood things and not damage other `SHOOTABLE` things with the same value of `H`.

#### Examples
```
Thing 12 (Imp)
Projectile group = -1
Projectile group 2 = 2
Infighting group = 1

Thing 16 (Baron of Hell)
Projectile group = 2
Infighting group = 1
Infighting group 2 = 2

Thing 18 (Hell Knight)
Projectile collision group = 1

Thing 21 (Arachnotron)
Infighting group = 2

Thing 36 (Arachnotron plasma ball)
Projectile collision group = 1
```

In this example:
* Imp projectiles now damage other Imps, but they will not infight with their own kind.
* Imps are immune to Baron projectiles, and Barons are immune to Imp projectiles.
* Barons and Imps will not infight even if they somehow take damage from one another.
* Barons and Arachnotrons will not infight.
* Arachnotron plasma balls will not impact with or deal direct damage to Hell Knights.

## New Thing Flags

* Add `MBF25 Bits = X` in the Thing definition.
* The format is the same as the existing `Bits`, `MBF21 Bits`, and `ID24 Bits` fields.
* Example: `MBF25 Bits = NODAMAGE+ANTITELEFRAG`.
* The value can also be given as a number (sum of the individual flag values below).

| Flag              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                              | Value            |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------|
| NODAMAGE          | Thing does not lose health when taking damage.                                                                                                                                                                                                                                                                                                                                                                                                           | 0x0000000000001  |
| NOCRUSH           | If Thing is a `CORPSE`, it does not turn into gibs when crushed. If Thing is `DROPPED`, it does not disappear when crushed. (Thing will still take damage from crushers.)                                                                                                                                                                                                                                                                                | 0x0000000000002  |
| NOTAUTOAIMED      | Player hitscan autoaim (but not BFG-style spray attacks) ignore this Thing completely.                                                                                                                                                                                                                                                                                                                                                                   | 0x0000000000004  |
| NOSPRAY           | `A_BFGSpray` and `A_Spray` ignore this Thing completely.                                                                                                                                                                                                                                                                                                                                                                                                 | 0x0000000000008  |
| ANTITELEFRAG      | Things that would telefrag this thing are telefragged instead.                                                                                                                                                                                                                                                                                                                                                                                           | 0x0000000000010  |
| PUSHABLE          | `SOLID` Thing can be pushed by other `SOLID` Things. This uses an identical method to Hexen.                                                                                                                                                                                                                                                                                                                                                             | 0x0000000000020  |
| CANNOTPUSH        | `SOLID` Thing cannot push other `PUSHABLE` objects.                                                                                                                                                                                                                                                                                                                                                                                                      | 0x0000000000040  |
| FLOATBOB          | GZDoom-style renderer-only floatbobbing.                                                                                                                                                                                                                                                                                                                                                                                                                 | 0x0000000000080  |
| DEADFLOAT         | Thing does not lose the `NOGRAVITY` flag upon death. This is turned on for Lost Souls by default.                                                                                                                                                                                                                                                                                                                                                        | 0x0000000000100  |
| ONLYSLAMSOLID     | When charging, Thing does not lose momentum when colliding with a non-`SOLID` Thing.                                                                                                                                                                                                                                                                                                                                                                     | 0x0000000000200  |
| KEEPCHARGETARGET  | Thing remembers its target after colliding after a charge.                                                                                                                                                                                                                                                                                                                                                                                               | 0x0000000000400  |
| NOSTOPCHARGE      | When charging, Thing does not lose momentum or stop charging when it receives non-lethal damage.                                                                                                                                                                                                                                                                                                                                                         | 0x0000000000800  |
| TUNNEL            | Ripper projectile remembers the last Thing it damaged and will not damage the same Thing again until it damages a different Thing.                                                                                                                                                                                                                                                                                                                       | 0x0000000001000  |
| NORIPBLOOD        | Ripper projectile does not produce blood or puffs when damaging a `SHOOTABLE` Thing.                                                                                                                                                                                                                                                                                                                                                                     | 0x0000000002000  |
| ACTIVATOR         | Thing can activate most player-activated linedefs as though it were a voodoo doll.                                                                                                                                                                                                                                                                                                                                                                       | 0x0000000004000  |
| NOWARP            | Thing does not trigger Z teleporters.                                                                                                                                                                                                                                                                                                                                                                                                                    | 0x0000000008000  |
| COPYONWARP        | If Thing triggers a Z teleporter, instead of teleporting, it spawns an exact duplicate of itself in the destination spot. This can be used to create seamless (or at least, less seamful) visual transitions when using Z teleporters.                                                                                                                                                                                                                   | 0x0000000010000  |
| SOLIDMISSILE      | `MISSILE`, `BOUNCES`, or `BOUNCER` thing collides with all impassible linedefs, including lower/upper sky sidedefs.                                                                                                                                                                                                                                                                                                                                      | 0x0000000020000  |
| LOYAL             | Thing will never infight with other Things of the same "faction." For example, a non-`FRIENDLY` monster with this flag may take damage from other non-`FRIENDLY` monsters, but it will never target them deliberately. A `FRIENDLY` monster with this flag will never deliberately target a Player or another `FRIENDLY` monster after taking damage.                                                                                                    | 0x0000000040000  |
| KEEPTARGET        | Once Thing obtains a target, it will not change its target until the original target is dead.                                                                                                                                                                                                                                                                                                                                                            | 0x0000000080000  |
| ONLYTARGETDMG     | If this Thing has a living target, then only damage sourced from its target can damage it. This implies `KEEPTARGET`.                                                                                                                                                                                                                                                                                                                                    | 0x0000000100000  |
| ISMONSTER         | Thing is considered a monster. It does not spawn when using the `-nomonsters` command-line parameter, and any cheat codes or source port-specific commands that affect all monsters, such as DSDA-Doom's `TNTEM` or GZDoom's `kill monsters` console command, should also affect this Thing.                                                                                                                                                             | 0x0000000200000  |
| BOUNCER           | Alternative to MBF's janky `BOUNCE` flag. Designed to be combined with the `MISSILE`, `BOUNCEONWALLS`, `BOUNCEONSOLID`,<br>`BOUNCEONSHOOTABLE`, `BOUNCEONFLOOR`,  `BOUNCEONCEILING`, and/or `BOUNCEONSKY` flags.                                                                                                                                                                                                                                         | 0x0000000400000  |
| BOUNCEONWALLS     | `BOUNCER` Thing bounces off of vertical surfaces, such as one-sided linedefs or upper and lower sidedefs. It will still pass through sky sidedefs if it is a `MISSILE`, the linedef does not have the `Block projectiles` flag, and the Thing does not have the `SOLIDMISSILE` and/or `BOUNCEONSKY` flags.                                                                                                                                               | 0x0000000800000  |
| BOUNCEONSOLID     | `BOUNCER` Thing bounces off of `SOLID` Things. If it is a `MISSILE`, it will still explode on collision with a `SHOOTABLE` Thing unless it also has the `BOUNCEONSHOOTABLE` flag.                                                                                                                                                                                                                                                                        | 0x0000001000000  |
| BOUNCEONSHOOTABLE | `MISSILE` and `BOUNCER` Thing bounces off of `SHOOTABLE` Things. It will still inflict damage upon impact.                                                                                                                                                                                                                                                                                                                                               | 0x0000002000000  |
| BOUNCEONFLOOR     | `BOUNCER` Thing bounces off of non-sky floors.                                                                                                                                                                                                                                                                                                                                                                                                           | 0x0000004000000  |
| BOUNCEONCEILING   | `BOUNCER` Thing bounces off of non-sky ceilings.                                                                                                                                                                                                                                                                                                                                                                                                         | 0x0000008000000  |
| BOUNCEONSKY       | If `BOUNCER` Thing has `BOUNCEONFLOOR`, will bounce on sky floors. If `BOUNCER` Thing has `BOUNCEONCEILING`, will bounce on sky ceilings.                                                                                                                                                                                                                                                                                                                | 0x0000010000000  |
| ANTIGRAV          | Thing will "fall" toward the ceiling and come at rest when it hits a the ceiling instead of the floor. This can be combined with `LOGRAV`.                                                                                                                                                                                                                                                                                                               | 0x00000020000000 |
| CANTLEAVEFLOORPIC | Thing cannot cross into a sector with a different floor texture.                                                                                                                                                                                                                                                                                                                                                                                         | 0x0000040000000  |
| FLOORHUGGER       | Projectile moves along the floor and will pass over all obstacles until it hits a solid wall (or, if not a ripper projectile, a `SOLID` and `SHOOTABLE` thing).                                                                                                                                                                                                                                                                                          | 0x0000080000000  |
| FORCEBLOOD        | `MISSILE` Thing will cause a `SOLID` `SHOOTABLE` Thing that it hits to produce blood or a puff as if it were a hitscan attack.                                                                                                                                                                                                                                                                                                                           | 0x0000100000000  |
| PAINLESS          | Things that receive radius or missile damage from this Thing will never enter their pain state.                                                                                                                                                                                                                                                                                                                                                          | 0x0000200000000  |
| TOUCHYTARGET      | Thing will set its target to the last living `SOLID` and `SHOOTABLE` thing to touch it.                                                                                                                                                                                                                                                                                                                                                                  | 0x0000400000000  |
| TOUCHYUSE         | Thing will set its target to the last living `SOLID` and `SHOOTABLE` thing to activate its Use state.                                                                                                                                                                                                                                                                                                                                                    | 0x0000800000000  |
| NOPAIN            | Thing will never enter its pain state while taking damage.                                                                                                                                                                                                                                                                                                                                                                                               | 0x0001000000000  |
| EXTREMEDEATH      | If Thing is a `MISSILE`, hitscan puff, telefrags another thing, or directly causes radius damage and a Thing is killed by it, it will always trigger the XDeath state.                                                                                                                                                                                                                                                                                   | 0x0002000000000  |
| NOEXTREMEDEATH    | If Thing is a `MISSILE`, hitscan puff, telefrags another thing, or directly causes radius damage and a Thing is killed by it, it will never trigger the XDeath state.                                                                                                                                                                                                                                                                                    | 0x0004000000000  |
| RESETONDEATH      | Thing's `Counter1`, `Counter2`, `Counter3`, and `Counter4` values are reset to their initial values upon death.                                                                                                                                                                                                                                                                                                                                          | 0x0008000000000  |
| COLLIDEONZMOVE    | Collision with other Things is checked even when a `MISSILE` object only has vertical momentum.                                                                                                                                                                                                                                                                                                                                                          | 0x0010000000000  |
| PASSMOBJ          | Even if it's `SOLID`, this Thing performs Z-collision checks with all other Things, and other Things perform Z-collision checks with it. If another Thing is standing on this Thing when its Z position moves, the other Thing will change its Z position as well. (To be determined: how this interacts with `SPAWNONCEILING` + `NOGRAVITY` objects on ceilings that move down and potentially cause collisions, or when anchored to a moving floor...) | 0x0020000000000  |
| HITOWNER          | `MISSILE` can collide with its owner. For best results, this should be added a couple of tics after the projectile spawns.                                                                                                                                                                                                                                                                                                                               | 0x0040000000000  |
| NODAMAGETHRUST    | Other Things that take damage directly from this Thing (such as if it's a `MISSILE`, a hitscan puff, an object calling `A_RadiusDamage`, or a monster calling a melee attack codepointer) are not pushed by the damage.                                                                                                                                                                                                                                  | 0x0080000000000  |
| NOTHRUST          | Sources of damage do not move this Thing at all.                                                                                                                                                                                                                                                                                                                                                                                                         | 0x0100000000000  |
| FRAGILE           | Any damage that would affect this Thing, as well as falling from a height, causes it to enter its Crash state.                                                                                                                                                                                                                                                                                                                                           | 0x0200000000000  |
| HURTONTOUCH       | Solid, non-`MISSILE` object inflicts damage on collision with `SHOOTABLE` objects as though it were a projectile, but doesn't enter its Death state.                                                                                                                                                                                                                                                                                                     | 0x0400000000000  |
| DONTHITTHINGS     | This Thing never collides with other Things. Useful for particle effects that might have `MISSILE` set.                                                                                                                                                                                                                                                                                                                                                  | 0x0800000000000  |
| GENERIC1          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x1000000000000  |
| GENERIC2          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x2000000000000  |
| GENERIC3          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x4000000000000  |
| GENERIC4          | Generic flag to be used for custom logic gates.                                                                                                                                                                                                                                                                                                                                                                                                          | 0x8000000000000  |


## Map Object Flags

`GENERIC1`, `GENERIC2`, `GENERIC3`, and `GENERIC4` should be exposed to the map editor like the `AMBUSH` and `FRIENDLY` flags. 

## New Thing Properties

### Initial sector tag
When a thing spawns (including when the map begins), it should store the tag of its current sector to a value called `initialsectortag`. This is only used for a couple of codepointers.

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

These counters are instanced per Thing. While `counter1` ... `counter4` can be manipulated at runtime via codepointers, `counter1max` ... `counter4max` are static values inherited entirely from the Thing's mobjinfo. Unless the Thing has the `RESETONDEATH` MBF25 flag set, the values of `counter1` ... `counter4` will persist even when a Thing dies and has been resurrected. On the other hand, it is expected that if a Thing dies and respawns via `-respawn` or Nightmare! difficulty, it will reinitialize its counter values, as this is technically an entirely new Thing.

When a codepointer would result in a counter being below 0 or above its maximum value, it will be set to 0 or the max value appropriately.

For more information on these, check the Thing Properties and Thing Codepointers subsections.

### Dropped Spawner
* Can be used in place of `Dropped item` to cause a thing to drop a random thing upon death.
* Add `Dropped spawner = X` in the Thing definition.
* `X` corresponds to a random spawner (see `spawners.md`).
* Defaults to -1 if not set, which means no random spawn.
* It is possible for both `Dropped spawner` and `Dropped item` to be set; both fields should be used, starting with the `Dropped item`.

### ID24HACKED Pickup Extensions
* To be determined (and scope altered, if ID24HACKED adopts `Pickup heath amount` and `Pickup armor amount`)

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

### Damage type
- When used on a puff or projectile, determines the damage type for the purposes of damage type factors and damage type Pain/Death/XDeath states.
- Defaults to 0 if not set, which means untyped damage.
- Add `Damage type = X` in the Thing definition.
- `X` is a nonnegative integer from 0 to 9.

### Damage type pain frames
* If the source of damage that triggers this Thing's pain state is typed, then the Thing will jump to this state instead of its default pain state, if available.
* Add `Injury frame Y = X` in the Thing definition.
* `Y` is a nonnegative integer from 0 to 9 and corresponds to a damage type.
* `X` represents a frame number.
* If not set, the thing will use its default pain state (if one exists).

### Damage type death frames
* If the source of damage that triggers this Thing's death state is typed, then the Thing will jump to this state instead of its default death state, if available.
* Add `Death frame Y = X` in the Thing definition.
* `Y` is a nonnegative integer from 0 to 9 and corresponds to a damage type.
* `X` represents a frame number.
* If not set, the thing will use its default death state.

### Damage type exploding frames
* If the source of damage that triggers this Thing's exploding state is typed, then the Thing will jump to this state instead of its default exploding state, if available.
* Add `Exploding frame Y = X` in the Thing definition.
* `Y` is a nonnegative integer from 0 to 9 and corresponds to a damage type.
* `X` represents a frame number.
* If not set, the thing will default to the death state of the same damage type if one exists. If not, it will default to its default exploding state if one exists. If not, it will default to its default death state.

### Damage factor
- Multiplies damage that the Thing receives from damage sources with the corresponding typed damage.
* Add `Damage factor Y = X` in the Thing definition.
* `X` Defaults to 65536 if not set, which is equivalent to 1.0.
* A value of 0 or 0.0 means that a Thing is completely immune to damage from the specified type.
* Multiple damage factors can be added with different values of `Y`, and if not provided, `Y` defaults to 0; `Damage factor = X` is equivalent to `Damage factor 0 = X`.
* `Y` is a nonnegative integer from 0 to 9.
* `X` has a minimum value of 0.0.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Wall horizontal bounce
- When a Thing is a `BOUNCER` and bounces off of a wall or solid obstacle, its X/Y momentum after bouncing will be multiplied by this value.
- Add `Wall horizontal bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 32768 (0.5) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Wall vertical bounce
- When a Thing is a `BOUNCER` and bounces off of a wall or solid obstacle, its Z momentum after bouncing will be multiplied by this value.
- Add `Wall vertical bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 65536 (1.0) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Floor horizontal bounce
- When a Thing is a `BOUNCER` and bounces off of a floor or ceiling, its X/Y momentum after bouncing will be multiplied by this value.
- Add `Floor horizontal bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 49152 (0.75) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Floor vertical bounce
- When a Thing is a `BOUNCER` and bounces off of a floor (unless the Thing has `ANTIGRAV`, in which case, if it bounces off of a ceiling) its Z momentum after bouncing will be multiplied by this value.
- Add `Floor vertical bounce = X`'` in the Thing definition.
- `X` is a nonnegative number and defaults to 32768 (0.5) if not set.
* Ideally, parsers should accept fixed notation as well as decimal values.

### Max bounces
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
    * Adds the specified thing flags to the caller. This command has been updated to support ID24 and MBF25 flags.
    * Args:
        * `flags (int)`: Standard actor flag(s) to add
        * `flags2 (int)`: MBF21 actor flag(s) to add
        * `flags3 (int)`: ID24 actor flag(s) to add
        * `flags4 (int)`: MBF25 actor flag(s) to add
    * Notes:
        * `flags3` and `flags4` will only be processed if the relevant compatibility levels are in use.

* **A_RemoveFlags(flags, flags2, flags3, flags4)**
    * Removes the specified thing flags from the caller. This command has been updated to support ID24 and MBF25 flags.
    * Args:
        * `flags (int)`: Standard actor flag(s) to remove
        * `flags2 (int)`: MBF21 actor flag(s) to remove
        * `flags3 (int)`: ID24 actor flag(s) to remove
        * `flags4 (int)`: MBF25 actor flag(s) to remove
    * Notes:
        * `flags3` and `flags4` will only be processed if the relevant compatibility levels are in use.

* **A_JumpIfFlagsSet(state, flags, flags2, flags3, flags4)**
    * Jumps to a state if the caller has the specified thing flags set. This command has been updated to support ID24 and MBF25 flags.
    * Args:
        * `state (uint)`: State to jump to.
        * `flags (int)`: Standard actor flag(s) to check
        * `flags2 (int)`: MBF21 actor flag(s) to check
        * `flags3 (int)`: ID24 actor flag(s) to check
        * `flags4 (int)`: MBF25 actor flag(s) to check
    * Notes:
        * `flags3` and `flags4` will only be processed if the relevant compatibility levels are in use.

* **A_MonsterMeleeAttack(damagebase, damagedice, sound, range, flatdamage, damagetype)**
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

* **A_MonsterBulletAttack(hspread, vspread, numbullets, damagebase, damagedice, flatdamage, hitscantype)**
    * Generic monster bullet attack. This command has been updated to support an optional flat damage parameter as well as hitscan type.
    * Args:
        * `hspread (fixed)`: Horizontal spread (degrees, in fixed point)
        * `vspread (fixed)`: Vertical spread (degrees, in fixed point)
        * `numbullets (uint)`: Number of bullets to fire; if not set, defaults to 1
        * `damagebase (uint)`: Base damage of attack; if not set, defaults to 3
        * `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to 5
        * `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to 0
        * `hitscantype (uint)`: Hitscan type to use for the bullet puff and damage type; if not set, defaults to 0 (the default hitscan type)
    * Notes:
        * Damage arg defaults are identical to Doom's monster bullet attack damage values.
        * Damage formula is: `damage = (damagebase * random(1, damagedice) + flatdamage)` unless either `damagebase` or `damagedice` is 0, in which case, `damage = flatdamage`.

### Actor Movement and Look Codepointers

* **A_LookEx(fov, range, makesound)**
    * Similar to `A_Look`, but a maximum distance and field of view can be applied. Optionally, the Thing's `Active sound` can be played as though it were moving.
    * Note that Things with `AMBUSH` enabled will always look in all directions for a target when their sector has a valid `soundtarget` (or `oldsoundtarget`, when `comp_nostickysoundtarget` is enabled).
    * Args:
        * `fov (fixed)`: Field-of-view, relative to calling actor's angle, to search for targets in. If zero, the search will occur in all directions. The default value is `180.0`.
        * `range (uint)`: Maximum distance at which an actor will be able to spot an enemy, in map units. The default value is 0, which means unlimited range.
        * `makesound (int)`: If nonzero, perform a random check using the same logic as though the actor were calling `A_Chase` to play the Thing's `Active sound`.
    * Notes:
        * This function will behave exactly the same as `A_Look` aside from the parameterized `fov` and `range`.

* **A_WanderEx(norandomturn, maxturn, nofacedir, stopblocked, makesound)**
    * Makes the calling actor move aimlessly without performing attack checks.
    * Args:
        * `norandomturn (int)`: If nonzero, the actor will not attempt to turn at random, but may still turn when it encounters an obstacle. This can be used to force a thing to move in its current facing direction.
        * `maxturn (fixed)`: Maximum angle in which the actor will turn in either direction. A value of `0.0` (or, practically, a value equal to or greater than `180.0`) will mean that the actor can turn in any direction. The default value is `0.0`.
        * `nofacedir (int)`: If nonzero, the actor will not change its angle to face the direction of travel.
        * `stopblocked (int)`: If nonzero, the actor will not automatically turn when colliding with an obstacle and will simply stop moving. It may turn randomly depending on the value of `norandomturn`, however.
        * `makesound (int)`: If nonzero, perform a random check using the same logic as though the actor were calling `A_Chase` to play the Thing's `Active sound`.
    * Notes:
        * This can be called whether or not the actor as a target.
        * Despite its name, this codepointer is designed to be used both within and outside of an actor's `Wander state` sequence.

* **A_ChaseEx(meleestate, missilestate, strafechance, straferange, strafespeed, norandomturn, maxturn, nodirectionturn, nofacedir, stopblocked, makesound)**
    * A parameterized version of `A_Chase`.
    * Args:
        * `meleestate (int)`: State to jump to when the actor performs a melee attack. A value of 0 means to use the actor's default `Melee state`, while a negative value means that no melee attack checks should be performed, and the random table should not move.
        * `missilestate (int)`: State to jump to when the actor performs a ranged attack. A value of 0 means to use the actor's default `Missile state`, while a negative value means that no ranged attack checks should be performed, and the random table should not move.
        * `strafechance (uint)`: Probability out of 255 that the actor will start strafing if it's within `straferange` of its target.
        * `strafespeed (uint)`: Speed at which the actor will move while it's strafing.
        * `straferange (fixed)`: Distance in map units from its target at which the actor will perform strafe checks.
        * `norandomturn (int)`: If nonzero, the actor will not attempt to turn at random, but may still turn when it encounters an obstacle. This can be used to force a thing to move in its current facing direction.
        * `maxturn (fixed)`: Maximum angle in which the actor will turn in either direction. A value of `0.0` (or, practically, a value equal to or greater than `180.0`) will mean that the actor can turn in any direction.
        * `nofacedir (int)`: If nonzero, the actor will not change its angle to face the direction of travel.
        * `stopblocked (int)`: If nonzero, the actor will not automatically turn when colliding with an obstacle and will simply stop moving. It may turn randomly depending on the value of `norandomturn`, however.
        * `makesound (int)`: If nonzero, perform a random check using the same logic as though the actor were calling `A_Chase` to play the Thing's `Active sound`.
    * Notes:
        * The strafing behavior is a parameterized version of that found in multiple Heretic/Hexen enemies.

* **A_Patrol(attemptfloat)**
    * Causes the calling actor to move its speed in its current direction. It will attempt to activate any linedef specials that it comes into contact with that it can activate and will cause the actor to round its movement to the nearest 45 degree angle and face its movement direction so long as its current frame is calling this codepointer. The actor will also snap to the X and Y position of any `MT_PATROLPOINT` objects that it comes within 16 map units of and change its direction to the `MT_PATROLPOINT` object's direction, though it will not perform the snap and turn if it is already facing in this direction.
    * Args:
        * `attemptfloat(uint)`: If nonzero and the calling actor has `NOGRAVITY` and `FLOAT` enabled, if it collides with an upper sidedef or lower sidedef that is not impassible to it and there is room for it to fit vertically into the sector on the other side, it will attempt to lower or raise its Z position until it is able to enter the sector by a maximum value of its `Speed` (and will use a smaller increment if using the full `Speed` value would cause it to overshoot).
    * Notes:
        * It is best to not use this codepointer outside of the `Patrol state` sequence. For more information on patrolling things, see Map Object Flags.

* **A_ChangeVelocityEx(velx, vely, velz, flags)**
    * Sets caller's own velocity, either to absolute values or relative values.
    * Args:
        * `velx (fixed)`: Amount of X (forward/back) momentum to apply
        * `vely (fixed)`: Amount of Y (left/right) momentum to apply
        * `velz (fixed)`: Amount of Z (up/down) momentum to apply
        * `flags (int)`: Allows further customization:
            * `DONTIGNOREZERO (0x001)`: Typically, `velx`, `vely`, and `velz` values of 0 are ignored, but this will cause them to be parsed when setting absolute momentum or multiplying momentum.
            * `ABSOLUTEANGLE (0x002)`: Does not use caller's current angle for `velx` and `vely`.
            * `ADDXVELOCITY (0x004)`: Adds `velx` to current X velocity.
            * `ADDYVELOCITY (0x008)`: Adds `vely` to current Y velocity.
            * `ADDZVELOCITY (0x010)`: Adds `velz` to current Z velocity.
            * `MULTXVELOCITY (0x020)`: Multiplies current X velocity by `velx`. Supercedes `ADDXVELOCITY`.
            * `MULTYVELOCITY (0x040)`: Multiplies current Y velocity by `vely`. Supercedes `ADDYVELOCITY`.
            * `MULTZVELOCITY (0x080)`: Multiplies current Z velocity by `velz`. Supercedes `ADDZVELOCITY`.
        * Notes:
            * Subject to change (and seeking feedback).

### Actor Logic Codepointers

* **A_JumpIfTargetInSightEx(state, fov, flags)**
    * Jumps to a state if caller's target is in line-of-sight. Optionally checks for a direct path between the the caller and its target.
    * Args:
        * `state (uint)`: State to jump to
        * `fov (fixed)`: Field-of-view, relative to calling actor's angle, to check for target in. If zero, the check will occur in all directions.
        * `flags (int)`: Allows further customization:
            * `CHECKPATH (0x001)`: Checks for `SOLID` obstructions in the path between caller and its target.
            * `CHECKHITSCAN (0x002)`: Checks for a clear path for a hitscan. Will also check for hitscan-blocking linedefs (but not any self-referencing sectors).
            * `CHECKLINES (0x004)`: Checks for any impassible linedefs, including "Block monsters" linedefs or "Block land monsters" linedefs as appropriate given the caller's current flags.
        * Notes:
            * Additional flags may be added as needed in future spec revisions.

* **A_JumpIfTracerInSightEx(state, fov, flags)**
    * Jumps to a state if caller's tracer target is in line-of-sight. Optionally checks for a direct path between the the caller and its target.
    * Args:
        * `state (uint)`: State to jump to
        * `fov (fixed)`: Field-of-view, relative to calling actor's angle, to check for tracer target in. If zero, the check will occur in all directions.
        * `flags (int)`: Allows further customization:
            * `CHECKPATH (0x001)`: Checks for `SOLID` obstructions in the path between caller and its tracer target.
            * `CHECKHITSCAN (0x002)`: Checks for a clear path for a hitscan. Will also check for hitscan-blocking linedefs (but not any self-referencing sectors).
            * `CHECKLINES (0x004)`: Checks for any impassible linedefs, including "Block monsters" linedefs or "Block land monsters" linedefs as appropriate given the caller's current flags.
        * Notes:
            * Additional flags may be added as needed in future spec revisions.

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
    * Jumps to a state if the caller's target has the specified thing flags set. This command has been updated to support ID24 and MBF25 flags.
    * Args:
        * `state (uint)`: State to jump to.
        * `flags (int)`: Standard actor flag(s) to check
        * `flags2 (int)`: MBF21 actor flag(s) to check
        * `flags3 (int)`: ID24 actor flag(s) to check
        * `flags4 (int)`: MBF25 actor flag(s) to check
    * Notes:
        * Will no-op if the caller has no target.

* **A_JumpIfTracerFlagsSet(state, flags, flags2, flags3, flags4)**
    * Jumps to a state if the caller's tracer target has the specified thing flags set. This command has been updated to support ID24 and MBF25 flags.
    * Args:
        * `state (uint)`: State to jump to.
        * `flags (int)`: Standard actor flag(s) to check
        * `flags2 (int)`: MBF21 actor flag(s) to check
        * `flags3 (int)`: ID24 actor flag(s) to check
        * `flags4 (int)`: MBF25 actor flag(s) to check
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

* **A_JumpIfMapNum(mapnum, episodenum, state, flags)**
    * Jumps to state based on the current map number and/or episode number.
    * Args:
        * `mapnum (uint)`: Map number to check for. If 0, will only compare the `episodenum`.
        * `episodenum (uint)`: Episode number to check for. If 0, will only compare the `mapnum`.
        * `state (uint)`: State to jump to if conditions are met.
        * `flags (int)`: Affects the comparison method.
            * `MAPGREATER (0x01)`: If set and `mapnum` is being compared, then the current map number must be greater than `mapnum`. Otherwise, it must be equal.
            * `EPGREATER (0x02)`: If set and `episodenum` is being compared, then the current episode number must be greater than `epinum`. Otherwise, it must be equal.
            * `EITHER (0x04)`: If set and both `mapnum` and `episodenum` are being compared, then either condition being met will cause the jump to occur.
    * Notes:
        * Will no-op if neither `mapnum` or `episodenum` are nonzero.

* **A_JumpIfSkill(skillnum, state, orhigher)**
    * Jumps to state based on the current skill selection.
    * Args:
        * `skillnum (uint)`: ID of the skill to check for. By default, 0 = ITYTD, 1 = HNTR, 2 = HMP, 3 = UV, 4 = Nightmare.
        * `state (uint)`: State to jump to if conditions are met.
        * `orhigher (uint)`: If nonzero, will also jump if the current skill is higher than the given value.
    * Notes:
        * If custom skills are defined, this is based off of the ID of the skill as defined in `UMAPINFO`.
        * In the case of `ZMAPINFO`-defined skills, as it defines skills by name and not by ID, IDs will correlate to the skills in the order in which they're defined.

### Target and Tracer Manipulation Codepointers

* **A_AddTargetFlags(flags, flags2, flags3, flags4)**
    * Adds the specified thing flags to the caller's target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to add
        * `flags2 (int)`: MBF21 actor flag(s) to add
        * `flags3 (int)`: ID24 actor flag(s) to add
        * `flags4 (int)`: MBF25 actor flag(s) to add
    * Notes:
        * Will no-op if the caller has no target.

* **A_AddTracerFlags(flags, flags2, flags3, flags4)**
    * Adds the specified thing flags to the caller's tracer target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to add
        * `flags2 (int)`: MBF21 actor flag(s) to add
        * `flags3 (int)`: ID24 actor flag(s) to add
        * `flags4 (int)`: MBF25 actor flag(s) to add
    * Notes:
        * Will no-op if the caller has no tracer target.

* **A_RemoveTargetFlags(flags, flags2, flags3, flags4)**
    * Removes the specified thing flags from the caller's target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to remove
        * `flags2 (int)`: MBF21 actor flag(s) to remove
        * `flags3 (int)`: ID24 actor flag(s) to remove
        * `flags4 (int)`: MBF25 actor flag(s) to remove
    * Notes:
        * Will no-op if the caller has no target.

* **A_RemoveTracerFlags(flags, flags2, flags3, flags4)**
    * Removes the specified thing flags from the caller's tracer target.
    * Args:
        * `flags (int)`: Standard actor flag(s) to remove
        * `flags2 (int)`: MBF21 actor flag(s) to remove
        * `flags3 (int)`: ID24 actor flag(s) to remove
        * `flags4 (int)`: MBF25 actor flag(s) to remove
    * Notes:
        * Will no-op if the caller has no tracer target.

* **A_ClearTarget**
    * Clears the caller's target field.
    * No args.

### Parameterized Monster Attack Codepointers

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
            * `ALWAYSPLAYSOUND (0x01)`: Sound will play even if attack fails.
            * `ASSUMEPLAYERMASS (0x02)`: Regardless of the target's mass, it will receive the same amount of momentum as though it had the player's default mass.
            * `NOLINEOFSIGHT (0x04)`: The attack will succeed no matter what, even if the caller cannot see its target.

* **A_MonsterChargeAttack(angle, hspread, vspread, speed, nofacetarget)**
    * Parameterized version of `A_SkullAttack`.
    * Args:
        * `angle (fixed)`: Angle (degrees), relative to calling actor's angle
        * `hspread (fixed)`: Horizontal spread (degrees), in fixed point.
        * `vspread (fixed)`: Vertical spread (degrees), in fixed point.
        * `speed (int)`: Speed at which the caller will launch itself; defaults to 20 if not set
        * `nofacetarget`: If nonzero, do not call `A_FaceTarget` before attacking.
    * Notes:
        * As with projectiles, damage calculations and damage type are handled by the the thing's properties.

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
            * `SPAWNRANDOMAMOUNT (0x01)`: If `number` is more than 1, will randomly spawn between 1 and that value based on RNG. The RNG should be similar to generating "Damage dice" values and advance the RNG table identically if this flag is set.
            * `INHERITTARGET (0x02)`: Automatically sets the spawned thing's target to the caller's target if it has one.
    * Notes:
        * A hard-coded array of angles at which to spawn things are provided for each valid value of `number`.

* **A_MonsterSpitEx(type, speed, fallbackstate, flags)**
    * Parameterized version of `A_BrainSpit` that creates a thing and fires it toward a spawn target thing.
    * Args:
        * `type (uint)`: Thing type to fire.
        * `speed (int)`: Speed at which the created thing should move toward the spawn target.
        * `fallbackstate (int)`: If non-negative, state to jump to if no qualifying spawn target is found as soon as the attempt is made; defaults to -1 if not set
        * `flags (int)`: Allows further customization.
            * `USETAG (0x01)`: If the current map format is UDMF, then only look for spawn targets that share a Thing Tag with the caller; if not, then only look for spawn targets in a sector with the same tag as the caller's `initialsectortag` value.
            * `FINDCLOSEST (0x02)`: Will attempt to find the closest spawn target thing.
            * `USEMISSILERANGE (0x04)`: Will only look for spawn targets within the caller's max missile range.
            * `USELOS (0x08)`: Will only look for spawn targets that the caller can potentially see.
            * `USETARGETFORCHECKS (0x10)`: Affects the behavior of `FINDCLOSEST`, `USEMISSILERANGE`, and `USELOS`; for `FINDCLOSEST`, will attempt to find the closest spawn target thing to the caller's target; for `USEMISSILERANGE`, the distance check is performed from the caller's target but still uses the caller's max missile range; for `USELOS`, will only look for spawn targets that the caller's target can potentially see; causes the attempt to fail if the caller has no target.
    * Notes:
        * Heavily WIP.

* **A_SpawnFlyEx(spawner, effectthing)**
    * Parameterized version of `A_SpawnFly`.
    * Args:
        * `spawner (uint)`: Random spawner (see `spawners.md`) to use when thing reaches a spawn target.
        * `effectthing (uint)`: If nonnegative, thing type to spawn upon reaching a spawn target as an "effect" (such as spawn fire or teleport fog).
        * `sound (uint)`: If nonzero, sound to play.

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
            * `ONLYSPRAYONCE (0x01)`: Each thing found will only be hit by one tracer
            * `USEPROJECTILELOS (0x02)`: Instead of using the caller's target, will look in all 360 degrees from itself; best used with `ONLYSPRAYONCE`.
            * `DONTIGNORETARGET (0x04)`: Usually, the spray check will ignore the caller's target; if this is set, then the caller's target can be targeted as well
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

* **A_DropItem(thingtype, spawner, nodropped)**
    * Can be used to make a thing drop another thing and/or a random spawner, even outside of the death state.
    * Args:
        * `thingtype (int)`: Thing to drop. If negative, no thing is dropped (presumably to use a random spawner instead); if not set, defaults to -1
        * `spawner (int)`: Random spawner to drop. If negative, no random spawner is dropped (presumably to use a specific thing instead); if not set, defaults to -1
        * `nodropped (uint)`: If nonzero, the dropped thing/random spawner spawn thing will not have the `DROPPED` flag set.
    * Notes:
        * Functionally, just re-uses the existing drop item code.
        * 

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