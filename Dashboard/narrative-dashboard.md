---
tags:
  - dashboard
month: 2026-07
---

# Dashboard

## Calendar

```dataviewjs
const pages = dv.pages('"Daily"').where(p => p.date != null);
const weekPages = dv.pages('"Weekly"');

const dateMap = {};
for (const p of pages) {
    const d = p.date?.toFormat ? p.date.toFormat("yyyy-MM-dd") : String(p.date);
    dateMap[d] = p;
}

const weekSet = new Set(weekPages.map(p => p.file.name).array());

const monthVal = dv.current()?.month;
let year, monthIdx;
if (monthVal?.year) {
    year = monthVal.year;
    monthIdx = monthVal.month - 1;
} else if (typeof monthVal === "string") {
    const [y, m] = monthVal.split("-").map(Number);
    year = y; monthIdx = m - 1;
} else {
    const now = new Date();
    year = now.getFullYear(); monthIdx = now.getMonth();
}
const month = monthIdx + 1;
const firstDay = (new Date(year, monthIdx, 1).getDay() + 6) % 7;
const daysInMonth = new Date(year, monthIdx + 1, 0).getDate();
const monthNames = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];

function isoWeek(y, m, d) {
    const date = new Date(y, m, d);
    date.setDate(date.getDate() + 3 - (date.getDay() + 6) % 7);
    const week1 = new Date(date.getFullYear(), 0, 4);
    const wNum = 1 + Math.round(((date - week1) / 86400000 - 3 + (week1.getDay() + 6) % 7) / 7);
    return [date.getFullYear(), wNum];
}

let html = `<table style="border-collapse:collapse;width:100%;font-size:14px">`;
html += `<caption style="font-weight:bold;font-size:16px;margin-bottom:8px;text-align:left">${monthNames[monthIdx]} ${year}</caption>`;
html += `<tr>${["Mon","Tue","Wed","Thu","Fri","Sat","Sun",""].map((d,i) => i < 7
    ? `<th style="padding:6px;text-align:center;color:#888">${d}</th>`
    : `<th style="padding:6px;width:36px"></th>`
).join("")}</tr><tr>`;

for (let i = 0; i < firstDay; i++) html += `<td></td>`;

let col = firstDay;
for (let day = 1; day <= daysInMonth; day++) {
    const dateStr = `${year}-${String(month).padStart(2,"0")}-${String(day).padStart(2,"0")}`;
    const p = dateMap[dateStr];

    const dow = new Date(year, monthIdx, day).getDay();
    const isWeekend = dow === 0 || dow === 6;

    let bg = "transparent", color = "inherit", title = "";
    if (isWeekend) { color = "#444"; }
    else if (p?.narrative_result === "correct")        { bg = "#49a845"; color = "#fff"; }
    else if (p?.narrative_result === "partial")        { bg = "#e6b800"; color = "#fff"; }
    else if (p?.narrative_result === "wrong")          { bg = "#cc0000"; color = "#fff"; }
    if (p) title = `${p.narrative ?? ""} → ${p.narrative_result ?? ""}`;

    const opacity = isWeekend ? "opacity:0.3;" : "";
    const label = isWeekend ? `${day}`
        : p
            ? `<a class="internal-link" data-href="Daily/${dateStr}" href="Daily/${dateStr}" style="color:${color};text-decoration:none">${day}</a>`
            : `<span data-create="${dateStr}" style="cursor:pointer;color:${color}">${day}</span>`;
    html += `<td style="padding:6px;text-align:center;border-radius:6px;background:${bg};color:${color};${opacity}" title="${title}">${label}</td>`;
    col++;

    if (col === 7 || day === daysInMonth) {
        if (day === daysInMonth) {
            for (let i = col; i < 7; i++) html += `<td></td>`;
        }
        const [wYear, wNum] = isoWeek(year, monthIdx, day);
        const wStr = `${wYear}-W${String(wNum).padStart(2,"0")}`;
        const wExists = weekSet.has(wStr);
        const wCell = wExists
            ? `<a class="internal-link" data-href="Weekly/${wStr}" href="Weekly/${wStr}" style="color:#888;text-decoration:none">W${wNum}</a>`
            : `<span data-create-week="${wStr}" style="cursor:pointer;color:#888">W${wNum}</span>`;
        html += `<td style="padding:4px;text-align:center;font-size:12px">${wCell}</td>`;
        html += `</tr><tr>`;
        col = 0;
    }
}

html += `</tr></table>`;
html += `<p style="margin-top:8px;font-size:12px;color:#888">Đổi tháng: sửa <code>month</code> trong frontmatter (vd: 2026-05)</p>`;
this.container.innerHTML = html;

this.container.querySelectorAll('[data-create]').forEach(el => {
    el.addEventListener('click', () => {
        const dateStr = el.dataset.create;
        if (confirm(`Create daily note for ${dateStr}?`)) {
            app.workspace.openLinkText(`Daily/${dateStr}`, '', false);
        }
    });
});

this.container.querySelectorAll('[data-create-week]').forEach(el => {
    el.addEventListener('click', () => {
        const wStr = el.dataset.createWeek;
        if (confirm(`Create weekly review for ${wStr}?`)) {
            app.workspace.openLinkText(`Weekly/${wStr}`, '', false);
        }
    });
});
```

## Stats

```dataviewjs
const monthVal = dv.current()?.month;
let year, monthIdx;
if (monthVal?.year) { year = monthVal.year; monthIdx = monthVal.month - 1; }
else if (typeof monthVal === "string") { const [y,m] = monthVal.split("-").map(Number); year=y; monthIdx=m-1; }
else { const now = new Date(); year=now.getFullYear(); monthIdx=now.getMonth(); }

const pages = dv.pages('"Daily"')
    .where(p => p.narrative_result != null && p.narrative_result != ""
        && p.date?.year === year && p.date?.month === monthIdx + 1);

const total = pages.length;
const correct = pages.where(p => p.narrative_result == "correct").length;
const partial = pages.where(p => p.narrative_result == "partial").length;
const wrong   = pages.where(p => p.narrative_result == "wrong").length;

dv.paragraph(`**Total:** ${total} days &nbsp;|&nbsp; 🟢 Correct: ${correct} (${total ? Math.round(correct/total*100) : 0}%) &nbsp;|&nbsp; 🟡 Partial: ${partial} (${total ? Math.round(partial/total*100) : 0}%) &nbsp;|&nbsp; 🔴 Wrong: ${wrong} (${total ? Math.round(wrong/total*100) : 0}%)`);
```

## Detail

```dataviewjs
const monthVal = dv.current()?.month;
let year, monthIdx;
if (monthVal?.year) { year = monthVal.year; monthIdx = monthVal.month - 1; }
else if (typeof monthVal === "string") { const [y,m] = monthVal.split("-").map(Number); year=y; monthIdx=m-1; }
else { const now = new Date(); year=now.getFullYear(); monthIdx=now.getMonth(); }

const pages = dv.pages('"Daily"')
    .where(p => p.narrative_result != null && p.narrative_result != ""
        && p.date?.year === year && p.date?.month === monthIdx + 1)
    .sort(p => p.date, "desc");

dv.table(["Date", "Narrative", "Result"], pages.map(p => [
    p.file.link,
    p.narrative ?? "",
    p.narrative_result
]));
```
