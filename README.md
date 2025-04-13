# Controller-Footswitch-Arduino

#include <MIDIUSB.h>

// Pinos dos footswitches e LEDs
const int numSwitches = 6;
const int switchPins[numSwitches] = {2, 3, 4, 5, 6, 7};
const int ledPins[numSwitches] = {8, 9, 10, 11, 12, 13};

// Notas MIDI atribuídas aos switches
const byte midiNotes[numSwitches] = {60, 61, 62, 63, 64, 65}; // C4 a F4

bool lastSwitchState[numSwitches] = {false, false, false, false, false, false};

void setup() {
  for (int i = 0; i < numSwitches; i++) {
    pinMode(switchPins[i], INPUT_PULLUP); // Usa pull-up interno
    pinMode(ledPins[i], OUTPUT);
    digitalWrite(ledPins[i], LOW);
  }
}

void loop() {
  for (int i = 0; i < numSwitches; i++) {
    bool currentState = !digitalRead(switchPins[i]); // Ativo baixo

    if (currentState != lastSwitchState[i]) {
      lastSwitchState[i] = currentState;

      if (currentState) {
        noteOn(0, midiNotes[i], 127); // Canal 1, nota, velocidade
        digitalWrite(ledPins[i], HIGH);
      } else {
        noteOff(0, midiNotes[i], 0);
        digitalWrite(ledPins[i], LOW);
      }
      MidiUSB.flush(); // Envia os dados MIDI
    }
  }
}

// Funções auxiliares MIDI
void noteOn(byte channel, byte pitch, byte velocity) {
  midiEventPacket_t noteOn = {0x09, 0x90 | channel, pitch, velocity};
  MidiUSB.sendMIDI(noteOn);
}

void noteOff(byte channel, byte pitch, byte velocity) {
  midiEventPacket_t noteOff = {0x08, 0x80 | channel, pitch, velocity};
  MidiUSB.sendMIDI(noteOff);
}
