# Task 1 - Voice Assistant (Advanced tier)

A Python voice assistant that listens through the microphone, understands free-form sentences, and answers aloud.

## Setup

```bash
pip install -r requirements.txt
```

PyAudio needs a system library for the microphone:

| OS | Command |
|----|---------|
| Windows | `pip install pyaudio` (usually works directly) |
| macOS | `brew install portaudio` then `pip install pyaudio` |
| Linux | `sudo apt install portaudio19-dev python3-pyaudio espeak` |

Optional environment variables (use a **test/dummy** email account):

```bash
export OPENWEATHER_API_KEY="your_key"      # free key from openweathermap.org
export EMAIL_ADDRESS="test.account@gmail.com"
export EMAIL_PASSWORD="app_password"       # Gmail app password, not your real password
# optional: SMTP_SERVER (default smtp.gmail.com), SMTP_PORT (default 587)
```

On Windows PowerShell use `$env:OPENWEATHER_API_KEY="your_key"`.

## Run

```bash
python voice_assistant.py          # speak to it
python voice_assistant.py --text   # type commands (no microphone needed)
python voice_assistant.py --mute   # print replies without speaking
```

## Things to say

| Say | What happens |
|-----|--------------|
| "Hello" | Predefined greeting |
| "What time is it?" / "What's today's date?" | Current time / date |
| "Search for python tutorials" | Opens Google results in the browser |
| "What's the weather in Chennai?" | Live weather from OpenWeatherMap |
| "Remind me to drink water in 10 minutes" | Timer; audible beep plus spoken alert |
| "Send an email to test account" | Asks subject and message, confirms, then sends |
| "Who is Alan Turing?" / "What is photosynthesis?" | Local knowledge base, then Wikipedia |
| "Add a new command" | Asks for a phrase and response, saves to config |
| "Goodbye" | Exits |

If speech isn't understood, the assistant asks you to repeat.

## Feature checklist

**Beginner:** voice input, hello greeting, time and date, web search, repeat-on-error, pyttsx3 replies.
**Advanced:** intent parsing (stemmed keyword weighting plus regex entity extraction, so "could you tell me what the weather is like in Mumbai" works), email via `smtplib`, reminders with audible alert, weather API, QA (local knowledge base plus Wikipedia), custom commands (config file or voice prompt), privacy documentation (below).

## How the intent parsing works

1. Text is normalised and stemmed (NLTK `PorterStemmer`, with a tiny fallback if NLTK is missing).
2. Each intent has weighted keywords; the highest score above a threshold wins.
3. Question-like sentences that match no intent go to general QA.
4. Entities (city, search query, duration, recipient) are pulled out with regular expressions.

## Custom commands

Edit `assistant_config.json`:

```json
"custom_commands": {
  "open github": { "type": "url", "value": "https://github.com" },
  "who made you": { "type": "speak", "value": "A student built me." }
}
```

`contacts` maps spoken names to email addresses; `knowledge_base` maps questions to answers.

## Privacy: what data is processed and how

- **Audio:** captured from the microphone only while the assistant says "Listening...". It is sent to Google's free Web Speech API (`recognize_google`) for transcription. Audio is not saved to disk by this program.
- **Transcribed text:** processed locally for intent detection. It is not logged or stored, except custom commands you explicitly add (saved to `assistant_config.json`).
- **Weather:** the city name is sent to OpenWeatherMap together with your API key.
- **General questions:** the topic is sent to Wikipedia's public API when the local knowledge base has no answer.
- **Email:** recipient, subject and body go to your SMTP server. Credentials are read from environment variables and never written to files. Use a throwaway account.
- **Reminders** live in memory only and are lost when the program exits.
- To avoid sending audio to any service, use `--text` mode.


# Task 2 - BMI Calculator (Advanced tier)

Calculates Body Mass Index, classifies it, colour-codes the result, saves records per user in SQLite, and plots each user's BMI trend.

## Setup and run

```bash
pip install -r requirements.txt   # only matplotlib; tkinter and sqlite3 ship with Python
python bmi_calculator.py          # GUI
python bmi_calculator.py --cli    # Beginner command-line version
```

On Linux, if tkinter is missing: `sudo apt install python3-tk`.

## How to use the GUI

1. Type a **user name** (or pick an existing one), then enter **weight (kg)** and **height (m)**.
2. Press **Calculate** (or Enter). The result panel is colour-coded:
   blue = Underweight, green = Normal, orange = Overweight, red = Obese.
3. Press **Save record** to store it under that user. The history table updates.
4. Press **Show trend** for a matplotlib line chart of that user's BMI over time, with the category bands shaded.

## Formula and categories

`BMI = weight(kg) / height(m)^2`, shown to 2 decimal places.

| BMI | Category |
|-----|----------|
| < 18.5 | Underweight |
| 18.5 - 24.9 | Normal |
| 25 - 29.9 | Overweight |
| >= 30 | Obese |

## Feature checklist

**Beginner (`--cli`):** prompts for weight and height, formula, categories, 2-decimal output, validation (non-numeric, zero, negative, impossible values, and a hint if height looks like centimetres).
**Advanced (GUI):** tkinter window, labelled inputs and Calculate button, colour-coded result, multi-user records, SQLite persistence, matplotlib trend chart, database error handling (every database call is wrapped; failures show a message box instead of crashing).

## Data storage

Records are saved in `bmi_records.db` next to the script (table `records`: user, weight, height, BMI, category, timestamp). Nothing is sent over the network. Use `--db path.db` for a separate file.

BMI is a screening number only and is not a medical diagnosis.

# Task 3 - Random Password Generator (Advanced tier)

Generates strong random passwords with a GUI, a strength meter, ambiguous-character filtering, and automatic clipboard copy.

## Setup and run

```bash
pip install -r requirements.txt   # pyperclip (tkinter ships with Python)
python password_generator.py         # GUI
python password_generator.py --cli   # Beginner command-line version
```

On Linux, `pyperclip` needs `xclip` or `xsel` (`sudo apt install xclip`); if missing, the app falls back to Tk's built-in clipboard. If tkinter is missing: `sudo apt install python3-tk`.

## Features

**Beginner (`--cli`):** choose length (min 8 enforced), choose character types (at least 2), generate and display, validate bad input, generate again without restarting.

**Advanced (GUI):**
- Slider and spinbox for length (8-64), checkboxes for uppercase, lowercase, numbers, symbols
- `secrets` module for cryptographically secure generation (not `random`)
- Strength bar and label (Weak / Medium / Strong)
- Guaranteed at least one character from each selected type
- Password is copied to the clipboard automatically on generation (`pyperclip`)
- Checkbox to exclude ambiguous characters (`0 O l 1 I |`)
- Last 5 passwords shown for the session (double-click one to copy it)

## Strength rating

Estimated entropy = `length x log2(size of character pool actually used)`.

| Entropy | Rating |
|---------|--------|
| < 50 bits | Weak |
| 50 - 74 bits | Medium |
| >= 75 bits | Strong |

For example, 16 characters using all four types is about 100 bits (Strong); 8 letters is about 46 bits (Weak).

## Security notes

- Why `secrets`: `random` is predictable if its state is known; `secrets` draws from the operating system's secure source.
- Positions of the guaranteed characters are hidden with a secure shuffle.
- Session history is held in memory only. It is **never written to a file** and disappears when the app closes.
- Clipboard contents can be read by other programs, so paste the password promptly and clear your clipboard history if you can.


