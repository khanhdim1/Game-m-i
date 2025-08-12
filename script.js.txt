// script.js

// Thông tin tài khoản lưu trong localStorage:
// users = { username: { password, balance } }

const loginContainer = document.getElementById('login-container');
const mainContainer = document.getElementById('main-container');
const usernameInput = document.getElementById('username');
const passwordInput = document.getElementById('password');
const btnLogin = document.getElementById('btn-login');
const btnRegister = document.getElementById('btn-register');
const loginMsg = document.getElementById('login-msg');
const displayUsername = document.getElementById('display-username');
const displayBalance = document.getElementById('display-balance');
const btnLogout = document.getElementById('btn-logout');
const gameButtons = document.querySelectorAll('.game-btn');
const gameArea = document.getElementById('game-area');

let currentUser = null;

// Lấy danh sách user từ localStorage
function getUsers() {
  const usersStr = localStorage.getItem('users');
  return usersStr ? JSON.parse(usersStr) : {};
}

function saveUsers(users) {
  localStorage.setItem('users', JSON.stringify(users));
}

function showMessage(msg, color = 'red') {
  loginMsg.style.color = color;
  loginMsg.textContent = msg;
}

function updateUserBalanceUI() {
  if(currentUser) {
    const users = getUsers();
    displayBalance.textContent = users[currentUser].balance;
  }
}

// Đăng ký
btnRegister.onclick = () => {
  const username = usernameInput.value.trim();
  const password = passwordInput.value.trim();
  if(!username || !password) {
    showMessage('Vui lòng nhập đủ tên và mật khẩu');
    return;
  }
  const users = getUsers();
  if(users[username]) {
    showMessage('Tài khoản đã tồn tại');
    return;
  }
  users[username] = { password, balance: 1000 };
  saveUsers(users);
  showMessage('Đăng ký thành công, bạn có 1000 xu', 'green');
}

// Đăng nhập
btnLogin.onclick = () => {
  const username = usernameInput.value.trim();
  const password = passwordInput.value.trim();
  if(!username || !password) {
    showMessage('Vui lòng nhập đủ tên và mật khẩu');
    return;
  }
  const users = getUsers();
  if(!users[username] || users[username].password !== password) {
    showMessage('Sai tên tài khoản hoặc mật khẩu');
    return;
  }
  currentUser = username;
  loginContainer.classList.add('hidden');
  mainContainer.classList.remove('hidden');
  displayUsername.textContent = currentUser;
  updateUserBalanceUI();
  loginMsg.textContent = '';
  loadGame('slot'); // Mặc định vào game slot
}

// Đăng xuất
btnLogout.onclick = () => {
  currentUser = null;
  loginContainer.classList.remove('hidden');
  mainContainer.classList.add('hidden');
  usernameInput.value = '';
  passwordInput.value = '';
  loginMsg.textContent = '';
  gameArea.innerHTML = '';
}

// Xử lý chọn game
gameButtons.forEach(btn => {
  btn.onclick = () => {
    if (!currentUser) return;
    loadGame(btn.dataset.game);
  };
});

// Load game theo tên
function loadGame(name) {
  gameArea.innerHTML = ''; // Xóa vùng chơi

  switch(name) {
    case 'slot':
      loadSlotGame();
      break;
    case 'baccarat':
      loadBaccaratGame();
      break;
    case 'dogfight':
      loadDogFightGame();
      break;
    case 'lottery':
      loadLotteryGame();
      break;
    default:
      gameArea.textContent = "Game chưa có, đang phát triển...";
  }
}

// Lấy và cập nhật số dư
function getBalance() {
  const users = getUsers();
  return users[currentUser].balance;
}

function updateBalance(newBalance) {
  const users = getUsers();
  users[currentUser].balance = newBalance;
  saveUsers(users);
  updateUserBalanceUI();
}

// --- Demo game slot đơn giản ---
function loadSlotGame() {
  gameArea.innerHTML = `
    <h2>Nổ Hũ Slot Machine</h2>
    <div id="slot-machine" style="display:flex;gap:15px;justify-content:center;margin:20px 0;">
      <div class="slot" style="width:80px;height:80px;background:#222d44;border-radius:15px;display:flex;align-items:center;justify-content:center;font-size:50px;color:#0ff;">🍒</div>
      <div class="slot" style="width:80px;height:80px;background:#222d44;border-radius:15px;display:flex;align-items:center;justify-content:center;font-size:50px;color:#0ff;">🍋</div>
      <div class="slot" style="width:80px;height:80px;background:#222d44;border-radius:15px;display:flex;align-items:center;justify-content:center;font-size:50px;color:#0ff;">🔔</div>
    </div>
    <button id="spin-btn" style="padding:10px 30px;margin:10px;">Quay</button>
    <div id="slot-info" style="margin-top:15px;color:#0f0;font-weight:bold;"></div>
  `;

  const symbols = ['🍒', '🍋', '🔔', '🍉', '⭐', '💎'];
  let balance = getBalance();
  const spinBtn = document.getElementById('spin-btn');
  const slotElems = document.querySelectorAll('#slot-machine .slot');
  const slotInfo = document.getElementById('slot-info');

  function randomSymbol() {
    return symbols[Math.floor(Math.random() * symbols.length)];
  }

  function checkWin(slots) {
    if (slots[0] === slots[1] && slots[1] === slots[2]) return 100;
    if (slots[0] === slots[1] || slots[1] === slots[2] || slots[0] === slots[2]) return 20;
    return 0;
  }

  function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  spinBtn.onclick = async () => {
    balance = getBalance();
    if (balance < 10) {
      slotInfo.textContent = "Bạn không đủ xu để cược!";
      return;
    }
    balance -= 10;
    updateBalance(balance);
    slotInfo.textContent = "Đang quay...";
    for (let i = 0; i < 15; i++) {
      slotElems.forEach(e => e.textContent = randomSymbol());
      await sleep(100 + i * 20);
    }
    const finalSlots = Array.from(slotElems).map(e => randomSymbol());
    finalSlots.forEach((sym, i) => slotElems[i].textContent = sym);
    const winAmount = checkWin(finalSlots);
    if (winAmount > 0) {
      balance += winAmount;
      updateBalance(balance);
      slotInfo.textContent = `Bạn thắng ${winAmount} xu! 🎉`;
    } else {
      slotInfo.textContent = "Bạn không thắng lần này!";
    }
  };
}

// --- Demo baccarat đơn giản ---
function loadBaccaratGame() {
  gameArea.innerHTML = `
    <h2>Baccarat (Demo đơn giản)</h2>
    <div>
      <button id="btn-player">Cược Player</button>
      <button id="btn-banker">Cược Banker</button>
      <button id="btn-tie">Cược Hòa</button>
    </div>
    <div id="baccarat-result" style="margin-top: 15px; font-weight: bold; color: #0f0;"></div>
  `;

  const resultDiv = document.getElementById('baccarat-result');

  function randomScore() {
    return Math.floor(Math.random() * 10) + Math.floor(Math.random() * 10);
  }

  function mod10(n) {
    return n % 10;
  }

  function playBaccarat(betOn) {
    let balance = getBalance();
    if (balance < 10) {
      resultDiv.textContent = "Bạn không đủ xu để cược!";
      return;
    }
    balance -= 10;
    updateBalance(balance);

    let playerScore = mod10(randomScore());
    let bankerScore = mod10(randomScore());

    let winner = 'Tie';
    if (playerScore > bankerScore) winner = 'Player';
    else if (bankerScore > playerScore) winner = 'Banker';

    let winAmount = 0;
    if (betOn === winner) {
      winAmount = betOn === 'Tie' ? 50 : 20;
      balance += winAmount;
      updateBalance(balance);
      resultDiv.textContent = `Player: ${playerScore}, Banker: ${bankerScore} - Bạn thắng ${winAmount} xu! 🎉`;
    } else {
      resultDiv.textContent = `Player: ${playerScore}, Banker: ${bankerScore} - Bạn thua!`;
    }
  }

  document.getElementById('btn-player').onclick = () => playBaccarat('Player');
  document.getElementById('btn-banker').onclick = () => playBaccarat('Banker');
  document.getElementById('btn-tie').onclick = () => playBaccarat('Tie');
}

// --- Demo đá gà đơn giản ---
function loadDogFightGame() {
  gameArea.innerHTML = `
    <h2>Đá Gà (Demo đơn giản)</h2>
    <div>
      <button id="btn-gallo">Cược Gà 1</button>
      <button id="btn-galtw">Cược Gà 2</button>
    </div>
    <div id="dogfight-result" style="margin-top: 15px; font-weight: bold; color: #0f0;"></div>
  `;

  const resultDiv = document.getElementById('dogfight-result');

  function playDogFight(betOn) {
    let balance = getBalance();
    if (balance < 10) {
      resultDiv.textContent = "Bạn không đủ xu để cược!";
      return;
    }
    balance -= 10;
    updateBalance(balance);

    let winner = Math.random() < 0.5 ? 'Gà 1' : 'Gà 2';

    if (betOn === winner) {
      balance += 20;
      updateBalance(balance);
      resultDiv.textContent = `Gà thắng: ${winner} - Bạn thắng 20 xu! 🎉`;
    } else {
      resultDiv.textContent = `Gà thắng: ${winner} - Bạn thua!`;
    }
  }

  document.getElementById('btn-gallo').onclick = () => playDogFight('Gà 1');
  document.getElementById('btn-galtw').onclick = () => playDogFight('Gà 2');
}

// --- Demo số đề (lô đề) đơn giản ---
function loadLotteryGame() {
  gameArea.innerHTML = `
    <h2>Số Đề (Lô Đề) Demo</h2>
    <div>
      <input type="number" id="lottery-input" placeholder="Chọn số 00-99" min="0" max="99" style="width: 100px; font-size: 18px; padding: 5px;" />
      <button id="btn-lottery-play">Đặt cược 10 xu</button>
    </div>
    <div id="lottery-result" style="margin-top: 15px; font-weight: bold; color: #0f0;"></div>
  `;

  const input = document.getElementById('lottery-input');
  const resultDiv = document.getElementById('lottery-result');

  function playLottery(num) {
    let balance = getBalance();
    if (balance < 10) {
      resultDiv.textContent = "Bạn không đủ xu để cược!";
      return;
    }
    if (num < 0 || num > 99 || isNaN(num)) {
      resultDiv.textContent = "Số không hợp lệ, vui lòng chọn 00-99";
      return;
    }
    balance -= 10;
    updateBalance(balance);

    const winningNum = Math.floor(Math.random() * 100);

    if (num === winningNum) {
      const winAmount = 900;
      balance += winAmount;
      updateBalance(balance);
