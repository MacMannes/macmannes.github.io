---
date: "2025-04-06T17:14:18+02:00"
draft: true
title: "Cleaner Arduino Code With Object Calisthenics"
showHero: true
tags:
  - Arduino
  - Object Calisthenics
  - Clean Code
---

### Before

```cpp
switch (commandValue) {
    case 0: {
        Serial.println("Turning off the alarm");
        state = commandValue;
        applyState();
        playSuccessNotes();
        break;
    }
    case 1: {
        Serial.println("Putting the alarm into sleep mode");
        state = commandValue;
        applyState();
        playSuccessNotes();
        break;
    }
    case 2: {
        Serial.println("Putting the alarm in away mode");
        state = commandValue;
        applyState();
        playSuccessNotes();
        break;
    }
    case 8: {
        Serial.println("Putting the alarm on scheduled mode");
        state = commandValue;
        applyState();
        playSuccessNotes();
        break;
    }
    case 99: {
        // Change the PIN code
        int newPinIndex = command.indexOf('*', separatorIndex + 1);
        if (newPinIndex != -1) {
            String newPin = command.substring(newPinIndex + 1);
            pinCode = newPin;
                    Serial.print("PIN code changed to: ");
                    Serial.println(pinCode);

                    // Save the new PIN code to flash memory
                    savePinCodeToFlash(pinCode);
                } else {
                    Serial.println("Invalid command format");
                }

                playSuccessNotes();
                playSuccessNotes();

                break;
            }
            case 1990: {
                Serial.println("Playing Monkey Island Theme :-D");
                playMonkeyIslandTune(BUZZER_PIN);
                break;
            }
            default:
                Serial.println("Invalid command");
                playErrorNotes();
        }
```

### After

```cpp
// Lookup table for command handlers
const std::map<int, std::function<void()>> commandHandlers = {
    { HOME,     []() { changeAlarmState(HOME, "Turning off the alarm"); }                   },
    { AWAY,     []() { changeAlarmState(AWAY, "Putting the alarm in away mode"); }          },
    { SLEEP,    []() { changeAlarmState(SLEEP, "Putting the alarm into sleep mode"); }      },
    { ALERT,    []() { changeAlarmState(ALERT, "Putting the alarm into alert mode"); }      },
    { SCHEDULE, []() { changeAlarmState(SCHEDULE, "Putting the alarm on scheduled mode"); } },
    { 1990,     playMonkeyIslandTheme                                                       }
};

void executeCommand(int commandValue, const String &command) {
    if (commandValue == 99) {
        handlePinChange(command);
        return;
    }

    auto it = commandHandlers.find(commandValue);
    if (it == commandHandlers.end()) {
        invalidCommand();
        return;
    }
    it->second();  // Call the function mapped to the command
}

void changeAlarmState(int newState, const char *message) {
    Serial.println(message);
    state = newState;
    applyState();
    playSuccessNotes();
}

void playMonkeyIslandTheme() {
    Serial.println("Playing Monkey Island Theme :-D");
    playMonkeyIslandTune(BUZZER_PIN);
}

void invalidCommand() {
    Serial.println("Invalid command");
    playErrorNotes();
}
```
