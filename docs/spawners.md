# MBF2y Random Spawner Definition v0.2.3

Random spawners are tables of RNG values and corresponding thing types. They can be used in conjunction with the `A_DropItem` codepointer, the `Dropped spawner` thing field, or the `A_SpawnFlyEx` codepointer to randomize what will spawn.

(To be determined; format is up for discussion)

## Random Spawner Records

Each spawner type may have up to 255 records defined (though this is considered, in technical terms, overkill).

| Property    | Data type                     | Description                                                                                                 |
|-------------|-------------------------------|-------------------------------------------------------------------------------------------------------------|
| `threshold` | `uint` between 0 and 255      | RNG threshold; if the result of a `P_Random` check is this value or lower, then this row is what is spawned |
| `type`      | `int` representing a thing ID | Thing type to spawn; if -1, no thing is spawned instead                                                     |

## Format and example

Random spawners are stored in the new `[SPAWNERS]` block of a `DEHACKED` lump/file.

The below example is the internal definition to be used for the default random spawner, which is the default Icon of Sin spawner.

```
[SPAWNERS]

Spawner 0 [Icon of Sin Spawner]
51, 11 // Imp
91, 12 // Demon
121, 13 // Spectre
131, 22 // Pain Elemental
161, 14 // Cacodemon
163, 3 // Arch-vile
173, 5 // Revenant
193, 20 // Arachnotron
223, 8 // Mancubus
247, 17 // Hell knight
255, 15 // Baron of Hell
```

## Notes:

* This is still a very rough draft. Format is not intended to be even close to finalized yet.