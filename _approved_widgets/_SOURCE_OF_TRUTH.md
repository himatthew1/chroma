# CHROMA — Note Asset Source of Truth & Widget→Game Rule

This folder is the **immutable canonical archive** of every note/effect design the user
**explicitly approved** as a widget. The `.html` files are verbatim copies pulled from the
design-session transcript. They are the ONLY allowed source when (re)implementing a note,
pad, or effect in the game. **Never re-create a design from memory. Never "improve" it.**

---

## THE RULE (widget → game), enforced every time

1. **One asset, shared.** In-game and How-to-Play MUST call the **same** draw/update code.
   There is exactly ONE implementation per note type (the `NoteFX` shared module). How-to-Play
   is just that module rendered on the fx canvas. If the user asks to change a note on one
   screen, it changes on BOTH because there is only one asset. Never write a second copy.

2. **Port verbatim from the approved widget.** When implementing/fixing a note, open the
   matching file below and copy its functions character-for-character (palettes, helpers,
   particle system, easing, constants). Adapt ONLY coordinate/scale/timebase plumbing
   (canvas size, note position, song-time vs widget `t%T`). Do not redesign shapes, colors,
   counts, blur, easing, or mechanics.

3. **Mechanic = the widget's mechanic.** e.g. space-hold *charges to full and auto-resolves*
   (it is NOT release-to-judge). Meteor *reverses along its own drawn path*. Match exactly.

4. **Verify against the widget before deploy.** Screenshot the game element and compare to the
   widget file. If it diverges, it's a bug — fix to match, don't ship.

5. **New designs go through a widget first.** Propose via `show_widget`, get explicit approval
   ("좋아/푸쉬해/이대로 좋아"), archive the approved widget here, THEN port. No in-game design
   that wasn't approved as a widget.

---

## SOURCE OF TRUTH per element (file → approval quote)

| Element | Canonical file | User approval (verbatim) |
|---|---|---|
| **Normal (Orb)** note + hit burst | `ASSET_VIEWER_v3__ALL_NOTES.html` → `aNormal` | "노트는 동일하게 오브로" (l.11720) |
| **Hold (Hex)** note: shake/grow/spark → release burst | `ASSET_VIEWER_v3__ALL_NOTES.html` → `aHold`; shape `HOLD_NOTE__5_shapes.html` #3 | "육각형이 나을 것 같아 홀딩 노트말이야" (l.3176); "헥스모양으로...오래 누를수록 떨리면서 커지고 파티클" (l.11720) |
| **Meteor / Drag** note (comet entry, gray path, reverse-consume) | `METEOR_DRAG__v4__APPROVED.html` → `aDrag` | "어두운 패스도 같이 사라지면 돼!" approving v4 (l.12155); entry gray→reverse color+points pop (l.11978) |
| **Space note** (conic aurora ring, spawn small→rush, **ring blasts off-screen on hit**) | `SPACE_NOTE__ringexit__APPROVED.html`; in-game render `SPACE_NOTE__ingame_preview__APPROVED.html`; consolidated `ASSET_VIEWER_v3__ALL_NOTES.html` → `aSpace`/`spRing` | "색상환띠가 그냥 커지면서 화면밖으로 퇴장…파티클처럼" → "좋아 푸쉬해" (l.4778/4787); "좋아 이제 푸쉬해" (l.5713) |
| **Space HOLD** (HEXAGON, charge→100%→auto aurora supernova) | `SPACE_HOLD__v5_clean__APPROVED.html`; hex rim helper `spHex` in `SPACE_HOLD_HEX__viewer.html` | "좋아~! 이대로 좋아 이대로 푸쉬하고" (l.7462). Hex (not redesign): "일반과 마찬가지로 육각형" (l.11822) |
| **Pads A/D/J/L** (ring + bold pixel letter) | `SPACE_PAD_AND_PADS__boldpads.html` → `ringPad`/`aSpacePad` | pad=ring confirmed (l.11720); bold pixel labels (l.11978) |
| **Space pad** (notched space-bar with gaps; pads slightly overlap top; NO antenna, NO inner grid) | `SPACE_PAD_AND_PADS__boldpads.html` → `bar()` | cockpit lens REJECTED → "스페이스바처럼" wider, "안테나 제거 / 모눈 제거 / ADJL 상단에 살짝 걸치게 / 홈 파놓기" (l.11836/11850) |
| **Hit explosions** (burst color+white core / shock ring / radial flash; space=`burstAur` aurora) | `ASSET_VIEWER__HIT_EXPLOSIONS.html` | "각 모든 노트들의 타격시 폭발 이펙트" (l.11775) |
| **Hold explosion SOUND** | variant ④ in `HOLD_EXPLOSION__variant4_APPROVED.html` | "4번이 좋다" (l.3738) |

All-element references (for layout/comparison): `ALLREF__notes_effects.html`, `ALLREF__ingame_vs_widget.html`.

---

## Shared engine the widgets all use (port these once into `NoteFX`)
- Palettes: `COL=["#ff2e97","#22a7ff","#39e36b","#ffd23f"]`, `AUR=[[57,227,200],[111,240,160],[169,139,255],[255,143,208],[92,200,255]]`
- Path helpers: `hexp(c,x,y,r,rot)`, `star4(c,x,y,r,rot)`
- Particles: `step()`, `drawP()` (streak/dot, `lighter`), `drawS()` (shock rings), `burst()`, `burstAur(fast)`, `shock()`, `flashAt()`
- Space: `spConic()`, `spRing()` (circle), `spHex()` (hexagon) — conic rim R*0.2 faint + R*0.09 bright + white ring R*0.06 + inner R*0.04@0.6
- Note draws: `aNormal/aHold/aMeteor(aDrag)/aSpace/aSpaceHold/ringPad/aSpacePad`

## Unification status (shared NoteFX functions — in-game + How-to call identical code)
- ✅ **Space note / hold**: `NoteFX_spaceFrame` + `NoteFX_spaceCharge` (ctx-agnostic). In-game `drawSpaceNote` and How-to both call them. Ring-blast-off-screen despawn (`rs=1+6.8*eo`), v5_clean hex charge. Commit e043583.
- ✅ **Normal (orb) / Hold (hex)**: `NoteFX_noteBody` + `NoteFX_approach`. Both screens call them. Commit 70ff745.
- ✅ **Space pad**: in-game notched space-bar `barPath()` already matches `SPACE_PAD_AND_PADS__boldpads`.
- ✅ **Miss timing**: How-to now misses ~0.37s late (matches in-game `JW.good+LATEGRACE`), only when player hasn't engaged.
- ⏳ **Meteor/drag**: in-game (`drawSpaceNote` sibling, the `n.type==="slide"` block) is the faithful v4 port; How-to drag is a faithful recipe-mirror. NOT yet a single shared function because in-game draws in low-res pcanvas px (fixed 7px points etc.) while How-to draws in full-res fxc (r-relative) — a literal merge needs a unit-scale param + ONE visual-verification pass (blocked while screenshot capture is timing out this session). In-game stays u=1 (byte-identical) when merged, so the merge can't regress gameplay.
