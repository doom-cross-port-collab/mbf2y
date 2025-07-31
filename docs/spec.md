# MBF2y Specification v0.2.3
MBF2y supports the full spec of MBF21 and the weapon, ammo, thing, and frame properties used in ID24HACKED, as well as the new thing types and definitions embraced by the latter. The goals of the standard are:
* To expand upon the parameterized behaviors established in MBF21 and allow for further customizability of Thing and Weapon behavior;
* To incorporate features and capabilities found in Doom engine games like Heretic and Hexen not currently accessible outside of standards such as `DECORATE` and `ZScript`;
* To provide commonly requested quality of life improvements;
* To follow existing conventions when possible;
* To expand accessibility and capabilities of Doom modding without becoming unrecognizable as Doom;
* To make as many of the additions "opt-in" and non-obtrusive as possible.

## Generalized Sector Types

| Dec   | Bit | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-------|-----|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1024  | 10  | Suppress all sounds originating from Things within the sector.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 2048  | 11  | Suppress sounds made by this sector's floor or ceiling movement. (If this bit is set, then even if a parameterized crusher linedef special is used without the Silent flag, this sector will be silent when it moves.)                                                                                                                                                                                                                                                                                                                             |
| 16384 | 14  | Damaging floor effects also apply to `SHOOTABLE` objects without `NOSECTOR`, not just players. This includes voodoo dolls.                                                                                                                                                                                                                                                                                                                                                                                                                         |
| 32768 | 15  | Sounds made within this sector will only play for players within sound propagation range. If enabled in conjunction with Bit 10, instead of suppressing all sounds from Things within the sector, it will only suppress sounds if the player is not within sound propagation range. If enabled in conjunction with Bit 11, sector movement will only be suppressed if the player is not within sound propagation range. Having both bits enabled or neither bit enabled applies the effect to all sounds originating from the sector or within it. |


## Linedef flags

These pertain to the binary map format.

| Dec   | Bit | Description                                                                                                                                                                                                                                                                                           |
|-------|-----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1024  | 10  | 3D midtexture; adopted from Eternity Engine. Actors can walk on the midtexture. (The midtex blocks hitscan and projectiles by default.)                                                                                                                                                               |
| 2048  | 11  | Normally, this linedef will be reserved; however, when `comp_reservedlineflag` is disabled, if this flag is located on a Boom parameterized linedef that involves a texture and/or effect change along with a floor or ceiling move, it will reverse the order of sector movement vs. texture change. |
| 16384 | 14  | When used in conjunction with bit 10, the 3D midtexture allows hitscan and projectiles to pass through, but is still impassible to actors.                                                                                                                                                            |
| 32768 | 15  | Block hitscan and projectiles.                                                                                                                                                                                                                                                                        |



## General Improvements

### `SWANDEFS`

`SWANDEFS` is an optional lump that can be included to take advantage of advanced switch texture and animated texture definition.

(To be determined; this is currently in the prototyping phase.)

### Fast Thing Handling

High velocity and `Speed` values can cause accuracy and clipping issues in the Doom engine. This is because of the way that Thing movement is handled. While not perfect, in MBF2y, this behavior is improved considerably.

If a thing's total momentum along the horizontal plane (as in, the distance in map units that it will travel per tic) is greater than or equal to its `Radius` value, then instead of performing only one movement per tic, it will split its movement into two or more smaller checks. The formula to determine the number of sub-movement checks it will make is `submovements = ( abs(horizontal momentum) / radius ) + 1` (rounded to the next positive integer). On each sub-movement, it will move its total momentum divided by the the value of `submovements`, performing collision checks at each step.

Note that Z movement will also need to occur after each sub-movement instead of after all X/Y movement has been handled, including collision detection. There will need to be a separate Z-movement path for fast Things versus non-fast Things, and the amount of Z movement per sub-movement will be equal to the total z momentum divided by `submovements`.

This approach mirrors how GZDoom and other ports handle accurate collision detection. Note, however, that two fast objects moving toward one another may not collide accurately.

This behavior will be disabled if the `comp_fastthings` compflag is set to `0`.

Note that by fixing this behavior, with the exception of very large Things, this also fixes the wild angle miscalculations when a Thing tries to move with a very high `Speed` value.

### Decoupling of `A_BFGSpray` and `P_AimLineAttack`

In previous standards and vanilla Doom, the function `P_AimLineAttack` is called by the codepointer `A_BFGSpray` as well as by hitscan player autoaim. This doesn't cause any issues in that capacity, but with the parameterization of BFG-style spray attacks as well as the `NOSPRAY` and `NOTAUTOAIMED` MBF2y Thing flag, `A_BFGSpray` and the parameterized version, `A_Spray`, use a duplicate copy of `P_AimLineAttack` called `P_AimSprayAttack` when in the newer complevel.

### Improved `FRIENDLY` monster behavior

In MBF, `FRIENDLY` monsters were introduced, and in order to accommodate them, changes were made to monsters' targeting thresholds and infighting cooldowns. MBF21 reversed this change to restore vanilla infighting behavior, but this had the unintended side effect of making `FRIENDLY` Things behave in unintended ways.

By default, in MBF2y, Things with the `FRIENDLY` flag will always behave as though `comp_pursuit` is disabled.

This behavior is controlled by `comp_friendlyfix`.

## `SNDDEFS`

`SNDDEFS` is an optional lump that can be used to achieve the following:
* Define randomized sound groups, similar to Former Human alert and death sounds, where when one sound is specified, any of the others in the same group may play instead.
* Override default attenuation of a particular sound.
* Prevent a sound from being cut off if the Thing that triggered it no longer exists.
* Prevent a sound from cutting off other sounds made by the same thing, provided there are free sound channels.

(To be determined; this has not yet been formalized and is highly subject to change.)

## New Comp Flags

All comp flags below should be unparsed by complevels below MBF2y. Some ports, like GZDoom, may already contain these fixes optionalized by other means; in that case, these are for reference for a "MBF2y compatibility mode".

* `comp_fastmissilefix` (default: 1)
    * Improves movement and collision handling for all projectiles moving faster than their size (default)
    * Exception: `MISSILE` or `SKULLFLY` objects with the `NOFASTCOLLISION` MBF2y flag will never use the improved movement and collision handling, while non-`MISSILE` or `SKULLFLY` objects with the `FASTCOLLISION` MBF2y flag will use the improved collision detection.
* `comp_friendlyfix` (default: 1)
    * `FRIENDLY` monsters will always behave as though `comp_pursuit` is disabled, thus causing them to use the MBF infighting cooldown.
* `comp_lesscorpseslide` (default: 1)
    * Monster corpses hanging off of a dropoff will only attempt to slide off of the higher floor if the difference between the highest floor and the lowest floor is 24 map units or more.
* `comp_fixfriendlyautoaim` (default: 1)
    * Horizontal autoaim no longer uses the faulty logic for checking for FRIENDLY shootable things.

## New things
The following things have been added.

| Index          | DoomEdNum | Name                               | Description                                                                                                                                                                                                                                                                                             |
|----------------|-----------|------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0xC000 (49152) | 32001     | `MT_PATROL_POINT` - "Patrol point" | `Width = 16`<br>`Height = 16`<br>`Mass = 100`<br>`MBF2y Bits = NOSCROLL`<br><br>When a thing calling `A_Patrol` is overlapping with this object, if it is not already moving in the same direction as this object is pointed, it will snap to this object's x/y position and change its direction to match. |
