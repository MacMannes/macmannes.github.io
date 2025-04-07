---
date: "2025-04-07T16:00:18+02:00"
draft: false
title: "Cleaner Arduino Code With Object Calisthenics"
showHero: true
tags:
  - Arduino
  - Object Calisthenics
  - Clean Code
---

The lost couple of months I've been learning how to make cleaner code in a Coding Dojo. At the dojo,
we practiced
[Object Calisthenics](https://bolcom.github.io/student-dojo/legacy-code/DevelopersAnonymous-ObjectCalisthenics.pdf),
a set of programming rules designed to help developers make their code cleaner and more
maintainable. The rules are:

1. Only One Level of Indentation per Method
2. Don’t Use the `else` Keyword
3. Wrap All Primitives and Strings
4. Use Only One Dot per Line
5. Don’t Abbreviate
6. Keep All Entities Small
7. Don’t Use Any Classes with More Than Two Instance Variables
8. Use First-Class Collections
9. Don’t Use Getters and Setters

When I was looking at an Arduino project for an alarm keypad, which I haven't touched in about a
year, I thought it would be great opportunity to practice those rules on my code.

## Before refactoring

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

## After refactoring

A `switch` statement is in fact nothing more than an `if` statement with a lot of `else`s which can
be replaced with a lookup table. By doing this, I could also clean up some duplicated code.

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
```

I introduced the `executeCommand` function to lookup the command and execute it.

```cpp
void executeCommand(int commandValue, const String &command) {
    if (commandValue == 99) {
        handlePinChange(command);
        return;
    }

    auto commandHandler = commandHandlers.find(commandValue);
    if (commandHandler == commandHandlers.end()) {
        invalidCommand();
        return;
    }

    commandHandler->second();  // Call the function mapped to the command
}
```

The other functions are also cleaned up and look like this:

```cpp
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

There's room for improvement, as the `main.c` file remains quite large. In the future, I plan to
refactor the code by introducing an object-oriented approach to encapsulate the code into smaller,
more manageable pieces. For now, however, I am satisfied with the results.

In case you're curious about the playMonkeyIslandTheme function on the alarm keypad—it's simply an
Easter Egg that plays the theme from Monkey Island through a simple buzzer. Being my favorite
adventure game, adding its iconic music provides a fun and nostalgic touch to the project.

{{< youtubeLite id="6ORsT4Gs9hA" label="Monkey Island PC Speaker Theme on an Arduino" >}}
