<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        :root { --accent: #f06414; --bg: #05070a; --card: #151921; --pos: #00ff9d; --neg: #ff3e3e; --text: #e0e6ed; }
        body { font-family: -apple-system, sans-serif; background: var(--bg); color: var(--text); padding: 5px; margin: 0; text-align: center; -webkit-user-select: none; }
        
        .tabs { display: flex; gap: 2px; position: sticky; top: 0; background: var(--bg); z-index: 100; padding: 5px 0; }
        .tab-btn { flex: 1; padding: 12px 0; background: var(--card); border: 1px solid #2d3748; color: #718096; border-radius: 4px; font-size: 10px; font-weight: bold; }
        .tab-btn.active { background: var(--accent); color: #fff; border-color: #fff; }

        .role-section { display: none; grid-template-columns: repeat(4, 1fr); gap: 4px; padding: 5px 0; }
        .role-section.active { display: grid; }
        
        .btn { background: #1a202c; border: 1px solid #2d3748; border-radius: 6px; padding: 5px 1px; cursor: pointer; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 75px; }
        .btn-img { width: 48px; height: 48px; border-radius: 4px; object-fit: cover; margin-bottom: 3px; background: #2d3748; display: flex; align-items: center; justify-content: center; overflow: hidden; }
        .btn-img img { width: 100%; height: 100%; object-fit: cover; }
        .btn-name { font-size: 8px; font-weight: bold; width: 100%; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        
        .btn.active { background: var(--accent); border-color: #fff; box-shadow: 0 0 10px rgba(240,100,20,0.5); }

        #res-container { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-top: 10px; }
        .res-box { background: var(--card); padding: 8px; border-radius: 8px; border-top: 3px solid; min-height: 150px; text-align: left; }
        .res-title { font-size: 10px; font-weight: bold; margin-bottom: 8px; text-transform: uppercase; }
        
        .row { display: flex; align-items: center; justify-content: space-between; padding: 5px 0; border-bottom: 1px solid #2d3748; }
        .row-left { display: flex; align-items: center; gap: 5px; font-size: 10px; }
        .mini-img { width: 22px; height: 22px; border-radius: 2px; overflow: hidden; background: #2d3748; display: flex; align-items: center; justify-content: center; }
        .mini-img img { width: 100%; height: 100%; object-fit: cover; }
        
        .tag { font-weight: bold; font-size: 9px; padding: 1px 4px; border-radius: 4px; }
        .reset { width: 100%; margin-top: 10px; padding: 15px; background: #2d3748; color: #fff; border: none; border-radius: 6px; font-weight: bold; }
    </style>
</head>
<body>
    <div class="tabs">
        <div class="tab-btn active" onclick="tab('T')">TANK</div>
        <div class="tab-btn" onclick="tab('D')">DPS</div>
        <div class="tab-btn" onclick="tab('S')">SUP</div>
    </div>

    <div id="T" class="role-section active"></div>
    <div id="D" class="role-section"></div>
    <div id="S" class="role-section"></div>

    <div id="res-container">
        <div class="res-box" style="border-color: var(--pos);"><div id="pos-list"></div></div>
        <div class="res-box" style="border-color: var(--neg);"><div id="neg-list"></div></div>
    </div>

    <button class="reset" onclick="location.reload()">RESET SYSTEM</button>

<script>
// 送っていただいたリーパーのBase64データ
const reaperImg = "data:image/png;base64,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";

const heroDB = {
    // TANK
    "オリーサ": { r: "T", i: "", c: ["ザリア", "シンメトラ", "ファラ", "エコー"] },
    "ザリア": { r: "T", i: "", c: ["ラインハルト", "ウィンストン", "ファラ", "バスティオン", "リーパー"] },
    "マウガ": { r: "T", i: "", c: ["ディーバ", "シグマ", "バスティオン", "メイ", "アナ", "ゼニヤッタ"] },
    "ロードホッグ": { r: "T", i: "", c: ["ザリア", "ディーバ", "リーパー", "アナ", "ゼニヤッタ"] },
    "ディーバ": { r: "T", i: "", c: ["ザリア", "シグマ", "ベンチャー", "シンメトラ", "ブリギッテ"] },
    "ウィンストン": { r: "T", i: "", c: ["ロードホッグ", "リーパー", "バスティオン", "アナ", "ゼニヤッタ"] },
    "ドゥーム": { r: "T", i: "", c: ["オリーサ", "ロードホッグ", "ソンブラ", "キャスディ", "ファラ", "エコー", "アナ", "ゼニヤッタ"] },
    "ハザード": { r: "T", i: "", c: ["ディーバ", "ファラ", "エコー", "ソンブラ", "アナ", "ゼニヤッタ"] },
    "ボール": { r: "T", i: "", c: ["ロードホッグ", "マウガ", "ソンブラ", "メイ", "キャスディ", "ブリギッテ"] },
    "シグマ": { r: "T", i: "", c: ["ウィンストン", "ラインハルト", "ヴェンデッタ", "エコー", "シンメトラ", "メイ", "ソンブラ", "モイラ", "ルシオ"] },
    "JQ": { r: "T", i: "", c: ["ザリア", "メイ", "アッシュ", "アナ", "キリコ"] },
    "ラインハルト": { r: "T", i: "", c: ["オリーサ", "ラマットラ", "エコー", "ジャンクラット", "バスティオン", "ファラ"] },
    "ラマットラ": { r: "T", i: "", c: ["ロードホッグ", "オリーサ", "リーパー", "ファラ", "バスティオン", "アナ", "ゼニヤッタ"] },

    // DPS
    "リーパー": { r: "D", i: reaperImg, c: ["ディーバ", "オリーサ", "ファラ", "エコー", "キャスディ", "ジャンクラット", "アナ", "キリコ", "ルシオ", "ゼニヤッタ"] },
    "ヴェンデッタ": { r: "D", i: "", c: ["ロードホッグ", "エコー", "トールビョーン", "ファラ", "フレイヤ"] },
    "アッシュ": { r: "D", i: "", c: ["ディーバ", "ドゥーム", "ウィンストン", "ゲンジ", "ウィドウ"] },
    "ウィドウ": { r: "D", i: "", c: ["ディーバ", "ウィンストン", "シグマ", "ドゥーム", "ボール", "ゲンジ", "ソンブラ", "トレーサー", "ハンゾー", "ルシオ"] },
    "キャスディ": { r: "D", i: "", c: ["ディーバ", "ラマットラ", "ウィドウ", "アナ"] },
    "ソジョーン": { r: "D", i: "", c: ["ディーバ", "シグマ", "ウィンストン", "ウィドウ", "ゲンジ", "ソンブラ", "ルシオ"] },
    "ハンゾー": { r: "D", i: "", c: ["ディーバ", "シグマ", "ゲンジ", "ファラ", "エコー"] },
    "ゲンジ": { r: "D", i: "", c: ["ウィンストン", "ザリア", "ロードホッグ", "エコー", "トールビョーン", "シンメトラ", "ファラ", "メイ", "ブリギッテ", "モイラ"] },
    "トレーサー": { r: "D", i: "", c: ["ディーバ", "ザリア", "ロードホッグ", "キャスディ", "トールビョーン", "ベンチャー", "ファラ", "キリコ", "ブリギッテ", "イラリー"] },
    "ベンチャー": { r: "D", i: "", c: ["ロードホッグ", "ファラ", "エコー", "キャスディ"] },
    "ジャンクラット": { r: "D", i: "", c: ["ザリア", "ファラ", "エコー", "ジュノ", "バティスト"] },
    "シンメトラ": { r: "D", i: "", c: ["JQ", "エコー", "ファラ", "ソンブラ", "ソジョーン", "フレイヤ", "ウーヤン"] },
    "ソルジャー": { r: "D", i: "", c: ["ディーバ", "ハザード", "ウィドウ", "ハンゾー", "ゲンジ"] },
    "トールビョーン": { r: "D", i: "", c: ["ザリア", "ラマットラ", "ソジョーン", "ハンゾー", "ファラ", "フレイヤ", "ウーヤン", "ゼニヤッタ"] },
    "バスティオン": { r: "D", i: "", c: ["ディーバ", "シグマ", "ゲンジ", "ゼニヤッタ"] },
    "メイ": { r: "D", i: "", c: ["オリーサ", "ザリア", "アッシュ", "ウィドウ", "ファラ", "フレイヤ", "キリコ", "バティスト", "花"] },
    "エコー": { r: "D", i: "", c: ["マウガ", "ディーバ", "ウィドウ", "アッシュ", "キャスディ", "バティスト", "アナ"] },
    "ソンブラ": { r: "D", i: "", c: ["ウィンストン", "ディーバ", "ザリア", "ハンゾー", "トールビョーン", "キャスディ", "キリコ", "ブリギッテ"] },
    "ファラ": { r: "D", i: "", c: ["ディーバ", "マウガ", "ウィドウ", "アッシュ", "キャスディ", "エコー", "ソンブラ", "アナ", "バティスト", "イラリー", "ジュノ"] },
    "フレイヤ": { r: "D", i: "", c: ["ディーバ", "ウィドウ", "キャスディ", "ソルジャー", "キリコ", "バティスト"] },

    // SUPPORT
    "アナ": { r: "S", i: "", c: ["ディーバ", "ウィンストン", "ザリア", "ウィドウ", "キリコ"] },
    "ゼニヤッタ": { r: "S", i: "", c: ["ゲンジ", "ソンブラ", "トレーサー", "ウーヤン"] },
    "バティスト": { r: "S", i: "", c: ["ディーバ", "ロードホッグ", "ボール", "ハンゾー", "ウィドウ"] },
    "ルシオ": { r: "S", i: "", c: ["ドゥーム", "キャスディ", "エコー", "ファラ", "フレイヤ"] },
    "キリコ": { r: "S", i: "", c: ["ウィンストン", "ファラ"] },
    "マーシー": { r: "S", i: "", c: ["ディーバ", "ロードホッグ", "アッシュ", "キャスディ", "アナ", "イラリー", "バティスト"] },
    "モイラ": { r: "S", i: "", c: ["ディーバ", "キャスディ", "トールビョーン"] },
    "花": { r: "S", i: "", c: ["ウィンストン", "エコー", "ハンゾー", "ファラ", "フレイヤ"] },
    "イラリー": { r: "S", i: "", c: ["ディーバ", "シグマ", "ザリア", "ゲンジ", "キリコ"] },
    "ジュノ": { r: "S", i: "", c: ["ディーバ", "ウィンストン", "ソンブラ", "キャスディ", "ウィドウ", "アナ"] },
    "ブリギッテ": { r: "S", i: "", c: ["ラマットラ", "ファラ", "エコー", "ジャンクラット", "バスティオン", "フレイヤ"] },
    "ウーヤン": { r: "S", i: "", c: ["ディーバ", "エコー", "ゲンジ", "ファラ", "マーシー"] }
};

let sel = [];

function tab(r) {
    document.querySelectorAll('.role-section, .tab-btn').forEach(e => e.classList.remove('active'));
    document.getElementById(r).classList.add('active');
    event.currentTarget.classList.add('active');
}

function init() {
    for (let name in heroDB) {
        let btn = document.createElement('div');
        btn.className = 'btn';
        let imgTag = heroDB[name].i 
            ? `<img src="${heroDB[name].i}">` 
            : `<span>${name[0]}</span>`;
        
        btn.innerHTML = `<div class="btn-img">${imgTag}</div><span class="btn-name">${name}</span>`;
        btn.onclick = function() {
            this.classList.toggle('active');
            let i = sel.indexOf(name);
            if (i > -1) sel.splice(i, 1); else sel.push(name);
            update();
        };
        document.getElementById(heroDB[name].r).appendChild(btn);
    }
}

function update() {
    let pos = {}, neg = {};
    sel.forEach(en => {
        if(heroDB[en]) {
            heroDB[en].c.forEach(ally => pos[ally] = (pos[ally] || 0) + 1);
        }
        for (let hName in heroDB) {
            if (heroDB[hName].c.includes(en)) neg[hName] = (neg[hName] || 0) + 1;
        }
    });

    const draw = (id, data, title, color) => {
        const el = document.getElementById(id);
        if (sel.length === 0) { el.innerHTML = `<div class="res-title" style="color:${color}">${title}</div>`; return; }
        let sorted = Object.entries(data).sort((a,b) => b[1]-a[1]);
        el.innerHTML = `<div class="res-title" style="color:${color}">${title}</div>` + sorted.map(x => {
            let hero = heroDB[x[0]];
            let miniImg = hero && hero.i ? `<img src="${hero.i}">` : `<span>${x[0][0]}</span>`;
            return `
                <div class="row">
                    <div class="row-left"><div class="mini-img">${miniImg}</div><span>${x[0]}</span></div>
                    <span class="tag" style="background:${color}22; color:${color}">+${x[1]}</span>
                </div>
            `;
        }).join("");
    };

    draw('pos-list', pos, 'BEST PICKS', 'var(--pos)');
    draw('neg-list', neg, 'AVOID', 'var(--neg)');
}

init();
</script>
</body>
</html>
