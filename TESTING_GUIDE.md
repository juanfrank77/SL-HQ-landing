# Keyboard Accessibility Testing Guide

## Quick Start Testing

### 1. Open the Website
Open `index.html` in your browser:
- Double-click the file, or
- Right-click > Open with > Browser of choice

### 2. Basic Keyboard Navigation Test

#### Step-by-Step Testing:

1. **Refresh the page** (Ctrl/Cmd + R)

2. **Press Tab once**
   - ✅ You should see the "Skip to main content" link appear at the top-left
   - ✅ It should have a white outline

3. **Press Tab again**
   - ✅ Focus moves to "Join the Mission" button
   - ✅ Button should have white outline visible against dark background
   - ✅ Button should lift up slightly (hover effect)

4. **Press Tab again**
   - ✅ Focus moves to "Explore Intel" button
   - ✅ Button should have white outline

5. **Continue pressing Tab**
   - ✅ Tab order should be logical (top to bottom)
   - ✅ All interactive elements get focused
   - ✅ Focus indicators are always visible

6. **Press Shift + Tab**
   - ✅ Focus moves backward through elements
   - ✅ Can navigate back to skip link

7. **Test Skip Link**
   - Refresh page
   - Press Tab once (skip link appears)
   - Press Enter
   - ✅ Page should scroll to "You're Capable of More" section

8. **Test Link Activation**
   - Tab to any button
   - Press Enter
   - ✅ Link should activate (note: demo links may not work)

### 3. Visual Focus Indicator Test

Open browser DevTools and check:

```javascript
// Run in Console to highlight all focusable elements
document.querySelectorAll('a, button, [tabindex]').forEach(el => {
    el.style.border = '2px solid red';
});
```

**Expected focusable elements:**
1. Skip navigation link
2. "Join the Mission" button (hero)
3. "Explore Intel" button (hero)
4. "Subscribe Now" button (CTA)
5. "Learn More" button (CTA)

**Total: 5 interactive elements**

### 4. Contrast Ratio Test

Use browser DevTools or online tools:

**Focus Indicators to Test:**
- Blue outline on white background: Should be 3:1 or higher
- White outline on dark blue background: Should be 3:1 or higher

**Tool**: Use Chrome DevTools Accessibility pane:
1. Right-click element
2. Inspect
3. Check "Accessibility" tab
4. Look for contrast ratio

### 5. Screen Reader Test (Optional)

#### Windows (NVDA - Free)

1. Download [NVDA](https://www.nvaccess.org/download/)
2. Install and launch
3. Open website
4. Press `H` to navigate by headings
5. Press `L` to navigate by links
6. Press `D` to navigate by landmarks

**What you should hear:**
- "Skip to main content, link"
- "Main region" when entering main content
- All heading text announced
- "Content info" when reaching footer

#### macOS (VoiceOver - Built-in)

1. Press `Cmd + F5` to enable VoiceOver
2. Open website
3. Press `Control + Option + →` to navigate forward
4. Press `Control + Option + ←` to navigate backward

**What you should hear:**
- All text content
- "Link" announced for each button
- "Main content" when entering main section

---

## Advanced Testing

### Test Focus Visible vs Focus

Modern browsers support `:focus-visible` which only shows outlines for keyboard users.

**Test procedure:**

1. **Click** a button with mouse
   - ✅ Should NOT show outline (`:focus-visible` working)

2. **Tab** to a button with keyboard
   - ✅ Should show outline (`:focus-visible` working)

If outlines show on click, check browser support:
- Chrome/Edge 86+
- Firefox 85+
- Safari 15.4+

### Test on Different Backgrounds

The site has multiple background colors. Test focus visibility on each:

1. **Hero section** (dark blue gradient)
   - Focus indicator: White
   - Should be clearly visible

2. **Problem section** (light gray)
   - Focus indicator: Blue
   - Should be clearly visible

3. **Solution section** (dark blue)
   - Focus indicator: N/A (no interactive elements)

4. **CTA section** (dark blue)
   - Focus indicator: White
   - Should be clearly visible

### Test Tab Trapping

Ensure focus never gets trapped:

1. Tab through entire page
2. Should reach the last focusable element
3. Press Tab once more
4. ✅ Focus should return to browser UI (address bar)
5. Never trapped in page

### Test Skip Link Behavior

1. **Refresh page**
2. **Don't press Tab** - Link should be hidden
3. **Press Tab** - Link should appear with smooth transition
4. **Press Tab again** - Link should hide again
5. **Shift+Tab back** - Link should reappear

---

## Browser Compatibility Testing

Test on multiple browsers:

### Chrome/Edge (Chromium)
- [x] Focus indicators visible
- [x] Skip link works
- [x] Tab order correct
- [x] :focus-visible supported

### Firefox
- [x] Focus indicators visible
- [x] Skip link works
- [x] Tab order correct
- [x] :focus-visible supported

### Safari
- [x] Focus indicators visible
- [x] Skip link works
- [x] Tab order correct
- [x] :focus-visible supported (15.4+)

### Mobile Testing

**iOS Safari:**
- Connect keyboard via Bluetooth
- Test tab navigation
- Verify focus indicators

**Android Chrome:**
- Connect keyboard via Bluetooth/OTG
- Test tab navigation
- Verify focus indicators

---

## Automated Testing

### Using Browser Extensions

#### 1. axe DevTools (Recommended)

1. Install [axe DevTools](https://chrome.google.com/webstore/detail/axe-devtools-web-accessib/lhdoppojpmngadmnindnejefpokejbdd)
2. Open DevTools (F12)
3. Go to "axe DevTools" tab
4. Click "Scan ALL of my page"
5. Review results

**Expected: 0 violations for keyboard accessibility**

#### 2. WAVE

1. Install [WAVE](https://wave.webaim.org/extension/)
2. Click WAVE icon in toolbar
3. Review "Structure" tab
4. Check for landmarks, headings, links

**Expected:**
- Skip navigation link present
- Proper heading structure
- All links have accessible names

#### 3. Lighthouse

1. Open Chrome DevTools (F12)
2. Go to "Lighthouse" tab
3. Select "Accessibility"
4. Click "Analyze page load"

**Expected Score: 95-100**

---

## Regression Testing Checklist

Use this checklist when making future changes:

### Before Deploying:
- [ ] All buttons/links have focus indicators
- [ ] Focus indicators visible on all backgrounds
- [ ] Skip link appears on Tab
- [ ] Tab order is logical
- [ ] No keyboard traps
- [ ] Links activate with Enter
- [ ] Screen reader announces all content
- [ ] ARIA landmarks present
- [ ] Automated tests pass
- [ ] Manual keyboard test completed

---

## Common Issues & Solutions

### Issue: Focus indicators not visible

**Cause**: CSS overriding focus styles
**Solution**: Check for `outline: none` without replacement

```css
/* Bad */
button:focus {
    outline: none;
}

/* Good */
button:focus-visible {
    outline: 3px solid blue;
}
```

### Issue: Skip link not working

**Cause**: Target element not focusable
**Solution**: Add `tabindex="-1"` to target

```html
<section id="main-content" tabindex="-1">
```

### Issue: Tab order illogical

**Cause**: CSS positioning or flexbox order
**Solution**: Ensure DOM order matches visual order

### Issue: Focus trapped

**Cause**: JavaScript preventing default Tab behavior
**Solution**: Review event listeners, ensure Tab works

---

## Performance Impact

These accessibility changes have **zero performance impact**:
- CSS-only implementations
- No JavaScript required for basic navigation
- No additional HTTP requests
- Minimal CSS addition (~100 lines)

---

## Next Steps

After testing:

1. ✅ Verify all tests pass
2. ✅ Document any browser-specific issues
3. ✅ Test with real users if possible
4. ✅ Monitor for regressions
5. ✅ Consider adding more interactive elements (if needed)

## Questions?

Refer to:
- `KEYBOARD_ACCESSIBILITY.md` - Full implementation details
- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [WebAIM Keyboard Testing](https://webaim.org/articles/keyboard/)
