<!DOCTYPE html>

<html>

<head>

  <title>THE WINNER - Email Login</title>

  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>

  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-auth-compat.js"></script>

  <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>

</head>

<body>

  <h2>THE WINNER - Real Earning App</h2>


  <div id="authSection">

    <input type="email" id="email" placeholder="Email">

    <input type="password" id="password" placeholder="Password">

    <br><br>

    <button onclick="signUp()">Sign Up</button>

    <button onclick="login()">Login</button>

  </div>


  <div id="userSection" style="display:none;">

    <p>Balance: ₹<span id="balance">--</span></p>

    <p><span id="rewardStatus">Checking reward...</span></p>


    <h3>Withdraw ₹110</h3>

    <input type="text" id="upi" placeholder="Enter your UPI ID">

    <button onclick="requestWithdraw()">Request Withdraw</button>


    <p id="withdrawalMsg" style="color:green;"></p>

    <button onclick="logout()">Logout</button>

  </div>


  <script>

    const firebaseConfig = {

      apiKey: "AIzaSyDlhpoHPffa1pBQ0z8L7eyDTfWdMvtQeKQ",

      authDomain: window.location.hostname, // Automatic domain

      databaseURL: "https://the-winner-a25c3-default-rtdb.firebaseio.com/",

      projectId: "the-winner-a25c3",

      storageBucket: "the-winner-a25c3.appspot.com",

      messagingSenderId: "365924271664",

      appId: "1:365924271664:web:b856749b065c427f670ff5"

    };


    firebase.initializeApp(firebaseConfig);

    const auth = firebase.auth();

    const db = firebase.database();


    auth.onAuthStateChanged(user => {

      if (user) {

        document.getElementById('authSection').style.display = 'none';

        document.getElementById('userSection').style.display = 'block';

        initUser();

        loadBalance();

        checkDailyReward();

      } else {

        document.getElementById('authSection').style.display = 'block';

        document.getElementById('userSection').style.display = 'none';

      }

    });


    function isValidEmail(email) {

      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

      return re.test(email);

    }


    function signUp() {

      const email = document.getElementById('email').value.trim();

      const password = document.getElementById('password').value;


      if (!isValidEmail(email)) {

        alert("Please enter a valid email address.");

        return;

      }


      if (password.length < 6) {

        alert("Password must be at least 6 characters.");

        return;

      }


      auth.createUserWithEmailAndPassword(email, password).catch(e => alert(e.message));

    }


    function login() {

      const email = document.getElementById('email').value.trim();

      const password = document.getElementById('password').value;


      if (!isValidEmail(email)) {

        alert("Please enter a valid email address.");

        return;

      }


      auth.signInWithEmailAndPassword(email, password).catch(e => alert(e.message));

    }


    function logout() {

      auth.signOut();

    }


    function initUser() {

      const userId = auth.currentUser.uid;

      const userRef = db.ref('users/' + userId);

      userRef.once('value').then(snapshot => {

        if (!snapshot.exists()) {

          userRef.set({ balance: 0, lastLoginReward: "", withdrawals: {} });

        }

      });

    }


    function loadBalance() {

      const userId = auth.currentUser.uid;

      db.ref('users/' + userId + '/balance').on('value', snapshot => {

        document.getElementById('balance').innerText = snapshot.val() || 0;

      });

    }


    function checkDailyReward() {

      const userId = auth.currentUser.uid;

      const today = new Date().toISOString().split('T')[0];

      const userRef = db.ref('users/' + userId);


      userRef.once('value').then(snapshot => {

        const data = snapshot.val();

        if (data.lastLoginReward !== today) {

          const newBalance = (data.balance || 0) + 5;

          userRef.update({ balance: newBalance, lastLoginReward: today });

          document.getElementById('rewardStatus').innerText = "₹5 Daily Reward Added!";

        } else {

          document.getElementById('rewardStatus').innerText = "Today's reward already claimed.";

        }

      });

    }


    function requestWithdraw() {

      const userId = auth.currentUser.uid;

      const upi = document.getElementById('upi').value.trim();


      if (upi === "") {

        alert("Please enter UPI ID.");

        return;

      }


      db.ref('users/' + userId).once('value').then(snapshot => {

        const data = snapshot.val();

        if (data.balance >= 110) {

          const wid = db.ref().push().key;

          db.ref('users/' + userId + '/withdrawals/' + wid).set({

            method: "UPI",

            details: upi,

            amount: 110,

            status: "pending",

            requestedAt: Date.now()

          });

          db.ref('users/' + userId + '/balance').set(data.balance - 110);

          document.getElementById('withdrawalMsg').innerText = "Withdrawal Requested!";

        } else {

          alert("You need at least ₹110 to withdraw.");

        }

      });

    }

  </script>

</body>

</html>

