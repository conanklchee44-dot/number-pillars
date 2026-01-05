# Number Pillars
A simple math game I got and redesigned to be complex
## how to play
click a box to remove it from the stack. 
Try to make the stack total sum equal to the target number, displayed on the left of the screen
### more stuff
 - Change difficulty using the difficulty settings (EASY, MEDIUM, HARD)
 - Create your own levels using the level editor!
 - Check your stats in the stats viewer
 - Adjust settings in the settings menu
## nerd stuff
I just used nested dictionaries for the main levels
``` javascript
// game data
let quizzes = [{
        numbers: [13, 19, 7, 9, 1, 3],
        answer: 11
    },
    {
        numbers: [11, 18, 6, 15, 8, 29, 4],
        answer: 23
    },
    {
        numbers: [7, 9, 3, 5, 4, 6, 9, 10],
        answer: 18
    },
    {
        numbers: [4.5, 4.5, 5.5, 7.5, 1.5],
        answer: 11.5
    },
    {
        numbers: [3.7, 2.3, 5.5, 4.8, 1.2, 8.1],
        answer: 11.5
    },

    {
        numbers: [2.25, 7.5, 5.25, 1.7, 2.3, 0.25],
        answer: 11.75
    },
    {
        numbers: [0.125, 3.375, 2.75, 1.25, 0.875, 0.125, 1.625, 3.75, 1.25, 3.475, 3.275],
        answer: 11.75
    },
];
```
Custom quizzes are stored in localStorage like so:
``` javascript
function loadCustomQuizzes() {
    try {
        const stored = localStorage.getItem('customQuizzes');
        if (stored) {
            customQuizzes = JSON.parse(stored);
            console.log(customQuizzes);
            // push to quizzes
            customQuizzes.forEach(q => quizzes.push(q));
        }
    } catch (e) { customQuizzes = []; }
    renderCustomList();
}

function storeCustomQuizzes() {
    localStorage.setItem('customQuizzes', JSON.stringify(customQuizzes));
}
```
All settings are also persisted in localStorage:
``` javascript
if (soundToggle) {
    soundToggle.addEventListener('click', () => {
        localStorage.setItem('soundMuted', soundMuted);
        updateSoundUI();
    });
}
if (confettiToggle) {
    confettiToggle.addEventListener('click', () => {
        confettiEnabled = !confettiEnabled;
        localStorage.setItem('confettiEnabled', confettiEnabled);
        updateConfettiUI();
    });
}
if (popupSizeToggle) {
    popupSizeToggle.addEventListener('click', () => {
        popupLarge = !popupLarge;
        localStorage.setItem('popupLarge', popupLarge);
        updatePopupSizeUI();
    });
}
```

The js uses showPopup() and attachTooltip() functions to be modular:
``` javascript
// popup helper
function showPopup(message, opts = {}, color=null) {
    const popup = document.getElementById('popup');
    if (!popup) return;
    popup.textContent = message;
    popup.classList.remove('hidden', 'show', 'shake', 'large');
    if (popupLarge) popup.classList.add('large');
    void popup.offsetWidth; // force reflow
    if (opts.shake) popup.classList.add('shake');
    popup.classList.add('show');
    if (color) { popup.style.color = color; } else { popup.style.color = ''; }
    setTimeout(() => {
        popup.classList.remove('show', 'shake', 'large');
        setTimeout(() => popup.classList.add('hidden'), 400);
    }, opts.duration || 1800);
}
// tooltip helper
function attachTooltip(el, getMessage) {
    if (!el) return () => {};
    let _prevTitle = null;
    const resolveMessage = () => (typeof getMessage === 'function' ? getMessage() : getMessage);
    const show = () => {
        const msg = resolveMessage();
        _prevTitle = el.getAttribute('title');
        if (_prevTitle !== null) el.removeAttribute('title');
        if (msg) el.setAttribute('data-tooltip', msg);
        // mark visible for css
        el.setAttribute('data-tooltip-visible', 'true');
    };
    const hide = () => {
        el.removeAttribute('data-tooltip-visible');
        setTimeout(() => el.removeAttribute('data-tooltip'), 220);
        if (_prevTitle !== null) {
            el.setAttribute('title', _prevTitle);
            _prevTitle = null;
        }
    };
    el.addEventListener('mouseenter', show);
    el.addEventListener('focus', show);
    el.addEventListener('mouseleave', hide);
    el.addEventListener('blur', hide);
    // return a detach function
    return () => {
        el.removeEventListener('mouseenter', show);
        el.removeEventListener('focus', show);
        el.removeEventListener('mouseleave', hide);
        el.removeEventListener('blur', hide);
        hide();
    };
}

```
This was very useful for later on:
``` javascript
// attach generic tooltips
attachTooltip(resetBtn, 'Reset the level');
attachTooltip(easyBtn, 'EASY mode - 5 lives, hints enabled');
attachTooltip(mediumBtn, 'MEDIUM mode - 3 lives, hints enabled');
attachTooltip(hardBtn, 'HARD mode - 1 life, no hints. Sum is hidden');
attachTooltip(editorToggle, 'LEVEL EDITOR - Create and save your own levels!');
attachTooltip(settingsBtn, 'Open SETTINGS menu');
attachTooltip(settingsClose, 'Close SETTINGS menu');
attachTooltip(soundToggle, () => soundMuted ? 'Unmute all sounds' : 'Mute all sounds');
attachTooltip(confettiToggle, () => confettiEnabled ? 'Disable confetti effects' : 'Enable confetti effects');
attachTooltip(popupSizeToggle, () => popupLarge ? 'Use small popups' : 'Use large popups');
attachTooltip(restoreValues, 'Restore default settings');
// hint button with dynamic message
attachTooltip(hintBtn, () => {
    if (difficulty === 'hard') return 'Hints disabled in HARD mode';
    if (parseInt(xp.innerText) < hintCost) return `You need ${hintCost - parseInt(xp.innerText)} more XP for a hint`;
    return `Cost: ${hintCost} XP`;
});

attachTooltip(xp, ' Earn XP by completing levels. Spend your XP on hints!');
attachTooltip(lives, 'Number of LIVES remaining. Lose all lives and it\'s GAME OVER!');
attachTooltip(levelIndex, 'Current LEVEL. Advance by completing quizzes.');
attachTooltip(targetNum, 'TARGET number. You need to reach it by removing numbers from the stack.');
attachTooltip(answerSum, () => {
    if (difficulty === 'hard') return 'Sum hidden in HARD mode';
    return 'Current SUM of remaining numbers in the stack.';
}
);
```
### styling stuff for nerds
I used Walter Turncoat, Nanum Pen Script, and Sigmar One. This is the code for that:
``` html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Nanum+Pen+Script&family=Sigmar+One&family=Walter+Turncoat&display=swap" rel="stylesheet">
```
All fonts are sourced from Google Fonts.

These are the css variables and colors I used:
``` css
--default-bgcolor: #2e86c1;
--default-bghover: #138d75;
--default-menucolor: #10161c;
--default-transparent: rgba(0, 0, 0, 0.2);
--default-shadow: rgba(0, 0, 0, 0.4);
--default-font: "Nanum Pen Script", sans-serif;
--button-font: "Sigmar One";
--yellow: #ffd166;
```