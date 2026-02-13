# Whoosh z-index: Game Boy passing over the letter

## Problem

When the Game Boy whooshed back from the front view into the box, it moved **behind** the letter/envelope instead of over it. The animation looked wrong because the Game Boy appeared to slide under the letter as it returned to the lower-right.

**Cause:** The Game Boy and letter are in the same “box” layer but have different default z-index values:

- **Letter** (`#letter-inside-box`): `z-[17]` — so the letter is clickable everywhere and appears on top in the box.
- **Game Boy** (`#gameboy-inside-box`): `z-[14]` — so in normal box view the letter is above the Game Boy.

When the user clicked the Game Boy (in front view) to trigger whoosh-back, the code **removed** the class `gameboy-front-view-active` at the **start** of the whoosh-back. That class is what gives the Game Boy a higher z-index (`z-index: 18`). As soon as it was removed, the Game Boy dropped back to `z-[14]`, so it went **below** the letter (`z-[17]`) for the whole whoosh-back animation.

So the bug was: we were clearing the “on top” state too early, before the whoosh-back finished.

## Fix

**Rule:** The Game Boy should keep its “on top” z-index for the **entire** whoosh-back, and only return to normal stacking when it’s back in the box.

1. **Do not remove `gameboy-front-view-active` when whoosh-back starts**  
   In the Game Boy click handler, when the user clicks to whoosh back, we no longer call `gameboyInside.classList.remove('gameboy-front-view-active')`. So the Game Boy keeps `z-index: 18` for the whole animation.

2. **Remove `gameboy-front-view-active` only when whoosh-back finishes**  
   In the `animationend` handler for the Game Boy container whoosh-back, we still do:
   - `gameboyInside.classList.remove('gameboy-container-whoosh-back')`
   - `gameboyInside.classList.add('gameboy-in-box')`
   - **`gameboyInside.classList.remove('gameboy-front-view-active')`**

   So the “on top” state is cleared only after the Game Boy has landed back in the box.

## Result

- **Whoosh up:** Game Boy gets `gameboy-front-view-active` as soon as the whoosh starts, so it passes **over** the letter and other items.
- **Whoosh back:** Game Boy keeps `gameboy-front-view-active` until the back animation ends, so it still passes **over** the letter on the way back into the box.
- **Back in box:** When the animation ends, we remove `gameboy-front-view-active`, so the Game Boy returns to `z-[14]` and the letter stays on top again for normal box view.

## Relevant code

- **CSS:** `#gameboy-inside-box.gameboy-front-view-active { z-index: 18; }`
- **JS:** Add `gameboy-front-view-active` when starting the Game Boy whoosh **up**; remove `gameboy-front-view-active` only in the **animationend** handler for the Game Boy whoosh **back**.
