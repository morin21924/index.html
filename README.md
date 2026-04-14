<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <!-- リファラを隠してブロックを回避 -->
    <meta name="referrer" content="no-referrer">
    <style>
        :root { --accent: #f06414; --bg: #05070a; --card: #151921; --pos: #00ff9d; --neg: #ff3e3e; --text: #e0e6ed; }
        body { font-family: -apple-system, sans-serif; background: var(--bg); color: var(--text); padding: 5px; margin: 0; text-align: center; -webkit-user-select: none; }
        
        .tabs { display: flex; gap: 2px; position: sticky; top: 0; background: var(--bg); z-index: 100; padding: 5px 0; }
        .tab-btn { flex: 1; padding: 12px 0; background: var(--card); border: 1px solid #2d3748; color: #718096; border-radius: 4px; font-size: 10px; font-weight: bold; }
        .tab-btn.active { background: var(--accent); color: #fff; border-color: #fff; }

        .role-section { display: none; grid-template-columns: repeat(4, 1fr); gap: 4px; padding: 5px 0; }
        .role-section.active { display: grid; }
        
        .btn { background: #1a202c; border: 1px solid #2d3748; border-radius: 6px; padding: 5px 1px; cursor: pointer; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 75px; }
        .btn-img { width: 44px; height: 44px; border-radius: 4px; object-fit: cover; margin-bottom: 3px; background: #000; border: 1px solid #333; }
        .btn-name { font-size: 8px; font-weight: bold; width: 100%; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        
        .btn.active { background: var(--accent); border-color: #fff; box-shadow: 0 0 10px rgba(240,100,20,0.5); }

        #res-container { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-top: 10px; }
        .res-box { background: var(--card); padding: 8px; border-radius: 8px; border-top: 3px solid; min-height: 150px; text-align: left; }
        .res-title { font-size: 10px; font-weight: bold; margin-bottom: 8px; text-transform: uppercase; }
        
        .row { display: flex; align-items: center; justify-content: space-between; padding: 5px 0; border-bottom: 1px solid #2d3748; }
        .row-left { display: flex; align-items: center; gap: 5px; font-size: 10px; }
        .mini-img { width: 20px; height: 20px; border-radius: 2px; object-fit: cover; background: #000; }
        
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
 * iPhone対策の核心：大手サービスのプロキシを経由させる
 * i0.wp.com は WordPress の配信サーバーで、iPhone の Safari でもブロックされにくい性質があります。
 */
function getImg(id) {
    if(!id) return "";
    // 安定した画像ソース (Overfast API)
    const url = `overfast-api.tekrop.fr/static/heroes/icons/${id}.webp`;
    // WordPressのCDNを「中継点」として使う（これがiPhoneで表示させるコツ）
    return `https://i0.wp.com/${url}`;
}

const heroDB = {
    // TANK
    "オリーサ": { r: "T", i: getImg("orisa"), c: ["ザリア", "シンメトラ", "ファラ", "エコー"] },
    "ザリア": { r: "T", i: getImg("zarya"), c: ["ラインハルト", "ウィンストン", "ファラ", "バスティオン", "リーパー"] },
    "マウガ": { r: "T", i: getImg("mauga"), c: ["ディーバ", "シグマ", "バスティオン", "メイ", "アナ", "ゼニヤッタ"] },
    "ロードホッグ": { r: "T", i: getImg("roadhog"), c: ["ザリア", "ディーバ", "リーパー", "アナ", "ゼニヤッタ"] },
    "ディーバ": { r: "T", i: getImg("d-va"), c: ["ザリア", "シグマ", "ベンチャー", "シンメトラ", "ブリギッテ"] },
    "ウィンストン": { r: "T", i: getImg("winston"), c: ["ロードホッグ", "リーパー", "バスティオン", "アナ", "ゼニヤッタ"] },
    "ドゥーム": { r: "T", i: getImg("doomfist"), c: ["オリーサ", "ロードホッグ", "ソンブラ", "キャスディ", "ファラ", "エコー", "アナ", "ゼニヤッタ"] },
    "ハザード": { r: "T", i: getImg("hazard"), c: ["ディーバ", "ファラ", "エコー", "ソンブラ", "アナ", "ゼニヤッタ"] },
    "ボール": { r: "T", i: getImg("wrecking-ball"), c: ["ロードホッグ", "マウガ", "ソンブラ", "メイ", "キャスディ", "ブリギッテ"] },
    "シグマ": { r: "T", i: getImg("sigma"), c: ["ウィンストン", "ラインハルト", "ヴェンデッタ", "エコー", "シンメトラ", "メイ", "ソンブラ", "モイラ", "ルシオ"] },
    "JQ": { r: "T", i: getImg("junker-queen"), c: ["ザリア", "メイ", "アッシュ", "アナ", "キリコ"] },
    "ラインハルト": { r: "T", i: getImg("reinhardt"), c: ["オリーサ", "ラマットラ", "エコー", "ジャンクラット", "バスティオン", "ファラ"] },
    "ラマットラ": { r: "T", i: getImg("ramattra"), c: ["ロードホッグ", "オリーサ", "リーパー", "ファラ", "バスティオン", "アナ", "ゼニヤッタ"] },

    // DPS
    "ヴェンデッタ": { r: "D", i: getImg("reaper"), c: ["ロードホッグ", "エコー", "トールビョーン", "ファラ", "フレイヤ"] },
    "アッシュ": { r: "D", i: getImg("ashe"), c: ["ディーバ", "ドゥーム", "ウィンストン", "ゲンジ", "ウィドウ"] },
    "ウィドウ": { r: "D", i: getImg("widowmaker"), c: ["ディーバ", "ウィンストン", "シグマ", "ドゥーム", "ボール", "ゲンジ", "ソンブラ", "トレーサー", "ハンゾー", "ルシオ"] },
    "キャスディ": { r: "D", i: getImg("cassidy"), c: ["ディーバ", "ラマットラ", "ウィドウ", "アナ"] },
    "ソジョーン": { r: "D", i: getImg("sojourn"), c: ["ディーバ", "シグマ", "ウィンストン", "ウィドウ", "ゲンジ", "ソンブラ", "ルシオ"] },
    "ハンゾー": { r: "D", i: getImg("hanzo"), c: ["ディーバ", "シグマ", "ゲンジ", "ファラ", "エコー"] },
    "ゲンジ": { r: "D", i: getImg("genji"), c: ["ウィンストン", "ザリア", "ロードホッグ", "エコー", "トールビョーン", "シンメトラ", "ファラ", "メイ", "ブリギッテ", "モイラ"] },
    "トレーサー": { r: "D", i: getImg("tracer"), c: ["ディーバ", "ザリア", "ロードホッグ", "キャスディ", "トールビョーン", "ベンチャー", "ファラ", "キリコ", "ブリギッテ", "イラリー"] },
    "ベンチャー": { r: "D", i: getImg("venture"), c: ["ロードホッグ", "ファラ", "エコー", "キャスディ"] },
    "リーパー": { r: "D", i: getImg("reaper"), c: ["ディーバ", "オリーサ", "ファラ", "エコー", "キャスディ", "ジャンクラット", "アナ", "キリコ", "ルシオ", "ゼニヤッタ"] },
    "ジャンクラット": { r: "D", i: getImg("junkrat"), c: ["ザリア", "ファラ", "エコー", "ジュノ", "バティスト"] },
    "シンメトラ": { r: "D", i: getImg("symmetra"), c: ["JQ", "エコー", "ファラ", "ソンブラ", "ソジョーン", "フレイヤ", "ウーヤン"] },
    "ソルジャー": { r: "D", i: getImg("soldier-76"), c: ["ディーバ", "ハザード", "ウィドウ", "ハンゾー", "ゲンジ"] },
    "トールビョーン": { r: "D", i: getImg("torbjorn"), c: ["ザリア", "ラマットラ", "ソジョーン", "ハンゾー", "ファラ", "フレイヤ", "ウーヤン", "ゼニヤッタ"] },
    "バスティオン": { r: "D", i: getImg("bastion"), c: ["ディーバ", "シグマ", "ゲンジ", "ゼニヤッタ"] },
    "メイ": { r: "D", i: getImg("mei"), c: ["オリーサ", "ザリア", "アッシュ", "ウィドウ", "ファラ", "フレイヤ", "キリコ", "バティスト", "花"] },
    "エコー": { r: "D", i: getImg("echo"), c: ["マウガ", "ディーバ", "ウィドウ", "アッシュ", "キャスディ", "バティスト", "アナ"] },
    "ソンブラ": { r: "D", i: getImg("sombra"), c: ["ウィンストン", "ディーバ", "ザリア", "ハンゾー", "トールビョーン", "キャスディ", "キリコ", "ブリギッテ"] },
    "ファラ": { r: "D", i: getImg("pharah"), c: ["ディーバ", "マウガ", "ウィドウ", "アッシュ", "キャスディ", "エコー", "ソンブラ", "アナ", "バティスト", "イラリー", "ジュノ"] },
    "フレイヤ": { r: "D", i: getImg("echo"), c: ["ディーバ", "ウィドウ", "キャスディ", "ソルジャー", "キリコ", "バティスト"] },

    // SUPPORT
    "アナ": { r: "S", i: getImg("ana"), c: ["ディーバ", "ウィンストン", "ザリア", "ウィドウ", "キリコ"] },
    "ゼニヤッタ": { r: "S", i: getImg("zenyatta"), c: ["ゲンジ", "ソンブラ", "トレーサー", "ウーヤン"] },
    "バティスト": { r: "S", i: getImg("baptiste"), c: ["ディーバ", "ロードホッグ", "ボール", "ハンゾー", "ウィドウ"] },
    "ルシオ": { r: "S", i: getImg("lucio"), c: ["ドゥーム", "キャスディ", "エコー", "ファラ", "フレイヤ"] },
    "キリコ": { r: "S", i: getImg("kiriko"), c: ["ウィンストン", "ファラ"] },
    "マーシー": { r: "S", i: getImg("mercy"), c: ["ディーバ", "ロードホッグ", "アッシュ", "キャスディ", "アナ", "イラリー", "バティスト"] },
    "モイラ": { r: "S", i: getImg("moira"), c: ["ディーバ", "キャスディ", "トールビョーン"] },
    "花": { r: "S", i: getImg("lifeweaver"), c: ["ウィンストン", "エコー", "ハンゾー", "ファラ", "フレイヤ"] },
    "イラリー": { r: "S", i: getImg("illari"), c: ["ディーバ", "シグマ", "ザリア", "ゲンジ", "キリコ"] },
    "ジュノ": { r: "S", i: getImg("juno"), c: ["ディーバ", "ウィンストン", "ソンブラ", "キャスディ", "ウィドウ", "アナ"] },
    "ブリギッテ": { r: "S", i: getImg("brigitte"), c: ["ラマットラ", "ファラ", "エコー", "ジャンクラット", "バスティオン", "フレイヤ"] },
    "ウーヤン": { r: "S", i: getImg("kiriko"), c: ["ディーバ", "エコー", "ゲンジ", "ファラ", "マーシー"] }
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
        // referrerpolicy="no-referrer" でiPhoneのブロックを回避
        btn.innerHTML = `<img src="${heroDB[name].i}" class="btn-img" referrerpolicy="no-referrer"><span class="btn-name">${name}</span>`;
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
                <div class="row-left">
                    <img src="${heroDB[x[0]] ? heroDB[x[0]].i : ''}" class="mini-img" referrerpolicy="no-referrer">
                    <span>${x[0]}</span>
                </div>
                <span class="tag" style="background:${color}22; color:${color}">+${x[1]}</span>
            </div>
        `).join("");
    };

    draw('pos-list', pos, 'BEST PICKS', 'var(--pos)');
    draw('neg-list', neg, 'AVOID', 'var(--neg)');
}

init();
</script>
</body>
</html>
