# Random-Case-Reader
Randomizes letter case of page text.

How to install:

1. Add as a new script in your Tampermonkey Browser Extension.

2. Toggle to activate it in the Tampermoney menu.

3. Visit any Website.

```
// ==UserScript==
// @name         Random Case Reader
// @namespace    local.randomcase
// @version      1.0.0
// @description  Randomizes letter case of page text.
// @match        *://*/*
// @run-at       document-idle
// @grant        GM_registerMenuCommand
// ==/UserScript==

(() => {
  'use strict';

  // --- settings ---
  let enabled = true;

  const SKIP_TAGS = new Set(['SCRIPT', 'STYLE', 'NOSCRIPT', 'TEXTAREA', 'INPUT', 'SELECT', 'OPTION']);

  function isSkippableTextNode(textNode) {
    const p = textNode.parentNode;
    if (!p) return true;
    if (p.nodeType !== Node.ELEMENT_NODE) return false;
    const tag = p.nodeName;
    if (SKIP_TAGS.has(tag)) return true;
    if (p.isContentEditable) return true;
    return false;
  }

  function randomCase(str) {
    // Change only letters; keep punctuation/whitespace as-is
    return Array.from(str, ch => {
      if (!/\p{L}/u.test(ch)) return ch;
      return Math.random() < 0.5 ? ch.toLowerCase() : ch.toUpperCase();
    }).join('');
  }

  function processTextNode(n) {
    if (!enabled) return;
    if (!n || n.nodeType !== Node.TEXT_NODE) return;
    if (!n.nodeValue || !n.nodeValue.trim()) return;
    if (isSkippableTextNode(n)) return;

    // Avoid re-processing already processed nodes
    if (n.__randomCaseDone) return;

    n.nodeValue = randomCase(n.nodeValue);
    n.__randomCaseDone = true;
  }

  function processSubtree(root) {
    if (!enabled) return;
    if (!root) return;

    if (root.nodeType === Node.TEXT_NODE) {
      processTextNode(root);
      return;
    }

    const walker = document.createTreeWalker(root, NodeFilter.SHOW_TEXT);
    let node;
    while ((node = walker.nextNode())) processTextNode(node);
  }

  // Initial run
  if (document.body) processSubtree(document.body);

  // Observe dynamic changes
  const obs = new MutationObserver(mutations => {
    if (!enabled) return;
    for (const m of mutations) {
      for (const added of m.addedNodes) processSubtree(added);
    }
  });
  obs.observe(document.documentElement, { childList: true, subtree: true });

  // Menu toggle
  if (typeof GM_registerMenuCommand === 'function') {
    GM_registerMenuCommand('Toggle Random Case (on/off)', () => {
      enabled = !enabled;
      // If turning on, process whole page once more for any missed nodes.
      if (enabled && document.body) processSubtree(document.body);
      console.log('[RandomCase] enabled =', enabled);
    });
  }
})();

```
