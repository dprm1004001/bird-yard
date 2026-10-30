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

- `index.html` is the whole game, plus sound files in `sounds/`. It is one file: Three.js r128 from cdnjs plus inline CSS and JS. There is no build step.
- Deploy: GitHub Pages from `main` / root, at https://dprm1004001.github.io/bird-yard/. Every push redeploys it.
- To test locally, run `python3 -m http.server 8000` and open http://localhost:8000.
- `berry-yard.html` and `low-poly-birds.html` are gitignored old working copies. Don't edit them.
- Progress is saved in localStorage under the key `berry-yard-v1`, per browser.
- Title screen (`#title`): "Bird Yard" fades in over the white void, then a button appears.
  - The button reads 시작하기 on a first visit and 이어하기 once `berry-yard-visited` or a save exists.
  - Returning players also get a grey underlined "처음부터 다시하기" link. It opens a reddish warning dialog, and confirming runs `resetAll()`.
  - Easter egg (`titleFx`): tapping the empty title screen drops a grain on a small 3D stage behind the logo, and a bird flies in to eat it. The view is distant on purpose: the camera is far back, grain is drawn at .6 and birds at .75. At most 10 birds (`MAX_BIRDS`) and 12 grains (`MAX_SEEDS`) are on screen; extra taps are ignored.
    - First-time players only see 울새 and 까치. Returning players get birds from their encyclopedia (`S.seen`).
    - The main scene isn't rendered while the title is up.
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
- Changing place (◀ ▶, 이동하기, or opening a new place) goes through `switchPlace`. It is a crossfade, as the user asked: a still of the old place (`snapshot`, background included) is laid over the screen in `#veil` (a canvas under the HUD). The place switches underneath at once, and the still fades out over 0.15 s.
- Drag that starts on a food source shakes it. Drag that starts on a feather sweeps feathers. Any other drag rotates the view.
- Photo mode: the round camera button under the feather count (`photoBtn`) opens it (`enterPhoto`). The HUD hides and the game keeps running.
  - One finger or left drag turns (`camA`) and tilts (`photo.pitch`).
  - Wheel or pinch zooms (`photo.dist`). A held-wheel (middle) drag or a two-finger drag slides the view (`photoPan`, `photo.pan`).
  - `placeCamera` positions the camera in both modes.
  - The shutter (`takePhoto`) flashes, draws the scene over its background (the forest's dusk gradient is redrawn from `DUSK_STOPS`), saves `bird-yard-YYYYMMDD-HHMMSS.png`, and shows a thumbnail. In the claude.ai artifact it saves through the `downloads` capability (`window.claude.use("downloads")`, so the artifact is published with `capabilities: {downloads: true}`); elsewhere it uses a plain download link.
  - × or Escape leaves.
- Credits: a small round “i” button under the place name and stats (`infoBtn`) opens a card (`#infoBack`, `openInfo`/`closeInfo`). The card is 70% white over a dimmed screen and slides in. Tapping outside it or pressing Escape slides it away. It lists the user's Pixabay sound credits and a YouTube music link exactly as given. Add new credits there when new sounds come in.
- Sound toggle: the speaker button under the camera (`soundBtn`, `setMuted`) turns every sound off and on: the pumpkin knock, the ambient beds, and the siren-show music. While muted, the show music still plays silently to keep the beat clock. The choice is kept per device in localStorage `berry-yard-muted`.

## Places and per-place decor

Places, in order: `yard` 베리 마당, `pond` 작은 연못 (renamed from 작은 물가 at the user's request), `plaza` 광장 벤치, `river` 맑은 강가, `forest` 어두운 숲, `island` 바닷가 외딴섬.

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
- Decor (`islandDecor`):
  - 1+: a lime-green wash in the middle of the sand. The island's light bleaches pale colours, so mix colours deep there.
  - 2+: rocks bobbing in the sea.
  - 3+: scallop and cone shells.
  - 4+: plumeria thickets: tiny bush blobs, smaller than the yard's bush, with plumeria peeking out between them. There are 6 (the user cut 20% from 8), and birds walk straight through them: they were in the way.
- `HIDDEN_ZONE.island` is the front of the island, facing the starting view. It is kept empty of decor and items for the island's future hidden object. The feeder and cleaner spots were moved out of it.
- Its hidden object is the beach hut set (see Hidden objects).
- Flamingo (`flamingo`) is the biggest bird: `SIZE` 1.2 makes up for the island's .85 scale. Its bill has a hard black tip (`tipFrom`). It idles on one leg (`oneLeg`), and after eating it usually wades to `rt.wadeSpots`, three spots in the surf off the beach. It stands there with one leg folded up (rest kind `wade`).
- Pelican (`pelican`) uses the `pouch` beak part with a `pouchColor` throat pouch.
- Ambient sound: while you're on the island, `sounds/sea-seagull.mp3` (the user's waves-and-gulls file; it replaced the first sea track) loops at volume .15 (`AMBIENT`, `updateAmbient`; the user turned the beds down twice, .35 → .22 → .15; the plaza's quiet recording is at .3). It fades in and out over about a second when you arrive or leave. Nothing plays before the first tap (`soundOK`); that first tap also unlocks the audio for iPhones. 베리 마당 (yard), the first place you land in, has `sounds/yard-spring-noon.mp3` (the user's spring-noon forest wind). 광장 벤치 (plaza) has `sounds/plaza-town-square.mp3` (town-square ambience). 어두운 숲 (forest) has `sounds/forest-night-crickets.mp3` (night-field crickets); it fades out while the pumpkin siren show plays. 작은 연못 (pond) has `sounds/pond-spring.mp3` (the user's file, 107 s) the same way, also at .15. Per-file levels the user set by ear: sea .1, river stream .04 (a very loud recording), plaza .3. 맑은 강가 (river) plays two beds together, `sounds/river-forest-stream.mp3` and `sounds/river-bird-chirp.mp3`: an `AMBIENT` entry can be a list (`BEDS` flattens them). Other places can get a bed by adding an `AMBIENT` entry. Over the forest bed, an owl (`sounds/owl-hoot.mp3`, 3.6 s) hoots very softly (.055) after gaps of 8, 5, 13 s in a loop (`CALLS`, `updateCalls`). The cycle restarts from 8 s each time you arrive, and it pauses while muted or during the siren show.

Each place grows decor in stages as shop items are placed there. The stage is the count of items in `S.items[pid]`, from 0 to 5.
- yard: green ground (the same fresh grass greens as the river, `#78c266`/`#69b65c`/`#8bcf76`, at the user's request), grass, flowers blooming one by one, butterflies from stage 3.
- pond: the river's grass-green ground with blue-green tufts, a natural stone ring, lily pads, pink rock flowers, lotus at stage 5.
- plaza: colour-only brick floor. Stages add a 2nd lamp, a facing bench, stone planters (short stems, big blooms), back planters with cube hedges, then a clock tower showing the real time. The layout is deliberately symmetric and man-made.
- river (`riverDecor`) is a cool valley.
  - The river plane keeps only 6% of its height jitter and sits at y .01, so no part dips under the grass disk (grass used to show through as patches).
  - 1+: green ground and grass on both banks.
  - 2+: big boulders rise on the far bank, in two stacks with a gap.
  - 3+: a small waterfall pours through the gap into a pool, with foam at its foot.
  - 4+: moss caps on the rocks, and ferns.
  - 5: a rainbow in the spray.
- forest sky and light: dusk. The renderer has `alpha:true`; places with `ENV[pid].dusk` clear to transparent, and a CSS gradient on `body[data-place="forest"]` shows behind the canvas. It is muted dark purple with a faint orange glow placed behind the treeline (upper middle of the screen).
  - The hemisphere light's sky colour is lavender (`ENV.forest.sky`) and the sun is warm orange (`sunColor`). The floor fades from mossy green to dusk purple at the rim.
- forest (`forestDecor`):
  - 1+: dim grass tufts.
  - 2+: mushroom clusters at the stump, the log and the pines. Red caps have white dots. There are 6 more per item.
  - 3+: fireflies, 5 more per item. Each is a yellow-green blinking light, a little smaller than a feather, that drifts about.

Shop items, one of each per place:
- 횃대 (perch)
- 자동 모이통 (feeder)
- 새집 (birdhouse)
- 깃털 수집기 (feather collector)
- 청소부 이집트물떼새 (cleaner plover, in a blue cap and vest)

Hidden objects (`HIDDEN`):
- Once all five items are placed in a place, its hidden object unlocks. It shows as "???" in the shop, with a toast and a red dot on 상점.
- 베리 마당 (`yard`) has a small stump (`STUMP`, `placeStump`) in the open ground in front of the berry tree.
  - Its top is a clean flat cut: an open bark tube plus a ring face aligned to it (`thetaStart -PI/2`), in pale-yellow growth rings.
  - The user wants plain resting to be rare there; lying down, dancing and guitar should dominate.
    - Fed birds hop onto it often (`stumpSpot`, 55%).
    - 80% of them doze (`B.sleeping`). Half of those flop onto their side (`B.sleepSide`); the rest tuck their head back.
    - Awake birds on it break into dances now and then.
    - Every 10–18 s a bird may fly in just to dance on it (`danceOnArrival`, 80%).
  - The stump has a single spot, in the middle: only one bird at a time uses it, as the user asked.
  - Hidden motion: a bird stands on the stump strumming a tiny guitar with its wing (`startGuitarist`, `makeGuitar`).
    - It plays for 20 s, with a 55% chance every 10 s. A bird just resting on the stump flies off to make way.
    - While it plays, `sounds/guitar.mp3` (the user's file) plays softly (`GUITAR_VOL` .05; the user halved it from .1). It fades in over 1 s, fades out over the last 2 s, and stops if you leave.
- 작은 연못 (`pond`) has a picnic mat (`PICNIC`, `placePicnic`) in the grass in front of the bread basket. It's pale red and white checks, with a closed little basket and a plate holding one hotdog.
  - Birds sometimes peck at the hotdog (rest kind `hotdog`). Others sit side by side on the mat (rest kind `picnic`, 3 seats) holding a can of beer in a wing (`makeCan`).
  - Each sip (`picnicPose`, lifting the wing to the beak) reddens the face one step via an emissive head material (`setDrunk`, `drunkMats`, levels 1–3).
  - At level 3 the bird gets up (`getUp`) and staggers about, rolling and drifting.
  - `MAX_DRUNK` (2) caps fully drunk birds; a drinker past the cap stops at level 2.
  - When a drunk's wander ends, half the time it totters to the pond edge (`goDrinkWater` → `sipwater`) and sobers up. Otherwise it leaves on a weaving, rolling flight.
  - `B.postPose` runs after `applyPose` each frame, for poses (like sipping) that override the wings.
- The plaza has a longboard half-pipe that birds skate on.
- After it is bought, odd moments happen in the plaza:
  - A pigeon smokes at the foot of a lamp, holding the cigarette in its wing. This is rare: 5% every 10 s.
  - A couple (`startCouple`) sits on the left half of the snack bench (`KISS_SEAT`), where the user marked. They lean in for kiss after kiss, in pairs, with hearts each time. They hold that half's two loaf spots while there. The user wanted this often: every 8–14 s, a 55% chance.
- 맑은 강가 (`river`) has a little hot spring (`SPRING`, `placeSpring`) in the grass at the front left, kept clear of river decor.
  - It is a milky sky-blue pool ringed with dark rocks, with one rock in the water and a short dark wooden fence round the back. The user's reference was a Japanese outdoor onsen.
  - Steam: 14 white puffs keep welling up out of the water (`updateSteam`). Tapping the pool splashes, but no fish leaps there.
  - Fed birds often go to soak (`bathSpot`, 50%; rest kind `bath`, 4 spots, sit `float`).
    - About half lay a small white towel on the head (`makeTowel`) and sit still. The rest splash about, flapping and throwing droplets.
    - Soaking raises `B.heat`, which turns the whole bird redder in 8 emissive steps (`setHeat`, `heatMats`). Towel birds heat faster.
  - Some bathers stay until they're boiled red (`B.soakLong`). At full heat they scramble out (`hotExit`) and always end up floating on the river to cool (`coolSpot`: a free water spot, or a fresh patch of river). The user rejected drinking at the bank.
    - Half fly straight onto the river, fast.
    - The rest dash (`B.rush`) to the bank (`tocool`), then hop in and float.
    - The others get out while only a little pink.
  - Out of the spring, heat fades slowly back to the bird's own colours (about 16 s floating on the river, 22 s elsewhere). A hot bird puffs steam.
- 어두운 숲 (`forest`) has a giant carved pumpkin (`PUMPKIN`, `placePumpkin`) at the back right. The user had the two pines there removed as if they never existed.
  - It is a ribbed orange pumpkin with a stem and leaf. Its carved face (triangle eyes, nose, jagged grin) is flat glowing shapes laid onto the front (`pumpkinGlow`), turned to face the usual view.
  - A warm orange `PointLight` in front spills light onto the floor.
  - The model is built at its own size (`r`, `h`) inside an inner group scaled by `PUMPKIN.s` (2.15); the user asked for it this big.
  - Bird behaviour around it is still to be directed.
  - Tapping it (a plain tap, `tapPumpkin`) plays `sounds/wood-tap.wav` (the user's file) and squashes the pumpkin in a little wobble. Each tap clones the `Audio` so quick taps overlap.
  - Siren show (`startShow`, `updateShow`, `endShow`, timings in `SHOW`): 10+ taps within 3 s set it off (then a 10 s cooldown, at the user's request).
    - Prep, 1.6 s: the glow turns red, the stem and leaf (`H.stem`) sink into the pumpkin, and a twin-horn siren (`makeSiren`: mint box, red flared horns) rises onto the top. Every grounded bird freezes in place (`rt.show` check in `updateBird`).
    - Then `sounds/yararara.mp3` plays (the user's file). 0–3.95 s is the siren: it spins faster and faster, with the glow pulsing red.
    - 3.95–11.5 s is a 125 BPM club beat (`SHOW.beat` .48 s, measured from the file). The glow and the pumpkin's light change colour every beat (`CLUB_COLORS`).
      - Everyone dances via `startDance`, with `danceSync` driving one body circle per beat from the music's clock (`showClock`). Birds that land mid-club join in.
    - Then the siren sinks, the stem returns and the glow goes back to yellow. Leaving the place ends the show.
  - Buying it brings the bat (`bat`, 박쥐), which isn't a bird. It's a small-template species with `ears` and `batWing` (a scalloped membrane part, `P.batWing`).
    - `needsHidden:'forest'` keeps it out of `poolFor` until the pumpkin is placed.
    - `secret:true` makes its encyclopedia card "???" / "비밀 도감이에요." with no portrait until it's met. It's also left out of place lists and the place-open toast.
    - Once met, its card shows its `note`: "너는... 새가 아니지 않니?" (the user's line).
- 바닷가 외딴섬 (`island`) has a beach set (`HUT`, `placeHut`) in `HIDDEN_ZONE.island`, all turned to face the usual view. The user's reference was a bamboo beach gazebo.
  - The hut has bamboo posts, a split-bamboo floor, a slatted backrest, and a deep thatched roof with a straw fringe and a tuft. Colours are mixed deep because the island light bleaches them.
  - The roof is tall (`PH` .82) so a flamingo fits inside.
  - Two white sunbeds stand side by side to its right. They are objects only for now.
  - The hut doubles as a wardrobe. A fed bird with no outfit often (45%) walks to its door (`tohut`), hops in, twirls about a second, and hops back out wearing one outfit (`dress`, `makeOutfit`). Then it shows off for a while. One bird at a time (`rt.hut.busy`).
  - The four outfits (`OUTFITS`) are built from each bird's template dims, so they fit every species. They were checked on all six island birds.
    - `shades`: dark lenses over the side eyes, with a bridge.
    - `lei`: a ring of mixed flowers round the base of the neck.
    - `crown`: a flower ring on the head.
    - `hula`: a leaf fringe round the hips.

## Achievements and the bird unlock chain

The user wants achievements tied to the encyclopedia: a place never opens with all its birds.
- Each place starts with two fixed base birds (`BASE`): yard 울새·굴뚝새, pond 청둥오리·청둥오리 암컷, plaza 비둘기·참새, river 백할미새·이집트물떼새, forest 어치·오색딱따구리, island 갈매기·괭이갈매기.
- Every other bird is brought by one achievement in `CHAIN`, in this order per place:
  1. The base birds visit N times between them (10 in the yard, 12 elsewhere).
  2. 2 shop items are placed there.
  3. Birds eat 40 food there.
  4. 4 shop items are placed there.
  - The yard has only the first two steps: 멋쟁이새 for visits, 까치 for 2 items. That was the user's own example.
- `poolFor` only spawns base birds, birds whose unlocking achievement is done, and the bat (gated by its hidden object).
- Each place also has a "도감 완성" achievement (`DEX_DONE`, 30 feathers).
- The old yard achievements are kept, but they no longer bring birds:
  - 첫 손님: 5 feathers
  - 열매 비: bush
  - 북적북적: second tree
  - 단골 마당: bowl
- "히든 업적" group: until done, the name, count and reward read ???, but the subtitle always shows the user's sly hint line in quotes (`hint`). Each pays 50 feathers. The old 작은 새 도감 achievement was removed.
  - 공원 버스킹 (`busk`, "떠오르는 신인 뮤지션"), first in the list: tap or photograph the guitar-playing bird on the stump.
  - 월척이오! (`fish`, "이야, 여기 물고기 맛집이네"): tap the water until a fish leaps out.
  - 술이 웬수지 (`drunk`, "아저씨, 거 적당히 마시라니까!"): tap a drunk bird (`drunk >= 2`) or photograph one.
  - 과태료 10만원 (`smoke`, "흡연은 흡연구역에서만!"): tap or photograph the smoking pigeon.
  - 오모나세상에 (`kiss`, "조옿을 때다~"): tap or photograph the kissing couple.
  - 시원~허다 (`spring`, "거짓말! 뜨겁잖아!!"): tap or photograph a bird soaking in the hot spring, or one still red from it.
  - 핼러윈 파티! (`party`, "야라라라 잇포즈츠 윳쿠리"): set off the pumpkin siren show.
  - 알로하오예 (`aloha`, "맛있는~ 포장마차 가득가득~"): tap or photograph a bird wearing a beach-hut outfit.
  - `spotBird` checks one bird for drunk, spring and aloha.
  - Finds are kept in `S.found` via `spot(kind)`. Actors are pickable (`userData.actor`, `A.act` names the moment). `spotInFrame` checks a photo for those heads on screen.
- The achievement sheet is grouped by place.
- A locked dex card shows which achievement brings that bird.
- Old saves catch up quietly at boot (`checkAch(true)`).
- `iga` picks 이/가 for generated texts.

## Hidden motion: the dance

- `startDance(rt, len)` makes every bird in the place dance in sync. That includes birds standing, walking, eating or perched, and an idle cleaner. Flying and floating birds don't join.
- The user's reference was a hand-drawn clip. In it the head is pinned in place, and the whole round body, legs included, is dragged through a big circle.
- The circle goes **sideways**: left, up, right, down, seen from the front. A forward/back circle was tried and rejected.
- The circle's size is `DANCE_RADIUS` (.5) of the template's body radius side to side, and 3/4 of that up and down.
- The user has asked twice for a smaller circle: .85 → .7 → .5. Only the circle shrinks; the body itself never changes size.
- The speed is `DANCE_BEAT` (1.875) circles a second. The user asked for 1.25× the earlier 1.5.
- The head faces straight ahead at its usual angle. Don't bow or tilt it; the user rejected that.
- The head is pinned a little **forward** of its usual place, along the bird's front-back axis only: `DANCE_FWD` (.16) of the template's body height. Its height and angle stay exactly as normal; lowering it was rejected.
- The body never tilts; it only moves.
- `dancePose` moves `hips`, then moves the neck back so the head lands exactly where it was.
- Standing birds' legs stretch and squash so their feet stay on the ground. The down swing is capped at 65% of leg length.
- It was checked on all 26 species: the head moves 0 cm and turns 0°.
- Don't swap this back for a torso tilt or rump wiggle; the user rejected that.
- While `B.dance > 0`, a bird does nothing else. The pose resets cleanly afterwards.
- For now it happens on its own, rarely: about once every three minutes when three or more birds are on the ground. It is meant to be tied to a hidden object later.

## Water taps and the secret achievement

- In the pond and the river, a plain tap (not a drag) on the water (`rt.waterMeshes`) runs `tapWater`. It throws up droplets and a ripple ring (`splash`).
- 30% of the time a little light-navy low-poly fish leaps out in an arc and plops back in (`leapFish`). It counts in `S.fish`.
- Achievement `fish` "첨벙! 물고기" is **secret** (`secret:true`). It shows as ??? with no condition until it's earned. It's earned on the first fish and pays 50 feathers (an `a.feathers` reward).

## Behaviour notes

- Birds rest on perches and birdhouses. They also take special rests: floating on water (pond and river), loafing (식빵 굽기) on benches or logs, and in the yard's bowl.
- Solid scenery registers circle footprints with `addObstacle`. Birds, food and poop are pushed out with `pushOut`.
- A bird that is stuck for about 1.2 s hops over the obstacle (`skipTo`). Herons, gulls, raptors and owls hop slowly.
