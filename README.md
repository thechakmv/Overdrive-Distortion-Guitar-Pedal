# Dual-Stage Overdrive & Distortion Guitar Pedal

I created analog guitar effects pedal with two independent, switchable gain stages featuring deliberately opposite midrange voicings to achieve classic Thrash metal rhythm and lead tones: a soft-clipping overdrive with a **mid-boost** (+6 dB at 800 Hz) and a hard-clipping distortion with a **mid-scoop** (−6 to −10 dB across 400–800 Hz). The Pedal features a single-knob passive tone control, and volume control as well. Each stage has true-bypass switching, so the two can run independently or be cascaded for stacked high-gain tones.

## Highlights

- Designed and simulated in LTSpice; verified Bode plots, transient behavior, and FFT spectrum on hardware
- Two-stage analog signal chain built around TL072 JFET-input op-amps
- True-bypass per stage via 3PDT footswitches; both stages engage simultaneously for cascaded gain
- Passive Twin-T notch filter for the distortion mid-scoop voicing
- Single 9 V DC supply, ~30 mA total current draw
- PCB layout in KiCad with attention to signal integrity, star-ground topology, and inter-stage isolation

## Signal Flow

```
                                                         
 Guitar ─> Input ─┬─> OD Stage ──┬─> Distortion ──┬─> Tone ─> Volume ─> Output ─> Amp
          Buffer  │   (soft clip,│   (hard clip + │  (LPF)    (audio)   Buffer
          (TL072) │   mid-boost) │   Twin-T scoop)│
                  │              │                │
                  ▼              ▼                ▼
                SW1 3PDT        SW2 3PDT       Always
                true bypass     true bypass    in signal path
```

When both footswitches are engaged, the OD output feeds the distortion stage input, producing cascaded high-gain saturation. Each stage can also run alone for cleaner crunch or for chunkier rhythm tones.

## Design Specifications

| Parameter | Value |
|---|---|
| Supply | 9 V DC, center-negative, ~30 mA |
| Frequency response (clean path) | Flat ±0.5 dB across 20 Hz – 20 kHz |
| OD voicing | +6 dB peak at 800 Hz; corners at 720 Hz and ~6 kHz |
| OD gain range | 0 dB to +41 dB |
| OD clipping | Soft, symmetric, ±0.6 V (1N4148 anti-parallel in feedback) |
| Distortion voicing | −6 to −10 dB notch centered at ~700 Hz |
| Distortion gain range | +14 dB to +54 dB |
| Distortion clipping | Hard, symmetric, ±0.6 V (1N4148 to Vref) |

## Design Philosophy

Where most pedals use a single clipping topology, this design uses two intentionally distinct stages whose characters complement each other when cascaded.

The **overdrive stage** is a Tube Screamer-derived non-inverting topology with silicon diodes inside the feedback loop. The feedback RC network forms a band-pass response: lows roll off below 720 Hz and highs roll off above ~6 kHz at maximum gain. Inside the band the gradual diode conduction produces soft clipping that preserves the dry signal underneath the saturation, giving the characteristic transparent overdrive sound.

The **distortion stage** uses an inverting topology with diodes clamping the op-amp output directly to the Vref node. This produces hard, square-wave clipping rich in odd harmonics. The clipped signal then feeds a passive Twin-T notch filter centered near 700 Hz, carving out the boxy midrange while leaving low and high content intact — the classic scooped-mid metal voicing.

The two stages are independently switchable: OD alone for crunch, distortion alone for chunky rhythm work, or both stacked for full-saturation lead tones.

## Repository Structure

```
├── ltspice/
│   ├── od_stage.asc
│   ├── distortion_stage.asc
│   └── Overdrive_Distortion_Schematic.asc              
├── kicad/
│   ├── pedal.kicad_pro             Project file
│   ├── pedal.kicad_sch             Schematic
│   ├── pedal.kicad_pcb             Board layout
│   └── gerbers/                    Fabrication output (zipped Gerbers + drill)
└── images/
    ├── frequency_response.png      Measured Bode plots
    ├── pcb_schematic.png          Spectrum analyzer captures
    └── pcb_3d.png                  Oscilloscope captures
```

## Tools

**Software:** LTSpice, KiCad 7

**Test equipment:** oscilloscope (FFT capable), signal generator, multimeter, soldering iron

## Build Process

1. **Simulate.** Run AC and transient analyses in LTSpice for each stage. Verify gain curves, clipping shape, and Vref stability under load. Use `.step` to sweep gain pot values and see the response variation across the control range.

2. **Breadboard.** Build stage-by-stage with bench power. Validate against simulation results with scope and FFT. This is the stage to tune by ear — swap diodes (Si vs. LED vs. asymmetric), tweak the scoop center frequency, dial in the gain ranges.

3. **PCB.** Re-capture the validated schematic in KiCad, assign through-hole footprints, lay out the board with the panel-component headers anchored to enclosure positions. Generate Gerbers and fabricate via JLCPCB or OSH Park.


**In Progress:**
1. **Assemble.** Populate the PCB; wire off-board panel components (pots, footswitches, jacks); mount in enclosure.

2. **Verify.** Measure assembled pedal's frequency response and FFT; compare against simulation predictions.

## Testing & Verification

The hardware was validated against simulation using:

**Bode plots:** swept sine across 20 Hz – 20 kHz, output captured on oscilloscope, magnitude plotted in dB. Measured curve matches LTSpice AC analysis within ±2 dB across the band.

**FFT analysis:** 1 kHz sine input at each stage's clipping threshold, output FFT'd to verify harmonic content. The OD stage shows soft-clip characteristic harmonic roll-off; the distortion stage shows hard-clip odd-harmonic series with a clear notch at ~700 Hz after the Twin-T.


## Design Issues Encountered (and Fixed)

Three non-obvious problems came up during simulation and breadboarding that are worth documenting:

**Vref instability under heavy distortion.** The hard-clipping diodes were pumping asymmetric current pulses into the unbuffered Vref node, dragging it from 4.5 V down toward 1 V during sustained clipping. Fixed by adding a 1 kΩ series resistor between the distortion op-amp's output and the diode pair, limiting peak diode current and preserving Vref bias integrity.

**High-frequency oscillation at high gain.** The TL072 stages were prone to oscillation when driving capacitive loads (cable runs, coupling caps). Fixed by adding a 470 Ω series isolation resistor between each op-amp output and its capacitive load, with feedback taken before the resistor.

**Switch-engage pop.** Resolved by adding 1 MΩ pull-down resistors on both sides of every coupling cap that interfaces with a footswitch, ensuring no DC charge accumulates on the disconnected side of the cap during bypass.

## Skills Demonstrated

- Analog circuit design with op-amps (inverting, non-inverting, follower topologies)
- Frequency-response shaping via reactive feedback networks
- Soft-clipping vs. hard-clipping topology selection and harmonic analysis
- Single-supply audio biasing: Vref generation, AC coupling, bypass strategy
- Passive filter design (Twin-T notch with derivation of component ratios)
- SPICE simulation: AC sweep, transient analysis, FFT, parametric stepping
- PCB layout for analog audio: ground topology, decoupling, inter-stage isolation
- Hardware validation against simulation with scope and spectrum analyzer

