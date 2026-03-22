# MBF2y MVP Specification Proposal

This is pared down version of the draft MBF2y specification. The actual specification is very likely to have more features than this, and codepointers listed here are likely to have more capabilities and arguments than actually listed; however, this is what will be considered the "minimum viable product" in that these features are simple enough to implement and/or generally agreed to be useful enough that they warrant inclusion.

## Mission Statement

The goals of the specification are:

- Generalize as much hardcoded behavior as possible;
- Work within the framework of existing Doom engine functionality;
- Draw inspiration from Doom engine games such as Heretic and Hexen when appropriate;
- Preserve classic Doom gameplay feel by default in the same vein as MBF21 while giving modders extended capabilities;
- Build upon features introduced by ID24HACKED while being developer-, speedrunner-, modder-, and demo-friendly.

## General Mapping Enhancements

The following map enhancements, first referenced in the Boom spec and already adopted by Eternity Engine, will be formally adopted as part of the spec.

#### MBF2y UDMF Namespace
This namespace will require its own documentation. At a minimum, it will include all MBF21 and ID24 map features, converted to the UDMF space, plus the following features:
- New linedef flags
  - 3D midtex
  - Midtex blocks missiles
  - Midtex blocks hitscan
- New sector flags
  - Sector does not make noise when floor or ceiling moves
  - Sounds originating in sector do not play

## Frames

#### Args expansion
- In order to facilitate enhanced codepointers and parameterization, it was necessary to extend the amount of Args.
- When there are simple binary flags that can be set on a codepointer, they are combined into a `flags` Arg that accepts mnemonics or numeric values. DECLARATE parsers should avoid accepting mnemonics for these Args outside of the correct Args fields and associated codepointers if possible.
  - Though the documentation attempts to make this explicit, as a general rule, when two flags for the same codepointer contradict one another, the one with the larger value takes precedence over the other.
- As part of the transition to DECLARATE, there have been some adjustments to how sounds, states, and things are referenced when referenced by Args in codepointers, with the following datatypes to match:
  - Things are now defined by and referred to by thing names instead of index. These will be referred to as `thingtype`.
  - Sounds will be referred to by sound name or alias (depending on the outcome of sound definition discussions) instead of by sound number. These will be referred to as `sound`.
  - States will now be referred to by labels that are scoped only to the entity that contains those things, which prevents things from sharing states (such as `Spawn`, `See+3`, etc., with support for custom labels). These will be referred to as `state`.

## Flat damage support

Parameterized damage is enhanced to allow flat damage amounts, which can be used either independently or alongside traditional randomized damage. Previously, it would be considered an error state if `daamgedice` in a damage calculation were set to `0`; however, this is no longer the case. This will instead result in any non-`flatdamage` damage being considered `0`.

In general, when `flatdamage` is added to a parameterized damage calculation, the formula is: `damage = flatdamage + (damagebase * random(1, damagedice))`.

## Things

#### Internal Mobj Properties
- `damagedthing (mobj_t)`: Current thing that a ripper projectile is damaging. This is cleared when the projectile is no longer overlapping a thing and is updated every time a ripper projectile moves.
  - Potential enhancements: this could be leveraged for projectiles in general to allow for a codepointer to jump to a different state if the thing being damaged or triggering a death state is a specific thing.
- `floatbobphase (int)`: Current phase of a thing's FLOATBOB movement. This is randomized upon thing spawn/initialization via `M_Random` and increments every tic, rolling over to 0 when it hits 64. This is will be used to load from a new visual-only Z displacement array for visual float-bobbing effects.
  - Potential enhancements: allow parameterization of a thing's FLOATBOB distance, speed, and frequency.

#### Crush state
- State that a thing jumps to when its corpse has been crushed by a moving ceiling or floor.
- If a monster does not have a crush state set, it should jump to `S_GIBS`, as is vanilla behavior.
- For the time being, we should not assume that crush state should inherit blood translation. Modders using this attribute should use frame translation from ID24HACKED for this instead.

#### Blood thing
- The thing that gets spawned when a thing would bleed if it doesn't don't have NOBLOOD set.
- Defaults to the blood splat.
- Custom blood things must have the BLOODSTATES flag set if they want to emulate the behavior of skipping to their `Spawn+1` or `Spawn+2` frame if less damage is dealt.

#### Blood translation
- Replacement for `Blood color` that overrides the Translation lump that a thing's `Blood thing` uses.

#### Gib health
- Negative value of health at which a thing should enter its gib death (XDeath) state.
- If not set, things with an XDeath state will reach it at `(-0.5 * health)`.
- Formalized from Doom Retro and Crispy Doom.

#### Melee threshold
- Distance (map units, in fixed point) from which a thing chasing its target will attempt to use melee attacks instead of ranged attacks.
- Is overridden if the `LONGMELEE` MBF21 flag is set.
- Default value is `0.0`.
- Formalized from Doom Retro and Crispy Doom; converted to fixed point for MBF2y and DECLARATE.

#### Max attack range
- Maximum distance (map units, in fixed point) from which a thing will make ranged attacks at its target.
- Is overridden if the `SHORTMRANGE` MBF21 flag is set.
- Default value is `0.0`, which means unlimited.
- Formalized from Doom Retro and Crispy Doom; converted to fixed point for MBF2y and DECLARATE.

#### Min missile chance
- Minimum chance (out of 255) that a thing will make a ranged attack when given an opportunity to do so.
- Is overridden if the `HIGHERMPROB` MBF21 flag is set.
- Default value is `160`.
- Formalized from Doom Retro and Crispy Doom.

#### Missle chance mult
- Typically, the closer an enemy is to the player, the more likely they are to fire; some enemies, like the Cyberdemon, consider the player to be at half the distance from them that they actually are for this probability. This generalizes that value by applying to the actual distance between a thing and its target for the purposes of weapon attack probability (but not the max attack range) as a multiplier (in fixed point notation).
- Is overridden if the `RANGEHALF` MBF21 flag is set.
- Default value is `1.0`.
- Formalized from Doom Retro and Crispy Doom.

#### Damage dice
- Attack daamge random multiplier for projectiles (and lost souls/other things that cause damage upon impact).
- Default value is `8`.

#### Flat damage
- Flat damage amount for projectiles (and lost souls/other things that cause damage upon impact).
- Default value is `0`.

#### Pickup health amount (ID24HACKED extension)
- Parameterizes health amounts for SPECIAL health items.
- For the health bonus, stimpack, medikit, and soulsphere pickup types, if set, this will override the amount of health to add to the player's current health total.
- For the medikit pickup type, if the player's resulting health is equal to or less than twice this value, the "Picked up a medikit that you REALLY need!" message will display.
  - Potential enhancement: allow definition of this string as an additional attribute. This doesn't seem particularly useful, however.
- For the berserk and megasphere pickup types, if the player's health is less than this amount when they pick up this item, it will be set to this amount.

#### Pickup armor amount (ID24HACKED extension)
- Parameterizes armor amounts for SPECIAL armor items.
- For the armor bonus pickup type, if set, this will override the amount of armor to add to the player's current armor total.
- For the green armor and blue armor pickup types, if set, this will override the amount of armor that the player must be under in order to pick up the item and the value of armor that will be set when the item is picked up.

#### New Thing Flags

| Flag              | Value          | Description                                                                                                                                                                           |
|-------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ISMONSTER         | 0x000000000001 | Has no effect with COUNTKILL. Thing is considered a monster; does not spawn if `-nomonsters` is set and is blocked by monster-blocking and impassible linedefs. (Lost Souls)          |
| NODAMAGE          | 0x000000000002 | If SHOOTABLE, thing does not lose health and cannot die, but still reacts as though it has taken damage.                                                                              |
| ANTITELEFRAG      | 0x000000000004 | If thing would be dealt damage as a result of a telefrag, it instead deals the damage to the source.                                                                                  |
| NOTAUTOAIMED      | 0x000000000008 | Thing is completely ignored by all player autoaim. Note that this does not affect BFG sprays or monster attacks.                                                                      |
| NOTSPRAYED        | 0x000000000010 | Thing is completely ignored by all `A_BFGSpray` and parameterized equivalents.                                                                                                        |
| IMMOVABLE         | 0x000000000020 | Thing does not move when taking damage from any source.                                                                                                                               |
| NOPAIN            | 0x000000000040 | Thing does not enter its pain state when taking damage.                                                                                                                               |
| ONLYSLAMSOLID     | 0x000000000080 | If thing is charging (such as by calling `A_SkullAttack`), it will not stop upon contact with things unless they are SOLID and/or SHOOTABLE.                                          |
| KEEPCHARGETARGET  | 0x000000000100 | If thing is charging (such as by calling `A_SkullAttack`), it will continue to pursue its target upon collision with an obstacle unless its target is dead.                           |
| UNSTOPPABLECHARGE | 0x000000000200 | If thing is charging (such as by calling `A_SkullAttack`), it will not lose momentum upon taking damage unless it dies or enters its pain state.                                      |
| FLOATBOB          | 0x000000000400 | Renderer-only float bobbing, similar to what's seen in UZDoom. Does not actually change an object's Z position. This flag is removed by `A_Fall`.                                     |
| DEADFLOAT         | 0x000000000800 | Thing does not lose gravity upon calling `A_Fall`. (Lost Souls)                                                                                                                       |
| FASTPROJECTILE    | 0x000000001000 | Thing uses accurate hit detection. (To be determined: if this will take the form of splitting up the movement per gametic, or something even more accurate can be used.               |
| TUNNEL            | 0x000000002000 | If thing is a ripper projectile, it will only deal damage to each SHOOTABLE thing it touches once until it touches a different SHOOTABLE thing or isn't touching a SHOOTABLE thing.   |
| NORIPBLOOD        | 0x000000004000 | If thing is a ripper projectile, it does not cause things it hits to call `P_SpawnBlood` or `P_SpawnPuff`.                                                                            |
| LOYAL             | 0x000000008000 | Thing will never infight. If FRIENDLY, will never retaliate against other FRIENDLY things or the player; otherwise, will only ever attack FRIENDLY things or the player.              |
| NORADTHRUST       | 0x000000010000 | When thing deals radius damage via a codepointer, it will never apply thrust to damaged things.                                                                                       |
| NODMGTHRUST       | 0x000000020000 | When thing deals damage upon impact (such as when acting as projectile or charging Lost Soul), it will never apply thrust to damaged things.                                          |
| ANTIGRAV          | 0x000000040000 | Gravity moves this thing upward instead of downward. Can be combined with LOGRAV.                                                                                                     |
| STAYONFLAT        | 0x000000080000 | Thing will never move into a sector with a different floor texture.                                                                                                                   |
| SEMISOLID         | 0x000000100000 | Thing cannot be moved into by other things as though it is SOLID, but if SOLID things are overlapping it, both it and those things can still move normally.                           |
| DUMMY             | 0x000000200000 | Thing is treated as a player for the purposes of crossing special linedefs.                                                                                                           |
| BLOODSTATES       | 0x000000400000 | When thing is spawned via `P_SpawnBlood`, it will jump to `Spawn+1` if source damage is between `12` and `9` and `Spawn+2` if source damage is less than `9`.                         |
| HITIMPASSIBLE     | 0x000000800000 | Thing cannot cross impassible linedefs whatsoever; if it's a MISSILE, it explodes on impact.                                                                                          |
| NOCRUSH           | 0x000001000000 | Thing does not get turned into gibs or a Crush Thing (or, if DROPPED, get destroyed) when crushed by a moving ceiling or floor.                                                       |
| NOSCROLL          | 0x000002000000 | Thing is not moved by floor scrollers (or ceiling scrollers, if it has SPAWNCEILING and is on the ceiling).                                                                           |
| GENERIC1...4      | Last 4 values  | Generic thing flags that have no hardcoded purpose and can be used for binary logic trees.                                                                                            |

#### Updated Thing Codepointers

Besides the general changes made to codepointer parameterization as a result of the transition to DECLARATE, the following codepointers have had additional arguments appended:

- **A_JumpIfFlagsSet(state, flags, flags2, flags3, flags4)**
  - Jumps to a state if caller has the specified thing flags set.
  - Args:
    - `state (uint)`: State to jump to.
    - `flags (int)`: Standard actor flag(s) to check.
    - `flags2 (int)`: MBF21 actor flag(s) to check.
    - `flags3 (int)`: ID24HACKED actor flag(s) to check.
    - `flags4 (int)`: MBF2y actor flag(s) to check.
  - Notes:
    - If multiple flags are specified in a field, jump will only occur if all the flags are set (e.g. AND comparison, not OR).

- **A_AddFlags(flags, flags2, flags3, flags4)**
  - Adds the specified thing flags to the caller.
  - Args:
    - `flags (int)`: Standard actor flag(s) to add.
    - `flags2 (int)`: MBF21 actor flag(s) to add.
    - `flags3 (int)`: ID24HACKED actor flag(s) to add.
    - `flags4 (int)`: MBF2y actor flag(s) to add.

- **A_RemoveFlags(flags, flags2, flags3, flags4)**
  - Removes the specified thing flags from the caller.
  - Args:
    - `flags (int)`: Standard actor flag(s) to remove.
    - `flags2 (int)`: MBF21 actor flag(s) to remove.
    - `flags3 (int)`: ID24HACKED actor flag(s) to remove.
    - `flags4 (int)`: MBF2y actor flag(s) to remove.

#### New Thing Codepointers

- **A_LineEffectEx(special, tag)**
  - Enhanced version of MBF's `A_LineEffect` that works more reliably and can also be used to activate teleportation specials, which causes the caller to teleport to a teleport destination in the tagged sector.
  - Args:
    - `special (uint)`: Linedef special to activate. Must be a S1, SR, W1, WR, G1, or GR type.
    - `tag (int)`: Sector tag to trigger.
  - Notes:
    - Unlike `A_LineEffect`, all specials are considered repeatable.
    - For non-teleportation specials, the tagged sectors will be activated as though the caller were a player, with the sector that the caller is in being the sector on the front side of the linedef's dummy sector.
    - For teleportation specials, the caller will attempt to teleport to a teleport destination in a tagged sector if it is able to do so as though it had crossed an actual linedef. If it is unable to do so, either because the teleport destination is blocked or because it doesn't exist, it will no-op.

- **A_SpawnObjectEx(type, angle, x_ofs, y_ofs, z_ofs, x_vel, y_vel, z_vel, flags)**
  - Enhanced version of MBF21's `A_SpawnObject` with additional functionality.
  - Args:
    - `type (thingtype)`: Type (dehnum) of actor to spawn.
    - `angle (fixed)`: Angle (degrees), relative to calling actor's angle.
    - `x_ofs (fixed)`: X (forward/back) spawn position offset.
    - `y_ofs (fixed)`: Y (left/right) spawn position offset.
    - `z_ofs (fixed)`: Z (up/down) spawn position offset.
    - `x_vel (fixed)`: X (forward/back) velocity.
    - `y_vel (fixed)`: Y (left/right) velocity.
    - `z_vel (fixed)`: Z (up/down) velocity.
    - `flags (uint)`: Mnemonics that affect the following behavior:
      - `A_SPEX_INHERITTARGET (0x0001)`: Sets the spawnee's `target` to the spawner's `target`, regardless of whether nor not the spawnee is a missile.
      - `A_SPEX_TARGETISTRACER (0x0002)`: Sets the spawnee's `tracer` to the spawner's `target`, regardless of whether or not the spawnee is a missile.
      - `A_SPEX_SPAWNERISTARGET (0x0004)`: Sets the spawnee's `target` to the spawner, regardless of whether or not the spawnee is a missile. Supercedes `A_SPEX_INHERITTARGET`.
      - `A_SPEX_SPAWNERISTRACER (0x0008)`: Sets the spawnee's `tracer` to the spawner, regardless of whether or not the spawnee is a missile. Supercedes `A_SPEX_INHERITTRACER`.
      - `A_SPEX_SPAWNEETRACER (0x0010)`: Sets the spawner's `tracer` to the spawnee.
      - `A_SPEX_CHECKCOLLISION (0x0020)`: Causes the spawnee to be destroyed if it would collide with an obstacle upon creation, not including the spawner.
      - `A_SPEX_INHERITFRIENDLY (0x0040)`: Forces the spawnee to have the same FRIENDLY status as the spawner.
      - `A_SPEX_INHERITAMBUSH (0x0080)`: Forces the spawnee to have the same AMBUSH status as the spawner.
      - `A_SPEX_INHERITGENERIC1 (0x0100)`: Forces the spawnee to have the same GENERIC1 status as the spawner.
      - `A_SPEX_INHERITGENERIC2 (0x0200)`: Forces the spawnee to have the same GENERIC2 status as the spawner.
      - `A_SPEX_INHERITGENERIC3 (0x0400)`: Forces the spawnee to have the same GENERIC3 status as the spawner.
      - `A_SPEX_INHERITGENERIC4 (0x0800)`: Forces the spawnee to have the same GENERIC4 status as the spawner.
      - `A_SPEX_INHERITHEALTH (0x1000)`: Sets the spawnee's health to the spawner's health immediately upon spawn.
      - `A_SPEX_INHERITTRANSLATION (0x2000)`: Changes the spawnee's translation lump to the spawner's translation lump. Use case is for gore mods using colored blood.
  - Notes:
    - If the spawnee is a missile, the `tracer` and `target` pointers are set as follows (assuming that there isn't a `flags` option to contradict it):
      - If spawner is also a missile, `tracer` and `target` are copied to spawnee.
      - Otherwise, spawnee's `target` is set to the spawner and `tracer` is set to the spawner's `target`.

- **A_DropItem(thing, flags)**
  - Drops an item as if it were dropped by the thing upon death.
  - Args:
    - `thing (thingtype)`: Thing to drop.
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_DROP_NOTDROPPED (0x001)`: Does not add the DROPPED flag to the object.

- **A_JumpIfTargetFlagsSet(state, flags, flags2, flags3, flags4)**
  - Jumps to a state if caller if its target has the specified thing flags set.
  - Args:
    - `state (uint)`: State to jump to.
    - `flags (int)`: Standard actor flag(s) to check.
    - `flags2 (int)`: MBF21 actor flag(s) to check.
    - `flags3 (int)`: ID24HACKED actor flag(s) to check.
    - `flags4 (int)`: MBF2y actor flag(s) to check.
  - Notes:
    - If multiple flags are specified in a field, jump will only occur if all the flags are set (e.g. AND comparison, not OR).

- **A_JumpIfTracerFlagsSet(state, flags, flags2, flags3, flags4)**
  - Jumps to a state if caller if its tracer target has the specified thing flags set.
  - Args:
    - `state (uint)`: State to jump to.
    - `flags (int)`: Standard actor flag(s) to check.
    - `flags2 (int)`: MBF21 actor flag(s) to check.
    - `flags3 (int)`: ID24HACKED actor flag(s) to check.
    - `flags4 (int)`: MBF2y actor flag(s) to check.
  - Notes:
    - If multiple flags are specified in a field, jump will only occur if all the flags are set (e.g. AND comparison, not OR).

- **A_AddTargetFlags(flags, flags2, flags3, flags4)**
  - Adds the specified thing flags to the caller's target. Caution is advised when using this codepointer on anything that targets the player mobj, such as monsters or player projectiles, but there can be valid use cases.
  - Args:
    - `flags (int)`: Standard actor flag(s) to add.
    - `flags2 (int)`: MBF21 actor flag(s) to add.
    - `flags3 (int)`: ID24HACKED actor flag(s) to add.
    - `flags4 (int)`: MBF2y actor flag(s) to add.

- **A_RemoveTargetFlags(flags, flags2, flags3, flags4)**
  - Removes the specified thing flags from the caller's target. Caution is advised when using this codepointer on anything that targets the player mobj, such as monsters or projectiles, but there can be valid use cases.
  - Args:
    - `flags (int)`: Standard actor flag(s) to remove.
    - `flags2 (int)`: MBF21 actor flag(s) to remove.
    - `flags3 (int)`: ID24HACKED actor flag(s) to remove.
    - `flags4 (int)`: MBF2y actor flag(s) to remove.

- **A_AddTracerFlags(flags, flags2, flags3, flags4)**
  - Adds the specified thing flags to the caller's tracer target. Caution is advised when using this codepointer on anything that traces the player mobj, such as monster projectiles, but there can be valid use cases.
  - Args:
    - `flags (int)`: Standard actor flag(s) to add.
    - `flags2 (int)`: MBF21 actor flag(s) to add.
    - `flags3 (int)`: ID24HACKED actor flag(s) to add.
    - `flags4 (int)`: MBF2y actor flag(s) to add.

- **A_RemoveTracerFlags(flags, flags2, flags3, flags4)**
  - Removes the specified thing flags from the caller's tracer target. Caution is advised when using this codepointer on anything that traces the player mobj, such as monster projectiles, but there can be valid use cases.
  - Args:
    - `flags (int)`: Standard actor flag(s) to remove.
    - `flags2 (int)`: MBF21 actor flag(s) to remove.
    - `flags3 (int)`: ID24HACKED actor flag(s) to remove.
    - `flags4 (int)`: MBF2y actor flag(s) to remove.

- **A_MonsterRefire(state, noloschance, maxrange, flags)**
  - Generic monster refire codepointer that parameterizes the refiring behavior of `A_CposRefire` and `A_SpidRefire`. Addresses issues with `A_JumpIfTargetInSight` not checking if the target is alive when used for refiring, including checking to see if a friend has gotten in the way first.
  - Args:
    - `state (state)`: State to jump to if the refire check succeeds.
    - `noloschance (int)`: Chance out of 255 that, if the caller loses sight of its target, that the refire check will still succeed; if not set, defaults to `0`.
    - `maxrange (fixed)`: Maximum distance (map units, in fixed point) that the target can be from its caller for the refire check to still succeed; if not set, defaults to `0.0` (no limit).
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_MREF_IGNOREFRIENDS (0x001)`: Bypasses the check for a friendly monster before performing the refire check.
      - `A_MREF_USEMISSILERANGE (0x002)`: If `maxrange` is set to `0.0`, uses the thing's missile range (whether set explicitly, left at default values, or set via `SHORTMRANGE`).

- **A_JumpIfPlayerCloser(state, distance, flags)**
  - Jumps to the provided state if any living player is within the given distance of the caller.
  - Args:
    - `state (state)`: State label to jump to if true.
    - `distance (fixed)`: Distance (map units, in fixed point) to check.
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_JIPC_SETTARGET (0x001)`: Sets the closest qualifying player to the caller's target if one is found.
      - `A_JIPC_SETTRACER (0x002)`: Sets the closest qualifying player to the caller's tracer if one is found.

- **A_JumpIfTargetZCloser(state, distance, flags)**
  - Jumps to the provided state if the caller's target's Z position is within a certain distance of the caller's Z position.
  - Args:
    - `state (state)`: State label to jump to if true.
    - `distance (fixed)`: Distance to check (map units, in fixed point).
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_JTZC_TARGETMUSTBEHIGHER (0x001)`: Only returns true if the target is higher than the caller.
      - `A_JTZC_CONSIDERTARGETHEIGHT (0x002)`: Considers both the target's Z position and the target's Z-position + height; otherwise, only considers the Z position.
      - `A_JTZC_CONSIDERCALLERHEIGHT (0x004)`: Considers both the caller's Z position and the caller's Z-position + height; otherwise, only considers the Z position.

- **A_JumpIfTracerZCloser(state, distance, flags)**
  - Jumps to the provided state if the caller's tracer target's Z position is within a certain distance of the caller's Z position.
  - Args:
    - `state (state)`: State label to jump to if true.
    - `distance (fixed)`: Distance to check (map units, in fixed point).
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_JTZC_TARGETMUSTBEHIGHER (0x001)`: Only returns true if the tracer target is higher than the caller.
      - `A_JTZC_CONSIDERTARGETHEIGHT (0x002)`: Considers both the tracer target's Z position and the target's Z position + height; otherwise, only considers the Z position.
      - `A_JTZC_CONSIDERCALLERHEIGHT (0x004)`: Considers both the caller's Z position and the caller's Z position + height; otherwise, only considers the Z position.

- **A_StepEx(angle, flags)**
  - Makes the thing take a step in a given direction. This does not require the thing to have a target.
  - Args:
    - `angle (fixed)`: Angle (degrees), relative to caller's angle.
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_STEX_FACEDIR (0x001)`: Makes the thing change its facing direction/angle to the direction it steps.
      - `A_STEX_ACTSOUND (0x002)`: Has a chance to play the thing's active sound as though it were pursuing a target.
      - `A_STEX_USESPECIAL (0x004)`: Allows the thing to use any valid linedef specials (such as doors) that it collides with.

- **A_LookEx(fov, maxdistance, flags)**
  - Enhanced version of A_Look that allows for fine-tuning certain behaviors.
  - Args:
    - `fov (fixed)`: Field of view (degrees, in fixed point) that the enemy will look in; defaults to `180.0` if not set. Enemies with AMBUSH set that are in a sector with a valid soundtarget will look in all directions.
    - `maxdistance (fixed)`: Maximum view distance (map units, in fixed point) that the enemy will spot targets from, not including soundtargets; defaults to 0 (no maximum) if not set.
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_LKEX_ACTSOUND (0x001)`: Play the active sound a thing normally makes during A_Chase even though it's not yet active.
      - `A_LKEX_AMBUSHNOMAXDIST (0x002)`: Bypass maxdistance if AMBUSH is set and there is a valid soundtarget.
      - `A_LKEX_AMBUSHUSEFOV (0x004)`: Use fov even if the thing is set to AMBUSH and has a valid soundtarget.
      - `A_LKEX_NOSEESOUND (0x008)`: Do not play the thing's see sound if it spots a target

- **A_Wander(maxturn, turnchance, fov, flags)**
  - Generic movement codepointer for aimlessly wandering monsters that haven't yet obtained a target. Calls `A_LookEx`, and if a thing would move into a linedef that it can press such as a door, it will activate it.
  - Args:
    - `maxturn (uint)`: Maximum number of rotations a monster will make when it decides to turn in either direction (barring collisions), with a value of `0` meaning no turning unless a collision is detected; if not set, defaults to `1`. `4` is the greatest practical value, and any value greater than that should be interpreted as `4`.
    - `turnchance (uint)`: Likelihood that a monster will change directions without having first run into an obstacle, out of `255`. If not set, defaults to `6`.
    - `fov (fixed)`: `fov` value to pass through to `A_LookEx`.
    - `maxdistance (fixed)`: `maxdistance` value to pass through to `A_LookEx`.
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_LKEX_ACTSOUND (0x001)`: Play the active sound a thing normally makes during A_Chase even though it's not yet active. (Passes through to `A_LookEx`).
      - `A_LKEX_AMBUSHNOMAXDIST (0x002)`: Bypass maxdistance if AMBUSH is set and there is a valid soundtarget. (Passes through to `A_LookEx`).
      - `A_LKEX_AMBUSHUSEFOV (0x004)`: Use fov even if the thing is set to AMBUSH and has a valid soundtarget. (Passes through to `A_LookEx`).
      - `A_LKEX_NOSEESOUND (0x008)`: Do not play the thing's see sound if it spots a target. (Passes through to `A_LookEx`).

- **A_ChaseEx(movespeed, missilechance, straferange, strafespeed, strafechance, strafetime, flags)**
  - Enhanced version of `A_Chase` that inherits and parameterizes some behavior used by Hexen monsters while providing further enhancement.
  - Args:
    - `movespeed (fixed)`: Movement speed (map units, in fixed point) at which the caller will move; if not set, defaults to its `Speed` value.
    - `missilechance (uint)`: If non-zero, overrides the minimum ranged attack chance; a value of `-1` means that it will not perform any ranged attacks whatsoever.
    - `straferange (fixed)`: Range from its target (map units, in fixed point) at which the caller may begin to strafe; if not set, defaults to `640.0`.
    - `strafespeed (fixed)`: Movement speed (map units, in fixed point) at which the caller will move while strafing; if not set, defaults to `movespeed` or its `Speed` value.
    - `strafechance (uint)`: Chance out of `255` that the caller will begin to strafe if within `straferange` of its target; if not set, defaults to `0`, which means it will never strafe.
    - `strafetime (uint)`: Minimum number of steps that the caller will take when it decides to strafe; if not set, defaults to `0`.
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_CHEX_NOACTSOUND (0x001): Suppress the active sound a thing normally makes.
      - `A_CHEX_FACETARGET (0x002): Force thing to face its target regardless of movement direction.
      - `A_CHEX_STRAFETARGET (0x004): Force thing to face its target regardless of movement direction, but only if strafing.
  - Notes:
    - For an implementation of the strafe behavior, look at `A_FastChase` in the DSDA-Doom codebase, which is used for enemies from Hexen. This is roughly equivalent to `A_ChaseEx(0, 0, 640.0, 0, 100, 3, 0)`.

- **A_AddHealth(health, maxhealth, flags)**
  - Adds health to the caller's current health value. Supports negative values.
  - Args:
    - `health (int)`: Amount of health to add.
    - `maxhealth (uint)`: Maximum amount of health that health will be set to; if not set, defaults to the thing's spawn health.
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_ADDH_PERCENTAGE (0x001)`: Treats `health` as a percentage of the caller's spawn health instead.

- **A_SetHealth(health, flags)**
  - Sets the caller's current health value. Supports negative values.
  - Args:
    - `health (int)`: Amount of health to set.
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_SETH_PERCENTAGE (0x001)`: Treats `health` as a percentage of the caller's spawn health instead.

- **A_AddTargetHealth(health, maxhealth, flags)**
  - Adds health to the caller's target's current health value. Supports negative values.
  - Args:
    - `health (int)`: Amount of health to add.
    - `maxhealth (uint)`: Maximum amount of health that health will be set to; if not set, defaults to the thing's target's spawn health.
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_ADTH_PERCENTAGE (0x001)`: Treats `health` as a percentage of the caller's target's spawn health instead.
      - `A_ADTH_TRACER (0x002)`: Use the caller's tracer target instead of the target.

- **A_SetTargetHealth(health, flags)**
  - Sets the caller's target's current health value. Supports negative values.
  - Args:
    - `health (int)`: Amount of health to set.
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_STTH_PERCENTAGE (0x001)`: Treats `health` as a percentage of the caller's spawn health instead.
      - `A_STTH_TRACER (0x002)`: Use the caller's tracer target instead of the target.

- **A_MonsterProjectileEx(type, angle, pitch, hoffset, voffset, hspread, vspread, flags)**
  - Enhanced version of MBF21's `A_MonsterProjectile`.
  - Args:
    - `type (thingtype)`: Type of actor to spawn.
    - `angle (fixed)`: Angle (degrees), relative to calling actor's angle.
    - `pitch (fixed)`: Pitch (degrees), relative to calling actor's pitch.
    - `hoffset (fixed)`: Horizontal spawn offset, relative to calling actor's angle.
    - `voffset (fixed)`: Vertical spawn offset, relative to actor's default projectile fire height.
    - `hspread (fixed)`: Horizontal spread (degrees, in fixed point).
    - `vspread (fixed)`: Vertical spread (degrees, in fixed point).
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_MPEX_ADJUSTFORANGLE (0x001)`: Considers the `angle` value for autoaim purposes, looking for targets at the angle in which the target is fired instead of the player's angle.
      - `A_MPEX_ADJUSTFORHOFFSET (0x002)`: Considers the `hoffset` value for autoaim purposes, looking for targets based on the player's position adjusted for the `hoffset` value instead of just the player's position.
      - `A_MPEX_ADJUSTFORHOFFSET (0x004)`: Considers the `hoffset` value for autoaim purposes, looking for targets based on the player's Z-position adjusted for the `voffset` value instead of just the player's Z-position.
      - `A_MPEX_IGNORESHADOW (0x008)`: The attack does not become less accurate if the caller's target has SHADOW set.
  - Notes:
    - The spawned projectile's `tracer` pointer is always set to the spawner's `target`, for generic seeker missile support.

- **A_MonsterBulletAttackEx(hspread, vspread, numbullets, damagebase, damagedice, flatdamage, puffthing, flags)**
  - Enhanced version of MBF21's `A_MonsterBulletAttack`.
  - Args:
    - `hspread (fixed)`: Horizontal spread (degrees, in fixed point).
    - `vspread (fixed)`: Vertical spread (degrees, in fixed point).
    - `numbullets (uint)`: Number of bullets to fire; if not set, defaults to `1`.
    - `damagebase (uint)`: Base damage of attack; if not set, defaults to `3`.
    - `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to `5`.
    - `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to `0`.
    - `puffthing (thing)`: Puff to spawn upon hitscan impact; if not set, defaults to the basic bullet puff, but can be set to a null thing.
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_MBAX_ALWAYSPUFF (0x001)`: Spawn the puff thing even if a shootable thing is hit.
      - `A_MBAX_NOBLOOD (0x002)`: Don't spawn blood things when they're hit, or bullet puffs on things with NOBLOOD set.
      - `A_MBAX_NOTARGET (0x004)`: Skip `A_FaceTarget`, checking for a target, or aiming; instead, simply perform the attack in the thing's current direction without any aim adjustment.
      - `A_MBAX_IGNORESHADOW (0x008)`: The attack does not become less accurate if the caller's target has SHADOW set.
  - Notes:
    - Damage formula is: `damage = flatdamage + (damagebase * random(1, damagedice))`.

- **A_MonsterMeleeAttackEx(damagebase, damagedice, flatdamage, sound, range, flags)**
  - Enhanced version of MBF21's `
  - Args:
    - `damagebase (uint)`: Base damage of attack; if not set, defaults to `3`.
    - `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to `8`.
    - `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to `0`.
    - `sound (sound)`: Sound to play if attack hits.
    - `range (fixed)`: Attack range; if not set, defaults to calling actor's melee range property.
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_MMAX_SPAWNBLOOD (0x001)`: If target bleeds (has a blood thing), then spawn it on hit.
  - Notes:
    - Damage formula is: `damage = flatdamage + (damagebase * random(1, damagedice))`.

- **A_MonsterSpawnAttack(thingtype, state, angle, pitch)**
  - Parameterized version of the Pain Elemental attack. Spawns the specified enemy (destroying it if it would collide with an obstacle) and has it inherit the caller's target, if it has one.
  - Args:
    - `thingtype (thingtype)`: Enemy type to spawn. No check is performed to make sure that this is a valid enemy; this is the modder's responsibility.
    - `state (state)`: State that the spawned enemy should jump to. Default is "Missile".
    - `angle (fixed)`: Angle (degrees, in fixed point) at which the new enemy should be angled horizontally relative to the caller's angle.
    - `pitch (fixed)`: Pitch (degrees, in fixed point) at which the new enemy should be angled vertically relative to the caller's pitch.

- **A_FireEx(sound, flags)**
  - Parameterized version of `A_Fire` and `A_CrackleFire` that allows choosing a different sound. Causes an object to snap to its tracer target so long as its target can see its tracer target (by default).
  - Args:
    - `sound (soundname)`: Sound to play (optional).
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_FREX_FULLVOL (0x001)`: Play the sound at full volume.
      - `A_FREX_SKIPLOSCHECK (0x002)`: Perform the snap to the caller's tracer target even if the caller's target can't see the caller's tracer target.

- **A_MonsterFogTarget(fogtype, sound, flags)**
  - Parameterized version of `A_VileTarget`. Creates the specified thing, setting its target to the caller and its tracer to the caller's target. Also calls A_FireEx on behalf of the spawned object, passing through the sound and flags.
  - Args:
    - `fogtype (thingtype)`: Thing type to spawn on the caller's target.
    - `sound (soundname)`: Sound to pass through to `A_FireEx` on behalf of the spawned object.
    - `flags (uint)`: Mnemonics to pass through to `A_FireEx` on behalf of the spawned object.
      - `A_FREX_FULLVOL (0x001)`: Play the sound at full volume.
      - `A_FREX_SKIPLOSCHECK (0x002)`: Perform the snap to the caller's tracer target even if the caller's target can't see the caller's tracer target.

- **A_MonsterZapAttack(sound, damagebase, damagedice, flatdamage, radius, radiusdamage, thrust, flags)**
  - Parameterized version of `A_VileAttack`. Deals immediate damage to the caller's target based on line of sight. If the caller also has a tracer active and line of sight is present, will also deal radius damage.
  - Args:
     - `sound (soundname)`: Sound to play if radius damage is dealt (optional).
     - `damagebase (uint)`: Base damage of attack; if not set, defaults to `0`.
     - `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to `0`.
     - `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to `20`.
     - `radius (fixed)`: Radius damage size (map units, in fixed point); if not set, defaults to `70.0`.
     - `radiusdamage (uint)`: Damage to deal if radius damage is dealt; if not set, defaults to `70`.
     - `thrust (uint)`: Amount of upward thrust to apply to the target upon a successful attack; if not set, defaults to 1000.
     - `flags (uint)`: Mnemonics that adjust the following logic:
       - `A_MZAP_CONSTTHRUST (0x001)`: Applies thrust as if the object has the same mass as the player's default mass.
       - `A_MZAP_USETRACERLOS (0x002)`: All line of sight checks are performed from the caller's tracer instead of from the caller; causes the attack to no-op if the ta
       - `A_MZAP_ALWAYSPLAYSOUND (0x004)`: Always play the specified sound, even when radius damage is not dealt.

- **A_MonsterChargeAttack(speed, sound, flags)**
  - Parameterized version of A_SkullAttack.
  - Args:
    - `speed (uint)`: Speed at which to charge.
    - `sound (soundname)`: Sound to play. If not set, defaults to the caller's attack sound.
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_MCHA_FULLVOL (0x001)`: Play the attack sound at full volume.
      - `A_MCHA_SKIPFACETARGET (0x002)`: Do not call A_FaceTarget before initiating the attack.

- **A_MonsterChargeSeek(threshold, maxturnangle)**
  - Variant of `A_SeekTracer` specifically for charging monsters.
  - Args:
    - `threshold (fixed)`: If angle to target is lower than this, missile will 'snap' directly to face the target.
    - `maxturnangle (fixed)`: Maximum angle a misssile will turn towards the target if angle is above the threshold.

- **A_SprayAttack(angle, tracercount, distance, fov, spraything, damagebase, damagedice, flatdamage, flags)**
  - Parameterized version of `A_BFGSpray` that allows parameterization and uses a more consistent damage formula.
  - Args:
    - `angle (fixed)`: Base angle (degrees, in fixed point), relative to the caller's angle, at which to activate the spray; if not set, defaults to 0.0.
    - `tracercount (uint)`: Number of tracers to fire; defaults to `40` if not set.
    - `fov (fixed)`: Size of the cone (degrees, in fixed point), relative to the caller's angle and offset by the angle arg, in which cones will be fired. Maximum value is `360.0`; if not set, defaults to `90.0`.
    - `distance (fixed)`: Maximum distance (map units, in fixed point) from the source of damage (see flags) at which things will be hit; if not set, defaults to `1024.0`.
    - `spraything (thingtype)`: Thing type to spawn over things hit by the spray attack; defaults to no thing if not set.
    - `damagebase (uint)`: Base damage of attack; if not set, defaults to `2`.
    - `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to `20`.
    - `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to `47`.
    - `flags (uint)`: Mnemonics that adjust the following logic:
      - `A_SPRA_CALLERISORIGIN (0x001)`: Instead of using the BFG's logic where the player who fired the projectile (the caller's target) is used for LOS, line of sight will be called from the caller's position, adjusted backwards by 1/2 of its radius.
      - `A_SPRA_CANHITTARGET (0x002)`: By default, A_SprayAttack will never hit the caller's target. This bypasses this logic.
  - Notes:
    - Default damage values have been provided to approximate the same range as the BFG9000 tracer and provide a somewhat similar bell curve, but in order to faciliatate parameterization, the distribution will differ.
    - If the `spraything` is a missile (has `MISSILE` or `BOUNCES`), the `tracer` and `target` pointers are set as follows:
      - If caller is also a missile, `tracer` and `target` are copied to spawnee.
      - Otherwise, caller's `target` is set to the spawner and `tracer` is set to the caller's `target`.

#### New Weapon Codepointers

- **A_RemoveWeapon**
  - Removes the current weapon from the player's inventory and switches to the next preferred weapon.
  - Args:
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_RMWP_REMOVEAMMO (0x001)`: Also remove all ammunition associated with the weapon's ammotype (if it has one).
  - Notes:
    - No-ops on the fist. This is because a weaponless player is a bad state and should be avoided.

- **A_WeaponBulletAttackEx(hspread, vspread, numbullets, damagebase, damagedice, flatdamage, puffthing, sound, flashstate, flags)**
  - Enhanced version of MBF21's `A_WeaponBulletAttack` that enhances parameterization.
  - Args:
    - `hspread (fixed)`: Horizontal spread (degrees, in fixed point).
    - `vspread (fixed)`: Vertical spread (degrees, in fixed point).
    - `numbullets (uint)`: Number of bullets to fire; if not set, defaults to `1`.
    - `damagebase (uint)`: Base damage of attack; if not set, defaults to `5`.
    - `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to `3`.
    - `flatdamage (uint)`: Flat damage to add to damage; if not set, defaults to `0`.
    - `puffthing (thingtype)`: Puff to spawn upon hitscan impact; if not set, defaults to the basic bullet puff, but can be set to a null thing.
    - `sound (sound)`: Sound index to play (optional).
    - `flashstate (state)`: State to flash to (optional).
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_WBAX_ALWAYSPUFF (0x001)`: Spawn the puff thing even if a shootable thing is hit.
      - `A_WBAX_NOBLOOD (0x002)`: Don't spawn blood things when they're hit, or bullet puffs on things with NOBLOOD set.
      - `A_WBAX_NOPAIN (0x004)`: Don't send hit things to their pain state.
      - `A_WBAX_ALWAYSPAIN (0x008)`: Always send hit things to their pain state (assuming they have one).
      - `A_WBAX_WEAPONALERT (0x010)`: Also call `A_WeaponAlert`; is only called once no matter how many bullets are fired.
      - `A_WBAX_CONSUMEAMMO (0x020)`: Also call `A_ConsumeAmmo`, using the weapon's `ammopershot` value; is only called once no matter how many bullets are fired.
      - `A_WBAX_CHECKAMMO (0x040)`: Also call `A_CheckAmmo`, using the weapon's Lower state and `ammopershot` value; is only called once no matter how many bullets are fired.
      - `A_WBAX_NOAUTOAIM (0x080)`: Do not attempt to adjust the player's aim.
      - `A_WBAX_NOTHRUST (0x100)`: Damage dealt by the attack does not thrust enemies.
      - `A_WBAX_OVERKILL (0x200)`: Lethal damage dealt by the attack will always cause the XDeath state if one is present.
  - Notes:
    - Damage formula is: `damage = flatdamage + (damagebase * random(1, damagedice))`
    - Damage arg defaults are identical to Doom's weapon bullet attack damage values.
    - The order of operations when using the `A_WBAX_CHECKAMMO`, `A_WBAX_CONSUMEAMMMO`, and/or `A_WBAX_WEAPONALERT` flags is `A_CheckAmmo(Lower, ammopershot) > A_ConsumeAmmo(ammopershot) > A_WeaponAlert`.

- **A_WeaponProjectileEx(type, angle, pitch, hoffset, voffset, hspread, vspread, sound, flags)**
  - Enhanced version of MBF21's `A_WeaponProjectile` that enhances paramerization.
  - Args:
    - `type (thingtype)`: Type of actor to spawn.
    - `angle (fixed)`: Angle (degrees), relative to player's angle.
    - `pitch (fixed)`: Pitch (degrees), relative to player's pitch.
    - `hoffset (fixed)`: Horizontal spawn offset, relative to player's angle.
    - `voffset (fixed)`: Vertical spawn offset, relative to player's default projectile fire height.
    - `hspread (fixed)`: Horizontal spread (degrees, in fixed point).
    - `vspread (fixed)`: Vertical spread (degrees, in fixed point).
    - `sound (sound)`: Sound index to play (optional).
    - `flashstate (state)`: State to flash to (optional).
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_WPRX_ADJUSTFORANGLE (0x001)`: Considers the `angle` value for autoaim purposes, looking for targets at the angle in which the target is fired instead of the player's angle.
      - `A_WPRX_ADJUSTFORHOFFSET (0x002)`: Considers the `hoffset` value for autoaim purposes, looking for targets based on the player's position adjusted for the `hoffset` value instead of just the player's position.
      - `A_WPRX_ADJUSTFORHOFFSET (0x004)`: Considers the `hoffset` value for autoaim purposes, looking for targets based on the player's Z-position adjusted for the `voffset` value instead of just the player's Z-position.
      - `A_WPRX_NOHORIZAUTOAIM (0x008)`: Do not adjust the horizontal aim of the projectile, though off-center targets will still be considered for vertical aim adjustment.
      - `A_WPRX_WEAPONALERT (0x010)`: Also call `A_WeaponAlert`.
      - `A_WPRX_CONSUMEAMMO (0x020)`: Also call `A_ConsumeAmmo`, using the weapon's `ammopershot` value.
      - `A_WPRX_CHECKAMMO (0x040)`: Also call `A_CheckAmmo`, using the weapon's Lower state and `ammopershot` value.
      - `A_WPRX_NOAUTOAIM (0x080)`: Do not attempt to adjust the player's aim.
  - Notes:
    - The spawned projectile's `tracer` pointer is set to the player's autoaim target, if available.
    - The order of operations when using the `A_WPRX_CHECKAMMO`, `A_WPRX_CONSUMEAMMMO`, and/or `A_WPRX_WEAPONALERT` flags is `A_CheckAmmo(Lower, ammopershot) > A_ConsumeAmmo(ammopershot) > A_WeaponAlert`.

- **A_WeaponMeleeAttackEx(damagebase, damagedice, flatdamage, zerkfactor, puffthing, sound, range, hitstate, flags)**
  - Enhanced version of MBF21's `A_WeaponMeleeAttack` that enhances parameterization.
  - Args:
    - `damagebase (uint)`: Base damage of attack; if not set, defaults to `2`.
    - `damagedice (uint)`: Attack damage random multiplier; if not set, defaults to `10`.
    - `flatdamage (uint)`: Flat damage to add to attack; if not set, defaults to `0`.
    - `zerkfactor (fixed)`: Berserk damage multiplier; if not set, defaults to `1.0`.
    - `puffthing (thingtype)`: Puff thing to spawn on impact. Note that only the default bullet puff will skip frames based on `range`.
    - `sound (sound)`: Sound index to play if attack hits (optional).
    - `range (fixed)`: Attack range; if not set, defaults to player mobj's melee range property.
    - `hitstate (state)`: State to jump to if attack hits a target (optional).
    - `flags (uint)`: Mnemonics that adjust the following behavior:
      - `A_WMAX_HITMONSTERONLY (0x001)`: Behavior that occurs on hitting a target (such as playing the `sound`, turning the player to the struck thing, or jumping to the `hitstate`) only occurs if the struck thing has the `COUNTKILL` flag and/or `ISMONSTER` MBF2y flag.
      - `A_WMAX_NOTURNONHIT (0x002)`: Do not turn the player toward the thing that gets hit upon hit.
      - `A_WMAX_WEAPONALERTONHIT (0x004)`: Calls `A_WeaponAlert` on hit. This is affected by `A_WMAX_HITMONSTERONLY`.
      - `A_WMAX_CONSUMEAMMOONHIT (0x008)`: Calls `A_ConsumeAmmo` on hit. This is affected by `A_WMAX_HITMONSTERONLY`.
      - `A_WMAX_OVERKILL (0x010)`: Lethal damage dealt by the attack will always cause the XDeath state if one is present.
      - `A_WMAX_ZERKOVERKILL (0x020)`: Lethal damage dealt by the attack will always cause the XDeath state if one is present, but only if the berserk powerup is active.
  - Notes:
    - Damage formula is: `damage = flatdamage + (damagebase * random(1, damagedice))`; this is then multiplied by `zerkfactor` if the player has Berserk.

- **A_WeaponJumpIfBerserk(state)**
  - Jumps to a state if the player has the berserk powerup active.
  - Args:
    - `state (state)`: State label to jump to.
  - Notes:
    - For best results, this should be a zero-duration state called before `A_WeaponReady` frames.

## Miscellaneous
Intentionally, there are fewer compatibility changes in MBF2y than there were in MBF21, as MBF21 does an excellent job of retaining vanilla Doom feel. The changes made were very deliberate.

#### New comp flags
  - `comp_skyhitfix`:
    - When on (default): Use fixed logic for missiles colliding with back sidedefs leading to sky textures.
    - When off: Use current buggy logic instead.

## Potential Additions
The following items were left off of this draft of the spec and may or may not be added.

#### Very Likely Additions
These features have already been successfully implemented in prototypes and are highly likely to be included, but are not considered crucial parts of the specification.
  - **Thing counters:** Counters for things that can be manipulated and compared via codepointers.
  - **Parameterized projectile height:** Alter the default height from which actors fire projectiles.
  - **Parameterized step height:** Alter the default height which an actor can step up or down.
  - **Additional missile enhancements:** Thing flags for floor-hugging, ceiling-hugging, and step-climbing missiles.
  - **Usable things:** Things that can react to player Use presses (including a `Use` label).
  - **Idle state:** Optional state for monsters to return to when their target dies.
  - **Pushable objects:** Thing flags that allow things to be pushed by other things.
  - **Gamestate-based codepointers:** Codepointers that allow jumping to a different frame based on difficulty, the current level, gamemode, etc.

#### Hopeful Additions
These have already been implemented in prototypes to one extent or another and can be implemented, but the extent to which they will be done is still up for discussion, as is the exact nature of implementation.
  - **Damage type support:** Whether via thing flag or arbitary numeric values, some degree of support for different types of damage, with potential for alternative Death/XDeath states and damage resistances/vulnerabilities.
  - **Heal groups:** The ability to control what objects a thing calling `A_HealChase` will attempt to Raise, so that enemies that perform special actions with the codepointer can co-exist alongside standard enemies without resurrecting them.
  - **Patrol codepointer:** Wolfenstein 3-D and Quake-style Patrol movement option for dormant monsters that allows them to follow a set path until they discover a target; would require implementation of a pre-defined path object.
  - **Movement along path codepointer:** Movement option for monsters that forces them to move along a pre-determined path, stopping to attack their target.
  - **Granular projectile collision:** Allowing projectiles to ignore certain things or groups of things for collision purposes.
  - **Stationary damage collision:** Allow objects to deal damage to objects that touch them even when there is no movement involved, perhaps as a thing flag so that it can be toggled for things like stationary traps.
  - **Fix non-flying monsters not being launched by Arch-Vile attacks:** Right now, monsters without NOGRAVITY that attempt to move (such as by calling `A_Chase`) immediately snap to the floor, which prevents them from being launched into the air by Arch-Vile attacks. In some instances, you may even be able to see monsters hit by Arch-Vile attacks and surviving flashing back and forth between the ground and the sky. An internal thing flag that indicates these things have been launched into the air and gets cleared when they hit the ground can prevent this behavior without causing these monsters to do things like wander off ledges.

#### Nice To Haves
These features have not yet been attempted, but discussions have been had about how they might be achieved, and in theory, they should be doable.
  - **Expose GENERIC1, GENERIC2, GENERIC3, and GENERIC4 flags to level editors:** Allow mappers to set these flags (or others) on objects.
  - **Height-preserving teleport linedef specials:** Teleportatino linedefs that preserve the teleporting thing's Z position, a la Final Doom.
  - **Teleport linedef specials with multiple destinations:** Teleportation linedefs that can have multiple destinations with the same tag, selected at random, with non-telefragging objects being taken to the first random vacant destination selected.
  - **Toggle midtex and blocking status of linedef:** Linedef specials that can toggle the visibility and blocking status of a two-sided linedef's midtex (shootable glass, dedicated force fields, etc.).
  - **Changing tagged sector floors and/or ceilings in relation with linedef's sectors:** Linedef specials that can cause tagged sectors to move together with the linedef's sector to create stable multi-sector lifts.
  - **Changing tagged sector light levels in relation with linedef's sector:** Linedef specials that can cause light levels to change across multiple sectors without needing to worry about other neighboring sectors, including variants that only affect the floor or ceiling.
  - **Improved bouncing behavior:** New thing flags that represent improved bouncing behaviors, including the ability to set coefficients based on the surface being collided with.
  - **Support for projectile spread:** `A_MonsterBulletAttackEx`/`A_WeaponBulletAttackEx`-like multiple-projectile support for `A_MonsterProjectileEx`/`A_WeaponProjectileEx`.
  - **Pickup state for SPECIAL items:** Allow objects to jump to a state after being picked up; if they have this state defined, they will lose their SPECIAL flag and jump to it when picked up, while if they do not, they will disappear as normal; could be used to have items leave behind debris when picked up, have non-multiplayer items that respawn, or create booby traps.
  - **Better jumping monster support:** MBF leverages the BOUNCES, DROPOFF, and FLOAT flags to have monsters be able to jump up ledges to pursue their targets, but this is buggy and introduces unwanted behaviors. Having dedicated chase codepointers would be a better approach for modders.
  - **New thing relationships:** In addition to having `target` and `tracer`, borrowing `parent` and `child` ideas could be useful. This could be used to leverage ideas like having `child` monsters remain loyal to their `parent`, `child` things die when their `parent` dies, or facilitate dedicated movement codepointers where `child` things move in relation to their `parent`.

#### Other Potential Additions
These features have only been briefly discussed or suggested.
  - **Sector damage that affects the entire vertical sector:** Sector damage modes that affect flying enemies or players who are not touching the floor. In addition, having options for these to individually affect upper, middle, and lower sectors.
  - **Direct velocity manipulation codepointers:** Codepointers that allow things to affect their own velocity.
  - **Weapon enhancements:** Weapon upgrades that can be picked up. Would involve new pickup types, weapon properties, and weapon codepointers.
  - **Object trails:** Thing properties that allow objects, including fast projectiles, to leave a trail as they move without needing to manually spawn them, including parameterization of distance/frequency. Would allow for the implementation of railgun attacks via fast projectiles without needing absurd numbers of fast projectiles.
  - **Damage thresholds:** Allow definition of things that can shrug off small amounts of damage without losing health, or pain states that only trigger if a certain amount of damage is dealt at once.
  - **Custom powerup definition:** Allow defining of custom, mod-specific powerups. Challenges would include how to define the exact effect they have on the player while also not bloating the spec.
