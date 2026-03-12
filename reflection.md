# 💭 Reflection: Game Glitch Investigator

## 🐛 Bug Fixes Applied

During the debugging process, we identified and fixed 16 distinct bugs that were affecting the game's functionality, user experience, and logic. Here's a comprehensive summary of each issue and its resolution:

### 1. **String/Integer Comparison Bug**

**Issue:** On even-numbered attempts, the secret number was being converted to a string while user guesses remained integers, causing a TypeError that triggered incorrect "Go LOWER!" responses for all guesses.  
**Logic:** The comparison `guess > secret` failed when one was a string and one was an integer, falling back to flawed string-based logic.  
**Fix:** Ensured the secret always remains an integer for consistent numeric comparisons.

### 2. **New Game Ignores Difficulty Range**

**Issue:** The "New Game" button always generated secrets in the 1-100 range regardless of the selected difficulty level.  
**Logic:** The button used hardcoded `random.randint(1, 100)` instead of respecting the difficulty-specific ranges.  
**Fix:** Changed to use `random.randint(low, high)` where low/high are calculated from the current difficulty setting.

### 3. **Enter Key Doesn't Submit**

**Issue:** Pressing Enter in the input field didn't submit the guess - only clicking the button worked.  
**Logic:** The input was a standalone `st.text_input()` without form handling.  
**Fix:** Wrapped the input in `st.form()` so Enter key properly submits the form.

### 4. **Form Doesn't Reset After Submission**

**Issue:** After submitting a guess, the form didn't clear for the next attempt, causing confusion.  
**Logic:** Streamlit forms needed explicit page refresh to reset input state.  
**Fix:** Added `st.rerun()` after processing each guess to refresh the page and clear the form.

### 5. **Hints Not Displaying**

**Issue:** After form submission, hints disappeared due to page refresh timing issues.  
**Logic:** The `st.rerun()` was clearing the hint messages before they could be displayed.  
**Fix:** Removed the problematic `st.rerun()` calls and let Streamlit forms handle the refresh naturally.

### 6. **Error Messages for Invalid Input**

**Issue:** Invalid inputs (non-numbers, empty strings) showed harsh error messages that disrupted gameplay.  
**Logic:** Used `st.error()` for input validation, which was too aggressive for a game context.  
**Fix:** Replaced error messages with gentle `st.warning()` notifications.

### 7. **Session State Modification Error**

**Issue:** Attempting to manually clear the input field caused Streamlit to throw "cannot be modified after instantiation" errors.  
**Logic:** Tried to modify `st.session_state.guess_input` after the widget was already created.  
**Fix:** Removed manual session state manipulation and let Streamlit forms handle input clearing automatically.

### 8. **Inverted Hint Logic**

**Issue:** Hints were backwards - said "Go HIGHER!" when guess was too high, and "Go LOWER!" when guess was too low.  
**Logic:** Used `if guess > secret` for "Go HIGHER!" instead of the correct logic.  
**Fix:** Corrected to `if guess < secret` → "Go HIGHER!", `if guess > secret` → "Go LOWER!".

### 9. **New Game Button Incomplete Reset**

**Issue:** "New Game" only reset the secret and attempts, but left game status and history in inconsistent states.  
**Logic:** Missing reset of `status` and `history` session state variables.  
**Fix:** Added `st.session_state.status = "playing"` and `st.session_state.history = []` to fully reset the game.

### 10. **Missing Show Hint Checkbox**

**Issue:** The checkbox to toggle hints was removed when making hints mandatory, but users wanted the option.  
**Logic:** Removed the UI control entirely instead of keeping it as an optional feature.  
**Fix:** Restored the "Show hint" checkbox so users can choose whether to see hints.

### 11. **Secret Not Regenerating on Difficulty Change**

**Issue:** Changing difficulty level didn't update the secret number - kept the old range's secret.  
**Logic:** Secret was only generated once at session start, not when difficulty changed.  
**Fix:** Added logic to regenerate secret whenever `difficulty` changes by tracking `current_difficulty` in session state.

### 12. **Incorrect Attempt Counting**

**Issue:** Attempt counter started at 1 instead of 0, giving users one fewer attempt than intended.  
**Logic:** `st.session_state.attempts = 1` meant users got 7 attempts instead of 8 for Normal difficulty.  
**Fix:** Changed initial attempts to 0 for proper counting.

### 13. **Missing Warnings for Invalid Input**

**Issue:** Invalid inputs were silently ignored without any user feedback.  
**Logic:** Removed error messages entirely instead of providing helpful warnings.  
**Fix:** Added back `st.warning(err)` for invalid inputs to guide users.

### 14. **Unfun Title Caption**

**Issue:** The subtitle "Something is off" was negative and didn't match the game's fun nature.  
**Logic:** Original AI-generated caption was misleading about the game's quality.  
**Fix:** Changed to "Challenge your guessing skills!" for a more engaging and positive tone.

### 15. **Unclear Attempt Display**

**Issue:** Only showing "attempts left" made it confusing to understand the relationship between used and remaining attempts.  
**Logic:** Single metric didn't provide full context about attempt usage.  
**Fix:** Changed display to show both "Attempts used: X | Attempts left: Y" for clarity.

### 16. **Invalid Attempts Not Counted**

**Issue:** Only valid numeric guesses counted toward the attempt limit, allowing users to spam invalid inputs.  
**Logic:** Attempts only incremented in the `else` block for valid guesses.  
**Fix:** Moved `st.session_state.attempts += 1` to the top of the submit handler so all submissions count.

---

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

When I first ran the game, it was completely unplayable — the secret number kept changing every single time I clicked "Submit," which made it impossible to ever win. The hints were also completely backwards: if I guessed too low, the game told me to go lower, and if I guessed too high, it told me to go higher. On top of that, on every even-numbered attempt the secret was silently converted to a string, causing a TypeError that always triggered the wrong hint. Pressing Enter in the input box did nothing — only the button worked — and there was no way to reset the game state cleanly between rounds.

---

## 2. How did you use AI as a teammate?

I used Claude (Claude Code in the terminal) as my primary AI assistant throughout this project. One suggestion that was correct: when I asked how to keep the secret number from resetting on every click, Claude suggested wrapping it in `if "secret" not in st.session_state`, which I verified by running the app and confirming the secret stayed stable across multiple guesses. One suggestion that was misleading: Claude initially recommended adding `st.rerun()` after every guess to clear the form, which I tried — but it caused all the hint messages to disappear before the user could read them, so I had to remove those calls and let Streamlit's form handling reset the input naturally instead.

---

## 3. Debugging and testing your fixes

I decided a bug was really fixed by testing it manually in the browser and also by running `pytest` for the logic functions. For example, I ran `pytest tests/test_game_logic.py -v` which showed all three tests — `test_winning_guess`, `test_guess_too_high`, and `test_guess_too_low` — failing because `check_guess` in `logic_utils.py` was still a stub raising `NotImplementedError`. Once I moved the correct logic into `logic_utils.py`, all three passed, which confirmed the core comparison logic was working. Claude helped me understand the test structure and why the tests expected a plain string return value rather than a tuple.

---

## 4. What did you learn about Streamlit and state?

The secret number kept changing because Streamlit reruns the entire Python script from top to bottom every single time the user interacts with the app — clicking a button, changing a dropdown, anything. Without session state, `random.randint()` was called fresh on every rerun, generating a brand-new secret each time. I'd explain it to a friend like this: imagine your browser refreshes the whole page every time you click anything, and all your variables reset to their defaults — that's a Streamlit rerun. Session state is like a sticky notepad that survives those refreshes. The fix was wrapping the secret assignment in `if "secret" not in st.session_state`, so it only generates a new secret once per session (or when the difficulty changes).

---

## 5. Looking ahead: your developer habits

One habit I want to reuse is writing the logic functions separately from the UI code and testing them with `pytest` before worrying about the Streamlit interface — it's much faster to catch bugs that way than by clicking through the app manually every time. One thing I'd do differently: next time I use AI on a coding task, I'd ask it to explain *why* it's suggesting a change before I apply it, because a few suggestions (like the `st.rerun()` calls) seemed correct at first but had side effects I didn't expect. This project changed how I think about AI-generated code — I used to assume it was basically correct and just needed small tweaks, but now I treat every AI suggestion as a starting hypothesis that needs to be tested and verified, not a finished solution.
