# StellaSoraBuildmaker
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>ステラソラ 巡遊者ビルドメモ</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <h1>巡遊者ビルドメモ</h1>

  <div class="selectors">
    <label>主力：
      <select id="mainSelect"></select>
    </label>
    <label>支援1：
      <select id="support1Select"></select>
    </label>
    <label>支援2：
      <select id="support2Select"></select>
    </label>
  </div>

  <label class="toggle">
    <input type="checkbox" id="hideUnselected" />
    取得しない素質を非表示（スクショ用）
  </label>

  <div id="talentArea"></div>

  <script src="script.js"></script>
</body>
</html>
body {
  font-family: sans-serif;
  padding: 16px;
}

h2 {
  margin-top: 24px;
}

.selectors label {
  margin-right: 12px;
}

table {
  border-collapse: collapse;
  margin-bottom: 16px;
}

td, th {
  border: 1px solid #ccc;
  padding: 6px 10px;
  text-align: center;
}

.talent {
  cursor: pointer;
  user-select: none;
}

/* コア素質 */
.core.on {
  background: #9be7a4;
}
.core.off {
  background: #eee;
}

/* サブ素質 */
.sub.none { background: #eee; }
.sub.low { background: #bde9ff; }
.sub.one { background: #7cbcff; color: #fff; }
.sub.mid { background: #ffb347; }
.sub.high { background: #ff6b6b; color: #fff; }

.toggle {
  display: block;
  margin: 12px 0;
}
/********************
 * 巡遊者データ
 ********************/
const characters = {
  "巡遊者A": {
    main: {
      core: [
        { id: "a_mc1", name: "コア1", image: "" },
        { id: "a_mc2", name: "コア2", image: "" },
        { id: "a_mc3", name: "コア3", image: "" },
        { id: "a_mc4", name: "コア4", image: "" }
      ],
      sub: [
        { id: "a_ms1", name: "サブA", image: "" },
        { id: "a_ms2", name: "サブB", image: "" }
      ]
    },
    support: {
      core: [
        { id: "a_sc1", name: "支援コア1", image: "" },
        { id: "a_sc2", name: "支援コア2", image: "" },
        { id: "a_sc3", name: "支援コア3", image: "" },
        { id: "a_sc4", name: "支援コア4", image: "" }
      ],
      sub: [
        { id: "a_ss1", name: "支援サブA", image: "" }
      ]
    }
  }
};

/********************
 * 定数
 ********************/
const subStates = [
  { key: "none", label: "取得しない" },
  { key: "low", label: "余裕あれば" },
  { key: "one", label: "1枚必須" },
  { key: "mid", label: "優先強化" },
  { key: "high", label: "6枚必須" }
];

const storageKey = "stellar_sola_memo";

/********************
 * 状態
 ********************/
let state = JSON.parse(localStorage.getItem(storageKey)) || {
  main: "",
  support1: "",
  support2: "",
  talents: {},
  hideUnselected: false
};

/********************
 * 初期化
 ********************/
const mainSelect = document.getElementById("mainSelect");
const support1Select = document.getElementById("support1Select");
const support2Select = document.getElementById("support2Select");
const talentArea = document.getElementById("talentArea");
const hideToggle = document.getElementById("hideUnselected");

function initSelectors() {
  const names = Object.keys(characters);
  [mainSelect, support1Select, support2Select].forEach(sel => {
    sel.innerHTML = "<option value=''>---</option>";
    names.forEach(n => {
      const o = document.createElement("option");
      o.value = n;
      o.textContent = n;
      sel.appendChild(o);
    });
  });

  mainSelect.value = state.main;
  support1Select.value = state.support1;
  support2Select.value = state.support2;
  hideToggle.checked = state.hideUnselected;
}

function save() {
  localStorage.setItem(storageKey, JSON.stringify(state));
}

/********************
 * 描画
 ********************/
function render() {
  talentArea.innerHTML = "";
  if (state.main) renderCharacter(state.main, "main", "主力");
  if (state.support1) renderCharacter(state.support1, "support", "支援1");
  if (state.support2) renderCharacter(state.support2, "support", "支援2");
}

function renderCharacter(name, role, label) {
  const data = characters[name][role];
  const block = document.createElement("div");
  block.innerHTML = `<h2>${label}：${name}</h2>`;

  // コア
  block.appendChild(makeTable(data.core, true));

  // サブ
  block.appendChild(makeTable(data.sub, false));

  talentArea.appendChild(block);
}

function makeTable(list, isCore) {
  const table = document.createElement("table");
  list.forEach(t => {
    const tr = document.createElement("tr");
    const tdName = document.createElement("td");
    const tdVal = document.createElement("td");

    tdName.textContent = t.name;

    const current = state.talents[t.id] ?? (isCore ? "off" : "none");

    if (state.hideUnselected && (current === "off" || current === "none")) {
      tr.style.display = "none";
    }

    tdVal.textContent = isCore
      ? (current === "on" ? "取得" : "未取得")
      : subStates.find(s => s.key === current).label;

    tdVal.className = `talent ${isCore ? "core" : "sub"} ${current}`;

    tdVal.onclick = () => {
      if (isCore) toggleCore(t.id, list);
      else toggleSub(t.id);
      save();
      render();
    };

    tr.append(tdName, tdVal);
    table.appendChild(tr);
  });
  return table;
}

/********************
 * 操作処理
 ********************/
function toggleCore(id, list) {
  const onCount = list.filter(
    t => state.talents[t.id] === "on"
  ).length;

  state.talents[id] =
    state.talents[id] === "on"
      ? "off"
      : onCount < 2
      ? "on"
      : state.talents[id] || "off";
}

function toggleSub(id) {
  const idx = subStates.findIndex(s => s.key === (state.talents[id] || "none"));
  state.talents[id] = subStates[(idx + 1) % subStates.length].key;
}

/********************
 * イベント
 ********************/
mainSelect.onchange = () => {
  state.main = mainSelect.value;
  save();
  render();
};
support1Select.onchange = () => {
  state.support1 = support1Select.value;
  save();
  render();
};
support2Select.onchange = () => {
  state.support2 = support2Select.value;
  save();
  render();
};
hideToggle.onchange = () => {
  state.hideUnselected = hideToggle.checked;
  save();
  render();
};

/********************
 * 起動
 ********************/
initSelectors();
render();
