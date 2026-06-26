# Kaun Banega C-Programmer (KBC Quiz Game in C)

A fully playable, terminal-based clone of *Kaun Banega Crorepati* / *Who Wants to Be a Millionaire*, written in pure C. Built as a college systems-programming project with a focus on clean modular design, file I/O, and POSIX system calls — not just a toy script.

```
  _  __ ____   ____
 | |/ /|  _ \ / ___|
 | ' / | |_) | |
 | . \ |  _ <| |___
 |_|\_\|_| \_\\____|
        KAUN BANEGA C-PROGRAMMER
        (Who Wants to be a C Crorepati)
============================================================
Player: Aditi   |   Current Winnings: Rs. 320000

Lifelines: [50:50 USED] [POLL] [PHONE]
============================================================
Question 11 (Rs. 640000)
============================================================
Who discovered penicillin?

  A: Louis Pasteur
  C: Robert Koch
  D: Joseph Lister

Time left: 22 sec  (Answer: A/B/C/D, lifelines: 5050/POLL/PHONE, or QUIT) >
```

## Features

- **15-level prize ladder** from ₹1,000 up to the ₹1,00,00,000 grand prize, modeled after the real show, with two "safe" checkpoints (Q5, Q10) that guarantee a minimum payout.
- **75+ question bank** spanning 15 difficulty tiers, loaded from a plain-text data file (`data/questions.txt`) — a new random question per tier is selected every game, so replays stay fresh.
- **Three classic lifelines**, usable once each per game:
  - `50:50` — eliminates two incorrect options
  - `POLL` — simulated audience poll, weighted toward the correct answer (the bias scales down on harder questions, just like a real crowd would get shakier)
  - `PHONE` — simulated "phone a friend" with a confidence-based response
- **Real countdown timer** (30s/question) implemented with `select()` on `stdin` — no busy-waiting, no extra threads, live in-place countdown.
- **Colored terminal UI** via ANSI escape codes: prize ladder highlighting, color-coded correct/wrong feedback, sound cues (terminal bell) on key events.
- **Persistent leaderboard** saved to disk (`leaderboard.dat`), sorted and displayed across sessions — survives restarts.
- **Walk-away option** (`QUIT`) at any point, banking current winnings — exactly like the real show.
- Zero compiler warnings under `-Wall -Wextra`; verified leak-free with AddressSanitizer/UBSan across full playthroughs.

## Project Structure

```
kbc-game/
├── src/
│   ├── main.c           # Menu loop, entry point
│   ├── game.c            # Core game loop, scoring, safe-level logic
│   ├── game.h             # Shared structs, constants, prototypes
│   ├── questions.c     # Question bank loading + per-game random selection
│   ├── lifelines.c       # 50:50, audience poll, phone-a-friend logic
│   ├── leaderboard.c   # Score persistence (load/save/sort)
│   ├── timer_input.c   # select()-based timed stdin input
│   └── ui.c                # ANSI colors, banner, prize ladder rendering
├── data/
│   └── questions.txt    # Question bank (pipe-delimited, editable)
├── Makefile
└── README.md
```

## Build & Run

Requires a POSIX environment (Linux, macOS, or WSL on Windows) with `gcc` and `make`.

```bash
git clone https://github.com/<your-username>/kbc-game.git
cd kbc-game
make
./kbc_game
```

Or in one step:

```bash
make run
```

> Run the binary from the project root so it can find `data/questions.txt`.

To clean build artifacts and the saved leaderboard:

```bash
make clean
```

## How to Play

1. Choose **Play Game** from the main menu and enter your name.
2. Answer each question with `A`, `B`, `C`, or `D` before the 30-second timer runs out.
3. Use a lifeline if you're stuck: `5050`, `POLL`, or `PHONE` (each can only be used once).
4. Type `QUIT` at any time to walk away with your current winnings.
5. Cross Question 5 or Question 10 and that prize amount becomes your **guaranteed minimum** — even a wrong answer later won't take you below it.
6. Answer all 15 questions correctly to become the **C-Crorepati**.

## Adding Your Own Questions

`data/questions.txt` is plain text — pipe-delimited, one question per line:

```
level|question|optionA|optionB|optionC|optionD|correct
```

`level` is 1–15, `correct` is `A`/`B`/`C`/`D`. Add as many extra questions per level as you like; the game randomly picks one per level each playthrough, so a bigger bank means more replay variety.

## Design Notes / What This Project Demonstrates

This was built to go beyond a basic "if-else quiz" and actually exercise core C and systems concepts:

- **Modular architecture** — game logic, UI, I/O, and data are cleanly separated across translation units with a shared header, rather than one giant `main.c`.
- **File I/O & simple data persistence** — both the question bank and the leaderboard are read from / written to disk in a custom lightweight format, parsed with `strtok`/`sscanf`.
- **POSIX system calls** — `select()` is used to implement a non-blocking, interruptible countdown timer on `stdin`, instead of relying on threads or busy-loops.
- **Defensive parsing** — malformed lines in the question file are skipped rather than crashing the program; user input is trimmed/validated before use.
- **Memory safety** — verified with `-fsanitize=address,undefined` across full playthroughs (lifelines, timeouts, wins, losses) with zero issues.

## Possible Extensions

- Difficulty-based question categories (Science, History, Sports) selectable from the menu
- An "admin mode" to add/edit questions from inside the game instead of editing the text file
- Cross-platform timer support for native Windows builds (currently POSIX-only via `select()`)
- Export leaderboard to CSV / JSON for external analysis

## License

MIT — feel free to fork, extend, and use this in your own portfolio.
