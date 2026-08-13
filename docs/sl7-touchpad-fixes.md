# Surface Laptop 7 touchpad fixes

This downstream patch set is validated on the Microsoft Surface Laptop 7 13.8"
with Snapdragon X Elite / X1E80100 hardware and touchpad VID:PID `045E:0C77`.

The work is based on commit `3663e96e758145801c3cb1ce7c72f362d0d4a5f5`.

## Fixes

1. Preserve the selected singletouch state across a frame where the selected
   contact is still valid but temporarily reported as unstable. An unstable
   measurement is ignored without being treated as a physical contact lift.
2. Explicitly disable the touch device when the daemon stops, using the
   existing `TouchDevice` release and sync path before the uinput device is
   destroyed.

The detector, stabilizer smoothness, tracking, palm handling, and button
debounce behavior are intentionally unchanged. Smoothness tuning remains future
work and is not part of this patch.

## Physical acceptance

Validated on real hardware:

- pointer: PASS
- physical click: PASS
- drag-and-drop: PASS
- long-held drag: PASS
- direction changes: PASS
- two-finger scroll: PASS
- post-test touchpad click: PASS
- post-test external mouse click: PASS
- clean daemon shutdown: PASS

The repository has no existing automated test target. Direct unit testing of
these paths would require injecting or replacing `UinputDevice`, which is not
justified for this small lifecycle fix; the supported build and static quality
checks remain the regression checks for this downstream patch.
