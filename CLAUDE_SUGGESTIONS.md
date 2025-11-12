# GWORDLE - Code Analysis & Improvement Suggestions

## Purpose

GWORDLE is a Wordle clone game created as an educational project. Key goals:
- Allow unlimited plays (no daily limit like the original Wordle)
- Teach web development basics (HTML, CSS, JavaScript)
- Keep it as a single-file application

The game provides 5 attempts to guess a random 5-letter word, with color-coded feedback:
- **Green**: Correct letter in correct position
- **Yellow**: Correct letter in wrong position
- **Gray**: Letter not in the word

---

## Critical Bugs

### 1. Debugger Statement in Production Code
**Location**: `index.html:164`

```javascript
var word = chooseWord();
debugger;  // ← Remove this
```

**Fix**: Remove the `debugger;` statement from production code.

---

### 2. Duplicate Letter Handling Bug
**Location**: `index.html:314-323` in the `getStates()` function

**Issue**: The function doesn't properly account for duplicate letters. For example, if the word is "ABBEY" and you guess "ERASE", it may incorrectly mark both E's as out-of-place even though only one E exists in the target word.

**Current Logic Problem**:
```javascript
for(var guessIndex = 0; guessIndex < word.length; guessIndex++) {
    if (CORRECT == results[guessIndex]) { continue; }

    for(var wordIndex = 0; wordIndex < word.length; wordIndex++) {
        if (guess[guessIndex] == word[wordIndex] && BAD == results[guessIndex]) {
            results[guessIndex] = OUT_OF_PLACE;
            break;
        }
    }
}
```

This doesn't check if a letter has already been "used" by a correct match elsewhere.

**Suggested Fix**: Track which letters in the target word have been matched and don't allow duplicate OUT_OF_PLACE markings.

---

### 3. Wrong CSS Class for Alphabet Letters
**Location**: `index.html:270`

```javascript
case OUT_OF_PLACE:
    cell.classList.add('out-of-place');
    alphabetCell.classList.add('correct');  // ← Should be 'out-of-place' or different class
    alphabetCell.removeAttribute('data-not-set');
    break;
```

**Issue**: OUT_OF_PLACE letters get marked as 'correct' in the alphabet grid, which is misleading.

---

## Code Quality Improvements

### 4. Magic Numbers
**Locations**: Throughout the code

- Width (5) and height (5) are hardcoded in multiple places
- Should use constants at the top of the script:

```javascript
const GRID_WIDTH = 5;
const GRID_HEIGHT = 5;
```

---

### 5. No Input Validation
**Location**: `index.html:170-172`

**Issues**:
- Input field accepts any characters; should restrict to letters only
- No handling for lowercase input until button click
- Users can paste invalid content

**Suggested Fix**: Add real-time validation:
```javascript
document.querySelector('.guess-input').addEventListener('keyup', (e) => {
    let value = e.target.value.toUpperCase().replace(/[^A-Z]/g, '');
    e.target.value = value;
    guessInputValueChanged();
});
```

---

### 6. Poor User Experience with Alerts
**Location**: `index.html:243-245`

```javascript
if (!dictionary.includes(guessWord)) {
    alert('That is not a valid word.');  // ← Jarring UX
    document.querySelector('.guess-input').value = '';
    return;
}
```

**Fix**: Replace alerts with styled in-page notifications (toast messages or inline error messages).

---

### 7. External Word List Not Used
**Issue**: `safedict_full.txt` (2314 words) exists but the dictionary is hardcoded in JavaScript (~20KB embedded in HTML).

**Suggested Fix**: Load dictionary from external file:
```javascript
async function loadDictionary() {
    const response = await fetch('safedict_full.txt');
    const text = await response.text();
    return text.trim().toUpperCase().split('\n');
}
```

---

### 8. No Replay Functionality
**Issue**: Players must refresh the entire page to play again.

**Suggested Fix**: Add a "New Game" button that:
- Resets the game state
- Clears the grid
- Chooses a new word
- Resets the alphabet grid

---

### 9. Missing Accessibility Features
**Issues**:
- No ARIA labels for screen readers
- No keyboard shortcuts (e.g., Enter key to submit)
- Poor contrast ratios may fail WCAG standards
- No focus management

**Suggested Fixes**:
- Add `aria-label` attributes
- Add Enter key handler for submission
- Test color contrast ratios
- Manage focus states properly

---

### 10. No Mobile Responsiveness
**Locations**: `index.html:9-127` (CSS section)

**Issues**:
- Fixed pixel widths (100px cells, 824px alphabet grid)
- Will break on small screens
- No viewport meta tag

**Suggested Fixes**:
- Add viewport meta tag: `<meta name="viewport" content="width=device-width, initial-scale=1.0">`
- Use responsive units (%, em, rem, vw/vh)
- Add media queries for different screen sizes
- Use CSS Grid with `fr` units and `minmax()`

---

## Architecture Improvements

### 11. Global Variables
**Issue**: All variables are in global scope, risking naming collisions.

**Suggested Fix**: Use module pattern or class:
```javascript
class GwordleGame {
    constructor() {
        this.word = null;
        this.guessIndex = 0;
        this.gameState = GAME_RUNNING;
        // ... other state
    }

    init() { /* ... */ }
    makeGuess(word) { /* ... */ }
    // ... other methods
}

const game = new GwordleGame();
game.init();
```

---

### 12. Tight Coupling
**Issue**: DOM manipulation mixed with game logic throughout the code.

**Suggested Fix**: Separate concerns using MVC pattern:
- **Model**: Game state, word validation, scoring logic
- **View**: DOM manipulation, rendering
- **Controller**: Event handling, coordinating Model and View

---

### 13. No State Management
**Issue**: Game state is scattered across DOM and variables.

**Suggested Fix**: Centralize state:
```javascript
const gameState = {
    targetWord: '',
    guesses: [],
    currentGuess: '',
    gameStatus: 'playing', // 'playing', 'won', 'lost'
    usedLetters: new Set()
};
```

---

### 14. Code Organization
**Issue**: All code in one script block (300+ lines).

**Suggested Fix**:
- Extract functions logically
- Group related functionality
- Add comments for sections
- Consider splitting into multiple files

---

## Feature Enhancements

### 15. Statistics Tracking
**Suggested Feature**:
- Track wins/losses using `localStorage`
- Show current streak
- Display guess distribution histogram
- Show total games played

---

### 16. Difficulty Levels
**Suggested Feature**:
- 4-letter words (easier)
- 6-letter words (harder)
- Adjustable number of guesses
- Hard mode (must use revealed hints)

---

### 17. Visual Feedback & Animations
**Suggested Feature**:
- Animate tile flips like original Wordle
- Shake animation for invalid words
- Victory animation on win
- Bounce/pop effects on letter entry
- Smooth color transitions

---

### 18. Share Functionality
**Suggested Feature**: Generate shareable emoji grids:
```
Gwordle 3/6

⬜🟨⬜⬜🟨
🟨🟩⬜🟨⬜
🟩🟩🟩🟩🟩
```

Include "Copy to clipboard" button.

---

### 19. Dark Mode
**Suggested Feature**:
- Add theme toggle switch
- Store preference in `localStorage`
- Excellent teaching opportunity for CSS variables and JavaScript theming

---

### 20. Hint System
**Suggested Feature**:
- Optional hint button for younger players
- Could reveal one letter position
- Limit number of hints per game
- Could disable hints for "hard mode"

---

## Performance Considerations

### 21. Large Embedded Dictionary
**Issue**: 2314 words embedded in HTML = ~20KB of page weight.

**Suggested Fix**:
- Load from external file (already exists: `safedict_full.txt`)
- Consider compression
- Cache in `localStorage` after first load

---

## Suggested Priority Order

### High Priority (Do First)
1. ✅ Fix duplicate letter bug in `getStates()`
2. ✅ Remove `debugger` statement
3. ✅ Add "New Game" button
4. ✅ Fix alphabet cell CSS bug (line 270)
5. ✅ Add input validation (letters only)
6. ✅ Replace alerts with better UI
7. ✅ Add Enter key to submit guess

### Medium Priority
8. Load dictionary from external file
9. Add mobile responsiveness
10. Implement proper state management
11. Add keyboard shortcuts
12. Improve accessibility (ARIA labels)
13. Extract constants for magic numbers

### Low Priority (Nice to Have)
14. Add animations
15. Statistics tracking with localStorage
16. Share functionality
17. Dark mode
18. Refactor to use modules/classes
19. Add difficulty levels
20. Hint system

---

## Positive Notes

Despite the suggestions above, the code shows several strengths:

- ✅ **Clean, readable code** - Easy to understand logic
- ✅ **Functional game** - Core mechanics work correctly (except duplicate letter edge case)
- ✅ **Good educational project** - Achieves the teaching goals
- ✅ **Single-file design** - Meets the stated architectural goal
- ✅ **Proper use of CSS Grid** - Modern layout techniques
- ✅ **No external dependencies** - Pure HTML/CSS/JS
- ✅ **Proper word validation** - Checks against dictionary

The game is a solid foundation that works well for its intended purpose!
