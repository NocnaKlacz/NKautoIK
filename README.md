# NK Auto IK

Blender add-on that builds working IK controls for the legs and arms of a humanoid armature in one click.
It works out which way each knee and elbow should bend, so you no longer get reversed knees or arms that
fold the wrong way.

Tested with Blender 4.5.5.

## Features

- **Automatic limb detection** - finds the upper bone, lower bone and hand/foot bone of every arm and leg,
  using bone names (Mixamo, Rigify-style `.L/.R`, `lShin`, `Thigh_L`, ...) and falling back to geometry
  (height, direction, bone lengths) for unnamed bones. Fingers, clavicles and the spine are ignored.
- **Manual bone picking** - if detection gets it wrong, list the bones yourself (see below).
- **Correct bend direction** - the front of the character is detected from the feet, so knees point forward and
  elbows point backward regardless of the armature's orientation or object rotation.
- **Calibrated pole angle** - instead of using a formula, the add-on probes Blender's IK solver and picks the
  pole angle that actually produces the right bend direction, whatever the bone roll is.
- **Joint limits** - the knee/elbow is locked as a hinge that bends in one direction only.
- **Pre-bend** - Blender's IK cannot bend a perfectly straight chain, so a slight pose-level bend is added as
  the solver's starting point. The rest pose is not modified.
- **Self-verification** - every limb is tested after setup; the panel reports the pole angle, the bend direction
  error and the deviation from the rest pose, and warns if something looks wrong.
- **Extras** - IK/FK switch (driver-based `IK` property on the control), control shapes, a bone collection
  named "IK Controls", L/R colors, optional parenting to a root bone, optional IK stretch.

## What it creates

For every limb:

| Item | Description |
| --- | --- |
| `IK_<end bone>` | IK control placed at the hand/foot, with the same orientation as that bone |
| `POLE_<lower bone>` | Pole target placed in front of the knee / behind the elbow |
| IK constraint | On the lower bone (shin / forearm), chain length 2, with the pole target and calibrated pole angle |
| Copy Rotation | On the hand/foot bone, so it follows the IK control's rotation |
| IK limits | Hinge limits on the lower bone (optional) |

All generated bones and constraints are tagged, so they can be removed cleanly.


## Usage

1. Select the armature (Object or Pose mode).
2. Optional: click **Detect Limbs** to preview what will be used. Nothing is changed.
3. Click **Create IK**.
4. Switch to Pose mode and move the `IK_` controls. Use the `POLE_` bones to rotate the knees/elbows.

Running **Create IK** again first removes the previously generated IK, so you can safely tweak the options
and re-run. **Remove Generated IK** deletes everything the add-on created.

### Picking bones manually

Enable **Pick Bones Manually**. Automatic detection is then skipped and only the listed limbs are used.

- **Add Limb** - select 2-3 bones forming a parent > child chain (e.g. thigh, shin, foot) and click it;
  Upper / Lower / End, type and side are filled in for you. With nothing selected it adds an empty entry.
- **Fill From Detection** - replaces the list with the automatically detected limbs so you can correct just
  the wrong one.
- For each entry choose **Leg/Arm**, **Left/Right**, and the **Upper**, **Lower** and **End** bones.
  End is optional. The bones must be direct parent > child (`Lower` is a child of `Upper`, `End` a child of
  `Lower`).

## Options

| Option | Description |
| --- | --- |
| Arms / Legs | Which limbs to build (automatic mode) |
| Character Front | `Auto` (detected from feet / names) or a fixed world axis. Set this if the bends come out reversed |
| Pole Distance | Distance of the pole target from the joint, as a multiple of the limb length |
| Joint Limits | Lock knees/elbows as a one-direction hinge |
| Minimum Bend (deg) | Pre-bend for straight limbs; without it Blender's IK will not bend them |
| IK Stretch | IK stretch amount (0 = off) |
| IK/FK Switch | Adds an `IK` property (1 = IK, 0 = FK) on each control, driving the constraints |
| Control Shapes | Custom wire shapes for the controls |
| Control Parent | `Auto` parents controls to a root bone if the rig has one, otherwise no parent |

## Notes and limitations

- **Pre-bend lives in the pose.** If you clear the pose rotation of the knees/elbows (`Alt+R`) and a limb stops
  bending, click **Restore Pre-bend**.
- If the armature's rest pose bends a knee/elbow the wrong way, the IK corrects it, so the joint may shift by a
  few centimeters relative to the rest pose.
- Designed for humanoids with two arms and two legs. Other creatures can be handled with manual bone picking.
- Only parent > child chains of 2-3 bones are supported (upper, lower, optional end). Twist bones should not be
  placed between them.

## Testing

`test_nk_auto_ik.py` builds several synthetic armatures (T-pose, A-pose, random bone rolls, rotated object,
flipped front, reversed rest bend), runs the add-on and checks that knees point forward, elbows point backward
and the targets are reached:

```
blender -b --factory-startup --python test_nk_auto_ik.py
```

## License

GPL-3.0-or-later.
