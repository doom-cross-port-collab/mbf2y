# MBF25 Specification v0.2
MBF25 supports the full spec of MBF21 and ID24. The goals of the standard are:
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
| 16384 | 14  | Doom 64-style colored lighting; if a `COLORMAP` is applied to this sector, its flats, sidedef textures, and Things within it will have the `COLORMAP` applied to them even when the player is not in the sector or in a sector with a different `COLORMAP`. (`COLORMAP` transfer specials will still apply to this sector.)                                                                                                                                                                                                                        |
| 32768 | 15  | Sounds made within this sector will only play for players within sound propagation range. If enabled in conjunction with Bit 10, instead of suppressing all sounds from Things within the sector, it will only suppress sounds if the player is not within sound propagation range. If enabled in conjunction with Bit 11, sector movement will only be suppressed if the player is not within sound propagation range. Having both bits enabled or neither bit enabled applies the effect to all sounds originating from the sector or within it. |

## Linedef flags

| Dec   | Bit | Description                                                                                                                                                                                                                                                                                          |
|-------|-----|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1024  | 10  | 3D midtexture; adopted from Eternity Engine. `SOLID` objects will collide with the midtexture as though it was a solid surface, though hitscan and line of sight are unaffected. Things can move over or under the midtexture or walk on top of it.                                                  |
| 16384 | 14  | Blocks hitscan. Hitscan attacks, such as melee, tracers, and bullet attacks, will not pass through this linedef as though it were a one-sided linedef. Line of sight is unaffected (except for `A_JumpIfTargetInSightEx` and `A_JumpIfTracerInSightEx` codepointers, see codepointer documentation). |
| 32768 | 15  | Wrap midtex. If enabled on a two-sided linedef, the midtexture will repeat from the lowest ceiling to the highest floor.                                                                                                                                                                             |

## Linedef Types

Many of the new linedef specials introduced by this standard are designed to be activated "behind the scenes", such as via a voodoo doll, the `A_LineEffectEx` codepointer, or a Thing with the `ACTIVATOR` MBF25 flag.

### Global and Local `COLORMAP` Manipulation

These linedef specials allow for setting and manipulation of the default `COLORMAP`, either for an entire level or for tagged sectors. The order of priority for `COLORMAP`s is the following:

* If the sector is tagged and there has been a local `COLORMAP` defined via activated linedef specials 1097 through 1103 using the same tag, then use the `COLORMAP` of the most recently activated linedef. (Special 1097 is considered to activate upon level initialization.)
* If the sector is tagged and there is a linedef with linedef special 242 using the same tag, and that linedef has a `COLORMAP` as its front middle sidedef texture, then use the `COLORMAP` of that linedef.
* If there is a global `COLORMAP` defined via activated linedef specials 1090 through 1096 (special 1090 is considered to activate upon level initialization), and one of the following conditions is met, then use the global `COLORMAP`:
    * The sector is untagged, and there is a global `COLORMAP` defined via activated linedef specials 1090 through 1096 ;
    * The sector is tagged, but there is no activated linedef special 1097 through 1103 using the same tag and no `COLORMAP` defined on the front middle sidedef texture of any linedefs with linedef special 242 and the same tag;

The activated linedef specials, which allow for dynamic manipulation, rely upon the front upper sidedef texture if activated from the front and the front lower sidedef texture if activated from the back. If the texture is blank, then the relevant `COLORMAP` will be be cleared; if it is a non-`COLORMAP` texture, no change will occur, and the linedef should not be considered to have activated.

Bear in mind that when working with Boom sectors with fake ceilings and floors, only the "middle" `COLORMAP` can be altered via the linedef specials below.

Save games will need to be able to preserve which global and local `COLORMAP`s are active, if any. Only one global `COLORMAP` can be active at any given time per level, so it would be efficient to simply store this `COLORMAP` as part of the save game data. Local `COLORMAP`s may require storing, for each unique sector tag, which `COLORMAP` is currently active, if any. (Alternatively, you could store the `COLORMAP` state for every sector individually.)

The activateable linedefs can only be activated by a player mobj, the `A_LineEffectEx` codepointer, and Things with the `ACTIVATOR` MBF25 flag.

| Linedef Special | Trigger | Tag required? | Description                                                                                                                                                                           |
|-----------------|---------|---------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1090            | Static  | No            | Change the default `COLORMAP` for the whole level based on the front upper sidedef texture.                                                                                             |
| 1091            | W1      | No            | Change the default `COLORMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1092            | WR      | No            | Change the default `COLORMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1093            | S1      | No            | Change the default `COLORMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1094            | SR      | No            | Change the default `COLORMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1095            | G1      | No            | Change the default `COLORMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1096            | GR      | No            | Change the default `COLORMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1097            | Static  | Yes           | Change the local `COLORMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1098            | W1      | Yes           | Change the local `COLORMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1099            | WR      | Yes           | Change the local `COLORMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1100            | S1      | Yes           | Change the local `COLORMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1101            | SR      | Yes           | Change the local `COLORMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1102            | G1      | Yes           | Change the local `COLORMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1103            | GR      | Yes           | Change the local `COLORMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |

### Global and Local `TRANMAP` Manipulation

These linedef specials allow for setting and manipulation of the default `TRANMAP` used for untagged translucent linedefs throughout an entire map, as well as dynamic alteration of the `TRANMAP` used on tagged linedefs affected by linedef special 260. The ordor of priority for `TRANMAP`s for a given translucent linedef is the following:

* If the linedef is tagged and there has been a local `TRANMAP` defined via activated linedef specials 1111 through 1117 using the same tag, then use the `TRANMAP` of the most recently activated linedef. (Special 1111 is considered to activate upon level initialization.)
* If the linedef is tagged and there has not been a local `TRANMAP` defined via acitvated linedef specials, but there is a linedef with linedef special 260 using the same tag and it is located in a sector that has a `TRANMAP` as its floor, then use that `TRANMAP`.
* If the linedef either: is untagged and has linedef special 260 on it **OR** has the `Translucent` UDMF flag enabled; **AND** there has been a global `TRANMAP` defined via activated linedef specials 1104 through 1110, then use the `TRANMAP` of the most recently activated linedef. (Linedef 1104 is considered to activate upon level initialization.)
* If the linedef either: is untagged and has linedef special 260 on it **OR** has the `Translucent` UDMF flag enabled; **AND** there has not been a global `TRANMAP` defined via activated linedef specials 1104 through 1110, then use the in-built default transparency `TRANMAP`.

The activated linedef specials, which allow for dynamic manipulation, rely upon the front upper sidedef texture if activated from the front and the front lower sidedef texture if activated from the back. If the texture is blank, then the relevant `TRANMAP` will be be cleared; if it is a non-`TRANMAP` texture, no change will occur, and the linedef should not be considered to have activated.

The activateable linedefs can only be activated by a player mobj, the `A_LineEffectEx` codepointer, and Things with the `ACTIVATOR` MBF25 flag.

| Linedef Special | Trigger | Tag required? | Description                                                                                                                                                                            |
|-----------------|---------|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1104            | Static  | No            | Change the default `TRANMAP` for the whole level based on the front upper sidedef texture.                                                                                             |
| 1105            | W1      | No            | Change the default `TRANMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1106            | WR      | No            | Change the default `TRANMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1107            | S1      | No            | Change the default `TRANMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1108            | SR      | No            | Change the default `TRANMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1109            | G1      | No            | Change the default `TRANMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1110            | GR      | No            | Change the default `TRANMAP` for the whole level based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back. |
| 1111            | Static  | Yes           | Change the local `TRANMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1112            | W1      | Yes           | Change the local `TRANMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1113            | WR      | Yes           | Change the local `TRANMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1114            | S1      | Yes           | Change the local `TRANMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1115            | SR      | Yes           | Change the local `TRANMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1116            | G1      | Yes           | Change the local `TRANMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |
| 1117            | GR      | Yes           | Change the local `TRANMAP` for tagged sector(s) based on the front upper sidedef texture if activated from the front, and the front lower sidedef texture if activated from the back.  |

### Relative Light Level Changes

These linedef specials are, in particular, meant for "behind the scenes" changes, as they rely upon the absolute light level of the front back sector. If the linedef is activated from the front, then the brightness of tagged sectors is increased by the light level of the linedef's front sector; if activated from the back, then the brightness of tagged sectors is decreased by the light level of the linedef's front sector instead.

The activateable linedefs can only be activated by a player mobj, the `A_LineEffectEx` codepointer, and Things with the `ACTIVATOR` MBF25 flag.

These linedef specials will not decrease a sector's light level below 0 or increase it above 256.

| Linedef Special | Trigger | Tag required? | Description                                                                                                                                                                                   |
|-----------------|---------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1118            | W1      | Yes           | Increase the light level of tagged sector(s) by the light level of the linedef's front sector if activated from the front; decrease it by the same amount instead if activated from the back. |
| 1119            | WR      | Yes           | Increase the light level of tagged sector(s) by the light level of the linedef's front sector if activated from the front; decrease it by the same amount instead if activated from the back. |
| 1120            | S1      | Yes           | Increase the light level of tagged sector(s) by the light level of the linedef's front sector if activated from the front; decrease it by the same amount instead if activated from the back. |
| 1121            | SR      | Yes           | Increase the light level of tagged sector(s) by the light level of the linedef's front sector if activated from the front; decrease it by the same amount instead if activated from the back. |
| 1122            | G1      | Yes           | Increase the light level of tagged sector(s) by the light level of the linedef's front sector if activated from the front; decrease it by the same amount instead if activated from the back. |
| 1123            | GR      | Yes           | Increase the light level of tagged sector(s) by the light level of the linedef's front sector if activated from the front; decrease it by the same amount instead if activated from the back. |

### Texture Transfers

These linedef specials are, in partiucular, meant for "behind the scenes" changes, as they rely upon the textures of the activating linedefs. If the linedef is activated from the front, then the front sidedef textures of tagged linedefs (besides any linedef with one of the below texture transfer specials) will be replaced with the front sidedef textures of the activated linedef. If the linedef is activated from the back, then the back sidedef textures of tagged linedefs (besides any linedef with one of the below texture transfer specials) will be replaced with the front sidedef textures of the activated linedef. If it is required to change both at once, it is recommended to use two linedefs, one facing in each direction.

In the event of a blank (or `COLORMAP`/`TRANMAP`, though this is simply to safeguard against user error) sidedef texture, behavior will be different based on whether it's a blank upper/lower sidedef texture, a blank middle sidedef texture, and whether the linedef is one-sided or two-sided.

* A blank upper or lower sidedef texture has no effect in general, whether a linedef is one-sided or two-sided.
* A blank middle texture has no effect on one-sided linedefs, but it will remove the front or back (depending on activation side) middle sidedef texture for two-sided linedefs.

All sidedef offsets should be preserved upon activation. If it is required to change these, combine these linedef specials with one of the Boom scroller linedef specials.

If a texture is changed to an animated texture, it should reset its animation frame count and start the animation over from the beginning.

The activateable linedefs can only be activated by a player mobj, the `A_LineEffectEx` codepointer, and Things with the `ACTIVATOR` MBF25 flag.

| Linedef Special | Trigger | Tag required? | Description                                                                                                                                                                                                                                          |
|-----------------|---------|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1124            | W1      | Yes           | Change the sidedef textures of tagged linedefs without specials 1124 through 1129 to this linedef's front sidedef textures; change the front sidedef textures if activated from the front, and the back sidedef textures if activated from the back. |
| 1125            | WR      | Yes           | Change the sidedef textures of tagged linedefs without specials 1124 through 1129 to this linedef's front sidedef textures; change the front sidedef textures if activated from the front, and the back sidedef textures if activated from the back. |
| 1126            | S1      | Yes           | Change the sidedef textures of tagged linedefs without specials 1124 through 1129 to this linedef's front sidedef textures; change the front sidedef textures if activated from the front, and the back sidedef textures if activated from the back. |
| 1127            | SR      | Yes           | Change the sidedef textures of tagged linedefs without specials 1124 through 1129 to this linedef's front sidedef textures; change the front sidedef textures if activated from the front, and the back sidedef textures if activated from the back. |
| 1128            | G1      | Yes           | Change the sidedef textures of tagged linedefs without specials 1124 through 1129 to this linedef's front sidedef textures; change the front sidedef textures if activated from the front, and the back sidedef textures if activated from the back. |
| 1129            | GR      | Yes           | Change the sidedef textures of tagged linedefs without specials 1124 through 1129 to this linedef's front sidedef textures; change the front sidedef textures if activated from the front, and the back sidedef textures if activated from the back. |

### Multi-Sector Triggers

Multiple linedefs placed within a map can be activated at once via a variety of methods. These particular linedef specials will activate the linedef specials of all linedefs that have a sector with the same tag as its front sector. If this linedef is activated from the front, then the secondary linedefs will be activated as though they were also activated from the front; conversely, if activated from the back, then the secondary linedefs will also be activated as though they were also activated from the back.

Only linedefs with a W1/WR/S1/SR/G1/GR activation method will be activated. W1/S1/G1 linedefs that have already been activated by the time that the primary linedef has triggered will not re-activate; similarly, if such linedefs are activated by one of these linedef specials, they are considered to have already activated, even if a player directly attempts to re-trigger them.

Linedefs with specials 1130 through 1141 are ignored by this to avoid infinite looping.

When determining the order in which linedefs should be activated:
* Start with the lowest sector index with the matching tag (Sector Scan).
* Once a tagged sector has been found, starting with the lowest linedef index (Per-Sector Linedef Scan).
* Once all activateable linedefs adjacent to a tagged sector have been activated, find repeat the Sector Scan looking for the next highest index, and repeat the Per-Sector Linedef Scan for this sector.
* Repeat until no more sectors with the matching tag have been found.

The activateable linedefs can only be activated by a player mobj, the `A_LineEffectEx` codepointer, and Things with the `ACTIVATOR` MBF25 flag. The activator of the primary linedef is considered to be the activator of the secondary linedef, but the sector relative to the secondary linedef is used rather than the sector of the primary linedef.

| Linedef Special | Trigger | Tag required? | Description                                                                                                                |
|-----------------|---------|---------------|----------------------------------------------------------------------------------------------------------------------------|
| 1130            | W1      | Yes           | Activate all activateable linedef specials adjacent to tagged sectors; transfer activation side to the activated linedefs. |
| 1131            | WR      | Yes           | Activate all activateable linedef specials adjacent to tagged sectors; transfer activation side to the activated linedefs. |
| 1132            | S1      | Yes           | Activate all activateable linedef specials adjacent to tagged sectors; transfer activation side to the activated linedefs. |
| 1133            | SR      | Yes           | Activate all activateable linedef specials adjacent to tagged sectors; transfer activation side to the activated linedefs. |
| 1134            | G1      | Yes           | Activate all activateable linedef specials adjacent to tagged sectors; transfer activation side to the activated linedefs. |
| 1135            | GR      | Yes           | Activate all activateable linedef specials adjacent to tagged sectors; transfer activation side to the activated linedefs. |

### Multi-Linedef Triggers

Similarly to the Multi-Sector Triggers, these linedefs can be used to trigger multiple linedefs, with the caveat that all linedefs must share a tag. This makes these linedef specials a lot more simple to configure and for game logic to process, but also a lot more limited. Similar to the Multi Sector Triggers, if this linedef is activated from the front, then the secondary linedefs will be activated as though they were also activated from the front; conversely, if activated from the back, then the secondary linedefs will also be activated as though they were also activated from the back.

Only linedefs with a W1/WR/S1/SR/G1/GR activation method will be activated. W1/S1/G1 linedefs that have already been activated by the time that the primary linedef has triggered will not re-activate; similarly, if such linedefs are activated by one of these linedef specials, they are considered to have already activated, even if a player directly attempts to re-trigger them.

Linedefs with specials 1130 through 1141 are ignored by this to avoid infinite looping.

When determining the order in which linedefs should be activated, start with the tagged linedef with the lowest index and proceed upward until no more matching linedefs are found.

The activateable linedefs can only be activated by a player mobj, the `A_LineEffectEx` codepointer, and Things with the `ACTIVATOR` MBF25 flag. The activator of the primary linedef is considered to be the activator of the secondary linedef, but the sector relative to the secondary linedef is used rather than the sector of the primary linedef.

| Linedef Special | Trigger | Tag required? | Description                                                                                                       |
|-----------------|---------|---------------|-------------------------------------------------------------------------------------------------------------------|
| 1136            | W1      | Yes           | Activate all activateable linedef specials with the same tag; transfer activation side to the activated linedefs. |
| 1137            | WR      | Yes           | Activate all activateable linedef specials with the same tag; transfer activation side to the activated linedefs. |
| 1138            | S1      | Yes           | Activate all activateable linedef specials with the same tag; transfer activation side to the activated linedefs. |
| 1139            | SR      | Yes           | Activate all activateable linedef specials with the same tag; transfer activation side to the activated linedefs. |
| 1140            | G1      | Yes           | Activate all activateable linedef specials with the same tag; transfer activation side to the activated linedefs. |
| 1141            | GR      | Yes           | Activate all activateable linedef specials with the same tag; transfer activation side to the activated linedefs. |

### Linedef Flag Modification

To be determined.

### Sector Flat Modification

To be determined.

### Teleport Only Player to Destination

The following linedefs behave identically to the existing teleportation specials, with the exception that only players and non-Player Things with both the `ACTIVATOR` MBF25 flag and the `TELEPORT` flag can activate them.

These will have no effect if activated by other Things, but can be called via `A_LineEffectEx` if the Thing otherwise qualifies (including the player mobj, hypothetically, though you'd have to be insane to want to do this).

| Linedef Special | Trigger | Tag required? | Description                                                                                           |
|-----------------|---------|---------------|-------------------------------------------------------------------------------------------------------|
| 1166            | W1      | Yes           | Teleport to Teleport Destination in tagged sector (player and `TELEPORT` + `ACTIVATOR` objects only). |
| 1167            | WR      | Yes           | Teleport to Teleport Destination in tagged sector (player and `TELEPORT` + `ACTIVATOR` objects only). |
| 1168            | S1      | Yes           | Teleport to Teleport Destination in tagged sector (player and `TELEPORT` + `ACTIVATOR` objects only). |
| 1169            | SR      | Yes           | Teleport to Teleport Destination in tagged sector (player and `TELEPORT` + `ACTIVATOR` objects only). |
| 1170            | G1      | Yes           | Teleport to Teleport Destination in tagged sector (player and `TELEPORT` + `ACTIVATOR` objects only). |
| 1171            | GR      | Yes           | Teleport to Teleport Destination in tagged sector (player and `TELEPORT` + `ACTIVATOR` objects only). |

### Teleport Only Player to Linedef With Same Tag (silent, same angle)

To be determined.

### Dynamic Property Transfers

To be determined.

### One-Way Passable Linedefs

To be determined. (Example use case: create monster spawning areas that monsters can leave, but not re-enter.)

### Teleport (Preserve Z position)

The following linedefs behave identically to the existing teleportation specials, with the exception that the Z position of objects that teleport through is preserved, as in Final Doom.

To be determined.

### Z Teleporters

These linedefs are a special kind of teleporter that acts in pairs, requiring two control linedefs within a control sector with different linedef specials and tags, with one linedef corresponding with a "source" sector and one linedef corresponding with a "destination" sector (note that a sector can be both a source sector and a destination sector). Instead of activating when a player crosses a linedef, they are applied statically to a tagged sector. During the Z movement check, a Thing in the primary sector will silently teleport from one sector to another, preserving their angle, and momentum. The floor height of the control sector is used to determine the vertical point at which the teleportation occurs, while the ceiling height of the control sector is used to determine the vertical point at which teleporting objects arrive in the destination sector. The difference between the Thing's Z position and the floor height of the control sector is applied relative to the ceiling height of the control sector when determining which Z position the Thing ends up at in the destination sector.

Teleport Destinations are required in both the "source" sector and the "target" sector. For best results, the sectors should have the same horizontal geometry (or at the very least, the "source" sector should be able to fit within the "target" sector, though these are mapping considerations and should not be enforced. (It's theoretically possible that enterprising modders may be able to achieve effects with these linedef specials that are unseen.)

The repeatable versions of these linedef specials can be activated by any Thing moving along the Z axis, whether or not it is under gravity; use the MBF25 Thing flag `COPYONWARP` if you would like a Thing to create an exact copy in the destination sector without actually teleporting, or the MBF25 Thing flag `NOWARP` if you would like it to be completely unaffected by these linedefs.

When Things teleport from a source sector to a destination sector, their X and Y position relative to the Teleportation Destination in the source sector is used to place them in the same relative position to the Teleportation Destination in the target sector. Note that there is no strict requirement that this actually lands the Thing into the Target Sector. If either the source sector or the destination sector is missing a Teleport Destination, then no teleportation should occur.

If a Thing would teleport/spawn into a ceiling, its Z position should be corrected to the Z position of the lowest ceiling height of any sector it's overlapping subtracted by its `Height` value, unless this would cause it to spawn below the floor height. If a Thing would teleport/spawn into a floor, its Z position should be corrected to the Z position of the lowest floor of any sector it's overlapping with. These are considered mapping mistakes; it's up to the player to fix these issues. If a Thing would teleport outside of any sector, this is considered a mapping error and the teleportation should fail.

Linedef specials 1222 and 1223 should be used in pairs within a control sector, and only one of each linedef should be considered "valid" within a sector, regardless of its activation. Use the highest linedef index. If either linedef special is missing from the a linedef whose front sector is the control sector, then the teleportation/spawning should not occur. The same thing goes for linedef specials 1224 and 1225.

It is a valid, blue sky use case to put all four linedef specials in the same control sector in order to have a two-way Z-Teleporter.

A sector be both an upper-to-lower source and a lower-to-upper source, an upper-to-lower destination and a lower-to-upper destination, or even all four at once (or any of the three).

Multiple source sectors with the same tag are supported, but for the purposes of the destination sector, use traditional teleportation rules, using the same logic to determine which destination sector and Teleportation Destination is valid. It is considered a mapping error to have multiple destination sectors (though again, a sector can be both an upper-to-lower destination and a lower-to-upper destination).

A sector can even be both a destination and a source for itself, such as if the linedefs with specials 1222 and 1223 would have the same tag. Telefragging and thing duplication via `COPYONWARP` **does not** occur when this is the case.

If infinite thing height is enabled, then teleportation of `SOLID` Things into the same X/Y position as `SHOOTABLE` things will cause telefragging regardless of Z position. Telefragging will still occur even if infinite thing height is disabled if there is any Z overlap.

The inspiration for these linedefs was the Build engine as seen in Duke Nukem 3-D, though this approach, while requiring a little more setup than Build's equivalent functionality, is actually considerably more flexible and versatile.

These linedefs, importantly, cannot be activated by Multi Sector Triggers, Multi Linedef Triggers, or codepointers.

#### Example 1
In the below example, when a Thing reaches a Z height of below 512 while in Sector 1, it will be moved to the same relative position in Sector 2. Similarly, when a Thing reaches a Z height of above 512 while in Sector 2, it will be moved to the same relative position in Sector 1.

```
* = Teleport Destination (orientation unimportant)

Sector 1
Source Sector
Ceiling Height: 1024
Floor Height: 256
Tag: 1

------------------------
|                      |
|                      |
|          X           |
|                      |
|                      |
|                      |
------------------------

Sector 2
Destination Sector
Ceiling Height: 768
Floor Height: 0
Tag: 2

------------------------
|                      |
|                      |
|          X           |
|                      |
|                      |
|                      |
------------------------

Sector 3
Control Sector
Ceiling Height: 512
Floor Height: 512
Tag: Irrelevant

--A--
|   |
B   C
|   |
--D--

Linedef A
Special: 1222 (Set Z-Teleport Upper-to-Lower Source)
Tag: 1

Linedef B
Special: 1223 (Set Z-Teleport Upper-to-Lower Destination)
Tag: 2

Linedef C
Special: 1224 (Set Z-Teleport Lower-to-Upper Source)
Tag: 2

Linedef D
Special: 1225 (Set Z-Teleport Lower-to-Upper Destination)
Tag: 1
```

#### Example 2
In the below example, there are three different sectors, each of which is both a source sector and a destination sector. Only downward vertical movement triggers teleportation in any of the three sectors. When a Thing's Z-height position reaches below 256 in Sector 1, it is moved to the same relative X/Y position in Sector 2, but at a Z height equal to 768 minus the amount below the height of 256 that it had reached in Sector 1.

As the Thing continues falling, once its Z-position reaches below 256 in Sector 2, it is moved to the same relative X/Y position in Sector 3, but at a Z height equal to 3800 minus the amount below the height of 256 that it had reached in Sector 2.

Finally, once its Z-position in Sector 3 reaches below 2176 in Sector 3, it is moved to the same relative X/Y position in Sector 1, but at a Z height equal to 768 minus the amount below the height of 256 that it had reached in Sector 3.

```
* = Teleport Destination (orientation unimportant)

Sector 1
Upper-to-lower source of Sector 2
Upper-to-lower destination of Sector 3
Ceiling Height: 1024
Floor Height: 0
Tag: 1

------------------------
|                      |
|                      |
|          X           |
|                      |
|                      |
|                      |
------------------------

Sector 2
Upper-to-lower source of Sector 3
Upper-to-lower destination of Sector 1
Ceiling Height: 1024
Floor Height: 0
Tag: 2

------------------------
|                      |
|                      |
|          X           |
|                      |
|                      |
|                      |
------------------------

Sector 3
Destination Sector
Ceiling Height: 4096
Floor Height: 2048
Tag: 3

------------------------
|                      |
|                      |
|          X           |
|                      |
|                      |
|                      |
------------------------

Sector 4
Control Sector 1
Ceiling Height: 768
Floor Height: 256
Tag: Irrelevant

--A--
|   |
|   |
|   |
--B--

Linedef A
Special: 1222 (Set Z-Teleport Upper-to-Lower Source)
Tag: 1

Linedef B
Special: 1223 (Set Z-Teleport Upper-to-Lower Destination)
Tag: 2

Sector 5
Control Sector 2
Ceiling Height: 3800
Floor Height: 256
Tag: Irrelevant

--C--
|   |
|   |
|   |
--D--

Linedef C
Special: 1222 (Set Z-Teleport Upper-to-Lower Source)
Tag: 2

Linedef D
Special: 1223 (Set Z-Teleport Upper-to-Lower Destination)
Tag: 3

Sector 6
Control Sector 3
Ceiling Height: 768
Floor Height: 2176
Tag: Irrelevant

--E--
|   |
|   |
|   |
--F--

Linedef E
Special: 1222 (Set Z-Teleport Upper-to-Lower Source)
Tag: 3

Linedef F
Special: 1223 (Set Z-Teleport Upper-to-Lower Destination)
Tag: 1

```

#### Example 3
Finally, in the below example, when a Thing reaches a Z-position below 768 while in Sector 1, it will move to the same relative position in Sector 2 (maintaining the same Z position, as the control sector that affects this pair has the same floor and ceiling height).

While in Sector 2, once the Thing reaches a Z position below 128, its X and Y position will be unchanged, but its Z position will be changed to 768 minus whatever amount below 128 it had reached; similarly, if its Z-position becomes greater than 768, its X and Y position will be unchanged, but its Z position will be changed to 128 plus whatever amount above 768 it hard reached.

```
* = Teleport Destination (orientation unimportant)

Sector 1
Upper-to-lower source of Sector 2
Ceiling Height: 2048
Floor Height: 0
Tag: 1

------------------------
|                      |
|                      |
|          X           |
|                      |
|                      |
|                      |
------------------------

Sector 2
Upper-to-lower source of itself
Upper-to-lower destination of Sector 1 and itself
Ceiling Height: 1024
Floor Height: 0
Tag: 2

------------------------
|                      |
|                      |
|          X           |
|                      |
|                      |
|                      |
------------------------

Sector 3
Control Sector 1
Ceiling Height: 768
Floor Height: 768
Tag: Irrelevant

--A--
|   |
|   |
|   |
--B--

Linedef A
Special: 1222 (Set Z-Teleport Upper-to-Lower Source)
Tag: 1

Linedef B
Special: 1223 (Set Z-Teleport Upper-to-Lower Destination)
Tag: 2

Sector 4
Control Sector 2
Ceiling Height: 768
Floor Height: 128
Tag: Irrelevant

--C--
|   |
|   |
|   |
--D--

Linedef C
Special: 1222 (Set Z-Teleport Upper-to-Lower Source)
Tag: 2

Linedef D
Special: 1223 (Set Z-Teleport Upper-to-Lower Destination)
Tag: 2

Sector 5
Control Sector 3
Ceiling Height: 128
Floor Height: 768
Tag: Irrelevant

--E--
|   |
|   |
|   |
--F--

Linedef E
Special: 1224 (Set Z-Teleport Lower-to-Upper Source)
Tag: 2

Linedef F
Special: 1225 (Set Z-Teleport Lower-to-Upper Destination)
Tag: 2

```

## General Improvements

### `SWANDEFS`

`SWANDEFS` is an optional lump that can be included to take advantage of advanced switch texture and animated texture definition.

(To be determined; this is currently in the prototyping phase.)

### Fast Thing Handling

High velocity and `Speed` values can cause accuracy and clipping issues in the Doom engine. This is because of the way that Thing movement is handled. While not perfect, in MBF25, this behavior is improved considerably.

If a thing's total momentum along the horizontal plane (as in, the distance in map units that it will travel per tic) is greater than or equal to its `Radius` value, then instead of performing only one movement per tic, it will split its movement into two or more smaller checks. The formula to determine the number of sub-movement checks it will make is `submovements = ( abs(horizontal momentum) / radius ) + 1` (rounded to the next positive integer). On each sub-movement, it will move its total momentum divided by the the value of `submovements`, performing collision checks at each step.

Note that Z movement will also need to occur after each sub-movement instead of after all X/Y movement has been handled, including collision detection. There will need to be a separate Z-movement path for fast Things versus non-fast Things, and the amount of Z movement per sub-movement will be equal to the total z momentum divided by `submovements`.

This approach mirrors how GZDoom and other ports handle accurate collision detection. Note, however, that two fast objects moving toward one another may not collide accurately.

This behavior will be disabled if the `comp_fastthings` compflag is set to `0`.

Note that by fixing this behavior, with the exception of very large Things, this also fixes the wild angle miscalculations when a Thing tries to move with a very high `Speed` value.

(To be determined: does this also fix elastic collisions with level geometry?)

### Horizontal Auto-Aim Improvements

In standard Doom gameplay, projectile attacks use horizontal aim adjustment. While this was useful when Doom was first released (mouse/keyboard aiming wasn't particularly accurate and low resolutions made it much easier to aim off center from distant targets), these are frequent sources of frustration for players in the modern day, particularly for projectiles.

Horizontal auto-aim is now an opt-in weapon flag when a projectile is spawned via vanilla weapon codepointers or the `A_WeaponProjectileAttack` or `A_WeaponProjectileAttackEx` codepointers. Very fast projectiles, such as those with `Speed` values of 60 or greater, benefit from having this behavior preserved. (See Weapon Flags for more information.)

Note that the horizontal autoaim check will still occur, and if it detects a target that is off-center from the player's viewpoint, it will adjust the aim, but only vertically. This allows for rudimentary leading of flying enemies with projectile weapons and should drastically reduce the number of unintentional face rockets.

If a weapon has the `FORCEHORIZONTALAUTOAIM` flag enabled, then the old behavior will be preserved.

### Vertical melee attack improvements

In standard Doom gameplay, monsters with a melee attack will be able to strike a target using a melee attack even if the target is significantly higher or lower than the monster.  By default in MBF25, when performing a melee attack range check, a monster with a ranged attack will use it instead of a melee attack, while monsters without a ranged attack will be considered out of range.

The formula for determining if a target is within vertical melee reach of its aggressor is the following, presented in pseudo-code:

```
if (target->z + target->height >= attacker->z -attacker->meleerange
    && target->z <= attacker->z + attacker->height + attacker->meleerange)
    return true; // the attacker can hit the target with a melee attack
```

To restore the old behavior, disable the `comp_noinfinitemeleeheight` compflag.

### Decoupling of `A_BFGSpray` and `P_AimLineAttack`

In previous standards and vanilla Doom, the function `P_AimLineAttack` is called by the codepointer `A_BFGSpray` as well as by hitscan player autoaim. This doesn't cause any issues in that capacity, but with the parameterization of BFG-style spray attacks as well as the `NOSPRAY` and `NOTAUTOAIMED` MBF25 Thing flag, `A_BFGSpray` and the parameterized version, `A_Spray`, use a duplicate copy of `P_AimLineAttack` called `P_AimSprayAttack` when in the newer complevel.

### Non-sticky sector sound targets

In previous standards and vanilla Doom, and even in advanced source ports such as GZDoom, once sound or a noise alert corresponding to a player has propagated to a sector, the sector's `soundtarget` property will remain filled indefinitely. This means that if a sleeping monster is teleported into a sector where the player has previously made noise, that monster will immediately wake up, assuming that it does not have the `AMBUSH` flag.

By default, in MBF25, when a sector sets its `soundtarget` property, it will also set its `oldsoundtarget` property to the same mobj, and its `soundtargetcountdown` property will be set to 20. At the start of every game tic, the sector's `soundtargetcountdown` value decreases by 1, and when it reaches 0, the sector's `soundtarget` property will be cleared. Enemies with the `AMBUSH` flag (either via the Thing definition or the map thing flag) will still look in all directions for the `oldsoundtarget` if one exists for the sector. In addition, enemies with the `Reinforcement` map object flag will immediately wake up if their sector has a valid `oldsoundtarget`, which allows preserving of the old behavior if desired.

This behavior is controlled by `comp_nostickysoundtarget`.

### Improved `FRIENDLY` monster behavior

In MBF, `FRIENDLY` monsters were introduced, and in order to accommodate them, changes were made to monsters' targeting thresholds and infighting cooldowns. MBF21 reversed this change to restore vanilla infighting behavior, but this had the unintended side effect of making `FRIENDLY` Things behave in unintended ways.

By default, in MBF25, Things with the `FRIENDLY` flag will always behave as though `comp_pursuit` is disabled.

This behavior is controlled by `comp_friendlyfix`.

### Hitscan types

To allow for even further customization, the concept of hitscan types has been implemented and exposed to modders. These allow the customization of bullet puff behavior as well as even simple rail attack setups.

For more information, refer to `hitscan.md`.

### Damage types

A rudimentary damage type system has been implemented. Thing and hitscan type properties, as well as the new parameterized attack codepointers, include a "Damage type" property that can be used in the following methods:
* Enable custom pain, death, and gib states for enemies based on the triggering damage type;
* Damage resistances and immunities;
* Future expansion for additional standards.

### Optional BLOCKMAP improvements

One frequent request has been to implement improved blockmap handling to prevent hitscan attacks from missing enemies that they should hit, but there have also been voices concerned with this. Now, 

## Things

For a list of new Thing properties, flags, and codepointers, refer to things.md.

## Weapons

For a list of new Weapon properties, flags, and codepointers, refer to weapons.md.

## `SNDDEFS`

`SNDDEFS` is an optional lump that can be used to achieve the following:
* Define randomized sound groups, similar to Former Human alert and death sounds, where when one sound is specified, any of the others in the same group may play instead.
* Override default attenuation of a particular sound.
* Prevent a sound from being cut off if the Thing that triggered it no longer exists.
* Prevent a sound from cutting off other sounds made by the same thing, provided there are free sound channels.

(To be determined; this has not yet been formalized and is highly subject to change.)

## New Comp Flags

* `comp_disablehorizontalautoaim`
    * Disables horizontal autoaim adjustment for projectiles spawned by weapon codepointers (default)
    * Exception: weapons with the `FORCEHORIZONTALAUTOAIM` MBF25 flag will still adjust their aim horizontally.
* `comp_fastthings`
    * Improves movement and collision handling for all things moving faster than their size (default)
    * Exception: things with the `FASTCOLLISION` MBF25 flag will always use the improved movement and collision handling.
* `comp_noinfinitemeleeheight`
    * Disables infinite monster melee height (default)
    * Exception: things with the `INFINITEMELEEHEIGHT` MBF25 flag will be considered to have an infinite melee height regardless of this setting.
* `comp_nostickysoundtarget`
    * Clears the `soundtarget` of sectors after 20 in-game tics have elapsed, preventing sleeping enemies that are spawned into or teleport into a sector the player has previously made noise in from waking up immediately (default)
    * Exception: enemies with the `AMBUSH` flag or `Reinrocement` map object flag will still wake up immediately.
* `comp_friendlyfix`
    * `FRIENDLY` monsters will always behave as though `comp_pursuit` is disabled, thus causing them to use the MBF infighting cooldown.