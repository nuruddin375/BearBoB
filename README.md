<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>

<script>
//────────────────────────────────────────
//  Firebase Config
//────────────────────────────────────────
const firebaseConfig = {
  apiKey: "AIzaSyDemoKey",
  authDomain: "demo-app.firebaseapp.com",
  databaseURL: "https://demo-app-default-rtdb.firebaseio.com",
  projectId: "demo-app",
  storageBucket: "demo-app.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:demo"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

const demoUserId = "user1";

//────────────────────────────────────────
//  Monetag Ad Ready Checker (FULL FIX)
//────────────────────────────────────────
let adReady = false;

function checkAdLoaded(){
    if (typeof show_9804827 !== "undefined") {
        adReady = true;
    } else {
        setTimeout(checkAdLoaded, 800);
    }
}
checkAdLoaded();

//────────────────────────────────────────
//  Initialize User
//────────────────────────────────────────
function initUser() {
  db.ref("users/" + demoUserId).once("value").then(s => {
    let data = s.val();
    if (!data) {
      data = {
        name:"MD Sobuj",
        id:"@Sobuj_boy_T1",
        balance:0,
        photo:"https://i.ibb.co/0jqHpnp/profile.png",
        history:[],
        tasksCompleted:0,
        totalEarnings:0,
        referrals:0,
        referralEarnings:0,
        memberSince:2023,
        level:1
      };
      db.ref("users/" + demoUserId).set(data);
    }
    updateUI(data);
  });
}

//────────────────────────────────────────
//  Update UI
//────────────────────────────────────────
function updateUI(data) {
  document.getElementById("username").innerText = data.name;
  document.getElementById("userid").innerText = data.id;
  document.getElementById("balance").innerText = data.balance.toFixed(2);

  document.getElementById("profileName").innerText = data.name;
  document.getElementById("profileId").innerText = data.id;

  document.getElementById("userPhoto").src = data.photo;
  document.getElementById("profilePhoto").src = data.photo;
  
  document.getElementById("withdraw-balance").innerText = data.balance.toFixed(2);

  document.getElementById("total-tasks").innerText = data.tasksCompleted;
  document.getElementById("total-earnings").innerText = data.totalEarnings.toFixed(2);
  document.getElementById("member-since").innerText = data.memberSince;
  document.getElementById("user-level").innerText = data.level;

  document.getElementById("referral-count").innerText = data.referrals;
  document.getElementById("referral-earnings").innerText = data.referralEarnings.toFixed(2);

  // Dynamic referral link
  document.getElementById("referral-link").innerText = 
      "https://t.me/techfisms_bot?start=" + demoUserId;
}

//────────────────────────────────────────
//  Add Balance
//────────────────────────────────────────
function addBalance(amount, desc) {
  db.ref("users/" + demoUserId).once("value").then(s => {
    let data = s.val();
    data.balance += amount;
    data.totalEarnings += amount;
    data.tasksCompleted += 1;

    data.history.push("+" + amount + "$ " + desc);

    db.ref("users/" + demoUserId).set(data);
    updateUI(data);
  });
}

//────────────────────────────────────────
//  Withdraw
//────────────────────────────────────────
function withdrawPoints() {
  let amount = parseFloat(document.getElementById("withdraw-amount").value);
  let method = document.getElementById("payment-method").value;
  let account = document.getElementById("withdraw-account").value;

  if (!amount || amount <= 0) {
    showStatus("Enter a valid amount", "error");
    return;
  }
  if (amount < 5) {
    showStatus("Minimum withdrawal $5.00", "error");
    return;
  }
  if (!account) {
    showStatus("Enter account number", "error");
    return;
  }

  db.ref("users/" + demoUserId).once("value").then(s => {
    let data = s.val();

    if (amount > data.balance) {
      showStatus("Insufficient balance", "error");
      return;
    }

    data.balance -= amount;
    data.history.push("-" + amount + "$ Withdrawal to " + method);

    db.ref("users/" + demoUserId).set(data);
    updateUI(data);

    document.getElementById("withdraw-amount").value = "";
    document.getElementById("withdraw-account").value = "";

    showStatus("Withdrawal request sent!", "success");
  });
}

//────────────────────────────────────────
//  Task System  (AD FIXED 100%)
//────────────────────────────────────────
function completeTask(type) {
  let amount = 0;
  let desc  = "";

  if(type === "ad"){ amount = 0.50; desc = "Advertisement watched"; }
  if(type === "telegram"){ amount = 1.00; desc = "Telegram channel joined"; }
  if(type === "survey"){ amount = 2.00; desc = "Survey completed"; }

  //──────────────────────────── FIXED AD LOGIC
  if (type === "ad") {

      if (!adReady) {
          showStatus("Ad loading… Try again in 2s", "info");
          return;
      }

      show_9804827()
      .then(() => {
          addBalance(amount, desc);
          showStatus(`Task completed! +$${amount}`, "success");
      })
      .catch(() => {
          showStatus("Ad failed! Try again.", "error");
      });

      return;
  }

  // Other tasks
  addBalance(amount, desc);
  showStatus(`Task completed! +$${amount}`, "success");
}

//────────────────────────────────────────
//  Copy referral link
//────────────────────────────────────────
function copyReferralLink() {
  const link = document.getElementById("referral-link").textContent;
  navigator.clipboard.writeText(link);
  showStatus("Referral link copied!", "success");
}

//────────────────────────────────────────
//  View Control
//────────────

