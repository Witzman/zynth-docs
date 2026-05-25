# Architecture Decisions

## ADR-001 — USB audio over built-in headphone jack

**Context.** Zynthian Pi 4 has built-in 3.5mm headphone output (bcm2835) and USB audio option.

**Decision.** Use external USB audio device (e.g. Sound Blaster Play! 2) for better quality and lower latency.

**Consequence.** Device must be connected before boot; card index may vary — JACK configured by name (`hw:S2`), not number. udev quirk may be needed for some devices.

**Why card name not number:** USB enumeration order changes between boots depending on plug order.

