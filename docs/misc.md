# MBF25 MISC Enhancements v0.2.3

The `[MISC]` block of `DEHACKED` lumps has been extended with the following keys. Fields not provided within the block will not be changed.

## Max Green Armor
* Add `Max Green Armor = A` in the `[MISC]` block.
* `A` is a nonnegative integer.
* This only applies to the "armor repair" pickup item type (see [things.md](.\things.md)).
* Defaults to 100.