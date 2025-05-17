<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>無限波浪 RPG</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #111;
      color: #fff;
      text-align: center;
      padding: 20px;
    }
    #language-toggle {
      position: absolute;
      top: 10px;
      right: 10px;
    }
    .shop-item, .stats-upgrade {
      margin: 10px;
      padding: 10px;
      background-color: #333;
      border-radius: 5px;
    }
    .monster, .boss {
      background-color: red;
      padding: 10px;
      margin: 10px;
      display: inline-block;
    }
    .interface-box {
      background-color: #222;
      padding: 10px;
      margin: 10px;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div id="language-toggle">
    <button onclick="toggleLanguage()">切換語言</button>
  </div>

  <h1 id="title">無限波浪 RPG</h1>
  <div class="interface-box">
    <p id="player-name-label">請輸入玩家名稱：</p>
    <input type="text" id="player-name" />
    <button onclick="startGame()">開始</button>
  </div>

  <div id="game-interface" style="display:none">
    <div class="interface-box">
      <p id="wave-display">波浪：0</p>
      <p id="coins-display">金幣：0</p>
      <p id="rebirth-token-display">重生代幣：0</p>
    </div>

    <div class="interface-box">
      <button onclick="generateWave()">生成波浪</button>
    </div>

    <div id="monsters-area"></div>

    <div id="shop" class="interface-box">
      <h3>商店</h3>
      <div class="shop-item">火球 - 10000 金幣 <button onclick="buy('fireball')">購買</button></div>
      <div class="shop-item">弓箭 - 10000 金幣 <button onclick="buy('bow')">購買</button></div>
      <div class="shop-item">重生之劍 - 10 重生代幣 <button onclick="buy('rebirth_sword')">購買</button></div>
    </div>

    <div id="friends" class="interface-box">
      <h3>加好友</h3>
      <input type="text" id="friend-name" placeholder="輸入好友名稱">
      <button onclick="addFriend()">加好友</button>
      <ul id="friend-list"></ul>
    </div>

    <div id="rebirth" class="interface-box">
      <h3>重生系統</h3>
      <button onclick="rebirth()">重生</button>
    </div>

    <div id="stats" class="interface-box">
      <h3>統計升級</h3>
      <div class="stats-upgrade">提升傷害（1代幣）<button onclick="upgradeStat('damage')">升級</button></div>
      <div class="stats-upgrade">提升幸運（1代幣）<button onclick="upgradeStat('luck')">升級</button></div>
    </div>
  </div>

  <script>
    let language = 'zh';
    let player = {
      name: '',
      coins: 0,
      wave: 0,
      rebirthTokens: 0,
      stats: { damage: 1, luck: 1 },
      friends: []
    };

    function toggleLanguage() {
      language = language === 'zh' ? 'en' : 'zh';
      document.getElementById('title').textContent = language === 'zh' ? '無限波浪 RPG' : 'Endless Waves RPG';
      document.getElementById('player-name-label').textContent = language === 'zh' ? '請輸入玩家名稱：' : 'Enter Player Name:';
    }

    function startGame() {
      const name = document.getElementById('player-name').value;
      if (!name) return alert(language === 'zh' ? '請輸入名稱' : 'Please enter a name');
      player.name = name;
      document.getElementById('game-interface').style.display = 'block';
    }

    function generateWave() {
      player.wave++;
      document.getElementById('wave-display').textContent = `波浪：${player.wave}`;
      const area = document.getElementById('monsters-area');
      area.innerHTML = '';
      let isBoss = player.wave % 10 === 0;
      let monster = document.createElement('div');
      monster.className = isBoss ? 'boss' : 'monster';
      let hp = isBoss ? (player.wave === 100 ? 10000 : 1000) : 100;
      monster.textContent = (isBoss ? 'Boss' : '怪物') + ` HP: ${hp}`;
      monster.onclick = () => {
        hp -= player.stats.damage * (player.hasGodSword ? 100 : 25);
        monster.textContent = (isBoss ? 'Boss' : '怪物') + ` HP: ${hp}`;
        if (hp <= 0) {
          player.coins += 100;
          if (isBoss && Math.random() < 0.5) alert('掉落木劍！');
          if (player.wave === 100 && Math.random() < 0.1) {
            player.hasGodSword = true;
            alert('你獲得神劍！攻擊力+100');
          }
          document.getElementById('coins-display').textContent = `金幣：${player.coins}`;
          monster.remove();
        }
      };
      area.appendChild(monster);
    }

    function buy(item) {
      if (item === 'fireball' || item === 'bow') {
        if (player.coins >= 10000) {
          player.coins -= 10000;
          alert(`${item} 已購買`);
        }
      } else if (item === 'rebirth_sword') {
        if (player.rebirthTokens >= 10) {
          player.rebirthTokens -= 10;
          alert('重生之劍已購買！');
        }
      }
      document.getElementById('coins-display').textContent = `金幣：${player.coins}`;
      document.getElementById('rebirth-token-display').textContent = `重生代幣：${player.rebirthTokens}`;
    }

    function addFriend() {
      const name = document.getElementById('friend-name').value;
      if (name && !player.friends.includes(name)) {
        player.friends.push(name);
        const li = document.createElement('li');
        li.textContent = name;
        document.getElementById('friend-list').appendChild(li);
      }
    }

    function rebirth() {
      if (player.wave >= 100) {
        player.wave = 0;
        player.rebirthTokens++;
        player.coins = 0;
        document.getElementById('wave-display').textContent = `波浪：${player.wave}`;
        document.getElementById('rebirth-token-display').textContent = `重生代幣：${player.rebirthTokens}`;
        alert('你已重生！');
      } else {
        alert('需要達到100波才能重生');
      }
    }

    function upgradeStat(stat) {
      if (player.rebirthTokens > 0) {
        player.stats[stat]++;
        player.rebirthTokens--;
        alert(`${stat} 提升至 ${player.stats[stat]}`);
        document.getElementById('rebirth-token-display').textContent = `重生代幣：${player.rebirthTokens}`;
      }
    }
  </script>
</body>
</html>

