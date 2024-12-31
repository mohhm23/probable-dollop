<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>منصة الموهوبين</title>
    <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-firestore.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 20px;
        }
        form, .admin-panel, .student-panel {
            max-width: 400px;
            margin: auto;
            display: none;
            flex-direction: column;
            gap: 15px;
        }
        input, button {
            padding: 10px;
            font-size: 16px;
        }
        .visible {
            display: flex;
        }
    </style>
</head>
<body>
    <h1>منصة الموهوبين</h1>
    
    <!-- تسجيل الدخول -->
    <form id="loginForm">
        <input type="text" id="username" placeholder="اسم المستخدم" required>
        <input type="password" id="password" placeholder="كلمة المرور" required>
        <button type="submit">تسجيل الدخول</button>
    </form>

    <!-- لوحة التحكم للمسؤول -->
    <div class="admin-panel">
        <h2>لوحة التحكم</h2>
        <button onclick="showAddStudentForm()">إضافة طالب</button>
        <button onclick="showAddAssignmentForm()">إضافة واجب</button>
        <button onclick="logout()">تسجيل الخروج</button>

        <div id="addStudentForm" style="display:none;">
            <h3>إضافة طالب</h3>
            <input type="text" id="studentUsername" placeholder="اسم المستخدم">
            <input type="password" id="studentPassword" placeholder="كلمة المرور">
            <button onclick="addStudent()">إضافة</button>
        </div>

        <div id="addAssignmentForm" style="display:none;">
            <h3>إضافة واجب</h3>
            <input type="text" id="assignmentTitle" placeholder="عنوان الواجب">
            <input type="number" id="assignmentPoints" placeholder="النقاط">
            <button onclick="addAssignment()">إضافة</button>
        </div>
    </div>

    <!-- لوحة الطالب -->
    <div class="student-panel">
        <h2>مرحبًا بك في منصة الموهوبين</h2>
        <div id="assignments"></div>
        <button onclick="logout()">تسجيل الخروج</button>
    </div>

    <script>
        // Firebase configuration (لا يتم وضع المفاتيح هنا مباشرة)
        const firebaseConfig = {
            apiKey: process.env.FIREBASE_API_KEY,
            authDomain: process.env.FIREBASE_AUTH_DOMAIN,
            projectId: process.env.FIREBASE_PROJECT_ID,
            storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
            messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
            appId: process.env.FIREBASE_APP_ID,
            measurementId: process.env.FIREBASE_MEASUREMENT_ID
        };

        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        // تسجيل الدخول
        document.getElementById("loginForm").addEventListener("submit", async (e) => {
            e.preventDefault();
            const username = document.getElementById("username").value;
            const password = document.getElementById("password").value;

            const querySnapshot = await db.collection("users").where("username", "==", username).get();
            let loginSuccess = false;

            querySnapshot.forEach((doc) => {
                const userData = doc.data();
                if (userData.password === password) {
                    loginSuccess = true;
                    if (userData.role === "admin") {
                        document.querySelector(".admin-panel").classList.add("visible");
                    } else if (userData.role === "student") {
                        document.querySelector(".student-panel").classList.add("visible");
                        loadAssignments();
                    }
                    document.getElementById("loginForm").classList.remove("visible");
                }
            });

            if (!loginSuccess) {
                alert("اسم المستخدم أو كلمة المرور غير صحيحة.");
            }
        });

        // إضافة طالب
        function showAddStudentForm() {
            document.getElementById("addStudentForm").style.display = "block";
        }

        async function addStudent() {
            const username = document.getElementById("studentUsername").value;
            const password = document.getElementById("studentPassword").value;

            await db.collection("users").add({
                username,
                password,
                role: "student"
            });
            alert("تمت إضافة الطالب بنجاح.");
        }

        // إضافة واجب
        function showAddAssignmentForm() {
            document.getElementById("addAssignmentForm").style.display = "block";
        }

        async function addAssignment() {
            const title = document.getElementById("assignmentTitle").value;
            const points = document.getElementById("assignmentPoints").value;

            await db.collection("assignments").add({
                title,
                points
            });
            alert("تمت إضافة الواجب بنجاح.");
        }

        // عرض الواجبات للطلاب
        async function loadAssignments() {
            const assignmentsDiv = document.getElementById("assignments");
            assignmentsDiv.innerHTML = "";

            const querySnapshot = await db.collection("assignments").get();
            querySnapshot.forEach((doc) => {
                const data = doc.data();
                const div = document.createElement("div");
                div.textContent = `واجب: ${data.title} - النقاط: ${data.points}`;
                assignmentsDiv.appendChild(div);
            });
        }

        // تسجيل الخروج
        function logout() {
            location.reload();
        }
    </script>
</body>
</html>
# probable-dollop
