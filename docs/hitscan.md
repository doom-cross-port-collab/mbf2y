# MBF2y Hitscan Type Definition v0.2.3

**Please note that this particular area of the spec is still in an early draft and is under discussion. It may not make it to the finished specification.**

Hitscan types (sometimes called "pufftypes" due to the usual presence of a bullet puff) are a concept that adds some customization potential to various types of hitscan attacks. Hitscan types have the following properties:

| Property        | DEH Field        | Data type | Description                                                                                                                                                                                                                                        | Default Value                                            |
|-----------------|------------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------|
| `hitscannumber` | N/A              | `uint`    | Unique identifier, used to allow referencing by pointers.                                                                                                                                                                                          | 0 (the default hitscan type; properties are shown below) |
| `defaultpuff`   | `Default puff`   | `int`     | The default "bullet puff" actor to spawn when this hitscan type hits a wall or `SHOOTABLE + NOBLOOD` actor. A negative value means there is no puff object.                                                                                        | `MT_PUFF` (38)                                           |
| `damagetype`    | `Damage type`    | `uint`    | The damage type number to apply to attacks made via the hitscan type. Valid values are 0-9, with 0 being untyped damage.                                                                                                                           | 0                                                        |
| `trailpuff`     | `Trail puff`     | `int`     | If the hitscan type has the `HASTRAIL` flag, then it will spawn this object at an interval determined by the `trailinterval` property. A negative value means there is no trail object.                                                            | `MT_NULL` (-1)                                           |
| `trailinterval` | `Trail interval` | `uint`    | If the hitscan type has the `HASTRAIL` flag and a valid `trailpuff` thing assigned, this is the distance (in map units) at which the `trailpuff` will be spawned along the hitscan trajectory. If set to 0, then the `trailpuff` will never spawn. | 0                                                        |
| `penetration`   | `Penetration`    | `uint`    | The number of `SHOOTABLE` actors that the hitscan type can travel through. A negative value means there is no limit.                                                                                                                               | 0                                                        |
| `flags`         | `Bits`           | `int`     | See the flags below.                                                                                                                                                                                                                               | 0                                                        |

## Hitscan Type Flags

| Flag                 | Value   | Description                                                                                                                                                              |
|----------------------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `NOPUFFONNOBLOODHIT` | 0x00001 | The hitscan type's `defaultpuff` will not spawn when the hitscan type hits a `SHOOTABLE+NOBLOOD` object.                                                                 |
| `PUFFONBLOODHIT`     | 0x00002 | The hitscan type's `defaultpuff` will spawn when the hitscan type hits a a `SHOOTABLE` object without `NOBLOOD`.                                                         |
| `NOBLOODONHIT`       | 0x00004 | `SHOOTABLE` objects without `NOBLOOD` will not spawn their Blood thing when hit by this hitscan type.                                                                    |
| `NOPUFFONWALL`       | 0x00008 | The hitscan type's `defaultpuff` object will not spawn when the hitscan type hits level geometry. Despite the name, this also applies to flats and upper/lower sidedefs. |
| `HASTRAIL`           | 0x00010 | The hitscan type's `trailpuff` will spawn based on the hitscan type's `Trail interval`.                                                                                  |
| `STOPONSOLID`        | 0x00020 | The hitscan type will hit non-`SHOOTABLE` thingsthat have `SOLID`, though no damage will be dealt.                                                                       |
| `PIERCESOLID`        | 0x00040 | When used in conjunction with `STOPONSOLID`, instead of stopping on a `SOLID` thing, the hitscan type will pierce it if it is able to do so.                             |
| `NOPUFFONSOLID`      | 0x00080 | When used in conjunction with `STOPONSOLID`, the hitscan type will spawn its `defaultpuff` on `SOLID` things.                                                            |
| `GIBONPUFFDEATH`     | 0x00100 | If an attack using this hitscan type kills a Thing with an `Exploding frame` (aka XDeath state), it will trigger it.                                                     |
| `NEVERGIB`           | 0x00200 | If an attack using this hitscan type kills a Thing, it will never trigger the `Exploding frame` even if it reduces its health below the thing's `Gib health` property.   |
| `BLOCKMAPFIX`        | 0x00400 | Hitscan checks of this hitscan type will use a fixed version of the blockmap behavior, allowing them to much more reliably strike large objects.                         |

## Format and example

Hitscantypes are stored in the new `[HITSCAN]` block of a `DEHACKED` lump/file.

The below example is the internal definition to be used for the default hitscan type behavior, which represents all vanilla bullet or player melee attacks.

```
[HITSCAN]

Hitscantype 0 [Default]
Default puff = 38
Damage type = 0
Trail puff = -1
Trail interval = 0
Penetration = 0
Bits = 0
```
