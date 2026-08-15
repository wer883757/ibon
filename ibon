// ==========================================
// 💥 結界破解：強制開放 Shadow DOM
// ==========================================
(function() {
    const originalAttachShadow = Element.prototype.attachShadow;
    Element.prototype.attachShadow = function(init) {
        if (init && init.mode === 'closed') init.mode = 'open';
        return originalAttachShadow.call(this, init);
    };
})();

// ==========================================
// 🚀 內建極速攔截器 (阻擋不必要的追蹤腳本，提升渲染速度)
// ==========================================
const BLOCK_DOMAINS = [
    'clarity.ms', 'doubleclick.net', 'google-analytics.com', 'googletagmanager.com',
    'googlesyndication.com', 'facebook.com', 'fbcdn.net', 'popin.cc', 'tagtoo.co',
    'lndata.com', 'rollbar.com', 'anymind360.com', 'sentry.io'
];

const observer = new MutationObserver((mutations) => {
    for (const mutation of mutations) {
        for (const node of mutation.addedNodes) {
            if (node.tagName === 'SCRIPT' || node.tagName === 'IFRAME') {
                if (node.src && BLOCK_DOMAINS.some(domain => node.src.includes(domain))) {
                    node.remove();
                }
            }
        }
    }
});
observer.observe(document.documentElement, { childList: true, subtree: true });

(function () {
    'use strict';

    let isWaitingForResponse = false;
    let waitingTimeout = null;
    const STORAGE_KEY = 'ibon_autostart_state_v3';
    const SETTINGS_KEY = 'ibon_user_pref_settings_v3';
    const SCHEDULE_KEY = 'ibon_scheduled_time';

    let running = false;
    let loopId = null;
    let scheduleTimer = null;
    const LOOP_INTERVAL = 300;
    const RELOAD_COOLDOWN = 1000;
    let lastReload = 0;

    const pageLoadTime = Date.now();
    const EXCLUDE_KEYWORDS = ['愛心', '身障', '輪椅', '優待'];
    const alarm = new Audio("https://actions.google.com/sounds/v1/alarms/beep_short.ogg");
    const $ = id => document.getElementById(id);

    function optimizeRendering() {
        const style = document.createElement('style');
        style.innerHTML = `
            .single-header img, .event-description img, picture { display: none !important; }
        `;
        if(document.head) document.head.appendChild(style);
        else document.addEventListener("DOMContentLoaded", () => document.head.appendChild(style));
    }
    optimizeRendering();

    function queryAllDeep(selector, root = document) {
        let results = Array.from(root.querySelectorAll(selector));
        const allElements = root.querySelectorAll('*');
        for (let el of allElements) {
            if (el.shadowRoot) {
                results = results.concat(queryAllDeep(selector, el.shadowRoot));
            }
        }
        return results;
    }

    // ======== GUI 與設定機制 ========
    function updateUIState() {
        const startBtn = $("start");
        const pauseBtn = $("pause");
        if (!startBtn || !pauseBtn) return;
        if (running) {
            startBtn.style.background = "#28a745";
            startBtn.innerText = "▶ 執行中 (監控中)";
            pauseBtn.style.opacity = "1";
        } else {
            startBtn.style.background = "#0066ff";
            startBtn.innerText = "▶ 啟動";
            pauseBtn.style.opacity = "0.5";
        }
    }

    function saveSettings() {
        if(!$("targetDate")) return;
        const config = {
            targetDate: $("targetDate").value || "",
            areaKw: $("areaKw").value || "",
            p1: $("p1").value || "",
            num: $("num").value || "2",
            mode: $("mode").value || "random",
            answer: $("answer").value || "",
            startTime: $("startTime").value || "",
            ticketPref: $("ticketPref") ? $("ticketPref").value : "any"
        };
        GM_setValue(SETTINGS_KEY, JSON.stringify(config));
    }

    function loadSettings() {
        const saved = GM_getValue(SETTINGS_KEY);
        if (saved) {
            try {
                const config = JSON.parse(saved);
                if (config.targetDate && $("targetDate")) $("targetDate").value = config.targetDate;
                if (config.areaKw && $("areaKw")) $("areaKw").value = config.areaKw;
                if (config.p1 && $("p1")) $("p1").value = config.p1;
                if (config.num && $("num")) $("num").value = config.num;
                if (config.mode && $("mode")) $("mode").value = config.mode;
                if (config.answer && $("answer")) $("answer").value = config.answer;
                if (config.startTime && $("startTime")) $("startTime").value = config.startTime;
                if (config.ticketPref && $("ticketPref")) $("ticketPref").value = config.ticketPref;
            } catch(e) {}
        }
    }

    function makeDraggable(el, handle) {
        let pos1 = 0, pos2 = 0, pos3 = 0, pos4 = 0;
        handle.onmousedown = (e) => {
            e = e || window.event; e.preventDefault();
            pos3 = e.clientX; pos4 = e.clientY;
            document.onmouseup = () => { document.onmouseup = null; document.onmousemove = null; };
            document.onmousemove = (e) => {
                e = e || window.event; e.preventDefault();
                pos1 = pos3 - e.clientX; pos2 = pos4 - e.clientY;
                pos3 = e.clientX; pos4 = e.clientY;
                el.style.top = (el.offsetTop - pos2) + "px";
                el.style.left = (el.offsetLeft - pos1) + "px";
                el.style.right = "auto";
            };
        };
    }

    function initGUI() {
        if (document.getElementById("kenny-panel")) {
            updateUIState();
            return;
        }
        if (!document.body) { setTimeout(initGUI, 200); return; }

        const container = document.createElement("div");
        container.id = "kenny-panel";
        container.style.cssText = `
            background: #111111; color: #eeeeee; padding: 12px; position: fixed; top: 15%; right: 20px;
            z-index: 999999; width: 280px; font-size: 14px; font-family: sans-serif;
            border-radius: 8px; border: 1px solid #333; text-align: left; box-shadow: 0 0 15px rgba(0,0,0,0.8);
            cursor: default; box-sizing: border-box;
        `;

        container.innerHTML = `
            <h3 id="kenny-header" style="margin:0 0 10px 0; text-align:center; font-size:18px; font-weight:normal; cursor:move; user-select:none; background:#222; border-radius:4px; padding:6px; color:#ddd; border:1px solid #333;"> Kenny ibon v3.9 </h3>

            <label style="display:block; margin:0 0 2px 0;">場次日期 (空=不指定)</label>
            <input id="targetDate" class="pref-input" type="text" placeholder="例：08-30 或 日" style="width:100%; margin:0 0 8px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">

            <label style="display:block; margin:0 0 2px 0;">指定區域 (多詞可用空格隔開)</label>
            <input id="areaKw" class="pref-input" type="text" placeholder="例：黃1B 特區" style="width:100%; margin:0 0 8px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">

            <label style="display:block; margin:0 0 2px 0;">票價優先順序 (多個用逗號 , 隔開)</label>
            <input id="p1" class="pref-input" type="text" placeholder="例：3980, 2680 (空=熱賣優先)" style="width:100%; margin:0 0 8px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">

            <label style="display:block; margin:0 0 2px 0;">張數</label>
            <select id="num" class="pref-input" style="width:100%; margin:0 0 8px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">
                <option value="1">1</option><option value="2" selected>2</option><option value="3">3</option><option value="4">4</option>
            </select>

            <label style="display:block; margin:0 0 2px 0;">模式</label>
            <select id="mode" class="pref-input" style="width:100%; margin:0 0 8px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">
                <option value="top">由上而下</option><option value="bottom">由下而上</option><option value="random" selected>隨機</option>
            </select>

            <label style="display:block; margin:0 0 2px 0;">同區票種偏好 (ibon 專用)</label>
            <select id="ticketPref" class="pref-input" style="width:100%; margin:0 0 8px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">
                <option value="any">預設 (皆可)</option><option value="bonus">🎁 優先含【福利券】</option><option value="normal">🎟️ 僅限純【全票】</option>
            </select>

            <label style="display:block; margin:0 0 2px 0;">一般驗證碼 / 問答答案 (若無則空)</label>
            <input id="answer" class="pref-input" type="text" placeholder="例：DREAMING" style="width:100%; margin:0 0 8px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">

            <label style="display:block; margin:0 0 2px 0;">啟動時間 (空=立即)</label>
            <input id="startTime" class="pref-input" type="text" placeholder="HH:MM:SS" style="width:100%; margin:0 0 12px 0; font-size:15px; padding:6px; box-sizing:border-box; background:#fff; color:#000; border:none; border-radius:2px;">

            <button id="start" style="width:100%; padding:8px; margin-bottom:6px; background:#0066ff; color:white; font-size:16px; border-radius:4px; border:none; cursor:pointer; transition:0.2s;"> ▶ 啟動 </button>
            <button id="pause" style="width:100%; padding:8px; background:#8b0000; color:white; font-size:16px; border-radius:4px; border:none; cursor:pointer; opacity:0.5; transition:0.2s;"> ⏸ 暫停 </button>
        `;
        document.body.appendChild(container);

        makeDraggable(container, $("kenny-header"));
        document.querySelectorAll(".pref-input").forEach(el => el.addEventListener("input", saveSettings));

        $("start").onclick = () => {
            const T = $("startTime") ? $("startTime").value.trim() : "";
            if (!T) {
                running = true;
                GM_setValue(STORAGE_KEY, 'true');
                updateUIState();
                toast("🚀 立即啟動！開始每秒重整。");
                startLoop();
            } else {
                GM_setValue(SCHEDULE_KEY, T);
                updateUIState();
                toast(`⏳ 已鎖定！系統將於 ${T} 準時開搶...`);
                startScheduleLoop(T);
            }
        };

        $("pause").onclick = () => {
            running = false;
            isWaitingForResponse = false;
            if (scheduleTimer) clearInterval(scheduleTimer);
            GM_deleteValue(STORAGE_KEY);
            GM_deleteValue(SCHEDULE_KEY);
            updateUIState();
            toast("⏸ 暫停中 (已取消所有任務)", "error");
            stopLoop();
        };

        loadSettings();
        updateUIState();
    }

    function startScheduleLoop(targetTime) {
        if (scheduleTimer) clearInterval(scheduleTimer);
        scheduleTimer = setInterval(() => {
            const now = new Date().toTimeString().slice(0, 8);
            if (now >= targetTime) {
                clearInterval(scheduleTimer);
                GM_deleteValue(SCHEDULE_KEY);
                running = true;
                GM_setValue(STORAGE_KEY, 'true');
                toast("🔥 時間到，自動重整並光速啟動！");
                window.location.reload();
            }
        }, 100);
    }

    function toast(msg, type = "info") {
        if (!document.body) return;
        const div = document.createElement("div"); div.innerText = msg;
        let bgColor = "rgba(0,0,0,0.85)";
        if (type === "error") bgColor = "rgba(204, 0, 0, 0.9)";
        if (type === "warning") bgColor = "rgba(245, 158, 11, 0.95)";
        div.style.cssText = `position: fixed; top: 15%; right: 20px; z-index: 9999999; background: ${bgColor}; color: #fff; padding: 10px 16px; border-radius: 8px; font-size: 16px; box-shadow: 0 0 8px #000; pointer-events: none; transition: opacity 0.3s;`;
        document.body.appendChild(div);
        setTimeout(() => { div.style.opacity = "0"; setTimeout(() => div.remove(), 400); }, type === "info" ? 1500 : 3500);
    }

    function setWaitingState() {
        isWaitingForResponse = true;
        if (waitingTimeout) clearTimeout(waitingTimeout);
        waitingTimeout = setTimeout(() => {
            isWaitingForResponse = false;
        }, 5000);
    }

    function isQueuingOrAntiBot() {
        const url = window.location.href;
        const html = document.body ? document.body.innerHTML : '';
        const text = document.body ? document.body.innerText : '';
        const title = document.title;
        if (html.includes('cf-turnstile') || html.includes('cf-browser-verification') || title.includes('Just a moment') || title.includes('請稍候')) return true;
        if (url.includes('queue-it') || html.includes('queue-it') || document.getElementById('queue-it_log')) return true;
        if (text.includes('排隊中') || text.includes('正在排隊') || text.includes('驗證您是真人') || text.includes('Checking your browser')) return true;
        return false;
    }

    function executeIbonFlow() {
        const url = window.location.href;

        // 🛡️ 結帳頁面待命防護
        if (url.includes("UTK0206_") || url.includes("Cart") || document.querySelector('input[name*="Contact"]')) {
            if (!window.standbyNotified) {
                toast("✅ 已進入結帳頁面！腳本轉入待命狀態，請手手動填寫資料。");
                window.standbyNotified = true;
            }
            return;
        }

        // 階段一：活動主頁
        if (url.includes("ticket.ibon.com.tw/ActivityInfo/Details")) {
            const targetDate = ($("targetDate")?.value || "").trim();
            const bigBuyBtn = queryAllDeep('button, a, div').find(el =>
                ((el.id && el.id.includes('BuyTicketsNow')) || (el.innerText && el.innerText.trim() === '立即購票')) &&
                el.offsetParent !== null && !el.classList.contains('btn-buy')
            );

            if (bigBuyBtn && !bigBuyBtn.dataset.kennyClicked) {
                bigBuyBtn.dataset.kennyClicked = "true";
                bigBuyBtn.click();
            }

            let allBuyBtns = [];
            const appGame = document.querySelector('app-game');
            if (appGame && appGame.shadowRoot) {
                allBuyBtns = Array.from(appGame.shadowRoot.querySelectorAll('.btn-buy'));
            }
            allBuyBtns = [...new Set([...allBuyBtns, ...queryAllDeep('.btn-buy')])];

            const activeBottomBtns = allBuyBtns.filter(b => {
                if (b === bigBuyBtn) return false;
                if (b.disabled) return false;
                const txt = (b.innerText || "").replace(/\s+/g, '');
                if (txt.includes('剩') || txt.includes('開賣') || txt.includes('尚未') || txt.includes('售完')) return false;
                if (b.classList.contains('btn-buy') || txt.includes('線上購票') || txt.includes('購票')) return true;
                return false;
            });

            if (activeBottomBtns.length > 0) {
                let targetButton = null;
                if (targetDate) {
                    for (const btn of activeBottomBtns) {
                        const row = btn.closest('.d-flex, .row, .grid, tr');
                        if (row && row.innerText && row.innerText.includes(targetDate)) { targetButton = btn; break; }
                    }
                } else { targetButton = activeBottomBtns[0]; }

                if (targetButton) {
                    setWaitingState();
                    toast("👉 按鈕已解鎖！雙重點擊突破中...");
                    if (bigBuyBtn) bigBuyBtn.click();
                    targetButton.click();
                    return true;
                }
            }

            const now = Date.now();
            if (now - pageLoadTime > 800 && now - lastReload > RELOAD_COOLDOWN) {
                lastReload = now;
                toast("🔄 等待解鎖，每秒自動重整...");
                setTimeout(() => { if (running) window.location.reload(); }, 200);
            }
            return false;
        }

        // 階段二：選擇票區 / 張數頁面 (UTK02 流程)
        else if (url.includes("orders.ibon.com.tw") || url.includes("UTK02")) {
            if (isWaitingForResponse) return;

            const areaKw = $("areaKw")?.value || "";
            const rawPriceKw = $("p1")?.value || "";
            const ticketPref = $("ticketPref")?.value || "any";
            const desiredNum = parseInt($("num")?.value) || 2;
            const mode = $("mode")?.value || "random";

            const hasSelects = Array.from(document.querySelectorAll('table select, .ticket-wrap select, select[id*="AMOUNT"], select[name*="AMOUNT"]')).length > 0;

            let rows = queryAllDeep("tr").filter(row => row.offsetParent !== null && row.innerText.trim().length > 2);
            let validCandidates = [];

            // 🌟 1. 初步篩選與 TicketPlus 權重評分系統
            rows.forEach(row => {
                const text = row.innerText || '';

                if (text.includes('7-ELEVEN') || text.includes('注意事項') || text.includes('聯華食品') || text.includes('退票')) return;
                if (EXCLUDE_KEYWORDS.some(k => text.includes(k))) return;

                let statusText = text;
                const statusCell = row.querySelector('td.action, td[data-title="空位"], select');
                if (statusCell) statusText = statusCell.innerText || (statusCell.tagName==='SELECT' ? '可選' : statusText);

                if (text.includes('已售完') || text.includes('缺票') || statusText.includes('完')) return;

                let score = 0;

                // 導入 TicketPlus 權重邏輯
                if (statusText.includes('熱賣中') || text.includes('熱賣中')) score += 100;
                if (statusText.includes('剩餘') || text.includes('剩餘')) score -= 100;

                // 同區票種偏好
                if (ticketPref === "bonus" && text.includes("福利")) score += 300;
                if (ticketPref === "normal" && !text.includes("福利")) score += 300;

                validCandidates.push({ row, score, text });
            });

            if (validCandidates.length === 0) {
                if (Date.now() - pageLoadTime > 2500) window.location.reload();
                return;
            }

            // 🌟 2. 指定區域篩選 (支援多詞空格)
            if (areaKw.trim() !== "") {
                const areaKws = areaKw.split(/\s+/).filter(k => k);
                if (areaKws.length > 0) {
                    validCandidates = validCandidates.filter(item => {
                        return areaKws.every(kw => item.text.includes(kw));
                    });
                }
            }

            if (validCandidates.length === 0) {
                if (Date.now() - pageLoadTime > 2500) window.location.reload();
                return;
            }

            // 🌟 3. 逗號優先順序篩選
            let finalCandidates = [];
            if (rawPriceKw.trim() !== "") {
                let priceList = rawPriceKw.split(/[,，]/).map(p => p.replace(/[^a-zA-Z0-9\u4e00-\u9fa5]/g, '').trim()).filter(p => p);
                for (let price of priceList) {
                    let tempMatches = validCandidates.filter(item => {
                        const cleanText = item.text.replace(/[^a-zA-Z0-9\u4e00-\u9fa5]/g, '');
                        return cleanText.includes(price);
                    });
                    if (tempMatches.length > 0) {
                        finalCandidates = tempMatches;
                        break;
                    }
                }
                // 智慧降級
                if (finalCandidates.length === 0) finalCandidates = validCandidates;
            } else {
                finalCandidates = validCandidates;
            }

            // 🌟 4. 權重最高者鎖定與模式套用
            if (finalCandidates.length > 0) {
                const maxScore = Math.max(...finalCandidates.map(item => item.score));
                finalCandidates = finalCandidates.filter(item => item.score === maxScore);
            }

            let targetObj = null;
            if (finalCandidates.length > 0) {
                if (mode === "bottom") targetObj = finalCandidates[finalCandidates.length - 1];
                else if (mode === "random") targetObj = finalCandidates[Math.floor(Math.random() * finalCandidates.length)];
                else targetObj = finalCandidates[0]; // mode === "top"
            }

            // 🌟 5. 執行點擊
            if (targetObj) {
                setWaitingState();
                const targetRow = targetObj.row;
                const zoneName = targetRow.innerText.trim().substring(0, 10).replace(/\n/g, '');

                if (hasSelects) {
                    let targetSelect = targetRow.querySelector('select');
                    if (!targetSelect) {
                        targetSelect = document.querySelector('table select[id*="AMOUNT"], table select[name*="AMOUNT"]');
                    }

                    if (targetSelect) {
                        let options = Array.from(targetSelect.options);
                        let targetVal = desiredNum.toString();
                        if (!options.some(opt => opt.value === targetVal) && options.length > 0) {
                            targetVal = options[options.length - 1].value;
                        }

                        targetSelect.value = targetVal;
                        targetSelect.dispatchEvent(new Event('change', { bubbles: true }));

                        toast(`🎫 鎖定票種 [${zoneName}]，選擇 ${targetVal} 張...`);
                        alarm.play();

                        setTimeout(() => {
                            let nextBtn = queryAllDeep('a, button').find(el => {
                                let txt = el.innerText || "";
                                return txt.includes("下一步") || (el.id && el.id.includes("AddShopingCart")) || txt.includes("다음 단계");
                            });
                            if (nextBtn) {
                                nextBtn.click();
                                toast("✅ 已點擊下一步！");
                            }
                        }, 700);
                        return true;
                    }
                }
                else {
                    const delayTime = Math.floor(Math.random() * 400) + 400;
                    toast(`🎯 鎖定 [${zoneName}]，等待 ${delayTime}ms 安全點擊...`);

                    setTimeout(() => {
                        const clickable = targetRow.querySelector('.action a:not([onclick*="delete"]), a:not([href*="7-11"]):not([onclick*="delete"]), button:not([class*="delete"]), input[type="radio"]') || targetRow;
                        clickable.click();
                    }, delayTime);
                    return true;
                }
            }
        }
    }

    function main() {
        if (!running || isWaitingForResponse || !document.body) return;

        try {
            if (isQueuingOrAntiBot()) {
                if (!window.queueNotified) {
                    toast("⏳ 系統防護驗證中... 腳本已自動靜默暫停重整，請通過驗證！", "warning");
                    window.queueNotified = true;
                }
                return;
            } else {
                window.queueNotified = false;
            }
            executeIbonFlow();
        } catch (error) {
            console.error("Ibon 腳本執行錯誤：", error);
        }
    }

    function startLoop() { if (!loopId) loopId = setInterval(() => { if (running) main(); }, LOOP_INTERVAL); }
    function stopLoop() { if (loopId) { clearInterval(loopId); loopId = null; } }

    setTimeout(() => {
        initGUI();
        running = (GM_getValue(STORAGE_KEY) === 'true');

        const scheduledTime = GM_getValue(SCHEDULE_KEY);
        if (scheduledTime) {
            const now = new Date().toTimeString().slice(0, 8);
            if (now >= scheduledTime) {
                GM_deleteValue(SCHEDULE_KEY);
                running = true;
                GM_setValue(STORAGE_KEY, 'true');
                updateUIState();
                toast("🔥 排程已觸發！系統接管中...");
                startLoop();
            } else {
                updateUIState();
                toast(`⏳ 排程恢復：將於 ${scheduledTime} 自動啟動...`);
                startScheduleLoop(scheduledTime);
            }
        }
        else if (running) {
            updateUIState();
            toast("🔄 跨網域/重整喚醒！自動接續執行中...");
            startLoop();
        }
    }, 450);
})();
