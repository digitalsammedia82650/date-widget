/* date-widget.js
   Digital Sam Media - Widget engine (put this file in your GitHub repo)
   - Renders into an element with id="ds-date-widget"
   - Uses Shadow DOM so CSS is scoped and widget internals are not visible
   - Shows Date, Time, Page Load Time, and site credit
*/

(function () {
  'use strict';

  const HUNTER_INTERVAL = 50;   // ms to poll for container
  const HUNTER_TIMEOUT = 10000; // stop hunting after 10s
  const CREDIT_URL = 'https://www.digitalsammedia.com'; // Define credit URL once

  function safe(fn) {
    try { fn(); } catch (e) { console.error('DSM Widget error:', e); }
  }

  function computeLoadMs() {
    try {
      const navEntries = (performance.getEntriesByType && performance.getEntriesByType('navigation')) || [];
      const nav = navEntries[0] || null;

      if (nav && typeof nav.loadEventEnd === 'number') {
        if (nav.loadEventEnd > 0) return Math.round(nav.loadEventEnd - (nav.startTime || 0));
        return Math.round(performance.now());
      }
      if (performance.timing && performance.timing.navigationStart) {
        const t = performance.timing;
        if (t.loadEventEnd && t.loadEventEnd > 0) {
          return Math.round(t.loadEventEnd - t.navigationStart);
        }
      }
      return Math.round(performance.now());
    } catch (e) {
      return null;
    }
  }

  function formatLoad(ms) {
    if (ms === null || ms === undefined || Number.isNaN(ms)) return '—';
    if (ms < 1000) return ms + ' ms';
    return (ms / 1000).toFixed(2) + ' s';
  }
  
  function formatTime(d) {
    try {
      return d.toLocaleTimeString(undefined, {
        hour: 'numeric',
        minute: '2-digit'
      });
    } catch (e) {
      return d.toTimeString().substring(0, 5);
    }
  }

  function formatDate(d) {
    try {
      return d.toLocaleDateString(undefined, {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    } catch (e) {
      return d.toDateString();
    }
  }

  function createWidgetInside(container) {
    // Avoid double-init
    if (container.__dsm_widget_inited) return;
    container.__dsm_widget_inited = true;

    // Attach shadow to avoid CSS leaks and protect credit visibility
    const shadow = container.attachShadow ? container.attachShadow({ mode: 'open' }) : container;

    // styles inside shadow
    const style = document.createElement('style');
    style.textContent = `
      :host { display:inline-block; font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
      .dsm-root {
        background: #fff;
        color: #111;
        padding: 16px;
        border-radius: 12px;
        box-shadow: 0 6px 20px rgba(0,0,0,0.12);
        border-left: 5px solid #2563eb;
        min-width: 240px;
      }
      .dsm-time { font-weight:800; font-size: 1.8rem; line-height:1; margin-bottom:8px; }
      .dsm-date { font-weight:700; font-size: 0.95rem; margin-bottom:12px; opacity: 0.9; }
      .dsm-load-label { font-size:0.75rem; opacity:0.7; }
      .dsm-load { font-size:0.9rem; font-weight:600; line-height:1; }
      .dsm-credit { font-size:0.78rem; opacity:0.85; margin-top:12px; border-top: 1px solid rgba(0,0,0,0.08); padding-top: 8px; }
      .dsm-credit a { color: inherit; text-decoration:none; font-weight:700; }
    `;

    // root markup
    const root = document.createElement('div');
    root.className = 'dsm-root';
    root.innerHTML = `
      <div class="dsm-time" id="dsm-time">Loading time…</div>
      <div class="dsm-date" id="dsm-date">Loading date…</div>
      <div class="dsm-load-label">Page Load Time:</div>
      <div class="dsm-load" id="dsm-load">—</div>
      <div class="dsm-credit">
        Widget by <a id="dsm-credit-link" href="${CREDIT_URL}" target="_blank" rel="noopener">${CREDIT_URL.replace('https://www.', '')}</a>
      </div>
    `;

    // append
    shadow.appendChild(style);
    shadow.appendChild(root);

    // Get element references from the shadow root
    const timeEl = shadow.getElementById ? shadow.getElementById('dsm-time') : root.querySelector('#dsm-time');
    const dateEl = shadow.getElementById ? shadow.getElementById('dsm-date') : root.querySelector('#dsm-date');
    const loadEl = shadow.getElementById ? shadow.getElementById('dsm-load') : root.querySelector('#dsm-load');

    // Render values
    let initialLoadTimeRendered = false;
    function render() {
      const now = new Date();
      timeEl.textContent = formatTime(now);
      dateEl.textContent = formatDate(now);

      if (!initialLoadTimeRendered) {
        const ms = computeLoadMs();
        loadEl.textContent = formatLoad(ms);
        initialLoadTimeRendered = true;
      }
    }

    // initial render
    setTimeout(render, 50);

    // Update clock and date once per minute (60s * 1000ms)
    setInterval(() => {
      safe(render);
    }, 60 * 1000);

    // 🚩 CREDIT PROTECTION: Re-fetch the credit link element inside the interval
    setInterval(() => {
      try {
        const creditParent = root.querySelector('.dsm-credit');
        const creditLink = creditParent ? creditParent.querySelector('#dsm-credit-link') : null;

        // Check if link is missing or has wrong URL
        if (!creditLink || creditLink.getAttribute('href') !== CREDIT_URL) {
          if (creditParent) {
            creditParent.innerHTML = `Widget by <a id="dsm-credit-link" href="${CREDIT_URL}" target="_blank" rel="noopener">${CREDIT_URL.replace('https://www.', '')}</a>`;
          }
        }
      } catch (e) { /* ignore */ }
    }, 4000);
  }

  // Hunter: wait for element with id ds-date-widget (max 10s)
  (function hunt() {
    const start = Date.now();
    const interval = setInterval(() => {
      const container = document.getElementById('ds-date-widget');
      if (container) {
        clearInterval(interval);
        safe(() => createWidgetInside(container));
        return;
      }
      if (Date.now() - start > HUNTER_TIMEOUT) {
        clearInterval(interval);
      }
    }, HUNTER_INTERVAL);
  })();

})();
