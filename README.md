<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Password Manager</title>
<style>
  :root{
    --bg:#121212; --fg:#ffffff; --card:#1e1e1e; --accent:#00ff99; --accent-2:#00cc77;
    --muted:#9aa0a6; --danger:#ff5a5f; --warning:#ffb020; --ok:#34c759;
    --border:rgba(255,255,255,.08);
  }
  body.light{
    --bg:#f5f5f5; --fg:#111; --card:#ffffff; --border:rgba(0,0,0,.08);
  }

  /* Base */
  *{box-sizing:border-box}
  body{font-family:system-ui,Segoe UI,Arial,sans-serif; background:var(--bg); color:var(--fg); margin:0}
  h1{color:var(--accent); text-align:center; padding:20px 16px; margin:0}
  h2{margin:0 0 12px 0}

  /* Top bar */
  .topbar{display:flex; align-items:center; justify-content:flex-end; padding:10px 16px; position:sticky; top:0; background:linear-gradient( to bottom, rgba(0,0,0,.18), transparent);}
  .iconbtn{background:transparent; border:none; color:var(--fg); font-size:1.5rem; cursor:pointer; padding:6px 10px; border-radius:12px}
  .iconbtn:hover{background:var(--border)}
  #loginBtn{margin-left:auto}

  /* Layout */
  .main{display:grid; gap:16px; padding:0 16px 110px}
  @media(min-width:980px){
    .main{grid-template-columns:1fr 1fr; max-width:1100px; margin:0 auto}
  }

  /* Cards / containers */
  .card{background:var(--card); border:1px solid var(--border); border-radius:14px; padding:18px}
  .row{display:flex; gap:10px; flex-wrap:wrap}
  .stack{display:flex; flex-direction:column; gap:10px}
  .field label{display:block; font-size:.9rem; color:var(--muted); margin-bottom:4px}
  .field input, .field select{width:100%; padding:10px 12px; border:1px solid var(--border); border-radius:10px; background:transparent; color:var(--fg)}
  .pill{background:var(--accent); color:#000; border:none; padding:10px 14px; border-radius:999px; font-weight:700; cursor:pointer}
  .pill:hover{background:var(--accent-2)}
  .ghost{background:transparent; border:1px solid var(--border); color:var(--fg)}
  .ghost:hover{border-color:var(--accent)}
  .muted{color:var(--muted)}

  /* Actions strip */
  .actions{display:flex; justify-content:center; gap:10px; flex-wrap:wrap; margin:6px 0 16px}

  /* Strength */
  .strength{font-weight:700; text-align:center; margin-top:6px}
  .strength.weak{color:#ff5252}
  .strength.medium{color:#ffb020}
  .strength.strong{color:#00ff99}

  /* Saved list */
  #savedList .item{display:grid; grid-template-columns:1fr auto; gap:8px; align-items:center; padding:12px; border:1px solid var(--border); border-radius:12px; background:rgba(0,0,0,.05)}
  body.light #savedList .item{background:rgba(0,0,0,.03)}
  .meta{display:flex; gap:8px; align-items:center; flex-wrap:wrap}
  .badge{font-size:.75rem; padding:4px 8px; border-radius:999px; border:1px solid var(--border)}
  .btns{display:flex; gap:6px; flex-wrap:wrap}
  .chip{border:1px solid var(--border); background:transparent; color:var(--fg); padding:6px 10px; border-radius:999px; cursor:pointer; display:inline-flex; gap:6px; align-items:center}
  .chip:hover{border-color:var(--accent)}
  .name{font-weight:700}
  .secret{font-family: ui-monospace, SFMono-Regular, Menlo, monospace; letter-spacing: .6px}

  /* Inline forms (no popups) */
  .panel{display:none}
  .panel.show{display:block; animation:fade .18s ease-in}
  @keyframes fade{from{opacity:0; transform:translateY(-4px)} to{opacity:1; transform:none}}

  /* Bottom fixed buttons */
  .fixed{position:fixed; bottom:18px; z-index:20}
  #themeToggle{left:18px}
  #helpBtn{right:18px}
  .fab{width:52px; height:52px; border-radius:50%; border:none; background:var(--accent); color:#000; font-size:1.3rem; cursor:pointer; display:flex; align-items:center; justify-content:center; box-shadow:0 10px 30px rgba(0,0,0,.25)}
  .fab:hover{background:var(--accent-2)}

  /* Help modal */
  #helpModal{display:none; position:fixed; inset:0; background:rgba(0,0,0,.6); z-index:50; align-items:center; justify-content:center; padding:16px}
  #helpContent{background:#fff; color:#000; border-radius:14px; max-width:560px; width:100%; padding:18px}
  #helpContent h3{margin:0 0 10px 0}
  #helpContent p{margin:8px 0}

  /* Tooltips */
  .chip[data-tip]{position:relative}
  .chip[data-tip]:hover::after{
    content:attr(data-tip); position:absolute; bottom:calc(100% + 6px); left:50%; transform:translateX(-50%);
    background:#000; color:#fff; font-size:.75rem; padding:4px 8px; border-radius:6px; white-space:nowrap;
  }
</style>
</head>
<body>
  <h1>Password Manager</h1>

  <div class="topbar">
    <button id="loginBtn" class="iconbtn" title="Login / Logout">👤</button>
  </div>

  <div class="actions">
    <button id="saveOpenBtn" class="pill">🔑 Save Password</button>
    <button id="importOpenBtn" class="pill">📥 Import Password</button>
    <button id="viewSavedBtn" class="pill ghost">📂 Saved Passwords</button>
  </div>

  <div class="main">
    <!-- Left column: generator + inline panels -->
    <section class="stack">
      <div class="card">
        <h2>Password Generator</h2>
        <div class="row">
          <div class="field" style="flex:1">
            <label>Password length</label>
            <input type="number" id="passwordLength" value="12" min="4" max="64"/>
          </div>
          <div class="field" style="flex:1">
            <label>Password type</label>
            <select id="passwordType">
              <option value="letters">Letters only</option>
              <option value="lettersNumbers">Letters + numbers</option>
              <option value="lettersNumbersSymbols">Letters + numbers + symbols</option>
            </select>
          </div>
        </div>
        <div class="row">
          <button id="generatePasswordBtn" class="pill">Generate</button>
          <button id="copyGeneratedBtn" class="pill ghost">Copy</button>
        </div>
        <div class="stack" style="margin-top:8px">
          <div id="passwordResult" class="secret"></div>
          <div id="passwordStrength" class="strength"></div>
        </div>
      </div>

      <!-- Save inline panel -->
      <div id="savePanel" class="card panel">
        <h2>Save Password</h2>
        <div class="field">
          <label>Name (required)</label>
          <input id="saveName" placeholder="e.g. Google"/>
        </div>
        <div class="field">
          <label>Username (optional)</label>
          <input id="saveUsername" placeholder="e.g. user@gmail.com"/>
        </div>
        <div class="field">
          <label>Password</label>
          <input id="savePassword" class="secret" placeholder="autofilled from generator"/>
        </div>
        <div class="row" style="align-items:center">
          <label class="muted">Store in:</label>
          <label><input type="radio" name="saveWhere" value="local" checked/> Local</label>
          <label><input type="radio" name="saveWhere" value="account"/> Account</label>
        </div>
        <div class="row">
          <button id="saveDoBtn" class="pill">Save</button>
          <button id="saveCancelBtn" class="pill ghost">Cancel</button>
        </div>
        <div class="muted" id="saveHint" style="margin-top:6px"></div>
      </div>

      <!-- Import inline panel -->
      <div id="importPanel" class="card panel">
        <h2>Import Password</h2>
        <div class="field">
          <label>Name (required)</label>
          <input id="importName" placeholder="e.g. Facebook"/>
        </div>
        <div class="field">
          <label>Username (optional)</label>
          <input id="importUsername" placeholder="e.g. john@site.com"/>
        </div>
        <div class="field">
          <label>Password (required)</label>
          <input id="importPassword" class="secret" placeholder="Enter password"/>
        </div>
        <div class="row" style="align-items:center">
          <label class="muted">Store in:</label>
          <label><input type="radio" name="importWhere" value="local" checked/> Local</label>
          <label><input type="radio" name="importWhere" value="account"/> Account</label>
        </div>
        <div class="row">
          <button id="importDoBtn" class="pill">Import</button>
          <button id="importCancelBtn" class="pill ghost">Cancel</button>
        </div>
        <div class="muted" id="importHint" style="margin-top:6px"></div>
      </div>
    </section>

    <!-- Right column: saved list -->
    <section class="card">
      <h2>Saved Passwords</h2>
      <input id="searchSaved" placeholder="Search by name..." class="field" style="width:100%; padding:10px 12px; border:1px solid var(--border); border-radius:10px; background:transparent; color:var(--fg)"/>
      <div id="savedList" class="stack" style="margin-top:12px"></div>
      <p id="noPasswords" class="muted" style="text-align:center; display:none">You have no saved passwords.</p>
    </section>
  </div>

  <!-- Fixed buttons -->
  <button id="themeToggle" class="fab fixed">🌙</button>
  <button id="helpBtn" class="fab fixed">❓</button>

  <!-- Help modal -->
  <div id="helpModal">
    <div id="helpContent">
      <h3>Help & Info</h3>
      <p>🔑 Generate strong passwords and save them either locally on this device or to your account for sync.</p>
      <p>☁️ “Account” saves to your cloud (same login), so you can access from other devices.</p>
      <p>🕶 Passwords are masked by default in the list—use the eye button to reveal.</p>
      <p>🧭 Desktop shows a two-column layout, mobile stacks everything for clarity.</p>
      <div style="display:flex; gap:8px; justify-content:flex-end; margin-top:12px">
        <button id="helpClose" class="pill">Close</button>
      </div>
    </div>
  </div>

  <!-- Firebase SDK -->
  <script type="module">
    /* =========================
       Firebase (v9 modular)
    ==========================*/
    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
    import { getAuth, onAuthStateChanged, signInWithPopup, GoogleAuthProvider, signOut } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js";
    import { getFirestore, collection, addDoc, query, where, getDocs, deleteDoc, doc, updateDoc } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";

    const firebaseConfig = {
      apiKey: "AIzaSyD6wyq8fLh4QiW2PP0e22I9pyACRNvSPkc",
      authDomain: "pasword-manager-d4ab6.firebaseapp.com",
      projectId: "pasword-manager-d4ab6",
      storageBucket: "pasword-manager-d4ab6.appspot.com",
      messagingSenderId: "1070489046614",
      appId: "1:1070489046614:web:96f8fcd64b4a0a939e75f7"
    };

    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);
    const provider = new GoogleAuthProvider();

    let currentUser = null;

    onAuthStateChanged(auth, (user) => {
      currentUser = user;
      document.getElementById("loginBtn").textContent = user ? "🚪" : "👤";
      // Hint updates
      updateWhereHints();
    });

    /* =========================
       Theme & Help
    ==========================*/
    const themeToggle = document.getElementById("themeToggle");
    themeToggle.addEventListener("click", () => {
      const light = document.body.classList.toggle("light");
      themeToggle.textContent = light ? "☀️" : "🌙";
    });

    const helpBtn = document.getElementById("helpBtn");
    const helpModal = document.getElementById("helpModal");
    const helpClose = document.getElementById("helpClose");
    helpBtn.onclick = () => helpModal.style.display = "flex";
    helpClose.onclick = () => helpModal.style.display = "none";
    helpModal.addEventListener("click", (e)=>{ if(e.target===helpModal) helpModal.style.display="none"; });

    /* =========================
       Login / Logout
    ==========================*/
    const loginBtn = document.getElementById("loginBtn");
    loginBtn.addEventListener("click", async () => {
      if (!currentUser) {
        try { await signInWithPopup(auth, provider); } catch(e){ console.error(e); }
      } else {
        await signOut(auth);
      }
    });

    /* =========================
       Generator
    ==========================*/
    const genBtn = document.getElementById("generatePasswordBtn");
    const copyGenBtn = document.getElementById("copyGeneratedBtn");
    const resultEl = document.getElementById("passwordResult");
    const strengthEl = document.getElementById("passwordStrength");

    function generatePassword() {
      const length = parseInt(document.getElementById("passwordLength").value);
      const type = document.getElementById("passwordType").value;
      let chars =
        type === "letters" ? "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ" :
        type === "lettersNumbers" ? "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789" :
        "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()_+-=[]{}|;:,.<>?";
      let pw = "";
      for (let i=0;i<length;i++){ pw += chars.charAt(Math.floor(Math.random()*chars.length)); }
      resultEl.textContent = pw;
      checkStrength(pw, strengthEl);
      // Pre-fill Save form if open
      const savePw = document.getElementById("savePassword");
      if (savePw) savePw.value = pw;
    }
    genBtn.addEventListener("click", generatePassword);
    copyGenBtn.addEventListener("click", ()=> {
      if (!resultEl.textContent) return;
      navigator.clipboard.writeText(resultEl.textContent);
    });

    function checkStrength(pw, targetEl){
      let score = 0;
      if (pw.length >= 8) score++;
      if (/[A-Z]/.test(pw)) score++;
      if (/[0-9]/.test(pw)) score++;
      if (/[^A-Za-z0-9]/.test(pw)) score++;
      if (score<=1){ targetEl.textContent="Weak"; targetEl.className="strength weak"; }
      else if(score===2){ targetEl.textContent="Medium"; targetEl.className="strength medium"; }
      else{ targetEl.textContent="Strong"; targetEl.className="strength strong"; }
    }

    /* =========================
       Panels: Save & Import (inline)
    ==========================*/
    const savePanel = document.getElementById("savePanel");
    const importPanel = document.getElementById("importPanel");
    const saveOpenBtn = document.getElementById("saveOpenBtn");
    const importOpenBtn = document.getElementById("importOpenBtn");

    saveOpenBtn.addEventListener("click", ()=>{
      togglePanel(savePanel, true);
      togglePanel(importPanel, false);
      // prefill from generator if exists
      document.getElementById("savePassword").value = resultEl.textContent || "";
      updateWhereHints();
    });
    importOpenBtn.addEventListener("click", ()=>{
      togglePanel(importPanel, true);
      togglePanel(savePanel, false);
      updateWhereHints();
    });

    function togglePanel(el, show){
      if (!el) return;
      if (show) el.classList.add("show");
      else el.classList.remove("show");
    }

    // Hints for where to save
    function updateWhereHints(){
      const saveHint = document.getElementById("saveHint");
      const importHint = document.getElementById("importHint");
      const accTxt = currentUser ? `Account: ${currentUser.displayName || currentUser.email || "Signed in"}` : "Account: Sign in required";
      if (saveHint) saveHint.textContent = `Local: stored only on this device • ${accTxt}`;
      if (importHint) importHint.textContent = `Local: stored only on this device • ${accTxt}`;
    }

    // Save form
    document.getElementById("saveDoBtn").addEventListener("click", async ()=>{
      const name = document.getElementById("saveName").value.trim();
      const username = document.getElementById("saveUsername").value.trim();
      const password = document.getElementById("savePassword").value;
      const where = [...document.querySelectorAll('input[name="saveWhere"]')].find(r=>r.checked)?.value;

      if (!name){ shake(document.getElementById("saveName")); return; }
      if (!password){ shake(document.getElementById("savePassword")); return; }

      await storeCredential({name, username, password}, where);
      clearSaveForm(); togglePanel(savePanel,false);
      await refreshList(); // update UI
    });
    document.getElementById("saveCancelBtn").addEventListener("click", ()=>{ clearSaveForm(); togglePanel(savePanel,false); });
    function clearSaveForm(){
      document.getElementById("saveName").value="";
      document.getElementById("saveUsername").value="";
      // keep password prefilled from generator
    }

    // Import form
    document.getElementById("importDoBtn").addEventListener("click", async ()=>{
      const name = document.getElementById("importName").value.trim();
      const username = document.getElementById("importUsername").value.trim();
      const password = document.getElementById("importPassword").value;
      const where = [...document.querySelectorAll('input[name="importWhere"]')].find(r=>r.checked)?.value;

      if (!name){ shake(document.getElementById("importName")); return; }
      if (!password){ shake(document.getElementById("importPassword")); return; }

      await storeCredential({name, username, password}, where);
      clearImportForm(); togglePanel(importPanel,false);
      await refreshList();
    });
    document.getElementById("importCancelBtn").addEventListener("click", ()=>{ clearImportForm(); togglePanel(importPanel,false); });
    function clearImportForm(){
      document.getElementById("importName").value="";
      document.getElementById("importUsername").value="";
      document.getElementById("importPassword").value="";
    }

    function shake(el){ el.style.outline="2px solid #ff5252"; setTimeout(()=>el.style.outline="none", 500); }

    async function storeCredential(cred, where){
      if (where==="account"){
        if (!currentUser){
          try{ await signInWithPopup(auth, provider); }
          catch(e){ console.error(e); return; }
        }
        if (!currentUser) return;
        await addDoc(collection(db,"passwords"), {
          uid: currentUser.uid,
          name: cred.name,
          username: cred.username || "",
          password: cred.password
        });
      } else {
        const key = "passwords";
        const arr = JSON.parse(localStorage.getItem(key)||"[]");
        arr.push({ name: cred.name, username: cred.username || "", password: cred.password });
        localStorage.setItem(key, JSON.stringify(arr));
      }
    }

    /* =========================
       Saved list (render/search)
    ==========================*/
    const savedContainer = document.getElementById("savedList");
    const noPasswords = document.getElementById("noPasswords");
    const searchSaved = document.getElementById("searchSaved");
    document.getElementById("viewSavedBtn").addEventListener("click", refreshList);
    searchSaved.addEventListener("input", refreshList);

    async function getAllCredentials(){
      let list = [];
      // Cloud
      if (currentUser){
        const qy = query(collection(db,"passwords"), where("uid","==",currentUser.uid));
        const snap = await getDocs(qy);
        snap.forEach(d => list.push({ id:d.id, location:"Account", ...d.data() }));
      }
      // Local
      const locals = JSON.parse(localStorage.getItem("passwords")||"[]").map(p=>({ location:"Local", ...p }));
      return [...list, ...locals];
    }

    function mask(str){ return "•".repeat(Math.min(10, Math.max(4, Math.ceil((str||"").length*0.6)))) }

    async function refreshList(){
      const all = await getAllCredentials();
      const term = (searchSaved.value || "").toLowerCase();
      const filtered = all.filter(x => (x.name||"").toLowerCase().includes(term));

      savedContainer.innerHTML = "";
      noPasswords.style.display = filtered.length ? "none" : "block";

      filtered.forEach(item=>{
        const row = document.createElement("div");
        row.className = "item";

        // left meta
        const left = document.createElement("div");
        left.className="stack";
        const topMeta = document.createElement("div");
        topMeta.className="meta";

        const name = document.createElement("span");
        name.className="name";
        name.textContent = item.name;

        const badge = document.createElement("span");
        badge.className="badge";
        badge.textContent = item.location;

        const strengthSpan = document.createElement("span");
        strengthSpan.className="badge";
        const tmp = document.createElement("div");
        checkStrength(item.password||"", tmp); // reuse checker to set text + class
        strengthSpan.textContent = tmp.textContent;

        topMeta.appendChild(name);
        topMeta.appendChild(badge);
        topMeta.appendChild(strengthSpan);

        const uname = document.createElement("span");
        uname.className="muted";
        uname.textContent = item.username ? `Username: ${item.username}` : "Username: —";

        const pass = document.createElement("div");
        pass.className="secret";
        let visible = false;
        pass.textContent = mask(item.password);

        left.appendChild(topMeta);
        left.appendChild(uname);
        left.appendChild(pass);

        // right buttons
        const btns = document.createElement("div");
        btns.className="btns";

        const eye = document.createElement("button");
        eye.className="chip"; eye.setAttribute("data-tip","Show / Hide");
        eye.innerHTML = "👁";
        eye.onclick = ()=>{ visible = !visible; pass.textContent = visible ? (item.password || "") : mask(item.password); };

        const copy = document.createElement("button");
        copy.className="chip"; copy.setAttribute("data-tip","Copy");
        copy.innerHTML = "📋";
        copy.onclick = ()=> navigator.clipboard.writeText(item.password || "");

        const share = document.createElement("button");
        share.className="chip"; share.setAttribute("data-tip","Share");
        share.innerHTML = "📤";
        share.onclick = async ()=>{
          if (navigator.share){
            try{ await navigator.share({ title: item.name, text: item.password }); } catch(e){}
          } else {
            await navigator.clipboard.writeText(item.password || "");
            // no alert; keep UI quiet
          }
        };

        const edit = document.createElement("button");
        edit.className="chip"; edit.setAttribute("data-tip","Edit name/username");
        edit.innerHTML = "✏️";
        edit.onclick = ()=> startInlineEdit(item, row);

        const del = document.createElement("button");
        del.className="chip"; del.setAttribute("data-tip","Delete");
        del.innerHTML = "🗑";
        del.onclick = async ()=>{
          // keep one confirm only for delete
          if (!confirm("Delete this password?")) return;
          if (item.location==="Account"){
            await deleteDoc(doc(db,"passwords", item.id));
          } else {
            let locals = JSON.parse(localStorage.getItem("passwords")||"[]");
            locals = locals.filter(x => !(x.name===item.name && (x.username||"")===(item.username||"") && x.password===item.password));
            localStorage.setItem("passwords", JSON.stringify(locals));
          }
          await refreshList();
        };

        btns.appendChild(eye);
        btns.appendChild(copy);
        btns.appendChild(share);
        btns.appendChild(edit);
        btns.appendChild(del);

        row.appendChild(left);
        row.appendChild(btns);
        savedContainer.appendChild(row);
      });
    }

    function startInlineEdit(item, row){
      // Build small inline editor UI under the row
      if (row.querySelector(".inline-edit")) return;
      const editWrap = document.createElement("div");
      editWrap.className="inline-edit";
      editWrap.style.gridColumn = "1 / -1";
      editWrap.style.paddingTop = "6px";

      const inner = document.createElement("div");
      inner.className = "row";
      const nameF = document.createElement("input");
      nameF.value = item.name || ""; nameF.placeholder="Name"; nameF.className="field"; nameF.style.flex="1";
      const userF = document.createElement("input");
      userF.value = item.username || ""; userF.placeholder="Username (optional)"; userF.className="field"; userF.style.flex="1";

      const saveB = document.createElement("button");
      saveB.className="pill"; saveB.textContent="Save";
      const cancelB = document.createElement("button");
      cancelB.className="pill ghost"; cancelB.textContent="Cancel";

      inner.appendChild(nameF); inner.appendChild(userF); inner.appendChild(saveB); inner.appendChild(cancelB);
      editWrap.appendChild(inner);
      row.appendChild(editWrap);

      cancelB.onclick = ()=> editWrap.remove();
      saveB.onclick = async ()=>{
        const newName = nameF.value.trim();
        const newUser = userF.value.trim();
        if (!newName){ shake(nameF); return; }

        if (item.location==="Account"){
          await updateDoc(doc(db,"passwords", item.id), { name:newName, username:newUser });
        } else {
          let locals = JSON.parse(localStorage.getItem("passwords")||"[]");
          const idx = locals.findIndex(x => x.name===item.name && (x.username||"")===(item.username||"") && x.password===item.password);
          if (idx>-1){
            locals[idx].name = newName;
            locals[idx].username = newUser;
            localStorage.setItem("passwords", JSON.stringify(locals));
          }
        }
        await refreshList();
      };
    }

    // Initial render
    refreshList();
  </script>
</body>
</html>
