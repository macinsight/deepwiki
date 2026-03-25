# Modding

Modifications can be made using [Tampermonkey](https://www.tampermonkey.net/) by hooking into the games' API. See ##Examples for more details

## Examples

API-driven file-manager and bulk-seller by Rain

```tampermonkey
// ==UserScript==
// @name         DeepNet Data Dealer
// @namespace    https://macinsight.github.io/deepwiki/modding/
// @version      1.1
// @description  File management and bulk selling for DeepNet (API-driven)
// @author       Rain
// @match        https://deepnet.us/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function () {
    'use strict';

    const SETTINGS_KEY = 'dnfs-settings';

    function loadSettings() {
        try {
            const s = JSON.parse(localStorage.getItem(SETTINGS_KEY));
            if (s) {
                if (typeof s.uiScale !== 'number') s.uiScale = 0;
                return s;
            }
        } catch (_) {}
        return { uiScale: 0 };
    }
    function saveSettings() {
        try { localStorage.setItem(SETTINGS_KEY, JSON.stringify(settings)); } catch (_) {}
    }
    let settings = loadSettings();

    function getScale() {
        if (settings.uiScale > 0) return settings.uiScale;
        return Math.max(1.0, Math.min(1.8, 0.55 + window.innerHeight / 1600));
    }

    // ═══════════════════════════════════════════════════════════
    //  CREDENTIAL CAPTURE & API
    // ═══════════════════════════════════════════════════════════
    let machineId = null, token = null;
    const _origFetch = window.fetch;
    function installCredSniffer() {
        window.fetch = function (...args) {
            try {
                const url = typeof args[0] === 'string' ? args[0] : args[0]?.url;
                if ((url?.includes('api.php') || url?.includes('/api?')) && args[1]?.body) {
                    const b = JSON.parse(args[1].body);
                    if (b.machine_id) machineId = b.machine_id;
                    if (b.token) token = b.token;
                    if (machineId && token) {
                        window.fetch = _origFetch;
                    }
                }
            } catch (_) {}
            return _origFetch.apply(this, args);
        };
    }
    installCredSniffer();

    function reqId() {
        const buf = new Uint8Array(16);
        crypto.getRandomValues(buf);
        return [...buf].map(b => b.toString(16).padStart(2, '0')).join('');
    }

    function getApiBase() {
        // Use the game's configured API base if available, otherwise detect from endpoint
        try {
            if (window.CONFIG && window.CONFIG.API_BASE) return window.CONFIG.API_BASE;
            if (window.APP_CONFIG && window.APP_CONFIG.API_BASE) return window.APP_CONFIG.API_BASE;
        } catch (_) {}
        return 'api';
    }

    async function api(action, extra = {}) {
        if (!machineId || !token) return null;
        try {
            const r = await _origFetch(`https://deepnet.us/${getApiBase()}?action=${action}`, {
                method: 'POST', credentials: 'include',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ machine_id: machineId, token, request_id: reqId(), ...extra }),
            });
            return JSON.parse((await r.text()).replace(/^\ufeff/, ''));
        } catch (e) { console.error(`[SELL] API ${action}:`, e); return null; }
    }

    // ═══════════════════════════════════════════════════════════
    //  HELPERS
    // ═══════════════════════════════════════════════════════════
    function isLoggedIn() {
        const p = document.querySelector('#prompt');
        return p && p.textContent.includes('@deepnet') && !p.textContent.includes('guest@deepnet');
    }

    function qualityLabel(q) {
        const n = Number(q || 0);
        if (n >= 5) return { label: 'legendary', color: '#a335ee' };
        if (n >= 4) return { label: 'rare', color: '#0070dd' };
        if (n >= 3) return { label: 'magic', color: '#3ddc84' };
        return { label: 'normal', color: 'var(--dos-text-dim)' };
    }

    function timeLeft(expiresAt) {
        if (!expiresAt) return '';
        const ms = new Date(expiresAt + 'Z').getTime() - Date.now();
        if (ms <= 0) return 'expired';
        const h = Math.floor(ms / 3600000);
        const m = Math.floor((ms % 3600000) / 60000);
        return `${h}h ${m}m`;
    }

    // ═══════════════════════════════════════════════════════════
    //  STATE
    // ═══════════════════════════════════════════════════════════
    let files = [];
    let selected = new Set();
    let selling = false;
    let totalEarned = 0;

    // ═══════════════════════════════════════════════════════════
    //  CORE ACTIONS
    // ═══════════════════════════════════════════════════════════
    async function refreshFiles() {
        const data = await api('get_files');
        if (!data?.success) { logEvent('Failed to fetch files'); return; }
        files = (data.files || []).filter(f => f.sellable && !f.sold && !f.expired);
        files.sort((a, b) => Number(b.value) - Number(a.value));
        // Remove any selected ids that no longer exist
        const ids = new Set(files.map(f => f.id));
        for (const id of selected) { if (!ids.has(id)) selected.delete(id); }
        if (currentView === 'main') renderMain();
    }

    async function sellSelected() {
        if (selling || selected.size === 0) return;
        selling = true;
        const queue = files.filter(f => selected.has(f.id));
        const total = queue.length;
        let earned = 0;
        renderMain();

        for (let i = 0; i < queue.length; i++) {
            const f = queue[i];
            setStatus('working', `Selling ${i + 1}/${total}...`);
            const r = await api('sell_file', { fileId: f.id });
            if (r?.success) {
                const amt = Number(r.earned || 0);
                earned += amt;
                totalEarned += amt;
                logEvent(r.message || `Sold ${f.filename} for ${amt} RCH`);
            } else {
                logEvent(`Failed: ${f.filename} — ${r?.error || 'unknown'}`);
            }
        }

        selected.clear();
        setStatus('ready', `+${earned} RCH`);
        logEvent(`Batch complete: +${earned} RCH (session total: ${totalEarned})`);
        await refreshFiles();
        selling = false;
        renderMain();
    }

    // ═══════════════════════════════════════════════════════════
    //  UI
    // ═══════════════════════════════════════════════════════════
    let panelEl = null, logLines = [], currentView = 'main';

    function logEvent(msg) {
        const ts = new Date().toLocaleTimeString('en-GB', { hour12: false }).slice(0, 5);
        logLines.push({ ts, msg });
        if (logLines.length > 50) logLines.shift();
        renderLog();
    }

    function setStatus(state, text) {
        const el = document.getElementById('dnfs-status');
        if (el) { el.textContent = text || ''; el.className = `dnfs-status dnfs-s-${state}`; }
        const gfx = document.querySelector('.deepos-icon[data-app="dnfs"] .deepos-icon-gfx');
        if (gfx) {
            gfx.style.color = state === 'working' ? 'var(--dos-flag-active)' :
                              state === 'ready' ? '#4a8f4a' :
                              state === 'error' ? '#8f4a4a' : 'var(--dos-icon-color)';
        }
    }

    function injectStyles() {
        const style = document.createElement('style');
        style.textContent = `
            #dnfs-panel {
                display: none; position: fixed; z-index: 6000; width: 480px;
                flex-direction: column;
                font-family: var(--dos-font, Consolas, "Courier New", monospace);
                font-size: var(--dos-font-sm, 12px);
                color: var(--dos-text, #b3b3b3);
                background: var(--dos-bg-window, #0a0a0a);
                border: 1px solid var(--dos-border, #222);
                box-shadow: 4px 4px 12px var(--dos-shadow, rgba(0,0,0,0.6));
                cursor: none; transform-origin: top left;
            }
            #dnfs-panel.visible { display: flex; }
            .dnfs-titlebar {
                display: flex; align-items: center; padding: 0 8px; height: 26px;
                background: var(--dos-tab-bg, #1a1a1a);
                border-bottom: 1px solid var(--dos-border, #222);
                cursor: none; flex-shrink: 0; gap: 6px;
            }
            .dnfs-title {
                font-size: 11px; font-weight: normal; letter-spacing: 0.06em;
                text-transform: uppercase; color: var(--dos-text-label, #a3a3a3);
                flex: 1;
            }
            .dnfs-hdr-btn {
                width: 14px; height: 14px;
                background: var(--dos-border, #222); border: 1px solid var(--dos-border-hi, #333);
                cursor: none; display: flex; align-items: center; justify-content: center;
                font-size: 10px; color: var(--dos-text-dim, #7a7a7a); line-height: 1;
            }
            .dnfs-hdr-btn:hover { background: var(--dos-bg-hover); border-color: var(--dos-border-hi); color: var(--dos-text-hi); }
            .dnfs-hdr-btn.close:hover { background: var(--dos-close-bg); border-color: var(--dos-close-border); }
            .dnfs-body { padding: 10px 12px; display: flex; flex-direction: column; gap: 10px; }

            /* Info bar */
            .dnfs-info {
                display: flex; align-items: center; justify-content: space-between;
                padding: 6px 8px;
                background: var(--dos-bg-panel, #1a1a1a); border: 1px solid var(--dos-border-lo, #1a1a1a);
            }
            .dnfs-info-left { display: flex; gap: 12px; align-items: center; }
            .dnfs-info-count { font-size: 11px; color: var(--dos-text); }
            .dnfs-info-value { font-size: 11px; color: #3ddc84; }
            .dnfs-status { font-size: 11px; letter-spacing: 0.04em; }
            .dnfs-s-idle { color: var(--dos-text-dim); }
            .dnfs-s-working { color: var(--dos-flag-active, #884444); }
            .dnfs-s-ready { color: #5a8f5a; }
            .dnfs-s-error { color: #8f4a4a; }

            /* File list */
            .dnfs-list {
                border: 1px solid var(--dos-border-lo, #1a1a1a);
                background: var(--dos-bg-panel, #1a1a1a);
                max-height: 260px; overflow-y: auto;
            }
            .dnfs-list-empty {
                padding: 12px; text-align: center;
                color: var(--dos-text-xdim, #696969); font-style: italic; font-size: 11px;
            }
            .dnfs-file-hdr {
                display: flex; gap: 6px; padding: 3px 8px; font-size: 9px;
                letter-spacing: 0.08em; text-transform: uppercase;
                color: var(--dos-text-xdim); border-bottom: 1px solid var(--dos-border-lo);
                margin-left: 22px;
            }
            .dnfs-file-hdr span:nth-child(1) { flex: 1; }
            .dnfs-file-hdr span:nth-child(2) { width: 70px; }
            .dnfs-file-hdr span:nth-child(3) { width: 50px; text-align: right; }
            .dnfs-file-hdr span:nth-child(4) { width: 50px; text-align: right; }
            .dnfs-file {
                display: flex; align-items: center; gap: 6px;
                padding: 3px 8px; font-size: 11px;
                border-bottom: 1px solid var(--dos-border-lo, #1a1a1a);
                cursor: none;
            }
            .dnfs-file:last-child { border-bottom: none; }
            .dnfs-file:hover { background: var(--dos-bg-hover); }
            .dnfs-file.selected { background: var(--dos-bg-selected); }
            .dnfs-file-chk {
                width: 14px; height: 14px; flex-shrink: 0;
                border: 1px solid var(--dos-border-hi, #333);
                background: var(--dos-bg, #0c0c0c);
                display: flex; align-items: center; justify-content: center;
                font-size: 10px; color: var(--dos-accent-hi);
            }
            .dnfs-file.selected .dnfs-file-chk { border-color: var(--dos-accent); }
            .dnfs-file-name { flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; color: var(--dos-text); }
            .dnfs-file-cat { width: 70px; font-size: 10px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
            .dnfs-file-val { width: 50px; text-align: right; color: var(--dos-text-hi); }
            .dnfs-file-exp { width: 50px; text-align: right; font-size: 10px; color: var(--dos-text-xdim); }

            /* Buttons */
            .dnfs-btns { display: flex; gap: 4px; }
            .dnfs-btn {
                flex: 1; padding: 5px 0; text-align: center;
                font-family: inherit; font-size: 11px; font-weight: normal;
                letter-spacing: 0.06em; text-transform: uppercase;
                border: 1px solid var(--dos-border, #222);
                background: var(--dos-bg-panel, #1a1a1a);
                color: var(--dos-text-label, #a3a3a3);
                cursor: none; transition: background 0.15s, color 0.15s;
            }
            .dnfs-btn:hover { background: var(--dos-bg-hover); color: var(--dos-text-hi); border-color: var(--dos-border-hi); }
            .dnfs-btn:disabled { opacity: 0.4; pointer-events: none; }
            .dnfs-btn.sell { color: #3ddc84; border-color: #335533; }
            .dnfs-btn.sell:hover { background: rgba(16,40,16,0.5); border-color: #448844; }

            /* Log */
            .dnfs-log {
                border: 1px solid var(--dos-border-lo);
                background: var(--dos-bg, #0c0c0c);
                max-height: 80px; overflow-y: auto; font-size: 10px; padding: 4px 6px;
            }
            .dnfs-log-line { color: var(--dos-text-dim); line-height: 1.5; }
            .dnfs-log-ts { color: var(--dos-text-xdim); margin-right: 6px; }

            /* Settings */
            .dnfs-set-section {
                margin-top: 4px; font-size: 10px; letter-spacing: 0.1em;
                text-transform: uppercase; color: var(--dos-text-xdim); margin-bottom: 4px;
            }
            .dnfs-scale-row { display: flex; align-items: center; gap: 4px; }
            .dnfs-scale-btn {
                width: 22px; height: 22px;
                background: var(--dos-bg-panel); border: 1px solid var(--dos-border);
                color: var(--dos-text-label); font-size: 13px; font-family: inherit;
                display: flex; align-items: center; justify-content: center; cursor: none;
            }
            .dnfs-scale-btn:hover { border-color: var(--dos-border-hi); color: var(--dos-text-hi); background: var(--dos-bg-hover); }
            .dnfs-scale-label {
                font-size: 11px; color: var(--dos-text); min-width: 90px;
                text-align: center; letter-spacing: 0.04em;
            }

            /* CLI fallback */
            #dnfs-cli-btn {
                display: none; min-width: 18px; padding: 1px 5px;
                text-align: center; font-weight: bold; font-size: 13px; font-family: inherit;
                color: #9cf79c; border: 1px solid #4a8f4a; background: rgba(12,42,12,0.65);
                cursor: pointer; letter-spacing: 0.5px; white-space: nowrap;
            }
            #dnfs-cli-btn:hover { background: rgba(20,60,20,0.8); border-color: #6abf6a; color: #bfffbf; }
        `;
        document.head.appendChild(style);
    }

    function buildPanel() {
        panelEl = document.createElement('div');
        panelEl.id = 'dnfs-panel';
        panelEl.innerHTML = `
            <div class="dnfs-titlebar">
                <span class="dnfs-title">Data Dealer</span>
                <div class="dnfs-hdr-btn" id="dnfs-settings-btn" title="Settings">\u2699</div>
                <div class="dnfs-hdr-btn close" id="dnfs-close">\u00D7</div>
            </div>
            <div class="dnfs-body" id="dnfs-body"></div>`;
        document.body.appendChild(panelEl);
        makePanelDraggable(panelEl, panelEl.querySelector('.dnfs-titlebar'));

        document.getElementById('dnfs-close').addEventListener('click', hidePanel);
        document.getElementById('dnfs-settings-btn').addEventListener('click', () => {
            switchView(currentView === 'settings' ? 'main' : 'settings');
        });
    }

    function switchView(view) {
        currentView = view;
        if (view === 'main') renderMain(); else renderSettings();
        const btn = document.getElementById('dnfs-settings-btn');
        if (btn) btn.style.color = view === 'settings' ? 'var(--dos-accent-hi)' : '';
    }

    function centerPanel() {
        if (!panelEl) return;
        const s = getScale();
        const pw = 480 * s;
        const ph = (panelEl.offsetHeight || 400) * s;
        panelEl.style.left = Math.max(0, (window.innerWidth - pw) / 2) + 'px';
        panelEl.style.top = Math.max(0, (window.innerHeight - ph) / 2) + 'px';
        panelEl.style.transform = `scale(${s})`;
        delete panelEl.dataset.dragged;
    }

    function applyScale() {
        if (!panelEl) return;
        if (panelEl.dataset.dragged) panelEl.style.transform = `scale(${getScale()})`;
        else centerPanel();
    }

    // ── Main view ──
    function renderMain() {
        const body = document.getElementById('dnfs-body');
        if (!body || currentView !== 'main') return;

        const sellableCount = files.length;
        const selectedValue = files.filter(f => selected.has(f.id)).reduce((s, f) => s + Number(f.value || 0), 0);

        body.innerHTML = `
            <div class="dnfs-info">
                <div class="dnfs-info-left">
                    <span class="dnfs-info-count">${selected.size} / ${sellableCount} files</span>
                    <span class="dnfs-info-value">${selectedValue > 0 ? selectedValue + ' RCH' : ''}</span>
                </div>
                <span class="dnfs-status dnfs-s-idle" id="dnfs-status">idle</span>
            </div>
            <div class="dnfs-file-hdr">
                <span>Filename</span>
                <span>Category</span>
                <span>Value</span>
                <span>Expires</span>
            </div>
            <div class="dnfs-list" id="dnfs-list"></div>
            <div class="dnfs-btns">
                <button class="dnfs-btn" id="dnfs-selall">Select All</button>
                <button class="dnfs-btn" id="dnfs-selnone">Select None</button>
                <button class="dnfs-btn" id="dnfs-refresh">Refresh</button>
                <button class="dnfs-btn sell" id="dnfs-sell" ${selling || selected.size === 0 ? 'disabled' : ''}>Sell (${selected.size})</button>
            </div>
            <div class="dnfs-log" id="dnfs-log"></div>`;

        const listEl = document.getElementById('dnfs-list');
        if (files.length === 0) {
            listEl.innerHTML = '<div class="dnfs-list-empty">No sellable files</div>';
        } else {
            for (const f of files) {
                const sel = selected.has(f.id);
                const q = qualityLabel(f.quality);
                const row = document.createElement('div');
                row.className = `dnfs-file${sel ? ' selected' : ''}`;
                row.innerHTML =
                    `<div class="dnfs-file-chk">${sel ? '\u2713' : ''}</div>` +
                    `<span class="dnfs-file-name" style="color:${q.color}" title="${f.filename}">${f.filename}</span>` +
                    `<span class="dnfs-file-cat" style="color:var(--dos-text-xdim)">${f.category || ''}</span>` +
                    `<span class="dnfs-file-val">${f.value}</span>` +
                    `<span class="dnfs-file-exp">${timeLeft(f.expires_at)}</span>`;
                row.addEventListener('click', () => {
                    if (selling) return;
                    if (selected.has(f.id)) selected.delete(f.id); else selected.add(f.id);
                    renderMain();
                });
                listEl.appendChild(row);
            }
        }

        document.getElementById('dnfs-selall').addEventListener('click', () => {
            files.forEach(f => selected.add(f.id));
            renderMain();
        });
        document.getElementById('dnfs-selnone').addEventListener('click', () => {
            selected.clear();
            renderMain();
        });
        document.getElementById('dnfs-refresh').addEventListener('click', () => {
            if (!selling) refreshFiles();
        });
        document.getElementById('dnfs-sell').addEventListener('click', () => {
            if (!selling && selected.size > 0) sellSelected();
        });
        renderLog();
    }

    // ── Settings view ──
    function renderSettings() {
        const body = document.getElementById('dnfs-body');
        if (!body) return;
        const scaleLabel = settings.uiScale === 0 ? `Auto (${Math.round(getScale() * 100)}%)` : `${Math.round(settings.uiScale * 100)}%`;
        body.innerHTML = `
            <div class="dnfs-set-section">UI Scale</div>
            <div class="dnfs-scale-row">
                <div class="dnfs-scale-btn" id="dnfs-sc-down">\u2212</div>
                <span class="dnfs-scale-label" id="dnfs-sc-label">${scaleLabel}</span>
                <div class="dnfs-scale-btn" id="dnfs-sc-up">+</div>
                <div class="dnfs-scale-btn" id="dnfs-sc-auto" style="margin-left:4px;width:auto;padding:0 6px;${settings.uiScale===0?'color:var(--dos-accent-hi);border-color:var(--dos-accent)':''}">Auto</div>
            </div>`;

        document.getElementById('dnfs-sc-down').addEventListener('click', () => {
            settings.uiScale = Math.max(0.8, Math.round(((settings.uiScale || getScale()) - 0.1) * 10) / 10);
            saveSettings(); applyScale(); renderSettings();
        });
        document.getElementById('dnfs-sc-up').addEventListener('click', () => {
            settings.uiScale = Math.min(2.0, Math.round(((settings.uiScale || getScale()) + 0.1) * 10) / 10);
            saveSettings(); applyScale(); renderSettings();
        });
        document.getElementById('dnfs-sc-auto').addEventListener('click', () => {
            settings.uiScale = 0; saveSettings(); applyScale(); renderSettings();
        });
    }

    function renderLog() {
        const el = document.getElementById('dnfs-log');
        if (!el) return;
        el.innerHTML = '';
        for (const l of logLines) {
            const div = document.createElement('div');
            div.className = 'dnfs-log-line';
            div.innerHTML = `<span class="dnfs-log-ts">${l.ts}</span>${l.msg}`;
            el.appendChild(div);
        }
        el.scrollTop = el.scrollHeight;
    }

    function showPanel() {
        if (!panelEl) buildPanel();
        panelEl.classList.add('visible');
        currentView = 'main'; renderMain();
        requestAnimationFrame(() => centerPanel());
        if (!selling && machineId && token) refreshFiles();
    }

    function hidePanel() { if (panelEl) panelEl.classList.remove('visible'); }

    function makePanelDraggable(el, handle) {
        let ox, oy, dx, dy;
        handle.addEventListener('mousedown', (e) => {
            if (e.target.closest('.dnfs-hdr-btn')) return;
            const rect = el.getBoundingClientRect();
            el.style.transform = `scale(${getScale()})`;
            el.style.left = rect.left + 'px';
            el.style.top = rect.top + 'px';
            el.dataset.dragged = '1';
            ox = e.clientX; oy = e.clientY; dx = rect.left; dy = rect.top;
            const mv = (e2) => { el.style.left = (dx + e2.clientX - ox) + 'px'; el.style.top = (dy + e2.clientY - oy) + 'px'; };
            const up = () => { document.removeEventListener('mousemove', mv); document.removeEventListener('mouseup', up); };
            document.addEventListener('mousemove', mv); document.addEventListener('mouseup', up);
            e.preventDefault();
        });
    }

    // ═══════════════════════════════════════════════════════════
    //  DESKTOP ICON + CLI FALLBACK
    // ═══════════════════════════════════════════════════════════
    let sawDeepOSPending = false, isDesktopIcon = false, cliBtnEl = null;
    const ICON_POS_KEY = 'deepos_icon_positions';
    function isDeepOSPending() {
        const p = window._deeposBootPending || document.body.classList.contains('deepos-active') || (window._DOS && window._DOS.active);
        if (p) sawDeepOSPending = true; return p || sawDeepOSPending;
    }
    function loadIconPositions() { try { return JSON.parse(localStorage.getItem(ICON_POS_KEY)) || {}; } catch (_) { return {}; } }
    function saveIconPositions() {
        const p = {}; document.querySelectorAll('#deepos-icons .deepos-icon').forEach(i => {
            const id = i.dataset.app || i.dataset.intApp || i.dataset.sysApp;
            if (id && i.style.left) p[id] = { x: parseInt(i.style.left) || 0, y: parseInt(i.style.top) || 0 };
        }); try { localStorage.setItem(ICON_POS_KEY, JSON.stringify(p)); } catch (_) {}
    }
    function calcIconSlot(idx) {
        const rem = parseFloat(getComputedStyle(document.documentElement).fontSize) || 16, vw = window.innerWidth / 100;
        const w = Math.min(6 * rem, Math.max(3.8 * rem, 5.2 * vw)), gh = Math.min(3.7 * rem, Math.max(2.1 * rem, 3.3 * vw));
        const ch = gh + 2.4 * rem, mr = Math.max(1, Math.floor((window.innerHeight - 40) / ch));
        return { x: 0.3 * rem + Math.floor(idx / mr) * w, y: 0.3 * rem + (idx % mr) * ch };
    }
    function applyIconPos(icon, id) {
        const s = loadIconPositions(); icon.style.position = 'absolute';
        if (s[id]) { icon.style.left = s[id].x + 'px'; icon.style.top = s[id].y + 'px'; }
        else { const p = calcIconSlot(document.querySelectorAll('#deepos-icons .deepos-icon').length); icon.style.left = p.x + 'px'; icon.style.top = p.y + 'px'; }
    }
    function makeIconDraggable(icon) {
        let ox = 0, oy = 0, d = false, m = false, w = false;
        icon.addEventListener('mousedown', e => { if (e.button) return; d = true; m = false; ox = e.clientX - (parseInt(icon.style.left) || 0); oy = e.clientY - (parseInt(icon.style.top) || 0); e.preventDefault(); });
        document.addEventListener('mousemove', e => {
            if (!d) return;
            if (!m && Math.abs(e.clientX - (ox + parseInt(icon.style.left || 0))) < 3 && Math.abs(e.clientY - (oy + parseInt(icon.style.top || 0))) < 3) return;
            m = true; const p = icon.parentElement.getBoundingClientRect();
            icon.style.left = Math.max(0, Math.min(e.clientX - ox, p.width - icon.offsetWidth)) + 'px';
            icon.style.top = Math.max(0, Math.min(e.clientY - oy, p.height - icon.offsetHeight)) + 'px';
        });
        document.addEventListener('mouseup', () => { if (!d) return; if (m) { saveIconPositions(); w = true; setTimeout(() => { w = false; }, 400); } d = false; m = false; });
        return () => w;
    }

    function placeUI() {
        if (isDesktopIcon || (cliBtnEl && cliBtnEl.parentElement)) return true;
        const icons = document.querySelector('#deepos-icons');
        if (icons) {
            const icon = document.createElement('div'); icon.className = 'deepos-icon'; icon.dataset.app = 'dnfs';
            icon.innerHTML = '<div class="deepos-icon-gfx" style="background:var(--dos-icon-bg);color:var(--dos-icon-color);font-size:clamp(0.5rem,0.85vw,1rem);font-weight:bold;letter-spacing:0.03em;">RCH</div><span>Dealer</span>';
            applyIconPos(icon, 'dnfs'); const chk = makeIconDraggable(icon);
            icon.addEventListener('dblclick', () => { if (!chk()) showPanel(); });
            icons.appendChild(icon); isDesktopIcon = true; return true;
        }
        if (isDeepOSPending()) return false;
        cliBtnEl = document.createElement('span'); cliBtnEl.id = 'dnfs-cli-btn';
        cliBtnEl.textContent = 'SELL'; cliBtnEl.title = 'Data Dealer';
        cliBtnEl.addEventListener('click', showPanel);
        const sr = document.querySelector('#status-right'), sb = document.querySelector('#statusbar');
        if (sr && sb) sb.insertBefore(cliBtnEl, sr); else if (sb) sb.appendChild(cliBtnEl); else return false;
        return true;
    }

    // ═══════════════════════════════════════════════════════════
    //  BOOTSTRAP
    // ═══════════════════════════════════════════════════════════
    function init() {
        injectStyles();
        const bc = setInterval(() => {
            if (!isLoggedIn()) return; if (!placeUI()) return;
            clearInterval(bc);
            if (cliBtnEl) cliBtnEl.style.display = 'inline-block';
            console.log('[SELL] v1.1 placed as', isDesktopIcon ? 'icon' : 'cli');
        }, 500);
    }
    setTimeout(init, 2000);
})();

```
