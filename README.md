<!doctype html>
<html lang="zh-Hans">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>马来西亚大学查询（含学费/奖学金/课程/实习/合作企业）</title>
  <style>
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,"Noto Sans";margin:0;background:#f6f7fb;color:#111}
    header{padding:18px 16px;background:#fff;border-bottom:1px solid #e6e8ef;position:sticky;top:0}
    .wrap{max-width:1100px;margin:0 auto}
    h1{font-size:18px;margin:0 0 8px}
    .row{display:flex;gap:10px;flex-wrap:wrap}
    input,select{padding:10px 12px;border:1px solid #d7dbe7;border-radius:10px;background:#fff}
    input{flex:1;min-width:240px}
    select{min-width:180px}
    main{padding:16px}
    .grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(280px,1fr));gap:12px}
    .card{background:#fff;border:1px solid #e6e8ef;border-radius:14px;padding:12px;box-shadow:0 1px 2px rgba(0,0,0,.03)}
    .muted{color:#586174;font-size:12px}
    .pill{display:inline-block;padding:4px 8px;border:1px solid #e6e8ef;border-radius:999px;font-size:12px;margin-right:6px}
    button{padding:10px 12px;border:1px solid #d7dbe7;border-radius:10px;background:#fff;cursor:pointer}
    dialog{border:none;border-radius:16px;max-width:820px;width:92vw}
    dialog::backdrop{background:rgba(0,0,0,.35)}
    .sec{margin:10px 0}
    .sec h3{margin:0 0 6px;font-size:14px}
    .list{margin:0;padding-left:18px}
    a{color:#1a66ff;text-decoration:none}
    a:hover{text-decoration:underline}
  </style>
</head>
<body>
<header>
  <div class="wrap">
    <h1>马来西亚大学查询</h1>
    <div class="muted">搜索/筛选：大学类型、州属、是否有奖学金/实习信息；点击卡片看详情与来源链接。</div>
    <div class="row" style="margin-top:10px">
      <input id="q" placeholder="搜索：大学名 / 科系 / 课程 / 合作企业（例如：broadcast, mass comm, APU）" />
      <select id="type">
        <option value="">全部类型</option>
        <option value="Public">公立</option>
        <option value="Private">私立</option>
        <option value="Branch">海外分校</option>
      </select>
      <select id="state">
        <option value="">全部州属</option>
      </select>
      <select id="flags">
        <option value="">全部</option>
        <option value="scholarship">仅显示：有奖学金信息</option>
        <option value="internship">仅显示：有实习/工业培训信息</option>
        <option value="partners">仅显示：有合作企业信息</option>
        <option value="tuition">仅显示：有学费信息</option>
      </select>
      <button id="reset">重置</button>
    </div>
  </div>
</header>

<main class="wrap">
  <div id="count" class="muted" style="margin:4px 0 12px"></div>
  <div id="grid" class="grid"></div>
</main>

<dialog id="dlg">
  <div style="padding:14px 14px 10px">
    <div class="row" style="justify-content:space-between;align-items:flex-start">
      <div>
        <div id="d_name" style="font-size:18px;font-weight:700"></div>
        <div id="d_meta" class="muted"></div>
      </div>
      <button id="close">关闭</button>
    </div>

    <div class="sec">
      <h3>官网</h3>
      <div id="d_web"></div>
    </div>

    <div class="sec">
      <h3>科系 / 课程</h3>
      <div id="d_prog" class="muted"></div>
    </div>

    <div class="sec">
      <h3>学费</h3>
      <div id="d_tuition" class="muted"></div>
    </div>

    <div class="sec">
      <h3>奖学金</h3>
      <div id="d_sch" class="muted"></div>
    </div>

    <div class="sec">
      <h3>实习 / Industrial Training</h3>
      <div id="d_intern" class="muted"></div>
    </div>

    <div class="sec">
      <h3>合作企业 / Industry Partners</h3>
      <div id="d_partners" class="muted"></div>
    </div>

    <div class="sec">
      <h3>来源（建议只放官方网页/官方PDF/年报）</h3>
      <ul id="d_sources" class="list"></ul>
    </div>

    <div class="muted" id="d_verified" style="margin-top:8px"></div>
  </div>
</dialog>

<script src="app.js"></script>
</body>
</html>

let DATA = [];

const el = (id) => document.getElementById(id);

function uniq(arr){ return [...new Set(arr.filter(Boolean))].sort(); }

function hasFlag(u, flag){
  if(flag === "scholarship") return (u.scholarships || []).length > 0;
  if(flag === "internship") return !!(u.internships && (u.internships.has_industrial_training || (u.internships.source||[]).length));
  if(flag === "partners") return (u.industry_partners || []).length > 0;
  if(flag === "tuition") return !!(u.tuition && (u.tuition.range_per_year || (u.tuition.source||[]).length));
  return true;
}

function textBlob(u){
  const parts = [];
  parts.push(u.name, u.type, u.state, u.website);
  (u.programmes||[]).forEach(p=>{
    parts.push(p.area);
    (p.courses||[]).forEach(c=>parts.push(c));
  });
  (u.industry_partners||[]).forEach(p=>parts.push(p.name));
  (u.scholarships||[]).forEach(s=>parts.push(s.name, s.eligibility, s.value));
  return parts.filter(Boolean).join(" ").toLowerCase();
}

function render(){
  const q = el("q").value.trim().toLowerCase();
  const type = el("type").value;
  const state = el("state").value;
  const flag = el("flags").value;

  const list = DATA.filter(u=>{
    if(type && u.type !== type) return false;
    if(state && u.state !== state) return false;
    if(flag && !hasFlag(u, flag)) return false;
    if(q){
      const blob = textBlob(u);
      if(!blob.includes(q)) return false;
    }
    return true;
  });

  el("count").textContent = `结果：${list.length} / ${DATA.length}`;

  const grid = el("grid");
  grid.innerHTML = "";
  list.forEach(u=>{
    const div = document.createElement("div");
    div.className = "card";
    const pills = [
      `<span class="pill">${u.type || "—"}</span>`,
      u.state ? `<span class="pill">${u.state}</span>` : ""
    ].join("");

    const tags = [];
    if((u.scholarships||[]).length) tags.push("奖学金");
    if(u.internships && (u.internships.has_industrial_training || (u.internships.source||[]).length)) tags.push("实习");
    if((u.industry_partners||[]).length) tags.push("合作企业");
    if(u.tuition && (u.tuition.range_per_year || (u.tuition.source||[]).length)) tags.push("学费");
    const tagLine = tags.length ? tags.map(t=>`<span class="pill">${t}</span>`).join("") : `<span class="muted">（信息待补齐）</span>`;

    div.innerHTML = `
      <div style="display:flex;justify-content:space-between;gap:8px">
        <div style="font-weight:700">${u.name}</div>
      </div>
      <div style="margin:6px 0">${pills}</div>
      <div class="muted" style="margin:6px 0 10px;line-height:1.35">
        ${u.website ? u.website : ""}
      </div>
      <div>${tagLine}</div>
      <div class="muted" style="margin-top:8px">最后核对：${u.last_verified || "—"}</div>
    `;
    div.onclick = ()=>openDetail(u);
    grid.appendChild(div);
  });
}

function toLinks(items){
  if(!items || !items.length) return `<span class="muted">暂无</span>`;
  return `<ul class="list">` + items.map(x=>{
    if(typeof x === "string") return `<li>${x}</li>`;
    const title = x.title || x.name || x.url || "link";
    return `<li><a href="${x.url}" target="_blank" rel="noopener">${title}</a></li>`;
  }).join("") + `</ul>`;
}

function openDetail(u){
  el("d_name").textContent = u.name;
  el("d_meta").textContent = `${u.type || "—"} · ${u.state || "—"}`;

  el("d_web").innerHTML = u.website
    ? `<a href="${u.website}" target="_blank" rel="noopener">${u.website}</a>`
    : `<span class="muted">暂无</span>`;

  // programmes
  if((u.programmes||[]).length){
    const html = u.programmes.map(p=>{
      const courses = (p.courses||[]).length ? p.courses.join(" / ") : "—";
      const mqa = (p.mqa_links||[]).map(url=>`<a href="${url}" target="_blank" rel="noopener">MQR</a>`).join(" ");
      return `<div><b>${p.area || "—"}</b><div class="muted">${courses} ${mqa ? " · " + mqa : ""}</div></div>`;
    }).join("<div style='height:8px'></div>");
    el("d_prog").innerHTML = html;
  } else {
    el("d_prog").innerHTML = `<span class="muted">暂无（可先从学院/课程目录页补齐）</span>`;
  }

  // tuition
  if(u.tuition){
    const r = u.tuition.range_per_year;
    const range = r ? `${u.tuition.currency || "MYR"} ${r.min ?? "?"} – ${r.max ?? "?"} / year` : "—";
    el("d_tuition").innerHTML = `<div>${range}</div>${toLinks(u.tuition.source || [])}`;
  } else el("d_tuition").innerHTML = `<span class="muted">暂无</span>`;

  // scholarships
  if((u.scholarships||[]).length){
    el("d_sch").innerHTML = `<ul class="list">` + u.scholarships.map(s=>{
      const link = s.url ? ` <a href="${s.url}" target="_blank" rel="noopener">详情</a>` : "";
      const meta = [s.value, s.eligibility].filter(Boolean).join(" · ");
      return `<li><b>${s.name || "Scholarship"}</b> <span class="muted">${meta}</span>${link}</li>`;
    }).join("") + `</ul>`;
  } else el("d_sch").innerHTML = `<span class="muted">暂无</span>`;

  // internships
  if(u.internships){
    const t = u.internships.has_industrial_training ? "有工业培训/实习模块" : "（未标注是否必修）";
    el("d_intern").innerHTML = `<div>${t}</div>${toLinks(u.internships.source || [])}`;
  } else el("d_intern").innerHTML = `<span class="muted">暂无</span>`;

  // partners
  if((u.industry_partners||[]).length){
    el("d_partners").innerHTML = `<ul class="list">` + u.industry_partners.map(p=>{
      const link = p.evidence_url ? ` <a href="${p.evidence_url}" target="_blank" rel="noopener">证据</a>` : "";
      return `<li>${p.name || "Partner"}${link}</li>`;
    }).join("") + `</ul>`;
  } else el("d_partners").innerHTML = `<span class="muted">暂无</span>`;

  // sources
  const sources = (u.sources || []).map(s=>({title: s.title, url: s.url}));
  el("d_sources").innerHTML = "";
  if(sources.length){
    sources.forEach(s=>{
      const li = document.createElement("li");
      li.innerHTML = `<a href="${s.url}" target="_blank" rel="noopener">${s.title || s.url}</a>`;
      el("d_sources").appendChild(li);
    });
  } else {
    const li = document.createElement("li");
    li.innerHTML = `<span class="muted">暂无</span>`;
    el("d_sources").appendChild(li);
  }

  el("d_verified").textContent = `最后核对日期：${u.last_verified || "—"}`;
  el("dlg").showModal();
}

async function init(){
  const res = await fetch("universities.json");
  DATA = await res.json();

  // state filter options
  const states = uniq(DATA.map(u=>u.state));
  const stateSel = el("state");
  states.forEach(s=>{
    const opt = document.createElement("option");
    opt.value = s; opt.textContent = s;
    stateSel.appendChild(opt);
  });

  ["q","type","state","flags"].forEach(id=>el(id).addEventListener("input", render));
  el("reset").onclick = ()=>{
    el("q").value=""; el("type").value=""; el("state").value=""; el("flags").value="";
    render();
  };
  el("close").onclick = ()=>el("dlg").close();

  render();
}

init();

[
  {
    "id": "um",
    "name": "Universiti Malaya",
    "type": "Public",
    "state": "Kuala Lumpur",
    "website": "https://www.um.edu.my",
    "programmes": [],
    "tuition": null,
    "scholarships": [],
    "internships": null,
    "industry_partners": [],
    "sources": [],
    "last_verified": "2026-01-19"
  },
  {
    "id": "apu",
    "name": "Asia Pacific University of Technology & Innovation (APU)",
    "type": "Private",
    "state": "Kuala Lumpur",
    "website": "https://www.apu.edu.my",
    "programmes": [
      {"area":"Computing / IT", "courses":["Software Engineering","Cyber Security"], "mqa_links":[]}
    ],
    "tuition": {
      "currency": "MYR",
      "range_per_year": null,
      "source": []
    },
    "scholarships": [],
    "internships": {
      "has_industrial_training": true,
      "source": []
    },
    "industry_partners": [],
    "sources": [],
    "last_verified": "2026-01-19"
  }
]
