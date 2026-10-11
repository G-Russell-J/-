# -<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="format-detection" content="telephone=no">
<meta name="theme-color" content="#0a0a0f">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<title>指尖导航 · 无障碍行走助手</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }

  html, body {
    width: 100%; height: 100%;
    background: #0a0a0f;
    color: #ffffff;
    font-family: "PingFang SC", "Microsoft YaHei", "Noto Sans SC", sans-serif;
    overflow: hidden;
    touch-action: manipulation;
    overscroll-behavior: none;
    -webkit-user-select: none;
    user-select: none;
  }

  .page { position: fixed; inset: 0; }

  /* ================= 第一页：授权引导页 ================= */
  #pageAuth {
    background: #0a0a0f;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
    padding: 24px;
    padding-top: calc(24px + env(safe-area-inset-top));
    padding-bottom: calc(24px + env(safe-area-inset-bottom));
    z-index: 20;
  }
  #pageAuth h1 {
    font-size: clamp(34px, 8vw, 56px);
    letter-spacing: 8px;
    margin-bottom: 22px;
  }
  #pageAuth .desc {
    font-size: clamp(19px, 4.2vw, 26px);
    line-height: 1.9;
    opacity: 0.9;
    max-width: 640px;
  }

  #permList {
    margin-top: 28px;
    width: min(460px, 92%);
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  .perm {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 20px;
    font-size: clamp(18px, 4vw, 24px);
    background: rgba(255,255,255,0.08);
    border: 1px solid rgba(255,255,255,0.2);
    border-radius: 12px;
  }
  .permState { font-weight: 700; }
  .perm.wait  .permState { color: #9ca3af; }
  .perm.doing .permState { color: #facc15; }
  .perm.ok    .permState { color: #4ade80; }
  .perm.fail  .permState { color: #f87171; }

  #authBtn {
    margin-top: 40px;
    padding: clamp(20px, 4vw, 28px) clamp(56px, 14vw, 110px);
    font-size: clamp(24px, 5.5vw, 34px);
    font-weight: 700;
    letter-spacing: 4px;
    color: #0a0a0f;
    background: #ffffff;
    border: none;
    border-radius: 18px;
    cursor: pointer;
    box-shadow: 0 0 0 4px rgba(255,255,255,0.25);
  }
  #authBtn:active { transform: scale(0.97); }
  #authBtn:disabled { opacity: 0.55; }
  #authMsg {
    margin-top: 26px;
    font-size: clamp(18px, 4vw, 24px);
    line-height: 1.8;
    color: #f87171;
    max-width: 640px;
    white-space: pre-line;
  }
  #pageAuth .footnote {
    position: absolute;
    bottom: calc(18px + env(safe-area-inset-bottom));
    left: 24px; right: 24px;
    font-size: clamp(14px, 3.2vw, 17px);
    opacity: 0.55;
    line-height: 1.7;
  }

  /* ================= 第二页：导航页 ================= */
  #pageNav { display: none; background: #000; }
  #pageNav.active { display: block; }

  #video {
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    object-fit: cover;
    background: #000;
  }
  #canvas { display: none; }

  #panel {
    position: fixed;
    top: 0; left: 0; right: 0;
    padding: 12px 20px;
    padding-top: calc(12px + env(safe-area-inset-top));
    background: rgba(10, 10, 20, 0.72);
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
    border-bottom: 2px solid rgba(255, 255, 255, 0.15);
    z-index: 10;
    text-align: center;
  }
  #panel h1 {
    font-size: clamp(16px, 3.6vw, 22px);
    letter-spacing: 4px;
    margin-bottom: 4px;
    opacity: 0.9;
  }
  #ratio {
    font-size: clamp(28px, 6.5vw, 42px);
    font-weight: 700;
    font-variant-numeric: tabular-nums;
    line-height: 1.25;
  }
  #status {
    display: inline-block;
    margin-top: 4px;
    padding: 4px 24px;
    font-size: clamp(22px, 5vw, 32px);
    font-weight: 800;
    letter-spacing: 8px;
    border-radius: 12px;
    border: 3px solid currentColor;
  }
  .status-safe    { color: #4ade80; }
  .status-warning { color: #facc15; }
  .status-danger  { color: #f87171; }

  #targetRow {
    margin-top: 6px;
    font-size: clamp(13px, 3.2vw, 17px);
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
  }
  #swatch {
    width: 20px; height: 20px;
    border-radius: 6px;
    border: 2px solid #ffffff;
    background: #555;
    flex: none;
  }
  .miniBtn {
    padding: 3px 10px;
    font-size: clamp(12px, 3vw, 15px);
    color: #fff;
    background: rgba(255,255,255,0.15);
    border: 1px solid rgba(255,255,255,0.4);
    border-radius: 8px;
    cursor: pointer;
  }
  .hidden { display: none !important; }

  /* 识别预览小窗：右下角 */
  #preview {
    position: fixed;
    right: 10px;
    bottom: calc(96px + env(safe-area-inset-bottom));
    width: clamp(96px, 22vw, 150px);
    height: auto;
    aspect-ratio: 4 / 3;
    border: 2px solid rgba(255,255,255,0.5);
    border-radius: 8px;
    z-index: 10;
    opacity: 0.9;
    background: #000;
  }

  #hint {
    position: fixed;
    bottom: 0; left: 0; right: 0;
    padding: 10px 20px;
    padding-bottom: calc(10px + env(safe-area-inset-bottom));
    background: rgba(10, 10, 20, 0.72);
    font-size: clamp(14px, 3.2vw, 17px);
    line-height: 1.55;
    text-align: center;
    z-index: 10;
  }
</style>
</head>
<body>

  <!-- ========== 第一页：授权引导 ========== -->
  <div id="pageAuth" class="page">
    <h1>指尖导航</h1>
    <p class="desc">
      本应用通过摄像头分析前方画面，识别桌椅沙发等障碍物，<br>
      用手机震动与语音提示方位。轻触下方按钮完成授权。
    </p >

    <div id="permList">
      <div class="perm wait" id="permCam"><span>摄像头</span><span class="permState">待授权</span></div>
      <div class="perm wait" id="permVib"><span>震动</span><span class="permState">待授权</span></div>
      <div class="perm wait" id="permSpk"><span>扬声器</span><span class="permState">待检测</span></div>
    </div>

    <button id="authBtn">轻触屏幕授权</button>
    <div id="authMsg"></div>
    <p class="footnote">扬声器无需系统授权，将以提示音实测可用性；摄像头功能需通过 HTTPS 或 localhost 访问。</p >
  </div>

  <!-- ========== 第二页：导航页 ========== -->
  <div id="pageNav" class="page">
    <video id="video" autoplay playsinline muted></video>
    <canvas id="canvas"></canvas>
    <canvas id="preview" width="160" height="120"></canvas>

    <div id="panel">
      <h1>指尖导航</h1>
      <div id="ratio">障碍物占比: --%</div>
      <div><span id="status" class="status-safe">当前状态: 安全</span></div>
      <div id="targetRow">
        <span id="swatch"></span><span id="modeText">自动模式：已识别背景，无需锁定</span>
        <button id="modeBtn" class="miniBtn hidden">切回自动</button>
        <button id="voiceBtn" class="miniBtn">语音: 开</button>
      </div>
    </div>

    <div id="hint">自动模式：识别与地面不同的物体并播报方位；点击画面可锁定指定颜色的障碍物。</div>
  </div>

<script>
(function () {
  "use strict";

  /* ============ 元素与常量 ============ */
  var pageAuth  = document.getElementById("pageAuth");
  var pageNav   = document.getElementById("pageNav");
  var authBtn   = document.getElementById("authBtn");
  var authMsg   = document.getElementById("authMsg");
  var permCam   = document.getElementById("permCam");
  var permVib   = document.getElementById("permVib");
  var permSpk   = document.getElementById("permSpk");

  var video     = document.getElementById("video");
  var canvas    = document.getElementById("canvas");
  var ctx       = canvas.getContext("2d", { willReadFrequently: true });
  var preview   = document.getElementById("preview");
  var pctx      = preview.getContext("2d");
  var ratioEl   = document.getElementById("ratio");
  var statusEl  = document.getElementById("status");
  var swatchEl  = document.getElementById("swatch");
  var modeText  = document.getElementById("modeText");
  var modeBtn   = document.getElementById("modeBtn");
  var voiceBtn  = document.getElementById("voiceBtn");

  var SAMPLE_INTERVAL = 500;
  var SAMPLE_W = 160;
  var SAMPLE_H = 120;
  var TOTAL = SAMPLE_W * SAMPLE_H;
  var TOL_MANUAL = 70;   // 手动锁定色的相近阈值（调高更灵敏）
  var TOL_AUTO   = 60;   // 自动模式：与背景色的差异阈值（调低更灵敏）

  // 检测灵敏度参数（调低面积门槛、放宽接地条件，小而近的障碍物也能触发）
  var MIN_AREA_RATIO  = 0.010;  // 最小面积占比 1.5% -> 1.0%
  var MIN_AREA_PX     = 40;     // 最小像素数   60 -> 40
  var GROUND_MAX_Y    = 0.50;   // 接地条件：底边达画面 55% -> 50%
  var GROUND_CY       = 0.40;   // 质心须在中下部 45% -> 40%

  var PROMPT_TEXT = "轻触屏幕授权手机摄像头和手机震动";
  var DENY_TEXT   = "未授权手机摄像头和手机震动";
  var GUIDE_TEXT  = "请将手机举至身前，摄像头对准前方";

  var supportVibrate = "vibrate" in navigator;
  var supportSpeech  = "speechSynthesis" in window;
  var voiceOn = true;
  var promptSpoken = false;
  var authStarted = false;
  var timer = null;
  var lastState = null;
  var mode = "auto";                 // auto=背景差分 | manual=锁定目标色

  var targetR = 100, targetG = 100, targetB = 100;  // 手动锁定色
  var groundR = 110, groundG = 110, groundB = 110;  // 自动模式背景色

  var mask    = new Uint8Array(TOTAL);
  var visited = new Uint8Array(TOTAL);
  var stack   = new Int32Array(TOTAL);
  var outImg  = pctx.createImageData(SAMPLE_W, SAMPLE_H);

  /* ============ 权限状态标注 ============ */
  function setPerm(el, state, label) {
    el.className = "perm " + state;
    el.querySelector(".permState").textContent = label;
  }

  /* ============ 语音 ============ */
  function speak(text, onend) {
    if (!voiceOn || !supportSpeech) { if (onend) setTimeout(onend, 300); return; }
    try {
      window.speechSynthesis.cancel();
      var u = new SpeechSynthesisUtterance(text);
      u.lang = "zh-CN";
      u.rate = 1.1;
      var done = false;
      function finish() { if (!done) { done = true; if (onend) onend(); } }
      u.onend = finish;
      u.onerror = finish;
      setTimeout(finish, Math.max(1800, text.length * 260));
      if (text === PROMPT_TEXT) u.onstart = function () { promptSpoken = true; };
      window.speechSynthesis.speak(u);
    } catch (e) { if (onend) onend(); }
  }

  voiceBtn.addEventListener("click", function () {
    voiceOn = !voiceOn;
    voiceBtn.textContent = "语音: " + (voiceOn ? "开" : "关");
    if (!voiceOn) { try { window.speechSynthesis.cancel(); } catch (e) {} }
  });

  /* ============ 扬声器实测 ============ */
  var audioCtx = null;
  function testSpeaker() {
    setPerm(permSpk, "doing", "检测中…");
    try {
      var AC = window.AudioContext || window.webkitAudioContext;
      if (!AC) { setPerm(permSpk, "fail", "不可用"); return; }
      audioCtx = audioCtx || new AC();
      if (audioCtx.state === "suspended") audioCtx.resume();
      var osc = audioCtx.createOscillator();
      var gain = audioCtx.createGain();
      osc.type = "sine";
      osc.frequency.value = 880;
      gain.gain.setValueAtTime(0.12, audioCtx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.35);
      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start();
      osc.stop(audioCtx.currentTime + 0.35);
      setTimeout(function () {
        if (audioCtx.state === "running") setPerm(permSpk, "ok", "已开启");
        else setPerm(permSpk, "fail", "被静音/阻塞");
      }, 400);
    } catch (e) { setPerm(permSpk, "fail", "不可用"); }
  }

  /* ============ 第一页：授权流程 ============ */
  window.addEventListener("load", function () {
    setTimeout(function () { speak(PROMPT_TEXT); }, 400);
  });
  function requestPermissions() {
    authMsg.textContent = "正在请求权限，请在弹窗中点击「允许」…";

    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      setPerm(permCam, "fail", "不支持");
      authMsg.textContent = "当前浏览器不支持摄像头（getUserMedia）。请改用 Chrome / Safari，并通过 HTTPS 或 localhost 访问。";
      speak(DENY_TEXT);
      authStarted = false;
      return;
    }

    setPerm(permCam, "doing", "请求中…");
    navigator.mediaDevices.getUserMedia({
      video: { facingMode: { ideal: "environment" } },
      audio: false
    }).then(function (stream) {
      setPerm(permCam, "ok", "已授权");
      video.srcObject = stream;
      return video.play();
    }).then(function () {
      enterNav();
    }).catch(function (err) {
      authStarted = false;
      authBtn.disabled = false;
      authBtn.textContent = "重新尝试授权";
      setPerm(permCam, "fail", "未授权");
      var reason = {
        NotAllowedError:  "权限被拒绝。请在浏览器设置中允许摄像头后重试。",
        NotFoundError:    "未检测到可用摄像头设备。",
        NotReadableError: "摄像头被其他应用占用，请关闭后重试。"
      }[err && err.name] || "授权失败：" + (err && err.message ? err.message : "未知错误");
      authMsg.textContent = reason;
      speak(DENY_TEXT);
    });
  }

  function enterNav() {
    canvas.width = SAMPLE_W;
    canvas.height = SAMPLE_H;
    pageAuth.classList.add("hidden");
    pageNav.classList.add("active");
    keepAwake();

    if (supportVibrate) navigator.vibrate(80);
    var okMsg = supportVibrate
      ? "授权成功，已进入导航页面"
      : "授权成功。但当前设备不支持震动，将使用语音提示代替。已进入导航页面";
    speak(okMsg, function () {
      speak(GUIDE_TEXT, function () {
        detectGround();                       // 先识别背景色
        timer = setInterval(analyze, SAMPLE_INTERVAL);
      });
    });
  }

  authBtn.addEventListener("click", function () {
    if (authStarted) return;
    authStarted = true;
    authBtn.disabled = true;
    authBtn.textContent = "请留意弹窗…";

    testSpeaker();
    if (supportVibrate) {
      navigator.vibrate(80);
      setPerm(permVib, "ok", "已开启");
    } else {
      setPerm(permVib, "fail", "不支持");
    }

    if (promptSpoken) requestPermissions();
    else speak(PROMPT_TEXT, requestPermissions);
  });

  /* ============ 屏幕常亮（行走演示） ============ */
  var wakeLock = null;
  function keepAwake() {
    if (!("wakeLock" in navigator)) return;
    navigator.wakeLock.request("screen").then(function (wl) {
      wakeLock = wl;
    }).catch(function () { /* 不支持或被拒绝则忽略 */ });
  }
  document.addEventListener("visibilitychange", function () {
    if (document.visibilityState === "visible") keepAwake();
  });

  /* ============ 背景色自动识别（取画面底部条带中位色） ============ */

  // 采样底部 5 个补丁的平均色；返回 null 表示补丁间分歧大（底部被障碍物占据）
  function sampleGroundPatches() {
    if (video.readyState < 2) return null;
    ctx.drawImage(video, 0, 0, SAMPLE_W, SAMPLE_H);
    var data = ctx.getImageData(0, 0, SAMPLE_W, SAMPLE_H).data;
    var xs = [16, 48, 80, 112, 144];
    var patches = [];
    for (var k = 0; k < xs.length; k++) {
      var rs = 0, gs = 0, bs = 0, n = 0;
      for (var dy = 4; dy <= 12; dy += 2) {
        for (var dx = -4; dx <= 4; dx += 2) {
          var px = Math.min(Math.max(xs[k] + dx, 0), SAMPLE_W - 1);
          var py = SAMPLE_H - 1 - dy;
          var idx = (py * SAMPLE_W + px) * 4;
          rs += data[idx]; gs += data[idx + 1]; bs += data[idx + 2]; n++;
        }
      }
      patches.push([rs / n, gs / n, bs / n]);
    }
    return patches;
  }

  function patchesAvg(ps) {
    var r = 0, g = 0, b = 0;
    for (var i = 0; i < ps.length; i++) { r += ps[i][0]; g += ps[i][1]; b += ps[i][2]; }
    return [r / ps.length, g / ps.length, b / ps.length];
  }

  function dist2(a, b) {
    var dr = a[0] - b[0], dg = a[1] - b[1], db = a[2] - b[2];
    return dr * dr + dg * dg + db * db;
  }

  // 带稳定性门控的背景更新：
  // 1) 5 个补丁颜色必须互相接近（否则说明底部被障碍物占据，拒绝采样）
  // 2) 新背景与当前背景差异过大时拒绝（防止"障碍物贴脸"把地面色带跑偏）
  function updateGround() {
    var ps = sampleGroundPatches();
    if (!ps) return false;
    for (var i = 0; i < ps.length; i++) {
      for (var j2 = i + 1; j2 < ps.length; j2++) {
        if (dist2(ps[i], ps[j2]) > 45 * 45) return false; // 分歧大，底部有障碍物
      }
    }
    var cand = patchesAvg(ps);
    var cur = [groundR, groundG, groundB];
    if (dist2(cand, cur) > 60 * 60) return false;       // 与已知背景差异过大
    groundR = Math.round(cand[0]);
    groundG = Math.round(cand[1]);
    groundB = Math.round(cand[2]);
    return true;
  }

  function detectGround(retry) {
    if (video.readyState < 2) {
      if (retry !== false) setTimeout(function () { detectGround(); }, 200);
      return;
    }
    // 首次识别：放宽时序约束，直接取补丁均值（若底部被挡则提示用户）
    var ps = sampleGroundPatches();
    if (ps) {
      var cand = patchesAvg(ps);
      groundR = Math.round(cand[0]);
      groundG = Math.round(cand[1]);
      groundB = Math.round(cand[2]);
    }
    mode = "auto";
    swatchEl.style.background = "rgb(" + groundR + "," + groundG + "," + groundB + ")";
    modeText.textContent = "自动模式：已识别背景，无需锁定";
    modeBtn.classList.add("hidden");
  }

  /* ============ 点击画面锁定目标色（进入手动模式） ============ */
  video.addEventListener("click", function (e) {
    if (video.readyState < 2) return;
    var rect = video.getBoundingClientRect();
    var vw = video.videoWidth, vh = video.videoHeight;
    if (!vw || !vh) return;

    var scale = Math.max(rect.width / vw, rect.height / vh);
    var vx = (e.clientX - rect.left - (rect.width - vw * scale) / 2) / scale;
    var vy = (e.clientY - rect.top  - (rect.height - vh * scale) / 2) / scale;
    if (vx < 0 || vy < 0 || vx >= vw || vy >= vh) return;

    var sx = Math.round(vx / vw * SAMPLE_W);
    var sy = Math.round(vy / vh * SAMPLE_H);

    ctx.drawImage(video, 0, 0, SAMPLE_W, SAMPLE_H);
    var data = ctx.getImageData(0, 0, SAMPLE_W, SAMPLE_H).data;
    var r = 0, g = 0, b = 0, n = 0;
    for (var dy = -2; dy <= 2; dy++) {
      for (var dx = -2; dx <= 2; dx++) {
        var px = sx + dx, py = sy + dy;
        if (px < 0 || py < 0 || px >= SAMPLE_W || py >= SAMPLE_H) continue;
        var idx = (py * SAMPLE_W + px) * 4;
        r += data[idx]; g += data[idx + 1]; b += data[idx + 2]; n++;
      }
    }
    targetR = Math.round(r / n);
    targetG = Math.round(g / n);
    targetB = Math.round(b / n);

    mode = "manual";
    swatchEl.style.background = "rgb(" + targetR + "," + targetG + "," + targetB + ")";
    modeText.textContent = "已锁定目标色（点击画面更换）";
    modeBtn.classList.remove("hidden");
    speak("已锁定障碍物颜色");
  });

  modeBtn.addEventListener("click", function () {
    detectGround();
    speak("已切换回自动模式");
  });

  /* ============ 连通域扫描（一次洪泛，收集全部色块统计） ============ */
  function scanBlobs() {
    visited.fill(0);
    var blobs = [];
    for (var i = 0; i < TOTAL; i++) {
      if (!mask[i] || visited[i]) continue;
      var sp = 0;
      stack[sp++] = i; visited[i] = 1;
      var area = 0, sumX = 0, sumY = 0;
      var minX = SAMPLE_W, minY = SAMPLE_H, maxX = 0, maxY = 0;
      while (sp > 0) {
        var p = stack[--sp];
        var x = p % SAMPLE_W, y = (p / SAMPLE_W) | 0;
        area++; sumX += x; sumY += y;
        if (x < minX) minX = x;
        if (x > maxX) maxX = x;
        if (y < minY) minY = y;
        if (y > maxY) maxY = y;
        if (x > 0            && mask[p - 1]        && !visited[p - 1])        { visited[p - 1] = 1;        stack[sp++] = p - 1; }
        if (x < SAMPLE_W - 1 && mask[p + 1]        && !visited[p + 1])        { visited[p + 1] = 1;        stack[sp++] = p + 1; }
        if (y > 0            && mask[p - SAMPLE_W] && !visited[p - SAMPLE_W]) { visited[p - SAMPLE_W] = 1; stack[sp++] = p - SAMPLE_W; }
        if (y < SAMPLE_H - 1 && mask[p + SAMPLE_W] && !visited[p + SAMPLE_W]) { visited[p + SAMPLE_W] = 1; stack[sp++] = p + SAMPLE_W; }
      }
      blobs.push({
        area: area,
        minX: minX, minY: minY, maxX: maxX, maxY: maxY,
        cx: sumX / area, cy: sumY / area
      });
    }
    return blobs;
  }

  /* ============ 主分析流程 ============ */
  function analyze() {
    if (video.readyState < 2) return;
    ctx.drawImage(video, 0, 0, SAMPLE_W, SAMPLE_H);
    var data = ctx.getImageData(0, 0, SAMPLE_W, SAMPLE_H).data;

    var i, j, dr, dg, db;
    if (mode === "manual") {
      // 相近色掩码
      var t2m = TOL_MANUAL * TOL_MANUAL;
      for (i = 0, j = 0; i < TOTAL; i++, j += 4) {
        dr = data[j] - targetR; dg = data[j + 1] - targetG; db = data[j + 2] - targetB;
        mask[i] = (dr * dr + dg * dg + db * db) <= t2m ? 1 : 0;
      }
    } else {
      // 自动模式：先尝试温和更新背景色（门控防污染），再做背景差分
      updateGround();
      var t2a = TOL_AUTO * TOL_AUTO;
      for (i = 0, j = 0; i < TOTAL; i++, j += 4) {
        dr = data[j] - groundR; dg = data[j + 1] - groundG; db = data[j + 2] - groundB;
        mask[i] = (dr * dr + dg * dg + db * db) > t2a ? 1 : 0;
      }
    }

    var blobs = scanBlobs();

    // 选块：手动=最大块；自动=「接地」的最大块（底边位置与质心达标即视为地面上的障碍物）
    var minArea = Math.max(TOTAL * MIN_AREA_RATIO, MIN_AREA_PX);
    var best = null;
    for (var k = 0; k < blobs.length; k++) {
      var b = blobs[k];
      if (b.area < minArea) continue;
      if (mode === "auto") {
        var grounded = (b.maxY >= SAMPLE_H * GROUND_MAX_Y) && (b.cy >= SAMPLE_H * GROUND_CY);
        if (!grounded) continue;
        if (!best || b.area > best.area) best = b;
      } else {
        if (!best || b.area > best.area) best = b;
      }
    }

    var percent = best ? Math.round((best.area / TOTAL) * 1000) / 10 : 0;
    ratioEl.textContent = "障碍物占比: " + percent + "%";

    drawPreview(best);

    // 方位：质心相对画面左/中/右
    var dirText = "";
    if (best) {
      if (best.cx < SAMPLE_W / 3) dirText = "左前方";
      else if (best.cx > SAMPLE_W * 2 / 3) dirText = "右前方";
      else dirText = "正前方";
    }
    applyState(percent, dirText);
  }

  /* ============ 预览小窗 ============ */
  function drawPreview(best) {
    var out = outImg.data;
    for (var k = 0, m = 0; k < TOTAL; k++, m += 4) {
      var inBest = false;
      if (best) {
        var x = k % SAMPLE_W, y = (k / SAMPLE_W) | 0;
        inBest = x >= best.minX && x <= best.maxX && y >= best.minY && y <= best.maxY;
      }
      if (mask[k]) {
        if (inBest) { out[m] = 74; out[m + 1] = 222; out[m + 2] = 128; }
        else        { out[m] = 90; out[m + 1] = 90;  out[m + 2] = 90; }
      } else {
        out[m] = 10; out[m + 1] = 10; out[m + 2] = 10;
      }
      out[m + 3] = 255;
    }
    pctx.putImageData(outImg, 0, 0);
    if (best) {
      pctx.strokeStyle = "#ffffff";
      pctx.lineWidth = 2;
      pctx.strokeRect(best.minX + 0.5, best.minY + 0.5,
                      best.maxX - best.minX, best.maxY - best.minY);
    }
  }

  /* ============ 状态与触觉反馈 ============ */
  function applyState(percent, dirText) {
    var state, text, speakText;
    if (percent < 5) {
      state = "safe";    text = "当前状态: 安全";
      ratioEl.style.color = "#4ade80";
      speakText = "已恢复安全";
    } else if (percent < 20) {
      state = "warning"; text = "当前状态: 警告";
      ratioEl.style.color = "#facc15";
      if (supportVibrate) navigator.vibrate(100);
      speakText = "警告，" + dirText + "检测到障碍物";
    } else {
      state = "danger";  text = "当前状态: 危险";
      ratioEl.style.color = "#f87171";
      if (supportVibrate) navigator.vibrate([200, 100, 200]);
      if (supportVibrate && lastState === "danger") navigator.vibrate([200, 100, 200]);
      speakText = "危险，" + dirText + "有障碍物，请立即绕行";
    }

    statusEl.textContent = text;
    statusEl.className = "status-" + state;

    // 仅状态切换时播报一次，避免重复打扰
    if (state !== lastState) {
      if (state !== "safe" || lastState === "warning" || lastState === "danger") {
        speak(speakText);
      }
    }
    lastState = state;
  }
})();
</script>
</body>
</html>
