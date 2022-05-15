# Notes from Terry P.

## 2022-04-08

- Set up Allotemplate for my project
- Final Project: implement [General Midi](https://www.wikiwand.com/en/General_MIDI) through Allolib.
	- /sidenote/ why is Gunshot a sound in general MIDI???
	- Allows users to give a midi file as an argument when running ./run.sh, which parses and plays that MIDI file
	- Any sort of customization/visualization once in file??
	- Use a combination of FM and subtractive

## 2022-04-14

- Start with a single file of [040_FrereJacquesTwoVoices.cpp](https://github.com/allolib-s21/demo1-pconrad/blob/master/tutorials/allolib-s21/040_FrereJacquesTwoVoices.cpp)
- There are many reasons starting with this file is useful:
	- This comes onboard with FM synthesis implemented
	- This shows an example of how Sequence and Notes objects are structured in order to create polyphony. Note objects are added to Sequence objects
	- There are some visualization features that I can dive into if I want to.
- For today I just wrote my own instance Seqence with some random notes, and made sure it can play.
- I also refactored everything so it's in own separate file, instead of having everything within main.cpp. Now the project is structured like a traditional cpp project.

## 2022-04-19

- [040_FrereJacquesTwoVoices.cpp](https://github.com/allolib-s21/demo1-pconrad/blob/master/tutorials/allolib-s21/040_FrereJacquesTwoVoices.cpp) at the top level plays Sequence objects, which you add Note objects to. Since the melody of Frere Jacques is not too long, the Notes that are added to Sequence objects are hardcoded:

```cpp
Sequence *sequenceFJPhrase3(float offset = 1.0)

{

TimeSignature t;

Sequence *result = new Sequence(t);

result->add(Note(G4 * offset, 0, 0.25, 0.2));

result->add(Note(A4 * offset, 0.5, 0.25, 0.3));

result->add(Note(G4 * offset, 1, 0.25, 0.4));

result->add(Note(F4 * offset, 1.5, 0.25, 0.45));

result->add(Note(E4 * offset, 2, 0.5, 0.5));

result->add(Note(C4 * offset, 3, 0.5, 0.25));

return result;

}
```

- However, my goal is to be able to import any midi file, so hard-coding will not work in the long run. But for now I want to start small and first deal with parsing the actual MIDI files.
- MIDI files are actually encoded in binary, unlike SynthSeqence files which are text/utf-8. So unless I write my own MIDI decoder, I will need to download a tool to help me to translate the MIDI into strings I can parse.
- [midi2asc](http://www.archduke.org/midi/) does exactly that. It is a C program that parses MIDI files into text files, where each line represents a single MIDI message.
- I use python (easier to code in) to write a parser that takes the translated MIDI text files from midi2asc and outputs cpp hardcode that adds notes like the code above. Then I can take that hard code and put it into a custom sequence in Allolib.
- This however, is a lot of steps to translate MIDI into Allolib. We so far have, MIDI file -> MIDI in text representation -> hard-coded cpp file -> copy paste the relevant lines within the cpp file. Ideally we'll want to shorten this process. Evan suggested Bash scripts. Or maybe I write everything in C++?

## 2022-04-20

- Presented in class today
- Professor Conrad suggested that I can directly output the code as an hpp file and include it in the relevant section.
- I also modified the Python parser so it ignores the metadata within the outputted MIDI text file, and only read the notes.

## 2022-04-25

- I went with writing a Bash script. Or rather, I modified the existing `run.sh` file to first run midi2asc.c, then run my python parser, and then finally runs Allolib.
- Instead of outputting hard code cpp, I instead outputted something more similar to SynthSequence files, where each line represents a note in time, along with the relevant parameters about that single note: amplitude, start time, duration, and frequency. The Allolib main.cpp then parses each line adds each note to a custom sequence object.
- Now you can do single voice MIDI translation.

## 2022-04-27

- 404... bricked my computer. Need to get it fixed

## 2022-05-02

- Trying to figure out polyphony, which is actually not too difficult. Sequences are played using the `playSequence` function. And if you call consecutive `playSequence` functions, they will all start playing at the same time. Thus in order to achieve polyphony, I just need to rewrite my python parser so that it recognizes the track number for each note and outputs the track number along with each note. Then assign each track to one sequence, then play all the sequences simultaneously.
- I added a function `createTracks()` within my `main.cpp` that separates each individual track's MIDI information into their own SynthSequence, which are stored in a vector.
- Thus, each individual part can be played by indexing the vector. Next I need to add some more synthesis options. Such as loading in presets.

## 2022-05-06

- Did a lot of research today on what the best way to load in presets is. Professor Conrad introduced Adam Schmieder's [preset collection](https://github.com/allolib-s21/notes-adamschmieder) a while back, and it'd be perfect for me to use.
- The most cpp way I can think of doing it (but I'm not sure if it's the most optimal). Is have presets be represented by a class, and using inheritance to represent preset types for each kind of synthesis.
- Then I create instances of the classes as a preset and use the `extern` keyword so it is accessible to all files.

## 2022-05-10

- Added subtractive synthesis and Adam Schmieder's presets as well.
