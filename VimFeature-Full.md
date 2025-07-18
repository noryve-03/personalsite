# Implementing Vim-like Navigation in JavaScript (Full Guide)

This guide provides a complete, step-by-step implementation for adding Vim-like navigation features to your website.

---

### **Step 1: Create the Cursor Element (HTML)**

First, we need an element that will represent our cursor. We'll add this inside the `<body>` tag. It will be positioned with JavaScript.

```html
<!-- Add this right after the opening <body> tag -->
<div id="vim-cursor"></div>
```

---

### **Step 2: Style the Cursor and Active Elements (CSS)**

Next, we'll style the cursor itself and create a class to highlight the element the cursor is currently on.

```css
/* Add this to your existing <style> block */

#vim-cursor {
    position: absolute;
    background-color: #00ff00; /* A bright, visible color */
    width: 10px; /* Block cursor width */
    height: 1.2rem; /* Match line-height */
    opacity: 0.7;
    z-index: 9999;
    pointer-events: none; /* So it doesn't interfere with mouse clicks */
    transition: top 0.05s, left 0.05s; /* Smooth movement */
}

/* A class to highlight the currently "active" navigable element */
.vim-active-element {
    background-color: rgba(0, 255, 255, 0.15);
    outline: 1px solid rgba(0, 255, 255, 0.5);
}

/* Styling for the visual selection */
.vim-visual-selection {
    background-color: rgba(70, 70, 120, 0.7);
    color: #fff;
}
```

---

### **Step 3: The Core Logic (JavaScript)**

This is where the main functionality lives. We'll manage state, handle key presses, move the cursor, and implement visual mode.

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
        if (this.currentItemIndex < 0 || this.currentItemIndex >= this.navigableItems.length) return;
        
        const currentElement = this.navigableItems[this.currentItemIndex];
        const rect = currentElement.getBoundingClientRect();
        
        this.cursorElement.style.left = `${rect.left + window.scrollX}px`;
        this.cursorElement.style.top = `${rect.top + window.scrollY}px`;
        this.cursorElement.style.height = `${rect.height}px`;
    }
    
    moveCursor(direction) {
        const oldIndex = this.currentItemIndex;
        let newIndex = this.currentItemIndex + direction;

        // Clamp the index to be within bounds
        if (newIndex < 0) newIndex = 0;
        if (newIndex >= this.navigableItems.length) newIndex = this.navigableItems.length - 1;

        this.currentItemIndex = newIndex;

        // Update active element class
        this.navigableItems[oldIndex].classList.remove('vim-active-element');
        const newElement = this.navigableItems[this.currentItemIndex];
        newElement.classList.add('vim-active-element');
        newElement.scrollIntoView({ behavior: 'smooth', block: 'center' });

        this.updateCursorPosition();
        
        if (this.vimMode === 'visual') {
            this.applyVisualSelection();
        }
    }
    
    applyVisualSelection() {
        // Clear previous selection
        document.querySelectorAll('.vim-visual-selection').forEach(el => {
            el.classList.remove('vim-visual-selection');
        });

        if (this.selectionStartIndex === -1) return;

        const start = Math.min(this.selectionStartIndex, this.currentItemIndex);
        const end = Math.max(this.selectionStartIndex, this.currentItemIndex);

        for (let i = start; i <= end; i++) {
            this.navigableItems[i].classList.add('vim-visual-selection');
        }
    }

    bindVimKeys() {
        document.addEventListener('keydown', (e) => {
            // We'll add the logic for h,j,k,l,v,y,enter here
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
        switch (e.key) {
            case 'j': // Move down
                e.preventDefault();
                this.moveCursor(1);
                break;
            case 'k': // Move up
                e.preventDefault();
                this.moveCursor(-1);
                break;
            case 'Enter': // Activate link
                e.preventDefault();
                const currentElement = this.navigableItems[this.currentItemIndex];
                if (currentElement.tagName === 'A' && currentElement.href) {
                    window.open(currentElement.href, '_blank');
                }
                break;
            case 'v': // Enter Visual Mode
                e.preventDefault();
                this.vimMode = 'visual';
                this.selectionStartIndex = this.currentItemIndex;
                this.applyVisualSelection();
                this.showStatus('VISUAL MODE');
                break;
        }
    }

    handleVisualModeKeys(e) {
        switch (e.key) {
            case 'j':
            case 'k':
                e.preventDefault();
                this.moveCursor(e.key === 'j' ? 1 : -1);
                break;
            case 'y': // Yank (Copy)
                e.preventDefault();
                this.yankSelection();
                break;
            case 'Escape':
                e.preventDefault();
                this.vimMode = 'normal';
                this.selectionStartIndex = -1;
                document.querySelectorAll('.vim-visual-selection').forEach(el => {
                    el.classList.remove('vim-visual-selection');
                });
                this.showStatus('NORMAL MODE');
                break;
        }
    }
    
    yankSelection() {
        const start = Math.min(this.selectionStartIndex, this.currentItemIndex);
        const end = Math.max(this.selectionStartIndex, this.currentItemIndex);
        
        let selectedText = '';
        for (let i = start; i <= end; i++) {
            selectedText += this.navigableItems[i].textContent.trim() + '\n';
        }

        navigator.clipboard.writeText(selectedText.trim()).then(() => {
            this.showStatus('Yanked to clipboard!');
            // Exit visual mode after yanking
            document.dispatchEvent(new KeyboardEvent('keydown', {'key': 'Escape'}));
        }).catch(err => {
            this.showStatus('Failed to yank.');
            console.error('Clipboard error:', err);
        });
    }
}

// Make sure to instantiate the class: new VimSite();
```

### **Step 4: Integration**

You need to call the new methods from the appropriate places in your existing `VimSite` class.

1.  Add the new properties to the `constructor`.
2.  Call `this.collectNavigableItems();`, `this.bindVimKeys();`, and `this.updateCursorPosition();` from the `init()` method.
3.  Make sure the `VimSite` class is instantiated at the end of the script.

This provides a solid foundation for Vim-like navigation on your site.
