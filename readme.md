# Modder's Best Friend aka MBF2y v0.2.3

MBF2y is a continuation of the MBF21 specification. The project has these goals:

* To expand upon the parameterized behaviors established in MBF21 and allow for further customizability of Thing and Weapon behavior;
* To incorporate low-hanging features and capabilities found in Doom engine games like Heretic and Hexen not currently accessible outside of standards such as `DECORATE` and `ZScript`;
* To provide commonly requested quality of life improvements;
* To follow existing conventions when possible;
* To expand accessibility and capabilities of Doom modding without becoming unrecognizable as Doom;
* To be relatively simple for ports with existing MBF21 or ID24 support to implement;
* To make as many of the additions "opt-in" and non-obtrusive as possible.

### Directory

Because of the amount of content added to this specification, information has been broken up into multiple pages for ease of navigation:

- [Overall specification](./docs/spec.md).
- [Frames specification](./docs/frames.md).
- [Things specification](./docs/things.md).
- [Frames specification](./docs/frames.md).
- [Weapons specification](./docs/weapons.md).
- [Linedef specification](./docs/linedefs.md).
- [Hitscan type specification](./docs/hitscan.md).
- [New powerup capabilities](./docs/powerups.md).
- [New UMAPINFO extensions](./docs/umapinfo.md).

### Version History
- v0.2.3
    - Removed more ambitious parts of the spec for future consideration.
    - Added `UMAPINFO` extensions.
- v0.2.2
    - Added weapon, frame, and powerups specifications.
    - Added multiple thing and weapon codepointers.
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