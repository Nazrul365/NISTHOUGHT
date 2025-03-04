# NISTHOUGHT
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Referral SignUp</title>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore.js"></script>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; }
        .button { padding: 10px 20px; margin: 10px 0; background-color: #2e3a87; color: white; border: none; cursor: pointer; }
        .button:hover { background-color: #f39c12; }
    </style>
</head>
<body>

    <h1>Create an Account</h1>
    <button id="google-signin" class="button">Sign Up with Google</button>
    <button id="phone-signin" class="button">Sign Up with Phone</button>
    <div id="referral-code-section">
        <h2>Have a referral code?</h2>
        <input type="text" id="referral-code" placeholder="Enter referral code">
        <button id="submit-referral" class="button">Submit</button>
    </div>
    <div id="user-info"></div>

    <script>
        // Firebase Configuration
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_PROJECT_ID.appspot.com",
            messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
            appId: "YOUR_APP_ID"
        };
        const app = firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        // Google Sign-In
        document.getElementById('google-signin').addEventListener('click', async () => {
            const provider = new firebase.auth.GoogleAuthProvider();
            try {
                const result = await auth.signInWithPopup(provider);
                const user = result.user;
                handleUserSignUp(user);
            } catch (error) {
                console.error(error.message);
            }
        });

        // Phone Number Authentication
        document.getElementById('phone-signin').addEventListener('click', async () => {
            const phoneNumber = prompt("Enter your phone number (with country code):");
            const appVerifier = new firebase.auth.RecaptchaVerifier('phone-signin', {
                size: 'invisible'
            });
            try {
                const confirmationResult = await auth.signInWithPhoneNumber(phoneNumber, appVerifier);
                const verificationCode = prompt("Enter the verification code sent to your phone:");
                const result = await confirmationResult.confirm(verificationCode);
                const user = result.user;
                handleUserSignUp(user);
            } catch (error) {
                console.error(error.message);
            }
        });

        // Handle User Sign-Up
        function handleUserSignUp(user) {
            // Save user info to Firestore
            const userRef = db.collection('users').doc(user.uid);
            userRef.set({
                uid: user.uid,
                email: user.email,
                phone: user.phoneNumber || null,
                referralCode: generateReferralCode(),
                referredBy: getReferralCodeFromURL()
            });

            // Show user info
            document.getElementById('user-info').innerHTML = `
                <p>Welcome, ${user.displayName}</p>
                <p>Your referral code: ${generateReferralCode()}</p>
            `;
        }

        // Generate a random referral code
        function generateReferralCode() {
            return Math.random().toString(36).substring(2, 10).toUpperCase();
        }

        // Get referral code from URL or input
        function getReferralCodeFromURL() {
            const urlParams = new URLSearchParams(window.location.search);
            return urlParams.get('ref') || '';
        }

        // Submit Referral Code
        document.getElementById('submit-referral').addEventListener('click', async () => {
            const referralCode = document.getElementById('referral-code').value;
            if (referralCode) {
                const userRef = db.collection('users').doc(auth.currentUser.uid);
                await userRef.update({ referredBy: referralCode });
                alert('Referral code applied successfully!');
            } else {
                alert('Please enter a valid referral code.');
            }
        });
    </script>

</body>
</html>
