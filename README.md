# tduch

## これなら助かる! .js の書き方

1. document.getElementById("exId").addEventListener は避けるか工夫する
   React では id も addEventListener も使えないので使わないか、切り分けて書く
   書きに login の場合の例を示す

   ```js:app_login.js
   // ログイン
   export async function login(e) { // 渡す関数は切り分けて書く
       try {
           const email = this.email.value; // thisを付ける
           const password = this.password.value;
           const userCredential = await signInWithEmailAndPassword(auth, email, password);
           await userCredential.user.reload();
           if (!userCredential.user.emailVerified) {
               alert("メール認証がまだ完了していません。");
           } else{
               window.location.href = "bbs.html"; // 認証OK→掲示板へ
           }
       } catch (e) {
           alert("ログインエラー：" + e.message);
       }

   };

   const emailInput = document.getElementById("email");
   const passwordInput = document.getElementById("password");
   document.getElementById("login").addEventListener("click",
       {email: emailInput, password: passwordInput, handleEvent: login}); //引数と関数をオブジェクトにする
   ```

2. 可能な限り DOM 操作や html 要素の取得は切り分ける
   投稿内容の取得と表示の場合の例を示す

   ```js:app_thread.js
   const q = query(messagesRef, orderBy("createdAt", "asc"));
   const messagesDiv = document.getElementById("messages");

   onSnapshot(q, (snapshot) => {
      const messages = snapshot.docs.map((doc) => { // オブジェクトの配列を返し
          const data = doc.data();
          return {
              name: data.name,
              message: data.message,
              date: data.createdAt?.toDate?.().toLocaleString() || "",
          };
      });
      createMessageDiv(messages); // DOM操作用のヘルパー関数に渡す
   });

   // メッセージ表示
   function createMessageDiv(messages) {
      messagesDiv.innerHTML = "";
      messages.forEach((message) => {
          const el = document.createElement("div");
          el.className = "message";
          el.innerHTML = `<strong>${message.name}</strong>: ${message.message}<br><span>${message.date}</span>`;
          messagesDiv.appendChild(el);
      })
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
   }
   ```

3. Firebase の設定ファイルを切り分けておく
   下記のように一つのファイルにまとめておくとファイルの見通しが良くなる
   これから作るファイルだけでも OK

   ```js:client.js
   import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
   import {
       getFirestore,
   } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";
   import {
       getAuth,
   } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js"

   const firebaseConfig = {
       apiKey: "AIzaSyBMdNIVlGBLlatRpX5Y1nDUThBhaomq1vI",
       authDomain: "dendenkeiji.firebaseapp.com",
       projectId: "dendenkeiji",
   };

   const app = initializeApp(firebaseConfig);
   export const auth = getAuth(app); // 他のファイルに渡す変数や関数はexportを付ける
   export const db = getFirestore(app);
   ```

   他のファイルにはインポート文を書いておけば OK

   ```js
   import { auth, db } from "./client.js";
   ```
