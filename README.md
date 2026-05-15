<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Super Platformer</title>
    <style>
        :root {
            --bg-color: #87CEEB; /* 空の青 */
            --main-color: #0000CD; /* UIの青 */
            --accent-color: #FF4500; /* アクセントのオレンジ */
            --text-color: #333333; /* ダークグレー */
            --coin-color: #FFD700; /* コインの黄色 */
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none; /* テキスト選択無効化 */
            -webkit-user-select: none;
        }

        body {
            background-color: #e0e0e0;
            color: var(--text-color);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            touch-action: none; /* スクロール防止 */
        }

        #game-container {
            position: relative;
            width: 100%;
            max-width: 1000px;
            height: 100vh;
            max-height: 600px;
            /* 背景はJSでステージに応じて動的に変更します */
            background: linear-gradient(to bottom, #87CEEB 0%, #E0F6FF 100%);
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
            overflow: hidden;
            transition: background 0.5s; /* 背景色切り替えを少し滑らかに */
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }

        /* UIオーバーレイ */
        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none; /* UIの奥をクリックできるようにする */
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
        }

        .header {
            display: flex;
            justify-content: space-between;
            font-size: 1.2rem;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            color: var(--main-color);
            text-shadow: 1px 1px 2px #fff;
        }

        /* メッセージ画面 (スタート, ゲームオーバー, クリア) */
        #message-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.85);
            backdrop-filter: blur(5px);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            pointer-events: auto;
            z-index: 10;
            transition: opacity 0.3s;
        }

        #message-screen.hidden {
            opacity: 0;
            pointer-events: none;
        }

        h1 {
            font-size: 3rem;
            color: var(--accent-color);
            text-shadow: 2px 2px 0px #fff, 4px 4px 0px rgba(0,0,0,0.1);
            margin-bottom: 20px;
            text-align: center;
        }

        p.inst {
            color: var(--text-color);
            margin-bottom: 30px;
            text-align: center;
            line-height: 1.5;
            font-weight: bold;
        }

        button.btn {
            background: var(--main-color);
            border: 2px solid var(--main-color);
            color: #fff;
            padding: 15px 40px;
            font-size: 1.2rem;
            font-weight: bold;
            text-transform: uppercase;
            cursor: pointer;
            border-radius: 30px;
            box-shadow: 0 5px 15px rgba(0, 0, 205, 0.3);
            transition: all 0.2s;
        }

        button.btn:hover, button.btn:active {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(0, 0, 205, 0.5);
        }

        /* モバイル用コントロール */
        #mobile-controls {
            position: absolute;
            bottom: 20px;
            left: 0;
            width: 100%;
            padding: 0 20px;
            display: none; /* デフォルトは非表示、タッチデバイスで表示 */
            justify-content: space-between;
            pointer-events: auto;
            z-index: 5;
        }

        .control-group {
            display: flex;
            gap: 15px;
        }

        .ctrl-btn {
            width: 60px;
            height: 60px;
            background: rgba(255, 255, 255, 0.5);
            border: 2px solid var(--main-color);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            color: var(--main-color);
            font-size: 24px;
            font-weight: bold;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
            -webkit-tap-highlight-color: transparent;
        }

        .ctrl-btn:active {
            background: rgba(255, 255, 255, 0.8);
            transform: translateY(2px);
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }

        @media (max-width: 768px) {
            #mobile-controls { display: flex; }
            .header { font-size: 1rem; }
            h1 { font-size: 2rem; }
            .ctrl-btn { width: 50px; height: 50px; font-size: 20px; } /* スマホでボタンがはみ出ないように少し縮小 */
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>
    
    <div id="ui-layer">
        <div class="header">
            <span id="score-display">SCORE: 0</span>
            <span id="deaths-display">DEATHS: 0</span>
        </div>
    </div>

    <div id="mobile-controls">
        <div class="control-group">
            <div class="ctrl-btn" id="btn-left">◀</div>
            <div class="ctrl-btn" id="btn-right">▶</div>
        </div>
        <div class="control-group">
            <div class="ctrl-btn" id="btn-down">▼</div>
            <div class="ctrl-btn" id="btn-jump">▲</div>
        </div>
    </div>

    <div id="message-screen">
        <h1 id="msg-title">SUPER PLATFORMER</h1>
        <p class="inst" id="msg-desc">
            [操作方法]<br>
            PC: 矢印キー または W/A/S/D<br>
            スマホ: 画面下のボタン<br><br>
            土管の上で「下」を押すとワープ！<br>
            ハテナブロックを下から叩いてコインをゲット！
        </p>
        <button class="btn" id="start-btn">GAME START</button>
    </div>
</div>

<script>
/**
 * スタイリッシュプラットフォーマー ゲームロジック
 */

// キャンバス設定
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const container = document.getElementById('game-container');

// UI要素
const msgScreen = document.getElementById('message-screen');
const msgTitle = document.getElementById('msg-title');
const msgDesc = document.getElementById('msg-desc');
const startBtn = document.getElementById('start-btn');
const scoreDisplay = document.getElementById('score-display');
const deathsDisplay = document.getElementById('deaths-display');

// リサイズ処理
let canvasWidth, canvasHeight;
function resize() {
    canvasWidth = container.clientWidth;
    canvasHeight = container.clientHeight;
    canvas.width = canvasWidth;
    canvas.height = canvasHeight;
}
window.addEventListener('resize', resize);
resize();

// ゲーム定数と設定
const PHYSICS = {
    gravity: 0.5,
    friction: 0.8,
    maxFallSpeed: 12
};

const THEME = {
    player: '#0000CD',      // 服の色 (青)
    skin: '#FFCC99',        // 肌の色
    ground: '#8B4513',      // 地面のブロック色 (土らしい濃い茶色)
    danger: '#FF4500',      // 危険物 (オレンジレッド)
    goal: '#FFD700',        // ゴール (ゴールド)
    particle: '#ffffff',
    item: '#FFD700',        // コイン
    brick: '#B22222',       // レンガ色
    question: '#FF8C00',    // ハテナブロック（ダークオレンジ）
    emptyBlock: '#8B4513',  // 叩いた後の空ブロック
    stone: '#A9A9A9',       // 硬い階段ブロック
    pipe: '#008000'         // 土管
};

// ゲーム状態管理
let gameState = 'START'; // START, PLAYING, WARPING_IN, WARPING_OUT, GAMEOVER, CLEAR
let currentStage = 'main'; // 'main' or 'underground'
let animationId;
let deaths = 0;
let score = 0;

// ワープ用変数
let warpTimer = 0;
let warpInfo = null;

// 雲の生成（背景装飾）
const clouds = [];
for (let i = 0; i < 15; i++) {
    clouds.push({
        x: Math.random() * 5000,
        y: Math.random() * 200 + 20,
        s: Math.random() * 0.8 + 0.4
    });
}

// 入力管理
const keys = {
    left: false,
    right: false,
    up: false,
    down: false
};

// キーボードイベント
window.addEventListener('keydown', (e) => {
    if (gameState !== 'PLAYING' && gameState !== 'WARPING_IN' && gameState !== 'WARPING_OUT') return;
    if (e.code === 'ArrowLeft' || e.code === 'KeyA') keys.left = true;
    if (e.code === 'ArrowRight' || e.code === 'KeyD') keys.right = true;
    if (e.code === 'ArrowDown' || e.code === 'KeyS') {
        keys.down = true;
        e.preventDefault(); // スクロール防止
    }
    if (e.code === 'ArrowUp' || e.code === 'KeyW' || e.code === 'Space') {
        keys.up = true;
        e.preventDefault(); // スクロール防止
    }
});

window.addEventListener('keyup', (e) => {
    if (e.code === 'ArrowLeft' || e.code === 'KeyA') keys.left = false;
    if (e.code === 'ArrowRight' || e.code === 'KeyD') keys.right = false;
    if (e.code === 'ArrowDown' || e.code === 'KeyS') keys.down = false;
    if (e.code === 'ArrowUp' || e.code === 'KeyW' || e.code === 'Space') keys.up = false;
});

// タッチイベント（モバイル）
const bindTouch = (id, key) => {
    const btn = document.getElementById(id);
    btn.addEventListener('touchstart', (e) => { e.preventDefault(); keys[key] = true; });
    btn.addEventListener('touchend', (e) => { e.preventDefault(); keys[key] = false; });
    // 指が外れた時の処理
    btn.addEventListener('touchcancel', (e) => { keys[key] = false; });
};
bindTouch('btn-left', 'left');
bindTouch('btn-right', 'right');
bindTouch('btn-jump', 'up');
bindTouch('btn-down', 'down');

// オブジェクト定義
class Player {
    constructor() {
        this.w = 20; // 人型のために細長くする
        this.h = 40;
        this.prevUp = false; // キーが押された瞬間を検知するため
        this.reset();
    }

    reset() {
        this.x = 100;
        this.y = 200;
        this.vx = 0;
        this.vy = 0;
        this.speed = 0.8;
        this.maxSpeed = 6;
        this.jumpForce = 11;
        this.grounded = false;
        this.onObject = null; // 現在乗っているオブジェクトを記憶
        this.jumpCount = 0; // 現在のジャンプ回数
        this.maxJumps = 2;  // 最大ジャンプ回数（2段ジャンプ）
        this.trail = []; // 軌跡エフェクト用
    }

    update() {
        // 左右の移動（加速度）
        if (keys.left) this.vx -= this.speed;
        if (keys.right) this.vx += this.speed;

        // 摩擦と速度制限
        this.vx *= PHYSICS.friction;
        if (this.vx > this.maxSpeed) this.vx = this.maxSpeed;
        if (this.vx < -this.maxSpeed) this.vx = -this.maxSpeed;

        // 重力
        this.vy += PHYSICS.gravity;
        if (this.vy > PHYSICS.maxFallSpeed) this.vy = PHYSICS.maxFallSpeed;

        // ジャンプの「押された瞬間」を検知
        const jumpPressed = keys.up && !this.prevUp;

        // ジャンプ処理（地上、または空中で2回目まで）
        if (jumpPressed && (this.grounded || this.jumpCount < this.maxJumps)) {
            // 空中ジャンプ（2回目）は少しジャンプ力を抑える
            const force = this.jumpCount === 0 ? this.jumpForce : this.jumpForce * 0.9;
            this.vy = -force;
            this.grounded = false;
            this.onObject = null;
            this.jumpCount++;
            
            // 1回目と2回目でパーティクルの色を変える演出
            const pColor = this.jumpCount === 1 ? '#ffffff' : THEME.item;
            createParticles(this.x + this.w/2, this.y + this.h, 10, pColor);
        }

        this.prevUp = keys.up; // 現在のキー状態を保存

        // 軌跡の保存
        this.trail.push({x: this.x, y: this.y});
        if (this.trail.length > 10) this.trail.shift();

        // X軸の移動と衝突判定
        this.x += this.vx;
        this.checkCollisions('x');

        // Y軸の移動と衝突判定
        this.y += this.vy;
        this.grounded = false;
        this.onObject = null; // 判定前に一度リセット
        this.checkCollisions('y');

        // アイテム取得判定
        this.checkItems();

        // 奈落判定
        if (this.y > 1500) {
            gameOver();
        }
    }

    checkItems() {
        for (let item of itemsData) {
            if (!item.collected) {
                // アイテムは円形、プレイヤーは矩形だが、簡単な矩形同士の衝突判定を行う
                const itemSize = 15; // アイテムの半径
                if (this.x < item.x + itemSize &&
                    this.x + this.w > item.x - itemSize &&
                    this.y < item.y + itemSize &&
                    this.y + this.h > item.y - itemSize) {
                    
                    item.collected = true;
                    score += 100;
                    createParticles(item.x, item.y, 10, THEME.item);
                }
            }
        }
    }

    checkCollisions(axis) {
        for (let obj of levelData) {
            // AABB(Axis-Aligned Bounding Box) 衝突判定
            if (this.x < obj.x + obj.w &&
                this.x + this.w > obj.x &&
                this.y < obj.y + obj.h &&
                this.y + this.h > obj.y) {
                
                // 危険物またはゴールとの接触
                if (obj.type === 'danger') {
                    gameOver();
                    return;
                }
                if (obj.type === 'goal') {
                    gameClear();
                    return;
                }

                // 壁・床との衝突解決
                if (axis === 'x') {
                    if (this.vx > 0) { // 右へ移動中
                        this.x = obj.x - this.w;
                        this.vx = 0;
                    } else if (this.vx < 0) { // 左へ移動中
                        this.x = obj.x + obj.w;
                        this.vx = 0;
                    }
                } else if (axis === 'y') {
                    if (this.vy > 0) { // 落下中（床に着地）
                        this.y = obj.y - this.h;
                        this.vy = 0;
                        this.grounded = true;
                        this.jumpCount = 0; // 着地したらジャンプ回数をリセット
                        this.onObject = obj; // 現在乗っているオブジェクトを記憶（ワープ判定用）
                    } else if (this.vy < 0) { // 上昇中（天井に衝突）
                        this.y = obj.y + obj.h;
                        this.vy = 0;

                        // ハテナブロックを下から叩いた時の処理
                        if (obj.type === 'block' && obj.subType === 'question' && !obj.hit) {
                            obj.hit = true; // 叩かれた状態にする
                            score += 100;   // スコア加算
                            // コインが飛び出すエフェクトを発生させる
                            coinEffects.push(new CoinEffect(obj.x + obj.w / 2, obj.y));
                        }
                    }
                }
            }
        }
    }

    draw(ctx, camX) {
        const drawX = this.x - camX;

        // 軌跡(トレイル)の描画
        for (let i = 0; i < this.trail.length; i+=2) {
            const t = this.trail[i];
            const alpha = i / this.trail.length * 0.3;
            ctx.fillStyle = `rgba(0, 0, 205, ${alpha})`;
            ctx.fillRect(t.x - camX + 4, t.y + 12, 12, 16); // 胴体サイズにあわせる
        }

        // 人型の描画
        // アニメーション計算
        let legAnim = 0;
        let armAnim = 0;
        if (this.grounded && Math.abs(this.vx) > 0.5) {
            legAnim = Math.sin(Date.now() / 80) * 8; // 走る足の振り
            armAnim = Math.cos(Date.now() / 80) * 6; // 走る腕の振り
        } else if (!this.grounded) {
            legAnim = 5; // ジャンプ中は足を上げる
            armAnim = -8; // 手を上げる
        }

        // ワープ中（沈む/浮上する）は気を付けの姿勢にする
        if (gameState === 'WARPING_IN' || gameState === 'WARPING_OUT') {
            legAnim = 0;
            armAnim = 0;
        }

        ctx.strokeStyle = '#333';
        ctx.lineWidth = 3;
        ctx.lineCap = 'round';

        // 後ろの腕と足
        ctx.beginPath();
        ctx.moveTo(drawX + 10, this.y + 25); // 股関節
        ctx.lineTo(drawX + 10 - legAnim, this.y + 38); // 左足
        ctx.moveTo(drawX + 10, this.y + 15); // 肩
        ctx.lineTo(drawX + 10 - armAnim, this.y + 25); // 左腕
        ctx.stroke();

        // 胴体
        ctx.fillStyle = THEME.player;
        ctx.fillRect(drawX + 4, this.y + 12, 12, 16);
        ctx.strokeRect(drawX + 4, this.y + 12, 12, 16);

        // 頭
        ctx.beginPath();
        ctx.arc(drawX + 10, this.y + 6, 7, 0, Math.PI * 2);
        ctx.fillStyle = THEME.skin;
        ctx.fill();
        ctx.stroke();

        // 前の腕と足
        ctx.beginPath();
        ctx.moveTo(drawX + 10, this.y + 25); // 股関節
        ctx.lineTo(drawX + 10 + legAnim, this.y + 38); // 右足
        ctx.moveTo(drawX + 10, this.y + 15); // 肩
        ctx.lineTo(drawX + 10 + armAnim, this.y + 25); // 右腕
        ctx.stroke();
    }
}

// パーティクルシステム
class Particle {
    constructor(x, y, color) {
        this.x = x;
        this.y = y;
        this.vx = (Math.random() - 0.5) * 5;
        this.vy = (Math.random() - 0.5) * 5;
        this.life = 1.0;
        this.decay = Math.random() * 0.05 + 0.02;
        this.color = color;
        this.size = Math.random() * 4 + 2;
    }
    update() {
        this.x += this.vx;
        this.y += this.vy;
        this.life -= this.decay;
    }
    draw(ctx, camX) {
        if (this.life <= 0) return;
        ctx.globalAlpha = this.life;
        ctx.fillStyle = this.color;
        ctx.shadowBlur = 10;
        ctx.shadowColor = this.color;
        ctx.fillRect(this.x - camX, this.y, this.size, this.size);
        ctx.globalAlpha = 1.0;
        ctx.shadowBlur = 0;
    }
}

// 叩いたときに出るコインのエフェクト
class CoinEffect {
    constructor(x, y) {
        this.x = x;
        this.y = y;
        this.vy = -10;       // 上に飛び出す力
        this.gravity = 0.6;  // 重力
        this.life = 1.0;
    }
    update() {
        this.y += this.vy;
        this.vy += this.gravity;
        // 落ち始めたら徐々に消える
        if (this.vy > 0) {
            this.life -= 0.05; 
        }
    }
    draw(ctx, camX) {
        if (this.life <= 0) return;
        ctx.globalAlpha = Math.max(0, this.life);
        
        // 早く回転するアニメーション
        const time = Date.now() / 50; 
        const scaleX = Math.abs(Math.sin(time));
        
        ctx.fillStyle = THEME.item;
        ctx.strokeStyle = '#B8860B';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.ellipse(this.x - camX, this.y, 10 * Math.max(scaleX, 0.1), 15, 0, 0, Math.PI * 2);
        ctx.fill();
        ctx.stroke();
        
        ctx.globalAlpha = 1.0;
    }
}

let particles = [];
let coinEffects = [];

function createParticles(x, y, count, color) {
    for (let i = 0; i < count; i++) {
        particles.push(new Particle(x, y, color));
    }
}

// レベルデータ構築
let levelData = [];
let itemsData = [];
function buildLevel(stage) {
    const b = 40; // 1ブロックの基本サイズ
    const floorY = 380; // 地面の高さ（画面の下より少し高めに変更）
    const lowY = floorY - 3 * b; // 低い空中のブロック
    const highY = floorY - 6 * b; // 高い空中のブロック

    levelData = [];
    itemsData = [];

    if (stage === 'main') {
        // --- 地上ステージ ---
        levelData = [
            // 地面 (原作の穴をあけつつ配置)
            { x: 0, y: floorY, w: 69 * b, h: 500, type: 'ground' },
            { x: 71 * b, y: floorY, w: 15 * b, h: 500, type: 'ground' },
            { x: 89 * b, y: floorY, w: 64 * b, h: 500, type: 'ground' },
            { x: 155 * b, y: floorY, w: 60 * b, h: 500, type: 'ground' },

            // 土管
            // 最初の土管（ワープ可能に設定）
            // spawnY は ワープ後に浮上してくる際にピッタリ土管の上に着地する座標 (floorY - 2*b - 40(player.h) + 40(浮上量))
            { x: 28 * b, y: floorY - 2 * b, w: 2 * b, h: 2 * b, type: 'pipe', warpTo: 'underground', spawnX: 2 * b + 10, spawnY: floorY - 2 * b }, 
            
            { x: 38 * b, y: floorY - 3 * b, w: 2 * b, h: 3 * b, type: 'pipe' },
            { x: 46 * b, y: floorY - 4 * b, w: 2 * b, h: 4 * b, type: 'pipe' },
            { x: 57 * b, y: floorY - 4 * b, w: 2 * b, h: 4 * b, type: 'pipe' },
            
            // 出口用の土管
            { x: 163 * b, y: floorY - 2 * b, w: 2 * b, h: 2 * b, type: 'pipe' },
            { x: 179 * b, y: floorY - 2 * b, w: 2 * b, h: 2 * b, type: 'pipe' },

            // ブロック群
            // 最初のハテナ
            { x: 16 * b, y: lowY, w: b, h: b, type: 'block', subType: 'question' },
            
            // 5連ブロック
            { x: 20 * b, y: lowY, w: b, h: b, type: 'block', subType: 'brick' },
            { x: 21 * b, y: lowY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 22 * b, y: lowY, w: b, h: b, type: 'block', subType: 'brick' },
            { x: 23 * b, y: lowY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 24 * b, y: lowY, w: b, h: b, type: 'block', subType: 'brick' },
            { x: 22 * b, y: highY, w: b, h: b, type: 'block', subType: 'question' },

            // 土管後のブロック
            { x: 77 * b, y: lowY, w: b, h: b, type: 'block', subType: 'brick' },
            { x: 78 * b, y: lowY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 79 * b, y: lowY, w: b, h: b, type: 'block', subType: 'brick' },

            { x: 80 * b, y: highY, w: 8 * b, h: b, type: 'block', subType: 'brick' },

            { x: 91 * b, y: highY, w: 3 * b, h: b, type: 'block', subType: 'brick' },
            { x: 94 * b, y: highY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 94 * b, y: lowY, w: b, h: b, type: 'block', subType: 'brick' },

            { x: 100 * b, y: lowY, w: 2 * b, h: b, type: 'block', subType: 'brick' },
            { x: 106 * b, y: lowY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 109 * b, y: lowY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 109 * b, y: highY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 112 * b, y: lowY, w: b, h: b, type: 'block', subType: 'question' },
            { x: 118 * b, y: lowY, w: b, h: b, type: 'block', subType: 'brick' },

            { x: 121 * b, y: highY, w: 3 * b, h: b, type: 'block', subType: 'brick' },

            // 階段（上り）
            { x: 134 * b, y: floorY - b, w: b, h: b, type: 'block', subType: 'stone' },
            { x: 135 * b, y: floorY - 2*b, w: b, h: 2*b, type: 'block', subType: 'stone' },
            { x: 136 * b, y: floorY - 3*b, w: b, h: 3*b, type: 'block', subType: 'stone' },
            { x: 137 * b, y: floorY - 4*b, w: b, h: 4*b, type: 'block', subType: 'stone' },
            // 階段（下り）
            { x: 140 * b, y: floorY - 4*b, w: b, h: 4*b, type: 'block', subType: 'stone' },
            { x: 141 * b, y: floorY - 3*b, w: b, h: 3*b, type: 'block', subType: 'stone' },
            { x: 142 * b, y: floorY - 2*b, w: b, h: 2*b, type: 'block', subType: 'stone' },
            { x: 143 * b, y: floorY - b, w: b, h: b, type: 'block', subType: 'stone' },

            // ゴール前の大階段
            { x: 181 * b, y: floorY - b, w: b, h: b, type: 'block', subType: 'stone' },
            { x: 182 * b, y: floorY - 2*b, w: b, h: 2*b, type: 'block', subType: 'stone' },
            { x: 183 * b, y: floorY - 3*b, w: b, h: 3*b, type: 'block', subType: 'stone' },
            { x: 184 * b, y: floorY - 4*b, w: b, h: 4*b, type: 'block', subType: 'stone' },
            { x: 185 * b, y: floorY - 5*b, w: b, h: 5*b, type: 'block', subType: 'stone' },
            { x: 186 * b, y: floorY - 6*b, w: b, h: 6*b, type: 'block', subType: 'stone' },
            { x: 187 * b, y: floorY - 7*b, w: b, h: 7*b, type: 'block', subType: 'stone' },
            { x: 188 * b, y: floorY - 8*b, w: 2*b, h: 8*b, type: 'block', subType: 'stone' },

            // ゴールエリア
            { x: 198 * b, y: floorY - 350, w: 10, h: 350, type: 'goal' }, // ゴールポール
            // お城(ブロックで表現)
            { x: 204 * b, y: floorY - 4*b, w: 5*b, h: 4*b, type: 'block', subType: 'stone' },
            { x: 204.5 * b, y: floorY - 5*b, w: b, h: b, type: 'block', subType: 'stone' },
            { x: 207.5 * b, y: floorY - 5*b, w: b, h: b, type: 'block', subType: 'stone' },
        ];

        itemsData = [
            // 適当な配置のコイン
            { x: 10 * b + 20, y: floorY - 40, collected: false },
            { x: 42 * b + 20, y: floorY - 40, collected: false },
            { x: 62 * b + 20, y: floorY - 40, collected: false },
            { x: 125 * b + 20, y: lowY, collected: false },

            // 穴を飛び越える時の誘導コイン
            { x: 70 * b + 20, y: floorY - 40, collected: false },
            { x: 88 * b + 20, y: floorY - 40, collected: false },
            { x: 154 * b + 20, y: floorY - 40, collected: false },
        ];

    } else if (stage === 'underground') {
        // --- 地下ステージ ---
        levelData = [
            // 床・壁・天井をブロックで囲む
            { x: 0, y: floorY, w: 30 * b, h: 500, type: 'ground' }, // 床
            { x: 0, y: 0, w: b, h: floorY, type: 'block', subType: 'stone' }, // 左壁
            { x: 29 * b, y: 0, w: b, h: floorY, type: 'block', subType: 'stone' }, // 右壁
            { x: 0, y: -b, w: 30 * b, h: b * 2, type: 'block', subType: 'stone' }, // 天井

            // 出現用土管
            { x: 2 * b, y: floorY - 2 * b, w: 2 * b, h: 2 * b, type: 'pipe' },
            
            // 地下の足場とコイン
            { x: 5 * b, y: floorY - 3 * b, w: 7 * b, h: b, type: 'block', subType: 'brick' },
            { x: 14 * b, y: floorY - 6 * b, w: 9 * b, h: b, type: 'block', subType: 'brick' },
            { x: 18 * b, y: floorY - 3 * b, w: 4 * b, h: b, type: 'block', subType: 'brick' },

            // 出口の土管（地上の163*bの土管へワープ）
            { x: 26 * b, y: floorY - 2 * b, w: 2 * b, h: 2 * b, type: 'pipe', warpTo: 'main', spawnX: 163 * b + 10, spawnY: floorY - 2 * b }
        ];

        // 地下にはコインをたくさん配置
        itemsData = [
            { x: 6 * b + 20, y: floorY - 3 * b - 20, collected: false },
            { x: 7 * b + 20, y: floorY - 3 * b - 20, collected: false },
            { x: 8 * b + 20, y: floorY - 3 * b - 20, collected: false },
            { x: 9 * b + 20, y: floorY - 3 * b - 20, collected: false },
            { x: 10 * b + 20, y: floorY - 3 * b - 20, collected: false },
            
            { x: 15 * b + 20, y: floorY - 6 * b - 20, collected: false },
            { x: 16 * b + 20, y: floorY - 6 * b - 20, collected: false },
            { x: 17 * b + 20, y: floorY - 6 * b - 20, collected: false },
            { x: 18 * b + 20, y: floorY - 6 * b - 20, collected: false },
            { x: 19 * b + 20, y: floorY - 6 * b - 20, collected: false },
            { x: 20 * b + 20, y: floorY - 6 * b - 20, collected: false },
            { x: 21 * b + 20, y: floorY - 6 * b - 20, collected: false },
            { x: 22 * b + 20, y: floorY - 6 * b - 20, collected: false }
        ];
    }
}

// カメラ（スクロール）変数
let cameraX = 0;

// インスタンス化
const player = new Player();

// ワープ実行関数
function startWarp(targetStage, spawnX, spawnY) {
    currentStage = targetStage;
    buildLevel(targetStage); // レベル再構築
    player.x = spawnX;
    player.y = spawnY;
    player.vx = 0;
    player.vy = 0;
    
    // カメラをワープ先に一瞬で合わせる
    cameraX = Math.max(0, player.x - canvasWidth / 3);

    // 背景色切り替え（地下は真っ暗、地上は空色）
    if (currentStage === 'underground') {
        container.style.background = '#000000';
    } else {
        container.style.background = 'linear-gradient(to bottom, #87CEEB 0%, #E0F6FF 100%)';
    }
}

// 描画関数
function draw() {
    ctx.clearRect(0, 0, canvasWidth, canvasHeight);

    // 雲の描画（パララックス効果） - 地上ステージのみ描画
    if (currentStage === 'main') {
        ctx.fillStyle = 'rgba(255, 255, 255, 0.9)';
        for (let c of clouds) {
            const cx = (c.x - cameraX * 0.3) % 5000;
            const drawX = cx < -200 ? cx + 5000 : cx;
            ctx.beginPath();
            ctx.arc(drawX, c.y, 30 * c.s, 0, Math.PI * 2);
            ctx.arc(drawX + 25 * c.s, c.y - 15 * c.s, 40 * c.s, 0, Math.PI * 2);
            ctx.arc(drawX + 55 * c.s, c.y - 5 * c.s, 35 * c.s, 0, Math.PI * 2);
            ctx.fill();
        }
    }

    // 配置されているアイテム（コイン）の描画
    const time = Date.now() / 150;
    for (let item of itemsData) {
        if (!item.collected) {
            const drawX = item.x - cameraX;
            if (drawX + 20 < 0 || drawX - 20 > canvasWidth) continue;

            const scaleX = Math.abs(Math.sin(time + item.x));
            ctx.fillStyle = THEME.item;
            ctx.strokeStyle = '#B8860B'; 
            ctx.lineWidth = 2;
            
            ctx.beginPath();
            ctx.ellipse(drawX, item.y, 10 * Math.max(scaleX, 0.1), 15, 0, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();
        }
    }

    // レベル（地形）の描画 ※土管以外を先に描画する
    for (let obj of levelData) {
        if (obj.type === 'pipe') continue; // 土管は後回し

        const drawX = obj.x - cameraX;
        if (drawX + obj.w < 0 || drawX > canvasWidth) continue;

        if (obj.type === 'ground') {
            // 地下の場合は暗いレンガ色にするなど色を変える演出
            const baseColor = currentStage === 'underground' ? '#2F4F4F' : THEME.ground;
            const lineColor = currentStage === 'underground' ? '#111' : '#000';
            const topColor = currentStage === 'underground' ? '#4a7575' : '#D2691E';

            ctx.fillStyle = baseColor;
            ctx.fillRect(drawX, obj.y, obj.w, obj.h);
            
            ctx.strokeStyle = lineColor;
            ctx.lineWidth = 1;
            const blockSize = 40;
            
            ctx.beginPath();
            for (let x = 0; x <= obj.w; x += blockSize) {
                ctx.moveTo(drawX + x, obj.y);
                ctx.lineTo(drawX + x, obj.y + obj.h);
            }
            for (let y = 0; y <= obj.h; y += blockSize) {
                ctx.moveTo(drawX, obj.y + y);
                ctx.lineTo(drawX + obj.w, obj.y + y);
            }
            ctx.stroke();

            for (let y = 0; y < obj.h; y += blockSize) {
                for (let x = 0; x < obj.w; x += blockSize) {
                    if (x + blockSize <= obj.w && y + blockSize <= obj.h) {
                        ctx.fillStyle = 'rgba(255, 255, 255, 0.15)';
                        ctx.fillRect(drawX + x, obj.y + y, blockSize, 4);
                        ctx.fillRect(drawX + x, obj.y + y, 4, blockSize);
                        
                        ctx.fillStyle = 'rgba(0, 0, 0, 0.3)';
                        ctx.fillRect(drawX + x, obj.y + y + blockSize - 4, blockSize, 4);
                        ctx.fillRect(drawX + x + blockSize - 4, obj.y + y, 4, blockSize);
                    }
                }
            }
            
            ctx.fillStyle = topColor;
            ctx.fillRect(drawX, obj.y, obj.w, 5);
            ctx.strokeStyle = '#222';
            ctx.lineWidth = 4;
            ctx.strokeRect(drawX, obj.y, obj.w, obj.h);
            
        } else if (obj.type === 'block') {
            if (obj.subType === 'question') {
                if (obj.hit) {
                    ctx.fillStyle = THEME.emptyBlock;
                    ctx.fillRect(drawX, obj.y, obj.w, obj.h);
                    ctx.strokeStyle = '#000';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(drawX, obj.y, obj.w, obj.h);
                    
                    ctx.fillStyle = '#000';
                    ctx.fillRect(drawX + 4, obj.y + 4, 4, 4);
                    ctx.fillRect(drawX + obj.w - 8, obj.y + 4, 4, 4);
                    ctx.fillRect(drawX + 4, obj.y + obj.h - 8, 4, 4);
                    ctx.fillRect(drawX + obj.w - 8, obj.y + obj.h - 8, 4, 4);
                } else {
                    ctx.fillStyle = THEME.question;
                    ctx.fillRect(drawX, obj.y, obj.w, obj.h);
                    ctx.fillStyle = '#fff';
                    ctx.font = 'bold 24px Arial';
                    ctx.textAlign = 'center';
                    ctx.fillText('?', drawX + obj.w/2, obj.y + 28);
                    ctx.strokeStyle = '#000';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(drawX, obj.y, obj.w, obj.h);
                }
            } else if (obj.subType === 'brick') {
                // 地下ステージのレンガは少し青っぽくする
                ctx.fillStyle = currentStage === 'underground' ? '#4682B4' : THEME.brick;
                ctx.fillRect(drawX, obj.y, obj.w, obj.h);
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 2;
                ctx.strokeRect(drawX, obj.y, obj.w, obj.h);
                ctx.beginPath();
                ctx.moveTo(drawX, obj.y + obj.h/2);
                ctx.lineTo(drawX + obj.w, obj.y + obj.h/2);
                ctx.moveTo(drawX + obj.w/2, obj.y);
                ctx.lineTo(drawX + obj.w/2, obj.y + obj.h/2);
                ctx.stroke();
            } else if (obj.subType === 'stone') {
                ctx.fillStyle = THEME.stone;
                ctx.fillRect(drawX, obj.y, obj.w, obj.h);
                ctx.strokeStyle = '#fff';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(drawX, obj.y + obj.h);
                ctx.lineTo(drawX, obj.y);
                ctx.lineTo(drawX + obj.w, obj.y);
                ctx.stroke();
                ctx.strokeStyle = '#000';
                ctx.beginPath();
                ctx.moveTo(drawX + obj.w, obj.y);
                ctx.lineTo(drawX + obj.w, obj.y + obj.h);
                ctx.lineTo(drawX, obj.y + obj.h);
                ctx.stroke();
            }
        } else if (obj.type === 'danger') {
            ctx.fillStyle = THEME.danger;
            ctx.fillRect(drawX, obj.y, obj.w, obj.h);
            ctx.fillStyle = '#FFA07A';
            for(let i = 0; i < obj.w; i += 20) {
                const heightOffset = Math.sin((Date.now() / 200) + i) * 5;
                ctx.beginPath();
                ctx.moveTo(drawX + i, obj.y + 5);
                ctx.lineTo(drawX + i + 10, obj.y - 5 + heightOffset);
                ctx.lineTo(drawX + i + 20, obj.y + 5);
                ctx.fill();
            }
        } else if (obj.type === 'goal') {
            ctx.fillStyle = '#C0C0C0';
            ctx.fillRect(drawX + 15, obj.y, 10, obj.h);
            ctx.fillStyle = THEME.goal;
            const flagWave = Math.sin(Date.now() / 150) * 10;
            ctx.beginPath();
            ctx.moveTo(drawX + 25, obj.y + 10);
            ctx.lineTo(drawX + 60 + flagWave, obj.y + 20);
            ctx.lineTo(drawX + 25, obj.y + 40);
            ctx.fill();
        }
    }

    // エフェクト群
    for (let i = coinEffects.length - 1; i >= 0; i--) {
        coinEffects[i].update();
        coinEffects[i].draw(ctx, cameraX);
        if (coinEffects[i].life <= 0) coinEffects.splice(i, 1);
    }
    for (let i = particles.length - 1; i >= 0; i--) {
        particles[i].update();
        particles[i].draw(ctx, cameraX);
        if (particles[i].life <= 0) particles.splice(i, 1);
    }

    // プレイヤーの描画
    player.draw(ctx, cameraX);

    // ★ プレイヤーが土管に入るときに手前に来るように、土管を最後に描画する
    for (let obj of levelData) {
        if (obj.type !== 'pipe') continue;
        
        const drawX = obj.x - cameraX;
        if (drawX + obj.w < 0 || drawX > canvasWidth) continue;

        ctx.fillStyle = THEME.pipe;
        ctx.fillRect(drawX + 4, obj.y + 20, obj.w - 8, obj.h - 20); // 胴体
        ctx.fillRect(drawX, obj.y, obj.w, 20);      // フチ
        ctx.strokeStyle = '#000';
        ctx.lineWidth = 2;
        ctx.strokeRect(drawX + 4, obj.y + 20, obj.w - 8, obj.h - 20);
        ctx.strokeRect(drawX, obj.y, obj.w, 20);
        
        ctx.fillStyle = 'rgba(255,255,255,0.4)';
        ctx.fillRect(drawX + 12, obj.y, 8, obj.h);
    }
}

// ゲームループ
function update() {
    if (gameState === 'PLAYING') {
        player.update();

        // ワープ判定（土管の上に乗っていて下キーが押されたら）
        if (player.grounded && keys.down && player.onObject && player.onObject.type === 'pipe' && player.onObject.warpTo) {
            gameState = 'WARPING_IN';
            warpInfo = {
                targetStage: player.onObject.warpTo,
                spawnX: player.onObject.spawnX,
                spawnY: player.onObject.spawnY
            };
            warpTimer = 0;
            player.vx = 0;
            // プレイヤーを土管の真ん中に合わせる
            player.x = player.onObject.x + player.onObject.w / 2 - player.w / 2;
        }
        
    } else if (gameState === 'WARPING_IN') {
        // 土管の中に沈んでいくアニメーション
        player.y += 2; 
        warpTimer++;
        if (warpTimer > 20) { // 十分沈んだらステージ切り替え
            startWarp(warpInfo.targetStage, warpInfo.spawnX, warpInfo.spawnY);
            gameState = 'WARPING_OUT';
            warpTimer = 0;
        }
    } else if (gameState === 'WARPING_OUT') {
        // 土管から浮上してくるアニメーション
        player.y -= 2; 
        warpTimer++;
        if (warpTimer > 20) { // 上がりきったらプレイ再開
            gameState = 'PLAYING';
            keys.down = false; // 押しっぱなし防止
        }
    }

    // カメラの追従（ワープ中も動かす）
    const targetCamX = player.x - canvasWidth / 3;
    cameraX += (Math.max(0, targetCamX) - cameraX) * 0.1;

    // UI更新 (スコアはアイテムなどで取得した純粋なスコアのみ)
    scoreDisplay.innerText = `SCORE: ${score}`;

    draw();
    animationId = requestAnimationFrame(update);
}

// ゲーム制御関数
function startGame() {
    gameState = 'PLAYING';
    msgScreen.classList.add('hidden');
    currentStage = 'main';
    container.style.background = 'linear-gradient(to bottom, #87CEEB 0%, #E0F6FF 100%)';
    buildLevel(currentStage);
    player.reset();
    score = 0; 
    particles = [];
    coinEffects = [];
    cameraX = 0;
    
    // キー入力をリセット
    keys.left = false;
    keys.right = false;
    keys.up = false;
    keys.down = false;

    cancelAnimationFrame(animationId);
    update();
}

function gameOver() {
    gameState = 'GAMEOVER';
    deaths++;
    deathsDisplay.innerText = `DEATHS: ${deaths}`;
    
    createParticles(player.x + player.w/2, player.y + player.h/2, 50, THEME.player);
    draw();

    setTimeout(() => {
        msgTitle.innerText = "GAME OVER";
        msgTitle.style.color = THEME.danger;
        msgTitle.style.textShadow = `2px 2px 0px #fff`;
        msgDesc.innerHTML = "やられてしまった！<br>もう一度挑戦する？";
        startBtn.innerText = "RETRY";
        startBtn.style.backgroundColor = THEME.danger;
        startBtn.style.borderColor = THEME.danger;
        
        msgScreen.classList.remove('hidden');
    }, 500);
}

function gameClear() {
    gameState = 'CLEAR';
    
    createParticles(player.x + player.w/2, player.y, 100, THEME.goal);
    draw();

    setTimeout(() => {
        msgTitle.innerText = "LEVEL CLEAR!";
        msgTitle.style.color = THEME.goal;
        msgTitle.style.textShadow = `2px 2px 0px #fff, 0 0 10px #FFD700`;
        msgDesc.innerHTML = `見事クリア！<br>FINAL SCORE: ${score}<br>DEATH COUNT: ${deaths}`;
        startBtn.innerText = "PLAY AGAIN";
        startBtn.style.backgroundColor = THEME.goal;
        startBtn.style.borderColor = THEME.goal;
        
        msgScreen.classList.remove('hidden');
    }, 1000);
}

// 初期化
startBtn.addEventListener('click', startGame);

// 背景のみを初期描画
buildLevel('main');
draw();

</script>
</body>
</html>
