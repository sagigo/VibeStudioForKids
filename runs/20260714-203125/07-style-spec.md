# Style spec

## Feel
Fast, exciting space-race adrenaline — not a quiet worksheet. The kid is
flying down a glowing highway making split-second lane decisions, so
everything should feel snappy and alive: glow, motion streaks, punchy
feedback on hits. But it's still a math tool for a kid who might miss and
lose a life, so the tone under the excitement stays encouraging, never
punishing — losing a life is "oops, try the next one," not "you failed."

## Palette
- **Background (deep space)** `#0a0e2a` — near-black navy canvas fill,
  with a faint `#241b4d` nebula glow used sparingly behind the starfield
  for depth (stars themselves: white dots, mixed opacity).
- **Lane dividers / highway glow (primary)** `#00e5ff` neon cyan — the
  signature "glowing lane" look; used for lane lines, car engine trail,
  and the score number in the HUD.
- **Target number on car / correct-match glow** `#ffd400` warm gold —
  reserved for the car's target number and the pulse/glow a tile gets the
  instant it's confirmed correct, so gold always means "this is the
  answer."
- **Correct hit feedback** `#3dff8a` neon green — brief flash/particle
  burst on a correct collection.
- **Wrong hit / hazard feedback** `#ff4d6d` neon red-pink — wrong-answer
  tile outline and the flash used when a life is lost; same red-pink
  tints the lives/heart icons so "life lost" reads instantly.
- **HUD text** `#ffffff` on a semi-opaque dark chip (`#0a0e2a` at ~70%
  opacity) behind it — keeps score/lives legible over a busy moving
  starfield regardless of what's happening on screen behind it.

## Typography & sizing
System font stack only: `-apple-system, "Segoe UI", "Noto Sans Hebrew",
Arial, sans-serif` — standard OS stacks already render Hebrew correctly
via system fallback, no font loading needed. Numerals in exercises and
the target number are always plain Western digits (0-9), never mirrored
or reordered, even inside RTL text.

Base HUD text ~20-24px (score/lives readable at a glance mid-gameplay
without breaking focus from the road). The car's target number and each
tile's exercise text are the largest, boldest elements in the scene
(~28-36px, bold) — they're the thing the kid must read fastest, so they
outrank every other label on screen. Game Over score is the single
biggest number in the app.

## Layout
Full-viewport canvas is the highway: lanes run vertically, tiles scroll
from top toward the car near the bottom, matching a forward-racing feel.
The HUD (score, lives-as-hearts) sits in a slim fixed bar/overlay at the
top of the screen, on a dark translucent chip so it never fully hides the
starfield behind it.

**RTL applies to:** the HUD's reading order and text alignment (score
label and lives arrange right-to-left to match Hebrew reading flow), all
Game Over screen text and its restart button (label, alignment, and its
position relative to the score readout), and any other menu/instructional
text. `dir="rtl"` on these UI chrome elements is correct and expected.

**RTL must NOT apply to:** the lane layout and lane order on the highway,
the car's on-screen position, tile spawn lanes, or the ArrowLeft/ArrowRight
control mapping. Left arrow always moves the car to the visually-left
lane and right arrow to the visually-right lane — these are absolute
physical directions tied to the keyboard, not "start/end" logical
directions, and must never be mirrored just because the surrounding text
is RTL. The canvas/game-world layer should stay `dir="ltr"` (or direction-
neutral) internally even while the HTML chrome around it is RTL.

## Microcopy tone
Hebrew, playful and energetic — a race announcer cheering the kid on, not
a teacher grading them. Correct collection: short, celebratory, one word
or exclamation (e.g. "מעולה!" / "יש!"), no explanation needed since the
gold glow already told them why. Wrong hit: light and forward-looking,
never blaming — something like "אופס, ניסיון הבא!" ("oops, next try!")
rather than naming the mistake. Game Over screen leads with the score as
the hero ("הניקוד שלך: X"), frames the end warmly ("טיסה טובה בחלל!" /
"good flight through space!"), and the restart button reads as an
invitation back into the race ("עוד סיבוב!" / "one more lap!") rather
than a flat "restart."
