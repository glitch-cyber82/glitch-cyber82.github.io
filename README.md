<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Aurora — Messages &amp; Video</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Sora:wght@500;600;700;800&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    * { box-sizing: border-box; }
    html, body {
      margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden;
      background: #06070C;
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }
    #myChatAppOverlay {
      --bm-bg: #0B0D14;
      --bm-bg-sec: #12141E;
      --bm-bg-elev: #171A26;
      --bm-bg-chat: #0B0D14;
      --bm-text: #F2F3F7;
      --bm-text-muted: #8A8FA3;
      --bm-border: #1F2230;
      --bm-border-input: #262A3B;
      --bm-msg-other: #1B1E2B;
      --bm-msg-other-text: #F2F3F7;
      --bm-accent-a: #7C6CF6;
      --bm-accent-b: #4FD8E8;
      --bm-accent-grad: linear-gradient(135deg, #7C6CF6 0%, #5B8DEF 55%, #4FD8E8 100%);
      --bm-danger: #FF5D6C;
      --bm-success: #3DDC97;
      --bm-radius-lg: 22px;
      --bm-radius-md: 16px;
      --bm-radius-sm: 11px;
      --bm-shadow-soft: 0 8px 30px rgba(0,0,0,0.35);
      width: 100%; height: 100vh; background: var(--bm-bg);
      display: flex; flex-direction: column; color: var(--bm-text); position: relative;
    }
    #myChatAppOverlay.bm-light {
      --bm-bg: #F6F7FB; --bm-bg-sec: #FFFFFF; --bm-bg-elev: #FFFFFF; --bm-bg-chat: #F0F1F7;
      --bm-text: #14151F; --bm-text-muted: #6D7186; --bm-border: #E7E8F1; --bm-border-input: #DEE0EC;
      --bm-msg-other: #EBEDF5; --bm-msg-other-text: #14151F; --bm-shadow-soft: 0 8px 24px rgba(30,32,60,0.08);
    }
    #myChatAppOverlay * { transition: background-color .25s ease, border-color .25s ease, color .25s ease; }
    #myChatAppOverlay ::placeholder { color: var(--bm-text-muted); opacity: 0.8; }
    #myChatAppOverlay input, #myChatAppOverlay button { font-family: 'Inter', sans-serif; }
    #myChatAppOverlay .bm-display { font-family: 'Sora', sans-serif; }
    #myChatAppOverlay .bm-scroll::-webkit-scrollbar { width: 5px; }
    #myChatAppOverlay .bm-scroll::-webkit-scrollbar-thumb { background: var(--bm-border-input); border-radius: 4px; }
    #myChatAppOverlay .bm-btn-grad { background: var(--bm-accent-grad); color: #06070C; border: none; cursor: pointer; font-weight: 700; letter-spacing: -0.01em; }
    #myChatAppOverlay .bm-btn-grad:active { transform: scale(0.97); }
    @keyframes bmPulse { 0%,100% { opacity:1; transform:scale(1);} 50% { opacity:.55; transform:scale(1.12);} }
    @keyframes bmSlideUp { from { opacity:0; transform:translateY(14px);} to { opacity:1; transform:translateY(0);} }
    @keyframes bmSpeakGlow { 0%,100% { box-shadow:0 0 0 2px var(--bm-success), 0 0 18px 4px rgba(61,220,151,0.55);} 50% { box-shadow:0 0 0 2px var(--bm-success), 0 0 28px 8px rgba(61,220,151,0.85);} }
    #myChatAppOverlay .bm-anim-in { animation: bmSlideUp .28s ease both; }
    #myChatAppOverlay .bm-tile.bm-speaking video { animation: bmSpeakGlow 1.1s ease-in-out infinite; }
    #myChatAppOverlay .bm-msg-del {
      opacity:0; transform:scale(0.85); transition:opacity .15s ease, transform .15s ease;
      width:22px;height:22px;border-radius:50%;background:rgba(255,93,108,0.15);color:var(--bm-danger);
      border:1px solid rgba(255,93,108,0.35);display:flex;align-items:center;justify-content:center;
      cursor:pointer;font-size:12px;flex-shrink:0;
    }
    #myChatAppOverlay .bm-msg-row:hover .bm-msg-del,
    #myChatAppOverlay .bm-msg-row.bm-show-del .bm-msg-del { opacity:1; transform:scale(1); }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
</head>
<body>

<div id="myChatAppOverlay">
  <div style="display:flex;flex-direction:column;height:100%;width:100%;">

    <div id="bmHeader" style="padding:16px 18px;background:var(--bm-bg);color:var(--bm-text);border-bottom:1px solid var(--bm-border);display:flex;justify-content:space-between;align-items:center;flex-shrink:0;">
      <div style="display:flex;align-items:center;gap:10px;">
        <div style="width:30px;height:30px;border-radius:9px;background:var(--bm-accent-grad);display:flex;align-items:center;justify-content:center;font-size:15px;box-shadow:0 4px 14px rgba(124,108,246,0.45);">✦</div>
        <span class="bm-display" style="font-weight:700;font-size:17px;letter-spacing:-0.01em;">Aurora</span>
      </div>
      <button id="bmThemeToggleBtn" style="background:var(--bm-bg-sec);border:1px solid var(--bm-border);width:34px;height:34px;border-radius:50%;font-size:15px;cursor:pointer;display:flex;align-items:center;justify-content:center;color:var(--bm-text);">🌙</button>
    </div>

    <!-- AUTH SCREEN -->
    <div id="bmAuthScreen" style="padding:32px 24px;display:flex;flex-direction:column;flex:1;overflow-y:auto;gap:12px;background:var(--bm-bg);justify-content:center;">
      <div class="bm-anim-in" style="max-width:340px;margin:0 auto;width:100%;">
        <div style="text-align:center;margin-bottom:26px;">
          <div style="width:52px;height:52px;border-radius:16px;background:var(--bm-accent-grad);margin:0 auto 14px;box-shadow:0 10px 28px rgba(124,108,246,0.4);"></div>
          <h1 class="bm-display" style="margin:0 0 6px 0;font-size:22px;font-weight:700;letter-spacing:-0.02em;">Welcome back</h1>
          <p style="margin:0;color:var(--bm-text-muted);font-size:13.5px;">Sign in to message and call your people</p>
        </div>
        <div style="display:flex;flex-direction:column;gap:10px;">
          <input type="text" id="bmUsername" placeholder="Username" style="width:100%;padding:14px 16px;border-radius:var(--bm-radius-sm);border:1px solid var(--bm-border-input);background:var(--bm-bg-sec);color:var(--bm-text);font-size:15px;outline:none;">
          <input type="password" id="bmPassword" placeholder="Password" style="width:100%;padding:14px 16px;border-radius:var(--bm-radius-sm);border:1px solid var(--bm-border-input);background:var(--bm-bg-sec);color:var(--bm-text);font-size:15px;outline:none;">
        </div>
        <div id="bmAuthStatus" style="font-size:0.8rem;color:var(--bm-danger);text-align:center;min-height:16px;margin-top:8px;"></div>
        <button id="bmSignInBtn" class="bm-btn-grad" style="width:100%;padding:14px;border-radius:var(--bm-radius-sm);font-size:15px;margin-top:6px;">Sign In</button>
        <button id="bmSignUpBtn" style="width:100%;padding:14px;background:transparent;color:var(--bm-text);border:1px solid var(--bm-border-input);border-radius:var(--bm-radius-sm);cursor:pointer;font-weight:600;font-size:15px;margin-top:8px;">Create Account</button>
      </div>
    </div>

    <!-- SETUP SCREEN -->
    <div id="bmSetupScreen" style="padding:18px;display:none;flex-direction:column;flex:1;overflow-y:auto;background:var(--bm-bg);gap:14px;" class="bm-scroll">
      <button id="bmBackToChatBtn" style="align-self:flex-start;background:none;border:none;color:var(--bm-text-muted);font-size:13.5px;font-weight:600;cursor:pointer;padding:0;">‹ Back to chat</button>
      <div class="bm-anim-in" style="display:flex;align-items:center;justify-content:space-between;background:var(--bm-bg-sec);border:1px solid var(--bm-border);padding:14px;border-radius:var(--bm-radius-lg);box-shadow:var(--bm-shadow-soft);">
        <div style="display:flex;align-items:center;width:100%;">
          <label for="bmHiddenFileInput" style="cursor:pointer;margin-right:14px;flex-shrink:0;position:relative;">
            <div style="padding:2px;border-radius:50%;background:var(--bm-accent-grad);">
              <img id="bmMyAvatar" src="data:image/svg+xml;charset=UTF-8,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100' height='100'%3E%3Crect width='100' height='100' fill='%231B1E2B'/%3E%3Ccircle cx='50' cy='35' r='20' fill='%238A8FA3'/%3E%3Cpath d='M20 100 C20 65 80 65 80 100' fill='%238A8FA3'/%3E%3C/svg%3E" alt="Avatar" style="width:50px;height:50px;border-radius:50%;object-fit:cover;display:block;border:2px solid var(--bm-bg-sec);">
            </div>
          </label>
          <div style="overflow:hidden;width:100%;">
            <div style="font-size:0.68rem;color:var(--bm-success);font-weight:700;margin-bottom:3px;letter-spacing:0.02em;">● Online</div>
            <div id="bmMyUsername" class="bm-display" style="font-weight:700;color:var(--bm-text);font-size:16px;"></div>
            <label for="bmHiddenFileInput" style="color:var(--bm-accent-b);font-size:0.75rem;cursor:pointer;margin-top:2px;display:block;font-weight:600;">Change photo</label>
          </div>
        </div>
      </div>

      <div class="bm-anim-in" style="background:var(--bm-bg-sec);border:1px solid var(--bm-border);border-radius:var(--bm-radius-lg);padding:14px;box-shadow:var(--bm-shadow-soft);">
        <h4 class="bm-display" style="margin:0 0 10px 0;font-size:0.78rem;color:var(--bm-text-muted);font-weight:600;letter-spacing:0.02em;">Friends</h4>
        <div style="display:flex;gap:8px;margin-bottom:10px;">
          <input type="text" id="bmAddFriendInput" placeholder="Add friend by username" style="flex:1;padding:11px 12px;font-size:0.9rem;border:1px solid var(--bm-border-input);border-radius:var(--bm-radius-sm);background:var(--bm-bg);color:var(--bm-text);outline:none;">
          <button id="bmSendReqBtn" class="bm-btn-grad" style="padding:11px 18px;border-radius:var(--bm-radius-sm);font-size:0.88rem;">Add</button>
        </div>
        <div id="bmFriendsList" class="bm-scroll" style="max-height:130px;overflow-y:auto;display:flex;flex-direction:column;gap:5px;"></div>
      </div>

      <button id="bmSignOutBtn" style="margin-top:2px;width:100%;padding:13px;background:transparent;color:var(--bm-danger);border:1px solid rgba(255,93,108,0.4);border-radius:var(--bm-radius-sm);cursor:pointer;font-size:0.88rem;font-weight:600;">Sign Out</button>
    </div>

    <!-- CHAT SCREEN -->
    <div id="bmChatScreen" style="display:none;flex-direction:column;flex:1;background:var(--bm-bg);overflow:hidden;position:relative;">
      <div style="display:flex;justify-content:space-between;align-items:center;padding:12px 16px;background:var(--bm-bg-sec);border-bottom:1px solid var(--bm-border);flex-shrink:0;">
        <button id="bmLeaveRoomBtn" title="Edit profile" style="background:none;color:var(--bm-text-muted);border:none;cursor:pointer;font-size:17px;padding:0;font-weight:600;">⚙</button>
        <div style="display:flex;flex-direction:column;align-items:center;text-align:center;">
          <span id="bmChatRoomTitle" class="bm-display" style="font-weight:700;font-size:15px;color:var(--bm-text);">Public Chat</span>
          <span id="bmTopUserId" style="font-size:0.7rem;color:var(--bm-accent-b);font-weight:600;">User: ...</span>
        </div>
        <div style="width:40px;"></div>
      </div>

      <div style="padding:10px 12px;background:var(--bm-bg-sec);border-bottom:1px solid var(--bm-border);display:flex;gap:7px;align-items:center;flex-shrink:0;flex-wrap:wrap;">
        <button id="bmJoinCallBtn" class="bm-btn-grad" style="padding:9px 16px;border-radius:var(--bm-radius-sm);font-size:0.85rem;display:flex;align-items:center;gap:6px;">📞 Call</button>
        <button id="bmPopoutBtn" title="Open in a new tab — needed for camera/mic when this page is embedded" style="display:none;padding:9px 12px;background:var(--bm-bg);border:1px solid var(--bm-border-input);color:var(--bm-text-muted);border-radius:var(--bm-radius-sm);font-size:0.8rem;font-weight:600;cursor:pointer;">↗ Open for calls</button>
      </div>

      <!-- Incoming call banner -->
      <div id="bmIncomingCallBanner" style="display:none;position:absolute;top:98px;left:12px;right:12px;z-index:30;background:var(--bm-bg-elev);border:1px solid var(--bm-accent-a);border-radius:var(--bm-radius-md);padding:12px 14px;box-shadow:0 12px 30px rgba(0,0,0,0.45);align-items:center;justify-content:space-between;">
        <div style="display:flex;align-items:center;gap:10px;">
          <div style="width:10px;height:10px;border-radius:50%;background:var(--bm-success);animation:bmPulse 1.1s infinite;"></div>
          <div>
            <div style="font-weight:700;font-size:0.88rem;" id="bmIncomingCallText">Incoming call…</div>
            <div style="font-size:0.72rem;color:var(--bm-text-muted);">Video &amp; audio</div>
          </div>
        </div>
        <div style="display:flex;gap:6px;">
          <button id="bmDeclineCallBtn" style="background:var(--bm-danger);color:#fff;border:none;padding:8px 12px;border-radius:10px;font-weight:700;font-size:0.78rem;cursor:pointer;">Decline</button>
          <button id="bmAcceptCallBtn" style="background:var(--bm-success);color:#06140F;border:none;padding:8px 12px;border-radius:10px;font-weight:700;font-size:0.78rem;cursor:pointer;">Accept</button>
        </div>
      </div>

      <!-- VIDEO CALL OVERLAY -->
      <div id="bmVideoContainer" style="display:none;position:absolute;top:0;left:0;width:100%;height:100%;background:radial-gradient(circle at 30% 20%, #14172a 0%, #03040A 65%);z-index:20;flex-direction:column;">
        <div style="display:flex;align-items:center;justify-content:space-between;padding:14px 16px;flex-shrink:0;">
          <div style="display:flex;align-items:center;gap:8px;background:rgba(255,255,255,0.08);backdrop-filter:blur(10px);padding:6px 12px;border-radius:20px;border:1px solid rgba(255,255,255,0.12);">
            <div style="width:7px;height:7px;border-radius:50%;background:var(--bm-danger);animation:bmPulse 1.4s infinite;"></div>
            <span id="bmCallTimer" style="color:#fff;font-size:0.78rem;font-weight:700;font-variant-numeric:tabular-nums;">00:00</span>
          </div>
          <div id="bmParticipantBadge" style="background:rgba(255,255,255,0.08);backdrop-filter:blur(10px);padding:6px 14px;border-radius:20px;border:1px solid rgba(255,255,255,0.12);color:#fff;font-size:0.78rem;font-weight:700;">1/5 in call</div>
        </div>

        <div id="bmVideoGrid" style="flex:1;display:grid;grid-template-columns:repeat(auto-fit, minmax(150px, 1fr));gap:12px;padding:6px 16px 16px;overflow-y:auto;align-items:center;justify-items:center;"></div>

        <div style="padding:22px;display:flex;justify-content:center;gap:16px;background:linear-gradient(to top, rgba(0,0,0,0.9), transparent);flex-shrink:0;">
          <button id="bmToggleMicBtn" style="background:rgba(255,255,255,0.1);backdrop-filter:blur(10px);color:#FFF;border:1px solid rgba(255,255,255,0.18);width:52px;height:52px;border-radius:50%;cursor:pointer;font-size:19px;box-shadow:0 6px 20px rgba(0,0,0,0.4);">🎤</button>
          <button id="bmToggleCamBtn" style="background:rgba(255,255,255,0.1);backdrop-filter:blur(10px);color:#FFF;border:1px solid rgba(255,255,255,0.18);width:52px;height:52px;border-radius:50%;cursor:pointer;font-size:19px;box-shadow:0 6px 20px rgba(0,0,0,0.4);">📷</button>
          <button id="bmEndCallBtn" style="background:linear-gradient(135deg,#FF5D6C,#E8384A);color:#FFF;border:none;padding:0 30px;height:52px;border-radius:26px;font-weight:700;cursor:pointer;box-shadow:0 8px 24px rgba(255,93,108,0.45);font-size:0.92rem;">Leave Call</button>
        </div>
      </div>

      <div id="bmMessagesBox" class="bm-scroll" style="flex:1;background:var(--bm-bg-chat);padding:16px;overflow-y:auto;display:flex;flex-direction:column;gap:10px;"></div>

      <div style="display:flex;gap:8px;align-items:center;padding:12px;background:var(--bm-bg-sec);border-top:1px solid var(--bm-border);flex-shrink:0;">
        <label for="bmMediaInput" title="Send photo or video" style="width:38px;height:38px;border-radius:50%;background:var(--bm-bg);border:1px solid var(--bm-border-input);display:flex;align-items:center;justify-content:center;cursor:pointer;flex-shrink:0;font-size:16px;color:var(--bm-text-muted);">📎</label>
        <input type="text" id="bmMsgInput" placeholder="Message…" style="flex:1;padding:11px 16px;border:1px solid var(--bm-border-input);border-radius:22px;background:var(--bm-bg);color:var(--bm-text);font-size:15px;outline:none;">
        <button id="bmSendBtn" class="bm-btn-grad" style="border-radius:50%;width:38px;height:38px;display:flex;align-items:center;justify-content:center;font-weight:bold;font-size:15px;">↑</button>
      </div>
      <input type="file" id="bmMediaInput" accept="image/*,video/*" style="display:none;">
    </div>
  </div>
  <input type="file" id="bmHiddenFileInput" accept="image/*" style="display:none;">
</div>

<script>
(function initApp() {
  function safeStorageGet(key) { try { return localStorage.getItem(key); } catch(e) { return null; } }
  function safeStorageSet(key, val) { try { localStorage.setItem(key, val); } catch(e) {} }
  function safeStorageRemove(key) { try { localStorage.removeItem(key); } catch(e) {} }

  var overlay = document.getElementById('myChatAppOverlay');
  var savedTheme = safeStorageGet('bm_theme');
  if (savedTheme === 'light') overlay.classList.add('bm-light');
  var themeBtn = document.getElementById('bmThemeToggleBtn');
  themeBtn.textContent = savedTheme === 'light' ? '☀️' : '🌙';
  themeBtn.onclick = function() {
    overlay.classList.toggle('bm-light');
    var isLight = overlay.classList.contains('bm-light');
    safeStorageSet('bm_theme', isLight ? 'light' : 'dark');
    this.textContent = isLight ? '☀️' : '🌙';
  };

  var SUPABASE_URL = "https://fkrinuqasoznkastjtag.supabase.co";
  var SUPABASE_KEY = "sb_publishable_cOpl_9b0Cxa5Te5Vc9DDng_EJIZp01Q";
  var client = null;
  var currentUsername = safeStorageGet("bm_username") || "";
  var currentRoom = "public";
  var myUniqueId = safeStorageGet("bm_peer_id");
  var myAvatarUrl = "";
  var defaultAvatar = "data:image/svg+xml;charset=UTF-8,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100' height='100'%3E%3Crect width='100' height='100' fill='%231B1E2B'/%3E%3Ccircle cx='50' cy='35' r='20' fill='%238A8FA3'/%3E%3Cpath d='M20 100 C20 65 80 65 80 100' fill='%238A8FA3'/%3E%3C/svg%3E";

  if(!myUniqueId) {
    myUniqueId = Math.random().toString(36).substring(2, 7).toUpperCase();
    safeStorageSet("bm_peer_id", myUniqueId);
  }

  function showScreen(name) {
    document.getElementById('bmAuthScreen').style.display = name === 'auth' ? 'flex' : 'none';
    document.getElementById('bmSetupScreen').style.display = name === 'setup' ? 'flex' : 'none';
    document.getElementById('bmChatScreen').style.display = name === 'chat' ? 'flex' : 'none';
  }
  showScreen(currentUsername ? "chat" : "auth");
  if (currentUsername) {
    document.getElementById('bmChatRoomTitle').textContent = "Public Chat";
    document.getElementById('bmTopUserId').textContent = currentUsername + " (ID: " + myUniqueId + ")";
  }

  function enterChat(){
    showScreen("chat");
    document.getElementById('bmChatRoomTitle').textContent = "Public Chat";
    document.getElementById('bmTopUserId').textContent = currentUsername + " (ID: " + myUniqueId + ")";
    loadMessages();
    if(window.bmMessageTimer) clearInterval(window.bmMessageTimer);
    window.bmMessageTimer = setInterval(loadMessages, 2000);
    startSignalingPoll();
    requestNotifyPermission();
  }

  function startSupabase() {
    if(window.supabase) {
      client = window.supabase.createClient(SUPABASE_URL, SUPABASE_KEY);
      if(currentUsername) { loadProfileData(); enterChat(); }
    }
  }
  if (!window.supabase) {
    var scriptEl = document.createElement('script');
    scriptEl.src = 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2';
    scriptEl.onload = startSupabase;
    document.head.appendChild(scriptEl);
  } else { startSupabase(); }

  document.getElementById('bmSignInBtn').onclick = async function(){
    if(!client) return;
    var u = document.getElementById('bmUsername').value.trim();
    var p = document.getElementById('bmPassword').value.trim();
    if(!u || !p) return;
    var res = await client.from("profiles").select("*").eq("username", u).eq("password", p).maybeSingle();
    if(res.data){
      currentUsername = u; safeStorageSet("bm_username", u);
      loadProfileData(); enterChat();
    } else { document.getElementById('bmAuthStatus').textContent = "Invalid login."; }
  };

  document.getElementById('bmSignUpBtn').onclick = async function(){
    if(!client) return;
    var u = document.getElementById('bmUsername').value.trim();
    var p = document.getElementById('bmPassword').value.trim();
    if(!u || !p) return;
    var res = await client.from("profiles").insert({username: u, password: p}).select().single();
    if(!res.error){
      currentUsername = u; safeStorageSet("bm_username", u);
      loadProfileData(); enterChat();
    } else { document.getElementById('bmAuthStatus').textContent = "Username taken or error."; }
  };

  document.getElementById('bmSignOutBtn').onclick = function(){
    safeStorageRemove("bm_username");
    currentUsername = "";
    if(window.bmMessageTimer) clearInterval(window.bmMessageTimer);
    stopSignalingPoll();
    showScreen("auth");
  };

  async function loadProfileData(){
    if(!client) return;
    document.getElementById('bmMyUsername').textContent = currentUsername;
    var res = await client.from("profiles").select("avatar_url").eq("username", currentUsername).maybeSingle();
    if(res.data && res.data.avatar_url){
      myAvatarUrl = res.data.avatar_url;
      document.getElementById('bmMyAvatar').src = res.data.avatar_url;
    }
  }

  document.getElementById('bmLeaveRoomBtn').onclick = function(){ showScreen("setup"); };
  document.getElementById('bmBackToChatBtn').onclick = function(){ showScreen("chat"); };

  /* MESSAGES ENGINE */
  var avatarCache = {};
  var knownPeersInRoom = {};

  async function loadMessages(){
    if(!currentRoom || !client) return;
    var res = await client.from("messages").select("*").eq("room", currentRoom).order("created_at", {ascending: true});
    var box = document.getElementById('bmMessagesBox');
    if(res.error){ console.error("loadMessages error:", res.error); return; }
    if(res.data){
      box.innerHTML = "";
      for (let m of res.data) {
        var isMe = m.username === currentUsername;
        var avatar = m.avatar_url || avatarCache[m.username] || defaultAvatar;
        if(m.avatar_url) avatarCache[m.username] = m.avatar_url;
        if(m.peer_id && !isMe) knownPeersInRoom[m.peer_id] = m.username;

        var row = document.createElement('div');
        row.className = 'bm-msg-row';
        row.style.cssText = 'display:flex; width:100%; gap:8px; align-items:flex-end; flex-direction:'+(isMe?'row-reverse':'row')+';';

        var avatarImg = document.createElement('img');
        avatarImg.src = avatar;
        avatarImg.style.cssText = 'width:28px;height:28px;border-radius:50%;object-fit:cover;flex-shrink:0;border:1.5px solid var(--bm-border-input);';

        var col = document.createElement('div');
        col.style.cssText = 'display:flex;flex-direction:column;max-width:72%;align-items:'+(isMe?'flex-end':'flex-start')+';';

        var meta = document.createElement('div');
        meta.style.cssText = 'font-size:0.68rem;color:var(--bm-text-muted);font-weight:600;margin:0 4px 3px;';
        meta.textContent = isMe ? 'You' : ((m.username || 'Unknown') + '  •  ID ' + (m.peer_id || '—'));
        col.appendChild(meta);

        var bubbleWrap = document.createElement('div');
        bubbleWrap.style.cssText = 'display:flex;align-items:center;gap:6px;flex-direction:'+(isMe?'row-reverse':'row')+';';

        if(m.media_url){
          var mediaEl;
          if(m.media_type === 'video'){
            mediaEl = document.createElement('video');
            mediaEl.controls = true;
            mediaEl.playsInline = true;
          } else {
            mediaEl = document.createElement('img');
          }
          mediaEl.src = m.media_url;
          mediaEl.style.cssText = 'max-width:220px;max-height:260px;border-radius:14px;display:block;box-shadow:0 2px 8px rgba(0,0,0,0.2);';
          bubbleWrap.appendChild(mediaEl);
        }

        if(m.message){
          var bubbleBg = isMe ? 'var(--bm-accent-grad)' : 'var(--bm-msg-other)';
          var bubbleColor = isMe ? '#06070C' : 'var(--bm-msg-other-text)';
          var bubble = document.createElement('div');
          bubble.style.cssText = 'background:'+bubbleBg+'; color:'+bubbleColor+'; padding:9px 14px; border-radius:18px; font-size:14.5px; line-height:1.35; font-weight:'+(isMe?'600':'400')+'; box-shadow:0 2px 8px rgba(0,0,0,0.12); word-break:break-word;';
          bubble.textContent = m.message;
          bubbleWrap.appendChild(bubble);
        }

        if(isMe){
          var delBtn = document.createElement('div');
          delBtn.className = 'bm-msg-del';
          delBtn.textContent = '✕';
          delBtn.title = 'Delete message';
          delBtn.onclick = (function(msgId){
            return function(){ deleteMessage(msgId); };
          })(m.id);
          bubbleWrap.appendChild(delBtn);
          row.onclick = function(){ row.classList.toggle('bm-show-del'); };
        }

        col.appendChild(bubbleWrap);
        row.appendChild(avatarImg);
        row.appendChild(col);
        box.appendChild(row);
      }
      box.scrollTop = box.scrollHeight;
    }
  }

  async function deleteMessage(id){
    if(!client || id === undefined) return;
    if(!confirm('Delete this message for everyone?')) return;
    var res = await client.from("messages").delete().eq("id", id);
    if(res.error){ alert('Could not delete: ' + res.error.message); }
    loadMessages();
  }

  document.getElementById('bmSendBtn').onclick = async function(){
    if(!client) return;
    var i = document.getElementById('bmMsgInput');
    var t = i.value.trim();
    if(t && currentRoom){
      i.value = "";
      var res = await client.from("messages").insert({
        username: currentUsername,
        room: currentRoom,
        message: t,
        avatar_url: myAvatarUrl || null,
        peer_id: myUniqueId
      });
      if(res.error){ console.error("send error:", res.error); alert('Could not send: ' + res.error.message); }
      document.getElementById('bmTopUserId').textContent = currentUsername + " (ID: " + myUniqueId + ")";
      loadMessages();
    }
  };

  document.getElementById('bmMsgInput').addEventListener('keydown', function(e){
    if(e.key === 'Enter') document.getElementById('bmSendBtn').click();
  });

  document.getElementById('bmMediaInput').addEventListener('change', async function(e){
    var file = e.target.files[0];
    if(!file || !client || !currentRoom) return;
    var msgInput = document.getElementById('bmMsgInput');
    var caption = msgInput.value.trim();
    msgInput.value = "";

    var isVideo = file.type.indexOf('video') === 0;
    var ext = (file.name.split('.').pop() || (isVideo ? 'mp4' : 'jpg')).toLowerCase();
    var path = currentUsername + '/' + Date.now() + '_' + Math.random().toString(36).slice(2,7) + '.' + ext;

    var up = await client.storage.from('chat-media').upload(path, file, { contentType: file.type || undefined });
    e.target.value = '';
    if(up.error){ alert('Upload failed: ' + up.error.message); return; }
    var pub = client.storage.from('chat-media').getPublicUrl(path);

    var res = await client.from("messages").insert({
      username: currentUsername,
      room: currentRoom,
      message: caption,
      media_url: pub.data.publicUrl,
      media_type: isVideo ? 'video' : 'image',
      avatar_url: myAvatarUrl || null,
      peer_id: myUniqueId
    });
    if(res.error){ alert('Could not send media: ' + res.error.message); }
    document.getElementById('bmTopUserId').textContent = currentUsername + " (ID: " + myUniqueId + ")";
    loadMessages();
  });

  /* WEBRTC CALLING ENGINE */
  var RTC_CONFIG = { iceServers: [{ urls: ['stun:stun.l.google.com:19302', 'stun:stun1.l.google.com:19302'] }] };
  var MAX_PARTICIPANTS = 5;

  var pcMap = {};
  var queuedIce = {};
  var participants = {};
  var localStream = null;
  var inCall = false;
  var pendingInvite = null;
  var lastSignalId = 0;
  var micOn = true, camOn = true;
  var callTimerInterval = null, callStartedAt = null;
  var heartbeatInterval = null, presenceCheckInterval = null;
  var audioCtx = null;
  var ringInterval = null, ringAudioCtx = null;

  function requestNotifyPermission(){
    try {
      if('Notification' in window && Notification.permission === 'default'){
        Notification.requestPermission();
      }
    } catch(e){}
  }

  function playRingBeep(){
    try {
      if(!ringAudioCtx) ringAudioCtx = new (window.AudioContext || window.webkitAudioContext)();
      var o = ringAudioCtx.createOscillator();
      var g = ringAudioCtx.createGain();
      o.type = 'sine';
      o.frequency.setValueAtTime(880, ringAudioCtx.currentTime);
      g.gain.setValueAtTime(0.001, ringAudioCtx.currentTime);
      g.gain.exponentialRampToValueAtTime(0.15, ringAudioCtx.currentTime + 0.02);
      g.gain.exponentialRampToValueAtTime(0.001, ringAudioCtx.currentTime + 0.35);
      o.connect(g); g.connect(ringAudioCtx.destination);
      o.start(); o.stop(ringAudioCtx.currentTime + 0.4);
    } catch(e){}
  }

  function startRinging(title, body){
    stopRinging();
    playRingBeep();
    ringInterval = setInterval(playRingBeep, 1600);
    try { if(navigator.vibrate) navigator.vibrate([300,150,300,150,300]); } catch(e){}
    try {
      if('Notification' in window && Notification.permission === 'granted' && document.hidden){
        new Notification(title, { body: body });
      }
    } catch(e){}
    window.bmRingTimeout = setTimeout(function(){
      stopRinging();
      document.getElementById('bmIncomingCallBanner').style.display = 'none';
      pendingInvite = null;
    }, 30000);
  }

  function stopRinging(){
    if(ringInterval){ clearInterval(ringInterval); ringInterval = null; }
    if(window.bmRingTimeout){ clearTimeout(window.bmRingTimeout); window.bmRingTimeout = null; }
    try { if(navigator.vibrate) navigator.vibrate(0); } catch(e){}
  }

  function showIncomingBanner(row, label){
    pendingInvite = row;
    document.getElementById('bmIncomingCallText').textContent = label;
    document.getElementById('bmIncomingCallBanner').style.display = 'flex';
    startRinging('Incoming call', label);
  }

  function startSignalingPoll(){
    stopSignalingPoll();
    lastSignalId = 0;
    window.bmSignalTimer = setInterval(pollSignaling, 1500);
    pollSignaling();
  }
  function stopSignalingPoll(){ if(window.bmSignalTimer) clearInterval(window.bmSignalTimer); }

  async function pollSignaling(){
    if(!client || !myUniqueId) return;
    var res = await client.from("facetime").select("*").eq("to_peer_id", myUniqueId).gt("id", lastSignalId).order("id", {ascending: true});
    if(!res.data || !res.data.length) return;
    for(let row of res.data){
      lastSignalId = Math.max(lastSignalId, row.id);
      await handleSignal(row);
      client.from("facetime").delete().eq("id", row.id).then(function(){});
    }
  }

  async function handleSignal(row){
    if(row.type === 'call-invite'){
      showIncomingBanner(row, (row.from_username || 'Someone') + ' invited you to the call');
    } else if(row.type === 'offer'){
      if(inCall){
        await acceptMeshOffer(row);
      } else {
        showIncomingBanner(row, (row.from_username || 'Someone') + ' is calling…');
      }
    } else if(row.type === 'answer'){
      var pc = pcMap[row.from_peer_id];
      if(pc){ await pc.setRemoteDescription(new RTCSessionDescription(row.payload)); flushQueuedIce(row.from_peer_id); }
    } else if(row.type === 'ice-candidate'){
      var pc2 = pcMap[row.from_peer_id];
      if(pc2 && pc2.remoteDescription){
        try { await pc2.addIceCandidate(new RTCIceCandidate(row.payload)); } catch(e){}
      } else {
        (queuedIce[row.from_peer_id] = queuedIce[row.from_peer_id] || []).push(row.payload);
      }
    } else if(row.type === 'call-end' || row.type === 'call-reject'){
      removeParticipant(row.from_peer_id);
      if(pendingInvite && pendingInvite.from_peer_id === row.from_peer_id){
        stopRinging();
        pendingInvite = null;
        document.getElementById('bmIncomingCallBanner').style.display = 'none';
      }
    }
  }

  function flushQueuedIce(peerId){
    var pc = pcMap[peerId];
    var q = queuedIce[peerId] || [];
    while(q.length && pc){
      var c = q.shift();
      pc.addIceCandidate(new RTCIceCandidate(c)).catch(function(){});
    }
  }

  function makePeerConnection(peerId){
    var conn = new RTCPeerConnection(RTC_CONFIG);
    conn.onicecandidate = function(e){
      if(e.candidate){
        client.from("facetime").insert({
          room: currentRoom, from_username: currentUsername, from_peer_id: myUniqueId,
          to_peer_id: peerId, type: 'ice-candidate', payload: e.candidate.toJSON()
        });
      }
    };
    conn.ontrack = function(e){ attachRemoteTile(peerId, e.streams[0]); };
    pcMap[peerId] = conn;
    return conn;
  }

  async function startLocalMedia(){
    localStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
    document.getElementById('bmVideoContainer').style.display = 'flex';
    renderLocalTile();
    watchSpeaking(myUniqueId, localStream);
  }

  function tileTemplate(peerId, label, isLocal){
    var wrap = document.createElement('div');
    wrap.className = 'bm-tile';
    wrap.id = 'bmTile_' + peerId;
    wrap.style.cssText = 'position:relative;width:100%;height:100%;max-height:280px;border-radius:16px;overflow:hidden;background:#111;';
    var video = document.createElement('video');
    video.autoplay = true; video.playsInline = true;
    if(isLocal) video.muted = true;
    video.style.cssText = 'width:100%;height:100%;object-fit:cover;border-radius:16px;border:2px solid '+(isLocal ? 'var(--bm-accent-a)' : 'var(--bm-border-input)')+';background:#000;transition:box-shadow .2s;';
    var tag = document.createElement('span');
    tag.style.cssText = 'position:absolute;bottom:8px;left:10px;font-size:0.7rem;background:rgba(0,0,0,0.6);padding:4px 10px;border-radius:8px;color:#fff;font-weight:600;backdrop-filter:blur(4px);';
    tag.textContent = label;
    wrap.appendChild(video); wrap.appendChild(tag);
    return { wrap: wrap, video: video };
  }

  function renderLocalTile(){
    var grid = document.getElementById('bmVideoGrid');
    var existing = document.getElementById('bmTile_' + myUniqueId);
    if(existing) existing.remove();
    var t = tileTemplate(myUniqueId, 'You', true);
    t.video.srcObject = localStream;
    grid.prepend(t.wrap);
    updateParticipantBadge();
  }

  function attachRemoteTile(peerId, stream){
    var grid = document.getElementById('bmVideoGrid');
    var existing = document.getElementById('bmTile_' + peerId);
    if(existing) existing.remove();
    var name = (participants[peerId] && participants[peerId].username) || peerId;
    var t = tileTemplate(peerId, name, false);
    t.video.srcObject = stream;
    grid.appendChild(t.wrap);
    watchSpeaking(peerId, stream);
    updateParticipantBadge();
  }

  function removeParticipant(peerId){
    if(pcMap[peerId]){ try { pcMap[peerId].close(); } catch(e){} delete pcMap[peerId]; }
    delete queuedIce[peerId];
    delete participants[peerId];
    var tile = document.getElementById('bmTile_' + peerId);
    if(tile) tile.remove();
    updateParticipantBadge();
  }

  function updateParticipantBadge(){
    var count = Object.keys(pcMap).length + 1;
    document.getElementById('bmParticipantBadge').textContent = count + '/' + MAX_PARTICIPANTS + ' in call';
  }

  function watchSpeaking(peerId, stream){
    try {
      if(!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      var source = audioCtx.createMediaStreamSource(stream);
      var analyser = audioCtx.createAnalyser();
      analyser.fftSize = 512;
      source.connect(analyser);
      var data = new Uint8Array(analyser.frequencyBinCount);
      (function loop(){
        var tile = document.getElementById('bmTile_' + peerId);
        if(!tile || !inCall){ return; }
        analyser.getByteFrequencyData(data);
        var avg = data.reduce(function(a,b){return a+b;}, 0) / data.length;
        tile.classList.toggle('bm-speaking', avg > 18);
        requestAnimationFrame(loop);
      })();
    } catch(e){}
  }

  function startCallTimer(){
    callStartedAt = Date.now();
    callTimerInterval = setInterval(function(){
      var secs = Math.floor((Date.now() - callStartedAt) / 1000);
      var m = String(Math.floor(secs / 60)).padStart(2,'0');
      var s = String(secs % 60).padStart(2,'0');
      document.getElementById('bmCallTimer').textContent = m + ':' + s;
    }, 1000);
  }

  async function getActiveParticipants(){
    var cutoff = new Date(Date.now() - 15000).toISOString();
    var res = await client.from("facetime").select("*").eq("room", currentRoom).eq("type", "presence").gt("created_at", cutoff).order("id", {ascending:false});
    var seen = {}, list = [];
    if(res.data){
      for(let r of res.data){
        if(seen[r.from_peer_id]) continue;
        seen[r.from_peer_id] = true;
        if(r.from_peer_id !== myUniqueId) list.push({ peer_id: r.from_peer_id, username: r.from_username });
      }
    }
    return list;
  }

  async function sendPresenceHeartbeat(){
    if(!client || !currentRoom) return;
    await client.from("facetime").insert({
      room: currentRoom, from_username: currentUsername, from_peer_id: myUniqueId,
      to_peer_id: myUniqueId, type: 'presence', payload: {}
    });
    client.from("facetime").delete().eq("room", currentRoom).eq("from_peer_id", myUniqueId).eq("type","presence").lt("created_at", new Date(Date.now()-20000).toISOString()).then(function(){});
  }

  async function joinCall(){
    if(inCall || !client || !currentRoom) return;
    try { await startLocalMedia(); } catch(err){ alert('Could not access camera/mic: ' + err.message); return; }

    var others = await getActiveParticipants();
    if(others.length >= MAX_PARTICIPANTS){
      alert('This call is full (' + MAX_PARTICIPANTS + '/' + MAX_PARTICIPANTS + ').');
      leaveCall(false);
      return;
    }

    inCall = true;
    startCallTimer();
    await sendPresenceHeartbeat();
    heartbeatInterval = setInterval(sendPresenceHeartbeat, 6000);
    presenceCheckInterval = setInterval(async function(){
      var active = await getActiveParticipants();
      var activeIds = active.map(function(p){ return p.peer_id; });
      Object.keys(pcMap).forEach(function(pid){
        if(activeIds.indexOf(pid) === -1) removeParticipant(pid);
      });
    }, 6000);

    for(let p of others){
      participants[p.peer_id] = { username: p.username };
      var pc = makePeerConnection(p.peer_id);
      localStream.getTracks().forEach(function(t){ pc.addTrack(t, localStream); });
      var offer = await pc.createOffer();
      await pc.setLocalDescription(offer);
      await client.from("facetime").insert({
        room: currentRoom, from_username: currentUsername, from_peer_id: myUniqueId,
        to_peer_id: p.peer_id, type: 'offer', payload: offer
      });
    }

    var alreadyHandled = others.map(function(p){ return p.peer_id; });
    Object.keys(knownPeersInRoom).forEach(function(peerId){
      if(peerId === myUniqueId || alreadyHandled.indexOf(peerId) !== -1) return;
      client.from("facetime").insert({
        room: currentRoom, from_username: currentUsername, from_peer_id: myUniqueId,
        to_peer_id: peerId, type: 'call-invite', payload: {}
      });
    });

    updateParticipantBadge();
  }

  async function acceptMeshOffer(row){
    if(!inCall){
      try { await startLocalMedia(); } catch(err){ alert('Could not access camera/mic: ' + err.message); return; }
      inCall = true; startCallTimer();
      await sendPresenceHeartbeat();
      heartbeatInterval = setInterval(sendPresenceHeartbeat, 6000);
      presenceCheckInterval = setInterval(async function(){
        var active = await getActiveParticipants();
        var activeIds = active.map(function(p){ return p.peer_id; });
        Object.keys(pcMap).forEach(function(pid){
          if(activeIds.indexOf(pid) === -1) removeParticipant(pid);
        });
      }, 6000);
    }
    participants[row.from_peer_id] = { username: row.from_username };
    var pc = makePeerConnection(row.from_peer_id);
    localStream.getTracks().forEach(function(t){ pc.addTrack(t, localStream); });
    await pc.setRemoteDescription(new RTCSessionDescription(row.payload));
    flushQueuedIce(row.from_peer_id);
    var answer = await pc.createAnswer();
    await pc.setLocalDescription(answer);
    await client.from("facetime").insert({
      room: currentRoom, from_username: currentUsername, from_peer_id: myUniqueId,
      to_peer_id: row.from_peer_id, type: 'answer', payload: answer
    });
  }

  document.getElementById('bmJoinCallBtn').onclick = joinCall;

  document.getElementById('bmPopoutBtn').onclick = function(){
    window.open(window.location.href, '_blank');
  };
  try {
    if (window.self !== window.top) {
      document.getElementById('bmPopoutBtn').style.display = 'inline-flex';
    }
  } catch(e) {
    document.getElementById('bmPopoutBtn').style.display = 'inline-flex';
  }

  document.getElementById('bmAcceptCallBtn').onclick = async function(){
    if(!pendingInvite) return;
    var invite = pendingInvite;
    stopRinging();
    document.getElementById('bmIncomingCallBanner').style.display = 'none';
    pendingInvite = null;
    if(invite.type === 'offer'){
      await acceptMeshOffer(invite);
    } else {
      await joinCall();
    }
  };

  document.getElementById('bmDeclineCallBtn').onclick = function(){
    stopRinging();
    if(pendingInvite && client){
      client.from("facetime").insert({
        room: currentRoom, from_username: currentUsername, from_peer_id: myUniqueId,
        to_peer_id: pendingInvite.from_peer_id, type: 'call-reject', payload: {}
      });
    }
    pendingInvite = null;
    document.getElementById('bmIncomingCallBanner').style.display = 'none';
  };

  document.getElementById('bmEndCallBtn').onclick = function(){ leaveCall(true); };

  function leaveCall(notifyPeers){
    if(notifyPeers){
      Object.keys(pcMap).forEach(function(peerId){
        client.from("facetime").insert({
          room: currentRoom, from_username: currentUsername, from_peer_id: myUniqueId,
          to_peer_id: peerId, type: 'call-end', payload: {}
        });
      });
      if(client && currentRoom){
        client.from("facetime").delete().eq("room", currentRoom).eq("from_peer_id", myUniqueId).eq("type","presence").then(function(){});
      }
    }
    Object.keys(pcMap).forEach(function(peerId){ try { pcMap[peerId].close(); } catch(e){} });
    pcMap = {}; queuedIce = {}; participants = {};
    if(localStream){ localStream.getTracks().forEach(function(t){ t.stop(); }); localStream = null; }
    document.getElementById('bmVideoGrid').innerHTML = '';
    document.getElementById('bmVideoContainer').style.display = 'none';
    if(callTimerInterval) clearInterval(callTimerInterval);
    if(heartbeatInterval) clearInterval(heartbeatInterval);
    if(presenceCheckInterval) clearInterval(presenceCheckInterval);
    document.getElementById('bmCallTimer').textContent = '00:00';
    inCall = false;
    micOn = true; camOn = true;
    document.getElementById('bmToggleMicBtn').style.opacity = 1;
    document.getElementById('bmToggleCamBtn').style.opacity = 1;
  }

  document.getElementById('bmToggleMicBtn').onclick = function(){
    if(!localStream) return;
    micOn = !micOn;
    localStream.getAudioTracks().forEach(function(t){ t.enabled = micOn; });
    this.style.opacity = micOn ? 1 : 0.45;
    this.textContent = micOn ? '🎤' : '🔇';
  };

  document.getElementById('bmToggleCamBtn').onclick = function(){
    if(!localStream) return;
    camOn = !camOn;
    localStream.getVideoTracks().forEach(function(t){ t.enabled = camOn; });
    this.style.opacity = camOn ? 1 : 0.45;
  };

})();
</script>

</body>
</html>
