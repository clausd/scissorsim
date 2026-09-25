# scissorsim

A web toy about why a hairdresser's income depends on what length is in fashion.

Open `index.html` in a browser. No build, no dependencies, no server.

## The idea

A friend's hairdresser was gloomy because longer hair was coming into fashion —
his regulars had started coming in less often. At first that seems wrong: hair
grows at the same rate whatever length it is, so the same amount of hair turns
up every month either way.

The catch is that nobody books a haircut by the centimetre. They book when their
hair looks *wrong*, and "wrong" is proportional. On a 1 cm crop, an extra
centimetre destroys the cut. At 15 cm it is invisible, and you only go in once
there are three or four centimetres to lose.

Constant growth, proportional tolerance — so the booking interval scales with
the fashionable length, and the hairdresser's week scales with its inverse.

## The model

Each head gets its own growth rate, its own take on the fashionable length, and
its own fussiness, all drawn around the town-wide sliders.

| | |
|---|---|
| Growth | 0.29 cm/week (±18% per head) |
| Personal length | fashion × 0.7–1.3 — nobody follows it exactly |
| Books a cut when | `length − target > max(tolerance × target, 0.35 cm)` |
| Minimum notice | 0.35 cm — below that nobody can tell, even on a buzz cut |
| Minimum gap | 2 weeks — you cannot book faster than that |
| Maximum gap | 26 weeks — split ends drag everyone in eventually |
| Booking delay | 0–2 weeks between wanting a cut and getting the slot |

The booking delay matters: without it, a swing in fashion sends the whole town
into the chair in the same week. With it, the spike spreads out the way a real
appointment book does.

Move the length slider and watch the chart. Dropping from 20 cm to 2 cm floods
the book; going the other way empties it for months.

## The drawing

Heads are hand-rolled canvas rather than rough.js or a physics engine — the
sketchy look comes from jittered strokes, and the hair from a solver written for
this one job.

A verlet chain with a bend constraint was the obvious approach and it was wrong:
the bend and distance constraints fight each other and the strands buckle into
kinks and spirals. What works instead is an analytic rest shape. Walking out
from the root, each joint turns toward vertical in proportion to how much hair
still hangs below it:

```
turn = min(TURN_MAX, TURN × segments_remaining × strand_limpness)
```

A 1 cm bristle has almost nothing below its root, so it stands straight out of
the scalp. A 20 cm strand has plenty, gives up within a joint or two, and falls.
That single rule produces the whole range, and it cannot buckle. The drawn
points then chase that shape on a damped spring, which is where the sway and the
settle-after-a-snip come from. Strands that would pass through the skull are
projected onto it, so long hair drapes over the crown and falls past the ears.

Segments are a fixed number of pixels, so longer hair simply has more of them.

One ruler governs everything on screen: a skull is 15 cm across and hair is
measured on that same scale, so a strand always reads at its true length
relative to the head. As the fashionable length grows the camera pulls back —
watch the scale bar in the corner.

## Controls

**Shuffle lengths** re-rolls the town: each head gets a new personal length, a
new growth rate, a new fussiness, and a new position in its own cut cycle —
some freshly cut, some already overdue. The same roll runs on load, so you
never start with twenty identical heads. It leaves the week counter and the
chart alone; only the people change.

Space plays and pauses, right arrow steps one week.

A starting state can be handed over in the URL:

```
index.html?length=18&heads=30&fuss=12
```
