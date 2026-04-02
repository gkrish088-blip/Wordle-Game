# Wordle 2.0

A browser-based word guessing game built from scratch — no frameworks, no libraries, just vanilla HTML, CSS, and JavaScript. The rules are simple: guess a six-letter word in five attempts. The twist is that the word changes every day, and your stats follow you across sessions.

---

## How to Play

You have five rows and six columns. Each row is one attempt. Type a six-letter word using the on-screen keyboard or your physical keyboard and press Enter. The game will color each tile based on how close your guess is:

- **Green** — the letter is correct and in the right position
- **Yellow** — the letter exists in the word but is in the wrong position
- **Dark grey** — the letter is not in the word at all

The keyboard at the bottom updates in real time to reflect the state of each letter, so you always know what you have eliminated and what is still in play.

---

## Features

**Daily word that changes automatically**
The game picks a new word every day without any server or backend. It does this by taking a fixed reference timestamp hardcoded into the game, computing how many days have passed since then using `Date.now()`, and feeding that number into a hash function. The hash output is then used as an index into the word list.

**DJB2 hashing for word selection**
The hash function used is DJB2, a classic non-cryptographic hash algorithm. It starts with a seed of 5381 and for each character of the input string, it does `hash = (hash * 33) XOR charCode`. The result is then made unsigned with `>>> 0` to prevent JavaScript from treating it as a negative number. The final value, modulo the length of the word list, gives a consistent index for the day. Because the input is always the number of days since the fixed timestamp, the same day always produces the same word — and nobody can tamper with it by changing their system clock in a useful way.

```js
function djb2Hash(str) {
  let hash = 5381;
  for (let i = 0; i < str.length; i++) {
    hash = (hash * 33) ^ str.charCodeAt(i);
  }
  return hash >>> 0;
}
```

**Correct yellow tile logic**
One of the trickier parts of building Wordle is handling yellow tiles accurately. A naive approach — color a tile yellow if the letter appears anywhere in the target — produces wrong results when a letter appears multiple times. For example, if the target is "ABBEY" and you guess "EERIE", only one E should be yellow, not all three.

The solution here is a two-pass system. In the first pass, all green matches are identified and those positions are nulled out from both the guess and the target. In the second pass, the remaining letters are compared, and a yellow is assigned only when there is a matching letter still available in the nulled target. This ensures that duplicate letters are counted fairly and tiles are not over-coloured.

**Persistent user stats across sessions**
When you visit for the first time, the game asks for your name. It then creates a user object with your name, total wins, and win streak, and stores it in `localStorage` as a JSON string. On every subsequent visit the overlay is skipped and your data is loaded immediately.

```js
const user = { username: "krish", totalWins: 0, winStreak: 0 };
localStorage.setItem("user", JSON.stringify(user));
```

When you win, `totalWins` and `winStreak` are both incremented and written back. When you lose, `winStreak` resets to zero but `totalWins` is preserved. This is the read-modify-write pattern — localStorage has no partial update method, so you always pull the full object, change what you need, and push it back.

**Win streak tracking**
The streak counter goes up by one each time you guess correctly and drops to zero the moment you run out of tries. It is stored in the same user object so it survives page refreshes and browser restarts.

**Total wins counter**
Separate from the streak. A loss resets the streak but does not touch total wins. This gives you an honest record of how many times you have ever solved the puzzle.

---

## Navigation

**Left side — three horizontal bars**
Clicking the bars icon in the top left takes you directly to the developer's portfolio. It opens in a new tab so you do not lose your game state.

**Right side — bar chart icon**
Clicking the chart icon on the right side of the nav opens a small stats panel that drops down just below the nav bar. It shows your username, total wins, and current win streak pulled live from localStorage. It does not cover the game grid or interrupt anything — it floats above the page using `position: absolute` and `z-index`, so the rest of the layout is completely unaffected.

---

## Word Validation

The game only accepts words that exist in its built-in word list. If you type a string that is not in the list and press Enter, a brief overlay flashes on screen saying "Not in the words list" and disappears after 500 milliseconds. The row is not advanced.

The word list is stored as a JavaScript array and converted into a `Set` at startup for O(1) lookup:

```js
const validWords = new Set(wordsarray);
```

---

## Name Validation

On first visit the name input only accepts letters. The validation uses a regular expression that rejects anything containing numbers, symbols, or whitespace. Spaces between words are stripped before testing, so "john doe" is treated as "johndoe" for the purpose of validation and both pass.

```js
const onlyLetters = /^[a-zA-Z]+$/.test(name.replaceAll(" ", ""));
```

---

## Tech Stack

- HTML, CSS, JavaScript — no build tools, no frameworks
- FontAwesome for icons
- Browser localStorage for persistence
- No backend, no database, no API calls

---

## Project Structure

```
wordle-2.0/
  index.html    — markup and keyboard layout
  index.css     — all styling
  script.js     — game logic, hashing, localStorage, input handling
```

---

## Running Locally

Clone or download the repo and open `index.html` in any modern browser. No install steps, no package manager, no server required.

```bash
git clone <repo-url>
cd wordle-2.0
open index.html
```

---

Built by Krish. Portfolio linked in the top left.