# Reusable Keyboard Interaction Patterns

Common keyboard-accessible components you can add to your website.

---

## Pattern 1: Accessible Modal Dialog

### HTML Structure

```html
<button id="openModal" class="btn-primary">Open Modal</button>

<div id="modal" class="modal" role="dialog" aria-modal="true" aria-labelledby="modalTitle" hidden>
    <div class="modal-overlay" aria-hidden="true"></div>
    <div class="modal-content">
        <div class="modal-header">
            <h2 id="modalTitle">Modal Title</h2>
            <button id="closeModal" class="modal-close" aria-label="Close modal">
                &times;
            </button>
        </div>
        <div class="modal-body">
            <p>Modal content goes here.</p>
            <button class="btn-primary">Confirm</button>
            <button class="btn-secondary">Cancel</button>
        </div>
    </div>
</div>
```

### CSS

```css
.modal {
    position: fixed;
    inset: 0;
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal[hidden] {
    display: none;
}

.modal-overlay {
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 0.75);
}

.modal-content {
    position: relative;
    z-index: 1;
    background: white;
    padding: 30px;
    border-radius: 16px;
    max-width: 600px;
    width: 90%;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.modal-close {
    position: absolute;
    top: 15px;
    right: 15px;
    background: none;
    border: none;
    font-size: 2rem;
    cursor: pointer;
    padding: 5px 10px;
    line-height: 1;
    color: #666;
}

.modal-close:focus-visible {
    outline: 3px solid var(--primary-blue);
    outline-offset: 3px;
}
```

### JavaScript

```javascript
class AccessibleModal {
    constructor(modalId) {
        this.modal = document.getElementById(modalId);
        this.openButton = document.getElementById('openModal');
        this.closeButton = document.getElementById('closeModal');
        this.overlay = this.modal.querySelector('.modal-overlay');
        this.focusableElements = null;
        this.previousFocus = null;

        this.init();
    }

    init() {
        this.openButton.addEventListener('click', () => this.open());
        this.closeButton.addEventListener('click', () => this.close());
        this.overlay.addEventListener('click', () => this.close());
        this.modal.addEventListener('keydown', (e) => this.handleKeyDown(e));
    }

    open() {
        // Save current focus
        this.previousFocus = document.activeElement;

        // Show modal
        this.modal.hidden = false;

        // Get focusable elements
        this.focusableElements = this.modal.querySelectorAll(
            'button:not([disabled]), [href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'
        );

        // Focus first element
        if (this.focusableElements.length > 0) {
            this.focusableElements[0].focus();
        }

        // Prevent body scroll
        document.body.style.overflow = 'hidden';
    }

    close() {
        // Hide modal
        this.modal.hidden = true;

        // Return focus
        if (this.previousFocus) {
            this.previousFocus.focus();
        }

        // Restore body scroll
        document.body.style.overflow = '';
    }

    handleKeyDown(e) {
        // Close on Escape
        if (e.key === 'Escape') {
            this.close();
            return;
        }

        // Trap focus with Tab
        if (e.key === 'Tab') {
            this.trapFocus(e);
        }
    }

    trapFocus(e) {
        const firstElement = this.focusableElements[0];
        const lastElement = this.focusableElements[this.focusableElements.length - 1];

        if (e.shiftKey) {
            // Shift + Tab
            if (document.activeElement === firstElement) {
                e.preventDefault();
                lastElement.focus();
            }
        } else {
            // Tab
            if (document.activeElement === lastElement) {
                e.preventDefault();
                firstElement.focus();
            }
        }
    }
}

// Initialize
const modal = new AccessibleModal('modal');
```

---

## Pattern 2: Dropdown Menu

### HTML Structure

```html
<div class="dropdown">
    <button
        id="menuButton"
        class="dropdown-toggle"
        aria-haspopup="true"
        aria-expanded="false"
        aria-controls="menuList">
        Menu
    </button>
    <ul
        id="menuList"
        class="dropdown-menu"
        role="menu"
        hidden>
        <li role="none">
            <a href="#" role="menuitem">Option 1</a>
        </li>
        <li role="none">
            <a href="#" role="menuitem">Option 2</a>
        </li>
        <li role="none">
            <a href="#" role="menuitem">Option 3</a>
        </li>
    </ul>
</div>
```

### CSS

```css
.dropdown {
    position: relative;
    display: inline-block;
}

.dropdown-toggle {
    padding: 12px 24px;
    background: var(--primary-blue);
    color: white;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-weight: 600;
}

.dropdown-toggle:focus-visible {
    outline: 3px solid var(--primary-blue);
    outline-offset: 3px;
}

.dropdown-menu {
    position: absolute;
    top: 100%;
    left: 0;
    margin-top: 8px;
    list-style: none;
    padding: 8px 0;
    background: white;
    border-radius: 8px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
    min-width: 200px;
    z-index: 100;
}

.dropdown-menu[hidden] {
    display: none;
}

.dropdown-menu a {
    display: block;
    padding: 12px 20px;
    color: #333;
    text-decoration: none;
    transition: background 0.2s;
}

.dropdown-menu a:hover,
.dropdown-menu a:focus {
    background: #f5f5f5;
}

.dropdown-menu a:focus-visible {
    outline: 3px solid var(--primary-blue);
    outline-offset: -3px;
}
```

### JavaScript

```javascript
class AccessibleDropdown {
    constructor(buttonId, menuId) {
        this.button = document.getElementById(buttonId);
        this.menu = document.getElementById(menuId);
        this.menuItems = this.menu.querySelectorAll('[role="menuitem"]');
        this.currentIndex = -1;

        this.init();
    }

    init() {
        this.button.addEventListener('click', () => this.toggle());
        this.button.addEventListener('keydown', (e) => this.handleButtonKeyDown(e));
        this.menu.addEventListener('keydown', (e) => this.handleMenuKeyDown(e));

        // Close on outside click
        document.addEventListener('click', (e) => {
            if (!this.button.contains(e.target) && !this.menu.contains(e.target)) {
                this.close();
            }
        });
    }

    toggle() {
        const isExpanded = this.button.getAttribute('aria-expanded') === 'true';

        if (isExpanded) {
            this.close();
        } else {
            this.open();
        }
    }

    open() {
        this.button.setAttribute('aria-expanded', 'true');
        this.menu.hidden = false;
        this.currentIndex = 0;
        this.menuItems[0].focus();
    }

    close() {
        this.button.setAttribute('aria-expanded', 'false');
        this.menu.hidden = true;
        this.currentIndex = -1;
        this.button.focus();
    }

    handleButtonKeyDown(e) {
        switch (e.key) {
            case 'Enter':
            case ' ':
            case 'ArrowDown':
                e.preventDefault();
                this.open();
                break;
            case 'ArrowUp':
                e.preventDefault();
                this.open();
                this.currentIndex = this.menuItems.length - 1;
                this.menuItems[this.currentIndex].focus();
                break;
        }
    }

    handleMenuKeyDown(e) {
        switch (e.key) {
            case 'Escape':
                e.preventDefault();
                this.close();
                break;
            case 'ArrowDown':
                e.preventDefault();
                this.currentIndex = (this.currentIndex + 1) % this.menuItems.length;
                this.menuItems[this.currentIndex].focus();
                break;
            case 'ArrowUp':
                e.preventDefault();
                this.currentIndex = this.currentIndex <= 0
                    ? this.menuItems.length - 1
                    : this.currentIndex - 1;
                this.menuItems[this.currentIndex].focus();
                break;
            case 'Home':
                e.preventDefault();
                this.currentIndex = 0;
                this.menuItems[0].focus();
                break;
            case 'End':
                e.preventDefault();
                this.currentIndex = this.menuItems.length - 1;
                this.menuItems[this.currentIndex].focus();
                break;
            case 'Tab':
                this.close();
                break;
        }
    }
}

// Initialize
const dropdown = new AccessibleDropdown('menuButton', 'menuList');
```

---

## Pattern 3: Accordion

### HTML Structure

```html
<div class="accordion">
    <div class="accordion-item">
        <h3>
            <button
                class="accordion-button"
                aria-expanded="false"
                aria-controls="panel1">
                <span>Section 1</span>
                <span class="accordion-icon" aria-hidden="true">+</span>
            </button>
        </h3>
        <div
            id="panel1"
            class="accordion-panel"
            role="region"
            hidden>
            <div class="accordion-content">
                <p>Content for section 1.</p>
            </div>
        </div>
    </div>

    <div class="accordion-item">
        <h3>
            <button
                class="accordion-button"
                aria-expanded="false"
                aria-controls="panel2">
                <span>Section 2</span>
                <span class="accordion-icon" aria-hidden="true">+</span>
            </button>
        </h3>
        <div
            id="panel2"
            class="accordion-panel"
            role="region"
            hidden>
            <div class="accordion-content">
                <p>Content for section 2.</p>
            </div>
        </div>
    </div>
</div>
```

### CSS

```css
.accordion-item {
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    margin-bottom: 12px;
    overflow: hidden;
}

.accordion-button {
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px;
    background: white;
    border: none;
    text-align: left;
    font-size: 1.1rem;
    font-weight: 700;
    cursor: pointer;
    transition: background 0.2s;
}

.accordion-button:hover {
    background: #f8f9fa;
}

.accordion-button:focus-visible {
    outline: 3px solid var(--primary-blue);
    outline-offset: -3px;
}

.accordion-button[aria-expanded="true"] .accordion-icon {
    transform: rotate(45deg);
}

.accordion-icon {
    font-size: 1.5rem;
    transition: transform 0.3s;
}

.accordion-panel {
    overflow: hidden;
    transition: max-height 0.3s ease;
}

.accordion-panel[hidden] {
    display: none;
}

.accordion-content {
    padding: 0 20px 20px;
}
```

### JavaScript

```javascript
class AccessibleAccordion {
    constructor(accordionElement) {
        this.accordion = accordionElement;
        this.buttons = this.accordion.querySelectorAll('.accordion-button');
        this.init();
    }

    init() {
        this.buttons.forEach((button, index) => {
            button.addEventListener('click', () => this.toggle(button));
            button.addEventListener('keydown', (e) => this.handleKeyDown(e, index));
        });
    }

    toggle(button) {
        const expanded = button.getAttribute('aria-expanded') === 'true';
        const panel = document.getElementById(button.getAttribute('aria-controls'));

        button.setAttribute('aria-expanded', !expanded);
        panel.hidden = expanded;
    }

    handleKeyDown(e, currentIndex) {
        let newIndex;

        switch (e.key) {
            case 'ArrowDown':
                e.preventDefault();
                newIndex = (currentIndex + 1) % this.buttons.length;
                this.buttons[newIndex].focus();
                break;
            case 'ArrowUp':
                e.preventDefault();
                newIndex = currentIndex === 0
                    ? this.buttons.length - 1
                    : currentIndex - 1;
                this.buttons[newIndex].focus();
                break;
            case 'Home':
                e.preventDefault();
                this.buttons[0].focus();
                break;
            case 'End':
                e.preventDefault();
                this.buttons[this.buttons.length - 1].focus();
                break;
        }
    }
}

// Initialize all accordions
document.querySelectorAll('.accordion').forEach(accordion => {
    new AccessibleAccordion(accordion);
});
```

---

## Pattern 4: Tabs

### HTML Structure

```html
<div class="tabs">
    <div role="tablist" aria-label="Content tabs">
        <button
            role="tab"
            aria-selected="true"
            aria-controls="tab1-panel"
            id="tab1"
            class="tab-button">
            Tab 1
        </button>
        <button
            role="tab"
            aria-selected="false"
            aria-controls="tab2-panel"
            id="tab2"
            class="tab-button"
            tabindex="-1">
            Tab 2
        </button>
        <button
            role="tab"
            aria-selected="false"
            aria-controls="tab3-panel"
            id="tab3"
            class="tab-button"
            tabindex="-1">
            Tab 3
        </button>
    </div>

    <div
        role="tabpanel"
        id="tab1-panel"
        aria-labelledby="tab1"
        class="tab-panel">
        <h3>Tab 1 Content</h3>
        <p>Content for tab 1.</p>
    </div>

    <div
        role="tabpanel"
        id="tab2-panel"
        aria-labelledby="tab2"
        class="tab-panel"
        hidden>
        <h3>Tab 2 Content</h3>
        <p>Content for tab 2.</p>
    </div>

    <div
        role="tabpanel"
        id="tab3-panel"
        aria-labelledby="tab3"
        class="tab-panel"
        hidden>
        <h3>Tab 3 Content</h3>
        <p>Content for tab 3.</p>
    </div>
</div>
```

### CSS

```css
[role="tablist"] {
    display: flex;
    gap: 8px;
    border-bottom: 2px solid #e0e0e0;
    padding: 0;
    margin: 0 0 24px 0;
}

.tab-button {
    padding: 12px 24px;
    background: none;
    border: none;
    border-bottom: 3px solid transparent;
    cursor: pointer;
    font-size: 1rem;
    font-weight: 600;
    color: #666;
    transition: all 0.2s;
}

.tab-button:hover {
    color: var(--primary-blue);
}

.tab-button[aria-selected="true"] {
    color: var(--primary-blue);
    border-bottom-color: var(--primary-blue);
}

.tab-button:focus-visible {
    outline: 3px solid var(--primary-blue);
    outline-offset: 3px;
}

.tab-panel {
    padding: 24px;
    background: white;
    border-radius: 8px;
    border: 1px solid #e0e0e0;
}

.tab-panel[hidden] {
    display: none;
}
```

### JavaScript

```javascript
class AccessibleTabs {
    constructor(tabsElement) {
        this.tabs = tabsElement;
        this.tabButtons = this.tabs.querySelectorAll('[role="tab"]');
        this.tabPanels = this.tabs.querySelectorAll('[role="tabpanel"]');
        this.currentIndex = 0;

        this.init();
    }

    init() {
        this.tabButtons.forEach((button, index) => {
            button.addEventListener('click', () => this.activateTab(index));
            button.addEventListener('keydown', (e) => this.handleKeyDown(e, index));
        });
    }

    activateTab(index) {
        // Deactivate all tabs
        this.tabButtons.forEach((button, i) => {
            const isActive = i === index;
            button.setAttribute('aria-selected', isActive);
            button.tabIndex = isActive ? 0 : -1;
            this.tabPanels[i].hidden = !isActive;
        });

        this.currentIndex = index;
        this.tabButtons[index].focus();
    }

    handleKeyDown(e, currentIndex) {
        let newIndex;

        switch (e.key) {
            case 'ArrowRight':
                e.preventDefault();
                newIndex = (currentIndex + 1) % this.tabButtons.length;
                this.activateTab(newIndex);
                break;
            case 'ArrowLeft':
                e.preventDefault();
                newIndex = currentIndex === 0
                    ? this.tabButtons.length - 1
                    : currentIndex - 1;
                this.activateTab(newIndex);
                break;
            case 'Home':
                e.preventDefault();
                this.activateTab(0);
                break;
            case 'End':
                e.preventDefault();
                this.activateTab(this.tabButtons.length - 1);
                break;
        }
    }
}

// Initialize all tab components
document.querySelectorAll('.tabs').forEach(tabs => {
    new AccessibleTabs(tabs);
});
```

---

## Pattern 5: Custom Select/Combobox

### HTML Structure

```html
<div class="combobox">
    <label id="combobox-label" for="combobox-input">Choose an option:</label>
    <div class="combobox-wrapper">
        <input
            type="text"
            id="combobox-input"
            role="combobox"
            aria-labelledby="combobox-label"
            aria-expanded="false"
            aria-controls="listbox"
            aria-autocomplete="list"
            autocomplete="off">
        <ul
            id="listbox"
            role="listbox"
            aria-labelledby="combobox-label"
            hidden>
            <li role="option" data-value="option1">Option 1</li>
            <li role="option" data-value="option2">Option 2</li>
            <li role="option" data-value="option3">Option 3</li>
            <li role="option" data-value="option4">Option 4</li>
        </ul>
    </div>
</div>
```

### CSS

```css
.combobox {
    margin: 20px 0;
}

.combobox label {
    display: block;
    margin-bottom: 8px;
    font-weight: 600;
}

.combobox-wrapper {
    position: relative;
}

#combobox-input {
    width: 100%;
    padding: 12px 16px;
    font-size: 1rem;
    border: 2px solid #ddd;
    border-radius: 8px;
    transition: border-color 0.2s;
}

#combobox-input:focus {
    outline: none;
    border-color: var(--primary-blue);
}

#listbox {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    margin-top: 4px;
    padding: 8px 0;
    background: white;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
    list-style: none;
    max-height: 200px;
    overflow-y: auto;
    z-index: 100;
}

#listbox[hidden] {
    display: none;
}

#listbox li {
    padding: 10px 16px;
    cursor: pointer;
}

#listbox li:hover,
#listbox li[aria-selected="true"] {
    background: #f0f7ff;
    color: var(--primary-blue);
}
```

### JavaScript

```javascript
class AccessibleCombobox {
    constructor(inputId, listboxId) {
        this.input = document.getElementById(inputId);
        this.listbox = document.getElementById(listboxId);
        this.options = this.listbox.querySelectorAll('[role="option"]');
        this.currentIndex = -1;

        this.init();
    }

    init() {
        this.input.addEventListener('input', () => this.filter());
        this.input.addEventListener('keydown', (e) => this.handleKeyDown(e));
        this.input.addEventListener('focus', () => this.open());

        this.options.forEach((option, index) => {
            option.addEventListener('click', () => this.select(index));
        });

        // Close on outside click
        document.addEventListener('click', (e) => {
            if (!this.input.contains(e.target) && !this.listbox.contains(e.target)) {
                this.close();
            }
        });
    }

    filter() {
        const query = this.input.value.toLowerCase();
        let visibleCount = 0;

        this.options.forEach(option => {
            const text = option.textContent.toLowerCase();
            const matches = text.includes(query);
            option.hidden = !matches;
            if (matches) visibleCount++;
        });

        if (visibleCount > 0) {
            this.open();
        } else {
            this.close();
        }
    }

    open() {
        this.input.setAttribute('aria-expanded', 'true');
        this.listbox.hidden = false;
    }

    close() {
        this.input.setAttribute('aria-expanded', 'false');
        this.listbox.hidden = true;
        this.currentIndex = -1;
    }

    select(index) {
        const option = this.options[index];
        this.input.value = option.textContent;
        this.close();
        this.input.focus();
    }

    handleKeyDown(e) {
        const visibleOptions = Array.from(this.options).filter(opt => !opt.hidden);

        switch (e.key) {
            case 'ArrowDown':
                e.preventDefault();
                if (!this.listbox.hidden) {
                    this.currentIndex = Math.min(this.currentIndex + 1, visibleOptions.length - 1);
                    this.highlightOption(visibleOptions[this.currentIndex]);
                } else {
                    this.open();
                }
                break;
            case 'ArrowUp':
                e.preventDefault();
                this.currentIndex = Math.max(this.currentIndex - 1, 0);
                this.highlightOption(visibleOptions[this.currentIndex]);
                break;
            case 'Enter':
                e.preventDefault();
                if (this.currentIndex >= 0) {
                    this.select(Array.from(this.options).indexOf(visibleOptions[this.currentIndex]));
                }
                break;
            case 'Escape':
                e.preventDefault();
                this.close();
                break;
        }
    }

    highlightOption(option) {
        this.options.forEach(opt => opt.removeAttribute('aria-selected'));
        if (option) {
            option.setAttribute('aria-selected', 'true');
            option.scrollIntoView({ block: 'nearest' });
        }
    }
}

// Initialize
const combobox = new AccessibleCombobox('combobox-input', 'listbox');
```

---

## Best Practices Summary

### For All Patterns:

1. **Focus Management**
   - Always return focus after closing
   - Trap focus in modals
   - Make first element focusable

2. **Keyboard Support**
   - `Escape` closes overlays
   - `Tab` navigates between components
   - Arrow keys navigate within components
   - `Enter`/`Space` activates

3. **ARIA Attributes**
   - Use appropriate roles
   - Update `aria-expanded` dynamically
   - Set `aria-controls` relationships
   - Provide labels with `aria-label` or `aria-labelledby`

4. **Visual Feedback**
   - Clear focus indicators
   - Hover states
   - Active/selected states

5. **Screen Reader Support**
   - Announce state changes
   - Provide context with ARIA labels
   - Use semantic HTML where possible

---

## Testing Each Pattern

For each component:

1. ✅ Tab to component
2. ✅ Use all keyboard shortcuts
3. ✅ Verify focus is visible
4. ✅ Test with screen reader
5. ✅ Check ARIA attributes in DevTools
6. ✅ Verify focus is trapped (if applicable)
7. ✅ Ensure focus returns correctly

---

## Additional Resources

- [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [Inclusive Components](https://inclusive-components.design/)
- [A11y Style Guide](https://a11y-style-guide.com/)
