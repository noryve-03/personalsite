# Implementing Vim-like Navigation (Practice Guide)

This guide provides the structure and comments to help you implement Vim-like navigation on your website. The core logic is left for you to complete as a practice exercise in JavaScript.

---

### **Step 1: Create the Cursor Element (HTML)**

First, we need an element that will represent our cursor. Add this inside the `<body>` tag.

```html
<!-- Add this right after the opening <body> tag -->
<div id="vim-cursor"></div>
```

---

### **Step 2: Style the Cursor and Active Elements (CSS)**

Next, we'll style the cursor and create classes for highlighting the active element and the visual selection.

```css
/* Add this to your existing <style> block */

#vim-cursor {
    position: absolute;
    background-color: #00ff00;
    width: 10px;
    height: 1.2rem;
    opacity: 0.7;
    z-index: 9999;
    pointer-events: none;
    transition: top 0.05s, left 0.05s;
}

.vim-active-element {
    background-color: rgba(0, 255, 255, 0.15);
    outline: 1px solid rgba(0, 255, 255, 0.5);
}

.vim-visual-selection {
    background-color: rgba(70, 70, 120, 0.7);
    color: #fff;
}
```

---

### **Step 3: The Core Logic (JavaScript)**

Here is the skeleton of the JavaScript logic. Your task is to fill in the missing pieces.

```javascript
// Add this inside your <script> tag, preferably within the VimSite class

class VimSite {
    constructor() {
        // ... existing constructor properties
        this.vimMode = 'normal'; // 'normal', 'visual'
        this.navigableItems = [];
        this.currentItemIndex = 0;
        this.cursorElement = document.getElementById('vim-cursor');
        this.selectionStartIndex = -1;
    }

    init() {
        // ... existing init() content
        this.collectNavigableItems();
        this.bindVimKeys();
        this.updateCursorPosition();
    }
    
    collectNavigableItems() {
        // Find all potential elements to navigate to
        this.navigableItems = Array.from(
            document.querySelectorAll('#about p, #about a, #blog a, #books a, #awesome-links a')
        );
        // Set the first item as active
        if (this.navigableItems.length > 0) {
            this.navigableItems[0].classList.add('vim-active-element');
        }
    }

    updateCursorPosition() {
        // TODO: Implement the logic to move the cursor element.
        // 1. Get the current active element from the `this.navigableItems` array using `this.currentItemIndex`.
        // 2. Get its position and dimensions using `getBoundingClientRect()`.
        // 3. Update the `style.left`, `style.top`, and `style.height` of `this.cursorElement`.
        //    (Hint: Remember to account for `window.scrollX` and `window.scrollY`).
    }
    
    moveCursor(direction) {
        // TODO: Implement the logic to change the current item.
        // 1. Store the old index.
        // 2. Calculate the new index based on the `direction` (+1 or -1).
        // 3. Make sure the new index is not out of bounds (it can't be less than 0 or greater than the array length - 1).
        // 4. Update `this.currentItemIndex`.
        // 5. Remove the 'vim-active-element' class from the old element.
        // 6. Add the 'vim-active-element' class to the new element.
        // 7. Scroll the new element into view.
        // 8. Call `this.updateCursorPosition()` to move the visual cursor.
        // 9. If the mode is 'visual', call `this.applyVisualSelection()`.
    }
    
    applyVisualSelection() {
        // TODO: Highlight the text between the start of the selection and the current cursor position.
        // 1. Clear any existing selection by removing the 'vim-visual-selection' class from all elements.
        // 2. Determine the start and end points of the selection using `this.selectionStartIndex` and `this.currentItemIndex`.
        // 3. Loop from the start to the end index.
        // 4. In the loop, add the 'vim-visual-selection' class to each navigable item.
    }

    bindVimKeys() {
        document.addEventListener('keydown', (e) => {
            // This part is done for you. It delegates key presses to the correct handler based on the mode.
            switch (this.vimMode) {
                case 'normal':
                    this.handleNormalModeKeys(e);
                    break;
                case 'visual':
                    this.handleVisualModeKeys(e);
                    break;
            }
        });
    }

    handleNormalModeKeys(e) {
        // TODO: Handle key presses in Normal mode.
        // Create a switch statement for `e.key`.
        // - case 'j': Move cursor down by calling `this.moveCursor(1)`.
        // - case 'k': Move cursor up by calling `this.moveCursor(-1)`.
        // - case 'Enter': Check if the current item is a link (`<a>` tag). If so, open its `href` in a new tab.
        // - case 'v': Change `this.vimMode` to 'visual', set `this.selectionStartIndex`, and update the status bar.
        // Remember to use `e.preventDefault()` for each case to stop the browser's default behavior.
    }

    handleVisualModeKeys(e) {
        // TODO: Handle key presses in Visual mode.
        // Create a switch statement for `e.key`.
        // - case 'j' & 'k': Move the cursor and update the selection.
        // - case 'y': Call `this.yankSelection()` to copy the text.
        // - case 'Escape': Exit visual mode, clear the selection, and reset the mode to 'normal'.
    }
    
    yankSelection() {
        // TODO: Copy the selected text to the clipboard.
        // 1. Determine the start and end of the selection.
        // 2. Create an empty string to hold the text.
        // 3. Loop through the selected items and append their `textContent` to your string.
        // 4. Use the `navigator.clipboard.writeText()` API to copy the string.
        // 5. On success, show a status message and exit visual mode.
    }
}

// Make sure to instantiate the class: new VimSite();
```

### **Step 4: Integration**

You need to call the new methods from the appropriate places in your existing `VimSite` class.

1.  Add the new properties to the `constructor`.
2.  Call `this.collectNavigableItems();`, `this.bindVimKeys();`, and `this.updateCursorPosition();` from the `init()` method.
3.  Make sure the `VimSite` class is instantiated at the end of the script.
