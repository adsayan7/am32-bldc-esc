# Bring-up log

Three problems showed up when I first powered the board. Here they are in order, with what I saw, what I thought was wrong, and what fixed it.

## 1. The 12 V supply for the gate driver was noisy

**What I saw:** On the first power-up I used a bench supply limited to 1 A, with no motor attached. The 3.3 V rail looked clean. The 12 V rail that powers the gate driver had a ripple on it that lined up with the PWM frequency.

**Why:** The gate driver pulls a quick burst of current every time it switches a transistor. The capacitors near it weren't close enough, or big enough, to supply those bursts, so the voltage wobbled each time. The buck converter making the 12 V was stable; the problem was the last few millimetres before the driver.

**Fix:** I added a larger capacitor right next to the driver's supply pin. The ripple dropped to a few tens of millivolts.

**Lesson:** Capacitors do their job best when they sit right next to the thing that needs the current.

## 2. The motor spun up, then lost sync around 30% throttle

**What I saw:** With the motor connected it started fine, but at about 30% throttle it stuttered and lost sync.

**Background:** This controller has no position sensors in the motor. It works out where the rotor is by watching the voltage on the motor wire that isn't being driven at that moment, and it times each step off that signal.

**What I suspected:** That the signal was getting corrupted. I probed the motor wire and the sensing input at the same time. After every switching event there was a burst of very fast ringing, like a short buzz. It leaked into the sensing circuit and looked like false position signals. The controller's timing drifted until the motor fell out of sync.

## 3. The ringing was also a voltage safety margin problem

**What I measured:** I looked at the voltage spike at the switching node directly. This board runs on a 25 V pack with 40 V transistors, and the ringing was using up a large part of the margin. So it was a reliability risk as well as a signal problem.

**Cause:** Switching edges that are very fast, together with small amounts of stray inductance in the loop between the capacitors and the transistors, make the voltage ring.

**Fix:**
- I raised the gate resistors from 10 ohm to 20 ohm. This slows the turn-on edge, and the overshoot dropped substantially.
- I adjusted the sensing filtering and checked the timing of when the controller samples, so it avoids the moment of switching.

After that, the motor held sync across the full throttle range.

**The trade-off:** Slower edges waste more energy as heat in the transistors. So I checked temperatures under load to make sure I hadn't just swapped a noise problem for a heat problem. Results are in the main README.

---

## If someone asks me to explain it out loud (30 seconds)

"My motor lost sync at around 30% throttle. The controller finds the rotor position by watching a voltage on the unused motor wire, so I suspected that signal. When I looked at it on the scope, there was fast ringing after every switch that looked like false position signals. The same ringing was also pushing the switch-node voltage close to the transistors' 40 volt limit, so I slowed the switching edges with bigger gate resistors. That fixed the ringing and the sync problem. The cost is a bit more heat, so I checked temperatures under load."

## Questions I should be ready for

**Why does a bigger gate resistor help?** It slows how quickly the transistor gate charges, so the transistor turns on more gradually and the voltage doesn't snap and ring as hard.

**What's the downside?** The transistor spends longer partly on, so it wastes more energy as heat.

**What's the root cause, really?** A fast edge plus stray inductance in the current loop. A better layout with a tighter loop would reduce it at the source, and I'd try that first in Rev 2.

**What else could you do about it?** A snubber (a small resistor-capacitor network across the switch), more local capacitors on the power rail, or a filter on the sensing input.

**How did you know it was the ringing and not something else?** I probed the motor wire and the sensing input at the same time and saw the burst line up with each switching event, then confirmed the motor held sync after slowing the edges.

**What would you change in Rev 2?** Tighter power loop layout, space for a snubber, a proper capacitor at the driver supply from the start (instead of adding it afterwards), and removing the redundant bootstrap diodes.
