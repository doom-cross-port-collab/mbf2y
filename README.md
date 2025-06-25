# Modder's Best Friend aka MBF25 v0.2.1

MBF25 is a continuation of the MBF21 specification. The project has these goals:

* To expand upon the parameterized behaviors established in MBF21 and allow for further customizability of Thing and Weapon behavior;
* To incorporate features and capabilities found in Doom engine games like Heretic and Hexen not currently accessible outside of standards such as `DECORATE` and `ZScript`;
* To provide commonly requested quality of life improvements;
* To follow existing conventions when possible;
* To expand accessibility and capabilities of Doom modding without becoming unrecognizable as Doom;
* To make as many of the additions "opt-in" and non-obtrusive as possible.

### Directory

Because of the amount of content added to this specification, information has been broken up into multiple pages for ease of navigation:

- [Overall specification](./docs/spec.md).
- [Things specification](./docs/things.md).
- [Frames specification](./docs/frames.md).
- [Weapons specification](./docs/weapons.md).
- [Hitscan type specification](./docs/hitscan.md).
- [Random spawners specification](./docs/spawners.md).

### Version History
- v0.2.1
    - Removed linedef flag 11 for now and added `A_ChangeVelocityEx`, `A_SelfRaise`, `A_JumpIfTargetInSightEx`, and `A_JumpIfTracerInSightEx` codepointers.
- v0.2
    - Initial draft spec (v0.1 is the old, defunct prototype).

##### Developers
- Bofu (project lead)
- elf_alchemist (linedefs, documentation, new lumps)

##### Collaborators
- Alaux
- Altazimuth
- Arsinikk
- boris
- Brad Harding
- c_oelckers
- ceski
- dasho
- Edward850
- electricbrass
- fabiangreffrath
- Lobo
- PBeGood
- Ralphis
- Redneckerz
- rfomin
- Xaser
