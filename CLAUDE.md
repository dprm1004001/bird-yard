# 베리 마당 (bird-yard): project notes for Claude

This is a bird-collecting web game. Start in a blank white space with a single berry tree.
Shaking a food source drops food, birds come to eat it, and they leave feathers, which are the currency.
Feathers open places and buy items, and the player fills the bird encyclopedia (도감).
The user directs the design in Korean. Reply in Korean.

## Working across two computers

The user alternates between two machines.
- At the start of a session, run `git pull`.
- After any finished change, commit and run `git push` to `main`.
- Never leave work only on one machine.

## Files and deploy

- `index.html` is the whole game. It is one file: Three.js r128 from cdnjs plus inline CSS and JS. There is no build step.
- Deploy: GitHub Pages from `main` / root, at https://dprm1004001.github.io/bird-yard/. Every push redeploys it.
- To test locally, run `python3 -m http.server 8000` and open http://localhost:8000.
- `berry-yard.html` and `low-poly-birds.html` are gitignored old working copies. Don't edit them.
- Progress is saved in localStorage under the key `berry-yard-v1`, per browser.
- Admin mode: 업적 sheet → 관리자 모드 → code `4575`. It gives 99999 feathers and opens every place and achievement. It does not fill the encyclopedia.

## Style rules the user has set

- Keep it **low-poly and "하찮은"**: as few polygons as possible, flat-shaded, colour only, no textures.
- **Birds share geometry.** There are two templates (`TPL.small` round, `TPL.large` tall) built once by `makeTemplate`. A species is a `SPECIES` entry: a 9-colour palette plus part variants and transforms. Add new birds as palette and parameter entries, not new meshes.
  - Large-template variants:
    - head: `point` or `round`
    - beak: `cone`, `wedge`, `flat`, `needle`, `dagger` or `hook`
    - feet: `toes` or `web`
    - extras: `neckLen`, `ears`, `plume`, `bars`, `stripes`, `headScale`, `eyeScale`, `tailLen`, `beakTip`, `capStart`
  - `rarity` makes a species rarer. `SIZE[id]` shrinks raptors and owls further.
- The global bird size is `BIRD_SCALE = .3`. The user shrank it twice, so keep birds small. `BK` scales walking and reach distances to match.
- **No chirp bubbles.** The 구구/찌찌 text was removed on request. Tapping a bird only makes it flap.
- Poop shows as soft grey overlapping ellipses, like rain spots. Dirt slows bird visits. "똥 치우기" and the cleaner plover clear it.
- Bottom HUD order, top to bottom: 똥 치우기 → ◀ ▶ (switch place) → the 장소/상점/도감/업적 menu, which stays on one row on phones.
- Drag that starts on a food source shakes it. Drag that starts on a feather sweeps feathers. Any other drag rotates the view.

## Places and per-place decor

Places, in order: `yard` 베리 마당, `pond` 작은 물가, `plaza` 광장 벤치, `river` 맑은 강가, `forest` 어두운 숲, `island` 바닷가 외딴섬.

The island (`buildIsland`):
- It is a round sand island (`ISLAND_R`) with sea all around and one leaning palm tree. Foam rings swell out from the beach. Birds can float on the sea.
- `rt.bound` keeps food and birds on the sand.
- `ENV.island.cam` pulls the camera back while you are there.
- The palm drops whole coconuts that birds can't eat. The player taps a fallen coconut (`crackNut`, picked in screen space by `nutNear`) to split it into two edible halves. Uncracked coconuts hop now and then to invite a tap.
- The feeder there drops ready halves (`feederFood:'half'`).
- Onboarding has two steps: `hint`, then `hint2`, which shows after the first drop until the first crack (`S.tried['island2']`).
- It is the last and most expensive place (250 feathers), after the forest.
- Birds there, including the cleaner, are drawn at `ENV.island.birdScale` (.85). Steps and reach scale with them.
- `tailBand` on a species gives a white tail with a black band mesh wrapped round it. 괭이갈매기 uses it.
- Its birds are 갈매기 `seagull` (removed from the pond), 괭이갈매기 `tailgull`, 바다쇠오리 `murrelet` (small template) and 가마우지 `cormorant`.
- No staged decor or hidden object yet.

Each place grows decor in stages as shop items are placed there. The stage is the count of items in `S.items[pid]`, from 0 to 5.
- yard: green ground, grass, flowers blooming one by one, butterflies from stage 3.
- pond: blue-green grass, a natural stone ring, lily pads, pink rock flowers, lotus at stage 5.
- plaza: colour-only brick floor. Stages add a 2nd lamp, a facing bench, stone planters (short stems, big blooms), back planters with cube hedges, then a clock tower showing the real time. The layout is deliberately symmetric and man-made.
- river and forest: **not decorated yet.** The user will direct each place one at a time, so don't decorate them unasked.

Shop items, one of each per place:
- 횃대 (perch)
- 자동 모이통 (feeder)
- 새집 (birdhouse)
- 깃털 수집기 (feather collector)
- 청소부 이집트물떼새 (cleaner plover, in a blue cap and vest)

Hidden objects (`HIDDEN`):
- Once all five items are placed in a place, its hidden object unlocks. It shows as "???" in the shop, with a toast and a red dot on 상점.
- Only the plaza has one so far: a longboard half-pipe that birds skate on.
- After it is bought, rare odd moments happen in the plaza: a pigeon smoking at the foot of a lamp with the cigarette held in its wing, and a bird couple kissing on the facing bench.
- The other places' hidden objects are still to be directed by the user.

## Behaviour notes

- Birds rest on perches and birdhouses. They also take special rests: floating on water (pond and river), loafing (식빵 굽기) on benches or logs, and in the yard's bowl.
- Solid scenery registers circle footprints with `addObstacle`. Birds, food and poop are pushed out with `pushOut`.
- A bird that is stuck for about 1.2 s hops over the obstacle (`skipTo`). Herons, gulls, raptors and owls hop slowly.
