<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="referrer" content="no-referrer">
    <style>
        :root { --accent: #f06414; --bg: #05070a; --card: #151921; --pos: #00ff9d; --neg: #ff3e3e; --text: #e0e6ed; }
        body { font-family: 'Avenir Next', -apple-system, sans-serif; background: var(--bg); color: var(--text); padding: 5px; margin: 0; text-align: center; -webkit-user-select: none; }
        .tabs { display: flex; gap: 2px; position: sticky; top: 0; background: var(--bg); z-index: 100; padding: 5px 0; }
        .tab-btn { flex: 1; padding: 12px 0; background: var(--card); border: 1px solid #2d3748; color: #718096; border-radius: 4px; font-size: 10px; font-weight: bold; }
        .tab-btn.active { background: var(--accent); color: #fff; border-color: #fff; }
        .role-section { display: none; grid-template-columns: repeat(4, 1fr); gap: 4px; padding: 5px 0; }
        .role-section.active { display: grid; }
        .btn { background: #1a202c; border: 1px solid #2d3748; border-radius: 6px; padding: 4px 1px; cursor: pointer; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 75px; }
        .btn-img { width: 48px; height: 48px; border-radius: 4px; object-fit: cover; margin-bottom: 2px; background: #000; display: block; }
        .btn-name { font-size: 8px; font-weight: bold; width: 100%; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        .btn.active { background: var(--accent); border-color: #fff; box-shadow: 0 0 10px rgba(240,100,20,0.5); }
        #res-container { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-top: 10px; }
        .res-box { background: var(--card); padding: 10px; border-radius: 8px; border-top: 3px solid; min-height: 150px; text-align: left; }
        .res-title { font-size: 10px; font-weight: bold; margin-bottom: 8px; text-transform: uppercase; }
        .row { display: flex; align-items: center; justify-content: space-between; padding: 5px 0; border-bottom: 1px solid #2d3748; }
        .row-left { display: flex; align-items: center; gap: 5px; font-size: 11px; }
        .mini-img { width: 20px; height: 20px; border-radius: 2px; object-fit: cover; }
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
/**
 * iPhone最強対策：WordPressの画像配信プロキシ(i0.wp.com)を経由させる
 * これにより、iPhoneのSafariは「WordPressの画像」だと認識し、ブロックを解除します
 */
function getIcon(id) {
    if(!id) return "https://via.placeholder.com/100/1a202c/ffffff?text=?";
    const rawUrl = `raw.githubusercontent.com/fomm/overwatch-hero-icons/master/icons/${id}.png`;
    return `https://i0.wp.com/${rawUrl}`;
}

const heroDB = {
    "オリーサ": { r: "T", i: getIcon("orisa"), c: ["ザリア", "シンメトラ", "ファラ", "エコー"] },
    "ザリア": { r: "T", i: getIcon("zarya"), c: ["ラインハルト", "ウィンストン", "ファラ", "バスティオン", "リーパー"] },
    "マウガ": { r: "T", i: getIcon("mauga"), c: ["ディーバ", "シグマ", "バスティオン", "メイ", "アナ", "ゼニヤッタ"] },
    "ロードホッグ": { r: "T", i: getIcon("roadhog"), c: ["ザリア", "ディーバ", "リーパー", "アナ", "ゼニヤッタ"] },
    "ディーバ": { r: "T", i: getIcon("dva"), c: ["ザリア", "シグマ", "ベンチャー", "シンメトラ", "ブリギッテ"] },
    "ウィンストン": { r: "T", i: getIcon("winston"), c: ["ロードホッグ", "リーパー", "バスティオン", "アナ", "ゼニヤッタ"] },
    "ドゥーム": { r: "T", i: getIcon("doomfist"), c: ["オリーサ", "ロードホッグ", "ソンブラ", "キャスディ", "ファラ", "エコー", "アナ", "ゼニヤッタ"] },
    "ボール": { r: "T", i: getIcon("wrecking-ball"), c: ["ロードホッグ", "マウガ", "ソンブラ", "メイ", "キャスディ", "ブリギッテ"] },
    "シグマ": { r: "T", i: getIcon("sigma"), c: ["ウィンストン", "ラインハルト", "ヴェンデッタ", "エコー", "シンメトラ", "メイ", "ソンブラ", "モイラ", "ルシオ"] },
    "JQ": { r: "T", i: getIcon("junker-queen"), c: ["ザリア", "メイ", "アッシュ", "アナ", "キリコ"] },
    "ラインハルト": { r: "T", i: getIcon("reinhardt"), c: ["オリーサ", "ラマットラ", "エコー", "ジャンクラット", "バスティオン", "ファラ"] },
    "ラマットラ": { r: "T", i: getIcon("ramattra"), c: ["ロードホッグ", "オリーサ", "リーパー", "ファラ", "バスティオン", "アナ", "ゼニヤッタ"] },
    "ハザード": { r: "T", i: "https://i0.wp.com/static.wikia.nocookie.net/overwatch_gamepedia/images/1/1a/Hazard_Portrait.png", c: ["ディーバ", "ファラ", "エコー", "ソンブラ", "アナ", "ゼニヤッタ"] },

    "アッシュ": { r: "D", i: getIcon("ashe"), c: ["ディーバ", "ドゥーム", "ウィンストン", "ゲンジ", "ウィドウ"] },
    "ウィドウ": { r: "D", i: getIcon("widowmaker"), c: ["ディーバ", "ウィンストン", "シグマ", "ドゥーム", "ボール", "ゲンジ", "ソンブラ", "トレーサー", "ハンゾー", "ルシオ"] },
    "キャスディ": { r: "D", i: getIcon("cassidy"), c: ["ディーバ", "ラマットラ", "ウィドウ", "アナ"] },
    "ソジョーン": { r: "D", i: getIcon("sojourn"), c: ["ディーバ", "シグマ", "ウィンストン", "ウィドウ", "ゲンジ", "ソンブラ", "ルシオ"] },
    "ハンゾー": { r: "D", i: getIcon("hanzo"), c: ["ディーバ", "シグマ", "ゲンジ", "ファラ", "エコー"] },
    "ゲンジ": { r: "D", i: getIcon("genji"), c: ["ウィンストン", "ザリア", "ロードホッグ", "エコー", "トールビョーン", "シンメトラ", "ファラ", "メイ", "ブリギッテ", "モイラ"] },
    "トレーサー": { r: "D", i: getIcon("tracer"), c: ["ディーバ", "ザリア", "ロードホッグ", "キャスディ", "トールビョーン", "ベンチャー", "ファラ", "キリコ", "ブリギッテ", "イラリー"] },
    "ベンチャー": { r: "D", i: getIcon("venture"), c: ["ロードホッグ", "ファラ", "エコー", "キャスディ"] },
    "リーパー": { r: "D", i: getIcon("reaper"), c: ["ディーバ", "オリーサ", "ファラ", "エコー", "キャスディ", "ジャンクラット", "アナ", "キリコ", "ルシオ", "ゼニヤッタ"] },
    "ジャンクラット": { r: "D", i: getIcon("junkrat"), c: ["ザリア", "ファラ", "エコー", "ジュノ", "バティスト"] },
    "シンメトラ": { r: "D", i: getIcon("symmetra"), c: ["JQ", "エコー", "ファラ", "ソンブラ", "ソジョーン", "フレイヤ", "ウーヤン"] },
    "ソルジャー": { r: "D", i: getIcon("soldier-76"), c: ["ディーバ", "ハザード", "ウィドウ", "ハンゾー", "ゲンジ"] },
    "トールビョーン": { r: "D", i: getIcon("torbjorn"), c: ["ザリア", "ラマットラ", "ソジョーン", "ハンゾー", "ファラ", "フレイヤ", "ウーヤン", "ゼニヤッタ"] },
    "バスティオン": { r: "D", i: getIcon("bastion"), c: ["ディーバ", "シグマ", "ゲンジ", "ゼニヤッタ"] },
    "メイ": { r: "D", i: getIcon("mei"), c: ["オリーサ", "ザリア", "アッシュ", "ウィドウ", "ファラ", "フレイヤ", "キリコ", "バティスト", "花"] },
    "エコー": { r: "D", i: getIcon("echo"), c: ["マウガ", "ディーバ", "ウィドウ", "アッシュ", "キャスディ", "バティスト", "アナ"] },
    "ソンブラ": { r: "D", i: getIcon("sombra"), c: ["ウィンストン", "ディーバ", "ザリア", "ハンゾー", "トールビョーン", "キャスディ", "キリコ", "ブリギッテ"] },
    "ファラ": { r: "D", i: getIcon("pharah"), c: ["ディーバ", "マウガ", "ウィドウ", "アッシュ", "キャスディ", "エコー", "ソンブラ", "アナ", "バティスト", "イラリー", "ジュノ"] },
    "ヴェンデッタ": { r: "D", i: "", c: ["ロードホッグ", "エコー", "トールビョーン", "ファラ", "フレイヤ"] },
    "フレイヤ": { r: "D", i: "", c: ["ディーバ", "ウィドウ", "キャスディ", "ソルジャー", "キリコ", "バティスト"] },

    "アナ": { r: "S", i: getIcon("ana"), c: ["ディーバ", "ウィンストン", "ザリア", "ウィドウ", "キリコ"] },
    "ゼニヤッタ": { r: "S", i: getIcon("zenyatta"), c: ["ゲンジ", "ソンブラ", "トレーサー", "ウーヤン"] },
    "バティスト": { r: "S", i: getIcon("baptiste"), c: ["ディーバ", "ロードホッグ", "ボール", "ハンゾー", "ウィドウ"] },
    "ルシオ": { r: "S", i: getIcon("lucio"), c: ["ドゥーム", "キャスディ", "エコー", "ファラ", "フレイヤ"] },
    "キリコ": { r: "S", i: getIcon("kiriko"), c: ["ウィンストン", "ファラ"] },
    "マーシー": { r: "S", i: getIcon("mercy"), c: ["ディーバ", "ロードホッグ", "アッシュ", "キャスディ", "アナ", "イラリー", "バティスト"] },
    "モイラ": { r: "S", i: getIcon("moira"), c: ["ディーバ", "キャスディ", "トールビョーン"] },
    "花": { r: "S", i: getIcon("lifeweaver"), c: ["ウィンストン", "エコー", "ハンゾー", "ファラ", "フレイヤ"] },
    "イラリー": { r: "S", i: getIcon("illari"), c: ["ディーバ", "シグマ", "ザリア", "ゲンジ", "キリコ"] },
    "ジュノ": { r: "S", i: getIcon("juno"), c: ["ディーバ", "ウィンストン", "ソンブラ", "キャスディ", "ウィドウ", "アナ"] },
    "ブリギッテ": { r: "S", i: getIcon("brigitte"), c: ["ラマットラ", "ファラ", "エコー", "ジャンクラット", "バスティオン", "フレイヤ"] },
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
        btn.innerHTML = `<img src="${heroDB[name].i}" class="btn-img" loading="lazy"><span class="btn-name">${name}</span>`;
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
        if(heroDB[en]) { heroDB[en].c.forEach(ally => pos[ally] = (pos[ally] || 0) + 1); }
        for (let hName in heroDB) { if (heroDB[hName].c.includes(en)) neg[hName] = (neg[hName] || 0) + 1; }
    });
    const draw = (id, data, title, color) => {
        const el = document.getElementById(id);
        if (sel.length === 0) { el.innerHTML = `<div class="res-title" style="color:${color}">${title}</div>`; return; }
        let sorted = Object.entries(data).sort((a,b) => b[1]-a[1]);
        el.innerHTML = `<div class="res-title" style="color:${color}">${title}</div>` + sorted.map(x => `
            <div class="row">
                <div class="row-left"><img src="${heroDB[x[0]] ? heroDB[x[0]].i : ''}" class="mini-img"><span>${x[0]}</span></div>
                <span class="tag" style="background:${color}22; color:${color}">+${x[1]}</span>
            </div>`).join("");
    };
    draw('pos-list', pos, 'BEST PICKS', 'var(--pos)');
    draw('neg-list', neg, 'AVOID', 'var(--neg)');
}
init();
</script>
</body>
</html>
