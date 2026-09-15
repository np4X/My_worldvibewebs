# draw.io electrical shape reference

`Sidebar-Electrical.js` in this folder is vendored straight from the
[draw.io repo](https://github.com/jgraph/drawio/blob/dev/src/main/webapp/js/diagramly/sidebar/Sidebar-Electrical.js)
(the source that builds the "Electrical" shape palette in the desktop/web
app). It's the ground truth for valid `shape=mxgraph.electrical.*` style
strings — search it by component name before adding a new shape to any
`.drawio` file here.

**Why this exists:** several shape names used in earlier commits didn't
actually exist (`shape=resistor`, `...switches.switch_1`,
`...electrical_symbols.battery_cell_2`, `...electrical_symbols.led_2`,
`...signal_sources.earth_ground`), so the CI renderer silently fell back to
plain boxes instead of erroring. There's no local drawio install to catch
this ahead of time, so double-check names here first.

## Verified style strings in use (`circuit_elements.drawio`, `buck_converter.drawio`)

| Component | Style |
|---|---|
| Resistor | `shape=mxgraph.electrical.resistors.resistor_1` |
| Capacitor | `shape=mxgraph.electrical.capacitors.capacitor_1` |
| Inductor (coil-loop) | `shape=mxgraph.electrical.inductors.inductor` (NOT `inductor_1`, which renders as a flat box) |
| Diode | `shape=mxgraph.electrical.diodes.diode;fillColor=strokeColor;` |
| Battery | `shape=mxgraph.electrical.miscellaneous.monocell_battery;fillColor=strokeColor;strokeWidth=1;` |
| Switch (single throw) | `shape=mxgraph.electrical.electro-mechanical.singleSwitch;aspect=fixed;elSwitchState=on;` |
| Switch (SPDT / two-way) | `shape=mxgraph.electrical.electro-mechanical.twoWaySwitch;aspect=fixed;elSwitchState=2;` |
| Ground | `shape=mxgraph.electrical.signal_sources.signal_ground` |
| DC source | `shape=mxgraph.electrical.signal_sources.dc_source_1` |
| LED | `shape=mxgraph.electrical.opto_electronics.led_2` |

## How to look up a new shape

1. Search `Sidebar-Electrical.js` for the component name (e.g. `grep -i mosfet`).
2. Each entry looks like `this.createVertexTemplateEntry(<prefix> + 'shape_key;', width, height, ...)`.
   The `<prefix>` variable (defined near the top of the file, e.g. `mei`,
   `mere`, `mess`) expands to `pointerEvents=1;verticalLabelPosition=bottom;
   verticalAlign=top;align=center;html=1;shape=mxgraph.electrical.<category>.`
   — concatenate it with the `shape_key` to get the full style string.
3. Some categories (switches, motors) use JS-drawn shapes with camelCase
   keys and extra params like `aspect=fixed;elSwitchState=...` instead of
   the underscore_case static stencil names — copy those styles verbatim
   rather than guessing.
4. To orient a horizontal two-terminal shape (resistor/capacitor/inductor)
   vertically, either add `rotation=90` (rotates the whole shape+label
   together, terminals stay at the natural left/right points) or
   `direction=south`/`north`/`west` (redraws the stencil in that
   orientation instead of applying a transform — the draw.io desktop app
   prefers `direction=` when you rotate a shape by hand, and it keeps
   labels upright).
5. After editing, push and check the Actions tab / pull the auto-rendered
   `.png` — there's no faster local way to verify a shape actually renders.

## House style (observed from hand-edits in the draw.io desktop app)

`buck_converter.drawio` is the base buck circuit and may get edited or
branched into variants later, so match its conventions when generating or
extending circuits in this repo:

- **Label components the way the source notes label them** — e.g. the
  switch is "D" because the PDF names it that; L/C/R and the source were
  left unlabeled (just symbols) because the notes don't caption them
  individually there. Don't blanket-add or blanket-remove labels — follow
  what's actually in the reference material for that diagram.
- **Orient with `direction=south`/`north`/`west`/`east`, not `rotation=90`.**
  That's what the desktop app itself writes when you rotate a shape by
  hand, so using it keeps generated files consistent with manual edits.
- **Prefer the flatter/slimmer stencil variant for inline placement**:
  `mxgraph.electrical.inductors.inductor_5` (h=14, flat coil) over
  `inductor` (h=34+), `resistors.resistor_2` over `resistor_1`. Check the
  sidebar file for size — smaller `height` values fit better on a tight
  rail.
- **Wires can be freeform.** A plain edge with just `sourcePoint`/
  `targetPoint` mxPoints (and an optional `points` array for a single jog)
  is fine and is exactly what manual click-drag wiring produces — edges
  don't need to be bound to `source=`/`target=` cell IDs with
  `entryX`/`exitX` constraints unless you want them to stay attached when
  a component moves.
- Match the palette's exact style baseline when possible:
  `pointerEvents=1;verticalLabelPosition=bottom;shadow=0;dashed=0;
  align=center;html=1;verticalAlign=top;shape=...` (include `shadow=0;
  dashed=0;` even though it's a no-op visually — it's what dragging the
  shape from the sidebar actually produces).
