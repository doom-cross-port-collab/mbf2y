# MBF2y Powerups Definition v0.2.3

**This feature is still being discussed. It does involve a bit of "original behavior" that could be considered contrary to the Doom experience, but if implemented, these will all be locked behind modders actually wanting to use them and implementing them in their maps.**

## New Powerups

The following powerup effects are available for use, though they will have no effect on standard gameplay unless `SPECIAL` items with their relevant pickup item types (see [things.md](.\things.md)) have been defined and placed. No pre-defined pickup items are designed for them, nor is there any expectation that these be accessible to the player without the use of custom `SPECIAL` objects. Ports that define custom objects for specific gameplay modes are welcome to create their own pre-defined pickup things for them, and ports that add non-standard cheat codes are welcome to expose them to cheat codes/console commands/etc., but there is no expectation of this.

The following have been added to the `player->powers` enum:

| Name              | Description                                                                                                                                                                                                                                                                                                                                                                 | Default duration | Palette to apply |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------|------------------|
| `pw_doubledamage` | All damage inflicted directly by the player is multiplied by 2. This does not include barrel explosions but does include explosion damage inflicted by player-fired projectiles, hitscan and melee attacks, and projectile damage.                                                                                                                                          | `(30 * TICRATE)` | TBD              |
| `pw_quaddamage`   | All damage inflicted directly by the player is multiplied by 4. This does not include barrel explosions but does include explosion damage inflicted by player-fired projectiles, hitscan and melee attacks, and projectile damage. Overrides `pw_doubledamage` if both are active.                                                                                          | `(30 * TICRATE)` | TBD              |
| `pw_leechhealth`  | 10% of all damage inflicted directly by the player is returned to the player up to the player's health maximum. This does not include barrel explosions but does include explosion damage inflicted by player-fired projectiles, hitscan and melee attacks, and projectile damage. The 10% is taken after factoring in the effects of `pw_doubledamage` or `pw_quaddamage`. | `(30 * TICRATE)` | TBD              |
| `pw_infiniteammo` | Ammunition is not subtracted from the player's inventory unless the `A_ConsumeAmmoEx` codepointer is used with the `BYPASSINFINITEAMMO` flag.                                                                                                                                                                                                                               | `(30 * TICRATE)` | TBD              |
| `pw_weaponspeed`  | Weapon frames with the `FASTWEAPON` bit will be reduced in duration by half (rounded up) while weapon frames with the `SKIPFASTWEAPON` bit will have their durations set to 0 instead.                                                                                                                                                                                      | `(30 * TICRATE)` | TBD              |


## New hardcoded strings

The following strings can be overridden by `DEHACKED` or `LANGUAGE` lumps, but are provided as defaults:

| Mnemonic           | Default text   |
|--------------------|----------------|
| `MBF2Y_GOTDOUBLE`  | Double damage! |
| `MBF2Y_GOTQUAD`    | Quad damage!   |
| `MBF2Y_GOTLEECH`   | Health leech!  |
| `MBF2Y_GOTINFAMMO` | Endless ammo!  |
| `MBF2Y_FASTWEAPON` | Fast attack!   |
