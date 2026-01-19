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
