# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable. 

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the broken app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.** - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

**Game Purpose:**
A number-guessing game where the player tries to guess a randomly generated secret number within a limited number of attempts. The game provides Higher/Lower hints and tracks score across guesses.

**Bugs Found and Fixed:**

| # | Bug | Fix |
|---|-----|-----|
| 1 | Secret number changed on every click (no session state) | Wrapped secret in `if "secret" not in st.session_state` |
| 2 | Hints were inverted (Go HIGHER when too high) | Corrected `<`/`>` comparison logic |
| 3 | Even-numbered attempts converted secret to string, causing TypeError | Always keep secret as integer |
| 4 | Enter key didn't submit the guess | Wrapped input in `st.form()` |
| 5 | `st.rerun()` cleared hints before user could read them | Removed `st.rerun()` calls; let form handle refresh |
| 6 | Invalid inputs showed harsh `st.error()` | Replaced with `st.warning()` |
| 7 | "New Game" always used 1–100 range, ignoring difficulty | Changed to `random.randint(low, high)` |
| 8 | "New Game" didn't reset `status` or `history` | Added full state reset |
| 9 | Attempt counter started at 1, giving one fewer attempt | Changed initial attempts to `0` |
| 10 | Changing difficulty didn't regenerate the secret | Track `current_difficulty` and regenerate on change |
| 11 | Attempt display only showed "attempts left" | Show both "Attempts used" and "Attempts left" |
| 12 | Invalid inputs didn't count toward attempt limit | Moved `attempts += 1` to top of submit handler |
| 13 | `logic_utils.py` functions were unimplemented stubs | Refactored all logic from `app.py` into `logic_utils.py` |

**Testing:**
Logic functions were tested with `pytest`. All 3 tests in `tests/test_game_logic.py` pass after refactoring `check_guess`, `parse_guess`, `get_range_for_difficulty`, and `update_score` into `logic_utils.py`.

## 📸 Demo

The fixed game running in Normal difficulty (range 1–100, 8 attempts):

- Hints correctly say **Go HIGHER!** or **Go LOWER!**
- Attempt counter shows both used and remaining attempts
- Enter key submits the guess (no button click needed)
- Changing difficulty regenerates the secret in the correct range
- "New Game" fully resets all state

To run locally: `python -m streamlit run app.py`

## 🚀 Stretch Features

**Challenge 1 – pytest Results:**

All 3 tests in `tests/test_game_logic.py` pass after refactoring logic into `logic_utils.py`:

```
tests/test_game_logic.py::test_winning_guess   PASSED
tests/test_game_logic.py::test_guess_too_high  PASSED
tests/test_game_logic.py::test_guess_too_low   PASSED

3 passed in 0.05s
```
