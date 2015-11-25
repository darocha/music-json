# Music JSON proposal

A proposal for a standard format for representing music in JSON, with the aim of
making emerging web apps using the new Web Audio and Web MIDI APIs interoperable.

This document is intended as a discussion starter. Please comment, propose ideas and
make pull requests.


## Example JSON

Here are the first two bars of Dolphin Dance represented in Music JSON:

    {
        "name": "Dolphin Dance",
        "events": [
            [2,   "note", 76, 0.8, 0.5],
            [2.5, "note", 77, 0.6, 0.5],
            [3,   "note", 79, 1, 0.5],
            [3.5, "note", 74, 1, 3.5],
            [10,  "note", 76, 1, 0.5],
            [0, "chord", "C", "∆", 4],
            [4, "chord", "G", "-", 4]
        ],
        "interpretation": {
            "time_signature": "4/4",
            "key": "C",
            "transpose": 0
        }
    }

## sequence

A sequence is an object with the properties <code>name</code> and <code>events</code>,
where <code>events</code> is an array of events.

    [event1, event2, ... ]

<code>interpretation</code> is optional, and is used to give hints to a renderer.

## event

An event is an array describing the time and type and the data needed to
describe the event.

    [time, type, data ...]

An event MUST have a start <code>time</code> and a <code>type</code>.
An event also contains extra data that is dependent on the type.

### time

<code>time</code> – FLOAT, describes a point in time from the start of the sequence.

<code>time</code> values are arbitrary – they describe time in beats, rather than
in absolute time, like seconds. The absolute time an event is played is dependent
upon the rate and the start time of the sequence it inhabits.

### type

<code>type</code> – STRING, the event type. The type determines the structure of the
rest of the data in the event array.

    [time, "note", number, velocity, duration]
    [time, "param", name, value, curve, duration]
    [time, "control", number, value]
    [time, "pitch", semitones]
    [time, "chord", root, mode, duration]
    [time, "sequence", id, address]

#### "note"

    [time, "note", number, velocity, duration]

<code>number</code> – INT [0-127], represents the pitch of a note<br/>
<code>velocity</code> – FLOAT [0-1], represents the force of the note's attack<br/>
<code>duration</code> – FLOAT [0-n], represents the duration at the sequence's current rate

<blockquote>We'd welcome feedback on the merits of using a "note" with a duration over
separate "noteon" and "noteoff" events (as in MIDI) <a href="http://github.com/soundio/music-json/issues">github.com/soundio/music-json/issues</a>.</blockquote> 

#### "param"

    [time, "param", name, value, curve, duration]

<code>name</code> – STRING, the name of the param to control<br/>
<code>value</code> – FLOAT, the new value of the param<br/>
<code>curve</code> – STRING ["step"|"linear"|"exponential"], represents the type of ramp to use to transition to <code>value</code><br/>
<code>duration</code> – NUMBER [seconds], where <code>curve</code> is not <code>"step"</code>, defines the duration of the ramp

#### "control"

Useful for MIDI apps, but it is preferred to use "param" events.

    [time, "control", number, value]

<code>number</code> – INT [0-127], represents the number of the control<br/>
<code>value</code> – FLOAT [0-1], represents the value of the control<br/>

#### "pitch"

    [time, "pitch", semitones]

<code>value</code> – FLOAT [semitones], represents a pitch shift in semitones

#### "chord"

A chord gives information about the current key centre and mode of the music. A chord event could
be used by a music renderer to display chord symbols, or could be interpreted by a music generator
to improvise music.

    [time, "chord", root, mode]

<code>root</code> – STRING ["A"|"Bb"|"B" ... "F#"|"G"|"G#"], represents the root of the chord<br/>
<code>mode</code> – STRING ["∆"|"-" ... TBD], represents the mode of the chord

#### "sequence"

    [time, "sequence", sequence, rate]

<code>sequence</code> – ARRAY|STRING, an array of events or name of a sequence(TBD)<br/>
<code>rate</code> – FLOAT [0-n], the rate at which to play back the sequence relative to the rate of the
current sequence.

Events in the child sequence should be played back on the target(s) of the current sequence.
The sequence event MAY have an optional final parameter, <code>address</code>, that defines
an alternative target to play the child sequence to.

    [time, "sequence", sequence, rate, address]

<code>address</code> – NUMBER|STRING, the id or path of an object to play the sequence to.<br/>

    // Trigger object id 3
    [0.5, "sequence", "groove", 1, 3]

<!--It is proposed that a near-CSS-like syntax be used to select objects in an app:

    // Trigger object id 3
    [0.5, "sequence", "groove", "objects[id=3]"]
    
    // Trigger all plugins of type "sampler"
    [0.5, "sequence", "groove", 1, "objects[type='sampler']"]
-->

## interpretation (object)

The optional interpret object contains meta information not directly needed to render the
music as sound, but required to render music as notation. A good renderer should
be capable of making intelligent guesses as to how to interpret Music JSON as
notation and none of these properties are required.

    {
        "time_signature": "4/4",
        "key": "C",
        "transpose": 0
    }

## Implementations

- <a href="http://github.com/soundio/midi">MIDI</a> Soundio's MIDI library converts MIDI events to Music JSON events with it's <code>normalise</code> method.
- <a href="http://github.com/soundio/sequence">Sequence</a> Soundio's Sequence object consumes and creates Music JSON. There's a demo at <a href="http://sound.io/sequencer/">sound.io/sequencer</a>.
- <a href="http://labs.cruncher.ch/scribe/">Scribe</a> is a music notation
interpreter and SVG renderer that consume (an old version of) Music JSON.

## References

- OSC spec: <a href="http://opensoundcontrol.org/spec-1_0">http://opensseqoundcontrol.org/spec-1_0</a>
- OSC example messages: <a href="http://opensoundcontrol.org/files/OSC-Demo.pdf">http://opensoundcontrol.org/files/OSC-Demo.pdf</a>
- Music XML: <a href="http://www.musicxml.com/for-developers/">http://www.musicxml.com/for-developers/</a>
- VexFlow: <a href="http://www.vexflow.com/">http://www.vexflow.com/</a>

## Contributions

Only a few very brief discussions have been had, which have served to identify a
basic need. People involved: Stephen Band, Stelio Tzonis, Al Johri and Jason Sigal.
