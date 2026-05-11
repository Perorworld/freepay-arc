<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>FreePay | Africa Freelancer Escrow</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.umd.min.js"></script>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: #0a0e1a; color: #f9fafb; font-family: 'Segoe UI', sans-serif; min-height: 100vh; transition: background 0.3s, color 0.3s; }
  body.light { background: #f3f4f6; color: #111827; }
  body.light header { border-color: #d1d5db; }
  body.light .card { background: #fff; border-color: #e5e7eb; }
  body.light .job-card { background: #f9fafb; border-color: #e5e7eb; }
  body.light input { background: #f9fafb; border-color: #d1d5db; color: #111827; }
  body.light .tab { border-color: #d1d5db; color: #6b7280; }
  body.light .stat { background: #fff; border-color: #e5e7eb; }
  body.light .feed { background: #fff; border-color: #e5e7eb; }
  body.light .feed-line { border-color: #e5e7eb; color: #374151; }
  body.light #toast { background: #1f2937; }
  body.light .logo { color: #1d4ed8; }
  body.light .arc-bg { opacity: 0.06; }
  body.light .arc-bg path:last-child { fill: #f3f4f6; }

  /* Arc logo background watermark */
  .arc-bg {
    position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%);
    width: 700px; height: 700px; opacity: 0.07; pointer-events: none; z-index: 0;
  }
  body > *:not(.arc-bg) { position: relative; z-index: 1; }

  header { display: flex; align-items: center; justify-content: space-between; padding: 18px 28px; border-bottom: 1px solid #1f2937; }
  .logo { font-size: 1.3rem; font-weight: 800; color: #2563eb; }
  .logo span { color: #10b981; }
  .meta { font-size: 0.75rem; color: #6b7280; text-align: right; }
  .meta strong { color: #f9fafb; display: block; }

  .hero { text-align: center; padding: 48px 20px 28px; }
  .hero h1 { font-size: 1.9rem; font-weight: 800; max-width: 520px; margin: 0 auto 12px; line-height: 1.25; }
  .hero h1 em { color: #10b981; font-style: normal; }
  .hero p { color: #6b7280; font-size: 0.88rem; max-width: 440px; margin: 0 auto 20px; line-height: 1.6; }
  .badge { display: inline-block; background: #1a2744; color: #60a5fa; border: 1px solid #1e3a6e; border-radius: 99px; padding: 5px 14px; font-size: 0.75rem; font-weight: 600; margin-bottom: 18px; }

  .stats { display: flex; gap: 14px; justify-content: center; flex-wrap: wrap; margin-bottom: 8px; }
  .stat { background: #111827; border: 1px solid #1f2937; border-radius: 10px; padding: 10px 18px; text-align: center; font-size: 0.78rem; color: #6b7280; }
  .stat strong { color: #f9fafb; display: block; font-size: 1rem; }

  .tabs { display: flex; gap: 8px; justify-content: center; padding: 24px 20px 20px; }
  .tab { padding: 9px 20px; border-radius: 8px; border: 1px solid #1f2937; background: transparent; color: #6b7280; cursor: pointer; font-size: 0.85rem; }
  .tab.active { background: #2563eb; color: #fff; border-color: #2563eb; }

  main { max-width: 680px; margin: 0 auto; padding: 0 20px 60px; }
  .panel { display: none; }
  .panel.active { display: block; }

  .card { background: #111827; border: 1px solid #1f2937; border-radius: 12px; padding: 24px; margin-bottom: 18px; }
  .card h2 { font-size: 0.95rem; font-weight: 700; margin-bottom: 18px; color: #e5e7eb; }

  label { display: block; font-size: 0.78rem; color: #6b7280; margin-bottom: 5px; }
  input { width: 100%; background: #0a0e1a; border: 1px solid #1f2937; color: #f9fafb; border-radius: 8px; padding: 11px 13px; font-size: 0.88rem; margin-bottom: 14px; outline: none; }
  input:focus { border-color: #2563eb; }

  .btn { width: 100%; padding: 13px; border-radius: 9px; border: none; font-weight: 700; font-size: 0.9rem; cursor: pointer; }
  .btn-primary { background: #2563eb; color: #fff; }
  .btn-success { background: #10b981; color: #fff; }
  .btn-danger { background: #ef4444; color: #fff; }
  .btn-warn { background: #f59e0b; color: #000; }
  .btn:disabled { opacity: 0.4; cursor: not-allowed; }

  .job-card { background: #0a0e1a; border: 1px solid #1f2937; border-radius: 10px; padding: 16px; margin-bottom: 12px; }
  .job-card h3 { font-size: 0.9rem; font-weight: 700; margin-bottom: 8px; }
  .chip { display: inline-block; font-size: 0.72rem; padding: 3px 10px; border-radius: 99px; font-weight: 600; margin-bottom: 10px; }
  .chip-funded { background: #1a2744; color: #60a5fa; }
  .chip-completed { background: #052e16; color: #4ade80; }
  .chip-disputed { background: #451a03; color: #fb923c; }
  .chip-cancelled { background: #1f1f1f; color: #6b7280; }
  .amount { font-size: 1rem; font-weight: 800; color: #10b981; }
  .ngn { font-size: 0.75rem; color: #6b7280; margin-bottom: 10px; }
  .addr { font-family: monospace; font-size: 0.72rem; color: #6b7280; word-break: break-all; margin-bottom: 12px; }
  .job-actions { display: flex; gap: 8px; flex-wrap: wrap; }
  .job-actions .btn { width: auto; padding: 7px 14px; font-size: 0.78rem; }

  .feed { background: #111827; border: 1px solid #1f2937; border-radius: 12px; padding: 20px; font-size: 0.8rem; color: #6b7280; min-height: 60px; }
  .feed-line { padding: 6px 0; border-bottom: 1px solid #1f2937; color: #9ca3af; }
  .feed-line:last-child { border-bottom: none; }

  #toast { position: fixed; bottom: 24px; right: 20px; background: #1f2937; color: #f9fafb; padding: 12px 18px; border-radius: 10px; font-size: 0.82rem; max-width: 300px; border-left: 4px solid #10b981; display: none; z-index: 999; }
  #connectBtn { background: #2563eb; color: #fff; border: none; padding: 9px 18px; border-radius: 8px; cursor: pointer; font-weight: 600; font-size: 0.82rem; }
  #darkToggle { background: transparent; border: 1px solid #1f2937; color: #9ca3af; padding: 7px 12px; border-radius: 8px; cursor: pointer; font-size: 0.82rem; transition: all 0.2s; }
  #darkToggle:hover { border-color: #4b5563; color: #f9fafb; }
  .empty { text-align: center; color: #6b7280; padding: 32px 0; font-size: 0.85rem; }
  #ngnPreview { font-size: 0.78rem; color: #6b7280; margin-top: -10px; margin-bottom: 14px; }
</style>
</head>
<body>

<!-- Arc Logo Background Watermark -->
<svg class="arc-bg" viewBox="0 0 200 220" fill="none" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="arcGrad" x1="60" y1="0" x2="140" y2="220" gradientUnits="userSpaceOnUse">
      <stop offset="0%" stop-color="#c8d6e8"/>
      <stop offset="100%" stop-color="#6b8aaa"/>
    </linearGradient>
  </defs>
  <!-- Outer arch shape -->
  <path d="
    M100 8
    C55 8, 18 48, 18 100
    L18 210
    L52 210
    L52 100
    C52 67, 73 42, 100 42
    C127 42, 148 67, 148 100
    L148 210
    L182 210
    L182 100
    C182 48, 145 8, 100 8 Z
  " fill="url(#arcGrad)"/>
  <!-- Inner notch cutout bottom-left of arch -->
  <path d="
    M52 155
    L52 210
    L95 210
    L95 175
    C95 163, 85 155, 73 155 Z
  " fill="#0a0e1a"/>
</svg>

<header>
  <div>
    <div class="logo">Free<span>Pay</span></div>
    <div style="font-size:0.7rem;color:#6b7280;margin-top:2px;">Africa Freelancer Escrow · Arc Network</div>
  </div>
  <div style="display:flex;align-items:center;gap:10px;">
    <button id="darkToggle" onclick="toggleTheme()">☀️ Light</button>
    <div class="meta">
      <strong id="walletShort">—</strong>
      Wallet
    </div>
    <div class="meta">
      <strong>Arc Testnet</strong>
      Network
    </div>
    <button id="connectBtn" onclick="connectWallet()">Connect Wallet</button>
  </div>
</header>

<div class="hero">
  <div class="badge">⚡ Powered by Arc Network · USDC Settlement</div>
  <h1>Get Paid as an African Freelancer. <em>No Middlemen.</em></h1>
  <p>Clients lock USDC in escrow. You get paid instantly on Arc when work is done. No Payoneer. No Paypal. No delays.</p>
  <div class="stats">
    <div class="stat"><strong id="ngnRate">—</strong>NGN / USDC</div>
    <div class="stat"><strong>~0.3s</strong>Settlement</div>
    <div class="stat"><strong>&lt;$0.01</strong>Gas Fee</div>
    <div class="stat"><strong>0%</strong>Platform Fee</div>
    <div class="stat"><strong id="contractShort">0x2481…1D4d</strong>Contract</div>
  </div>
</div>

<div class="tabs">
  <button class="tab active" onclick="switchTab('client')">I'm a Client</button>
  <button class="tab" onclick="switchTab('freelancer')">I'm a Freelancer</button>
  <button class="tab" onclick="switchTab('feed')">— Live Feed —</button>
</div>

<main>

  <div class="panel active" id="panel-client">
    <div class="card">
      <h2>📋 Post a Job & Deposit USDC</h2>
      <label>Job Title</label>
      <input id="jobTitle" placeholder="e.g. Build landing page for my startup" />
      <label>Freelancer Wallet Address</label>
      <input id="freelancerAddr" placeholder="0x..." />
      <label>Payment Amount (USDC)</label>
      <input id="jobAmount" type="number" placeholder="e.g. 50" min="1" />
      <div id="ngnPreview"></div>
      <button class="btn btn-primary" id="postBtn" onclick="postJob()" disabled>Connect wallet to post job</button>
    </div>
    <div class="card">
      <h2>📁 Your Posted Jobs</h2>
      <div id="clientJobs"><div class="empty">Connect your wallet to see jobs</div></div>
    </div>
  </div>

  <div class="panel" id="panel-freelancer">
    <div class="card">
      <h2>💼 Jobs Assigned to You</h2>
      <div id="freelancerJobs"><div class="empty">Connect your wallet to see jobs</div></div>
    </div>
  </div>

  <div class="panel" id="panel-feed">
    <div class="card">
      <h2>📡 Live Activity Feed</h2>
      <div class="feed" id="feedBox">
        <div class="feed-line">Connecting to Arc Testnet...</div>
      </div>
    </div>
  </div>

</main>

<div id="toast"></div>

<script>
const CONTRACT_ADDRESS = "0x2481e47190217C7E2bc96a40a9BECA4895Dc1D4d";
const ABI = [
  "function createJob(address _freelancer, uint256 _amount, string calldata _title) external returns (uint256)",
  "function releasePayment(uint256 jobId) external",
  "function cancelJob(uint256 jobId) external",
  "function raiseDispute(uint256 jobId) external",
  "function getClientJobs(address client) external view returns (uint256[])",
  "function getFreelancerJobs(address freelancer) external view returns (uint256[])",
  "function jobs(uint256) external view returns (uint256,address,address,uint256,string,uint8,uint256)",
  "function jobCount() external view returns (uint256)",
  "event JobCreated(uint256 indexed jobId, address indexed client, address indexed freelancer, uint256 amount, string title)",
  "event PaymentReleased(uint256 indexed jobId, address freelancer, uint256 amount)"
];
const USDC_ABI = [
  "function approve(address spender, uint256 amount) external returns (bool)"
];
const USDC_ADDRESS = "0x3600000000000000000000000000000000000000";
const ARC_TESTNET = {
  chainId: "0x4CD032",
  chainName: "Arc Testnet",
  rpcUrls: ["https://rpc.testnet.arc.network"],
  nativeCurrency: { name: "USDC", symbol: "USDC", decimals: 6 },
  blockExplorerUrls: ["https://testnet.arcscan.app"]
};

let provider, signer, account, contract, usdcContract;
let ngnRate = 1620;

async function connectWallet() {
  if (!window.ethereum) { toast("Install MetaMask to continue", "warn"); return; }
  try {
    await window.ethereum.request({ method: "eth_requestAccounts" });
    try {
      await window.ethereum.request({ method: "wallet_switchEthereumChain", params: [{ chainId: ARC_TESTNET.chainId }] });
    } catch(e) {
      if (e.code === 4902) await window.ethereum.request({ method: "wallet_addEthereumChain", params: [ARC_TESTNET] });
    }
    provider = new ethers.BrowserProvider(window.ethereum);
    signer = await provider.getSigner();
    account = await signer.getAddress();
    contract = new ethers.Contract(CONTRACT_ADDRESS, ABI, signer);
    usdcContract = new ethers.Contract(USDC_ADDRESS, USDC_ABI, signer);
    document.getElementById("walletShort").textContent = shortAddr(account);
    document.getElementById("connectBtn").textContent = "Connected ✓";
    document.getElementById("connectBtn").style.background = "#10b981";
    document.getElementById("postBtn").disabled = false;
    document.getElementById("postBtn").textContent = "Post Job & Deposit USDC";
    listenToEvents();
    await loadClientJobs();
    await loadFreelancerJobs();
    toast("Wallet connected ✓");
  } catch(e) { toast("Connection failed: " + e.message, "err"); }
}

async function fetchRate() {
  try {
    const r = await fetch("https://api.coingecko.com/api/v3/simple/price?ids=usd-coin&vs_currencies=ngn");
    const d = await r.json();
    ngnRate = d["usd-coin"].ngn;
    document.getElementById("ngnRate").textContent = "₦" + ngnRate.toLocaleString();
  } catch { document.getElementById("ngnRate").textContent = "₦1,620"; }
}
fetchRate();

document.getElementById("jobAmount").addEventListener("input", e => {
  const v = parseFloat(e.target.value);
  document.getElementById("ngnPreview").textContent = v > 0 ? `≈ ₦${(v * ngnRate).toLocaleString()} NGN` : "";
});

async function postJob() {
  const title = document.getElementById("jobTitle").value.trim();
  const freelancer = document.getElementById("freelancerAddr").value.trim();
  const amount = parseFloat(document.getElementById("jobAmount").value);
  if (!title || !freelancer || !amount) { toast("Fill all fields", "warn"); return; }
  if (!ethers.isAddress(freelancer)) { toast("Invalid freelancer address", "warn"); return; }
  const btn = document.getElementById("postBtn");
  try {
    btn.disabled = true; btn.textContent = "Approving USDC…";
    const amountParsed = ethers.parseUnits(amount.toString(), 6);
    const approveTx = await usdcContract.approve(CONTRACT_ADDRESS, amountParsed);
    await approveTx.wait();
    btn.textContent = "Posting job…";
    const tx = await contract.createJob(freelancer, amountParsed, title);
    await tx.wait();
    toast("Job posted & funds locked ✓");
    document.getElementById("jobTitle").value = "";
    document.getElementById("freelancerAddr").value = "";
    document.getElementById("jobAmount").value = "";
    btn.disabled = false; btn.textContent = "Post Job & Deposit USDC";
    await loadClientJobs(); await loadFreelancerJobs();
  } catch(e) {
    toast("Error: " + (e.reason || e.message), "err");
    btn.disabled = false; btn.textContent = "Post Job & Deposit USDC";
  }
}

async function loadClientJobs() {
  const el = document.getElementById("clientJobs");
  if (!contract) return;
  try {
    const ids = await contract.getClientJobs(account);
    if (ids.length === 0) { el.innerHTML = '<div class="empty">No jobs posted yet</div>'; return; }
    el.innerHTML = "";
    for (const id of [...ids].reverse()) {
      const job = await contract.jobs(id);
      el.innerHTML += renderJob(job, id, "client");
    }
  } catch { el.innerHTML = '<div class="empty">Could not load jobs</div>'; }
}

async function loadFreelancerJobs() {
  const el = document.getElementById("freelancerJobs");
  if (!contract) return;
  try {
    const ids = await contract.getFreelancerJobs(account);
    if (ids.length === 0) { el.innerHTML = '<div class="empty">No jobs assigned to you yet</div>'; return; }
    el.innerHTML = "";
    for (const id of [...ids].reverse()) {
      const job = await contract.jobs(id);
      el.innerHTML += renderJob(job, id, "freelancer");
    }
  } catch { el.innerHTML = '<div class="empty">Could not load jobs</div>'; }
}

function renderJob(job, id, role) {
  const statusMap = ["OPEN","FUNDED","COMPLETED","DISPUTED","CANCELLED"];
  const chipMap = ["chip-funded","chip-funded","chip-completed","chip-disputed","chip-cancelled"];
  const status = Number(job[5]);
  const amount = ethers.formatUnits(job[3], 6);
  const ngn = (parseFloat(amount) * ngnRate).toLocaleString();
  const created = new Date(Number(job[6]) * 1000).toLocaleDateString();
  let actions = "";
  if (role === "client" && status === 1) {
    actions += `<button class="btn btn-success" onclick="releasePayment(${id})">✅ Release Payment</button>`;
    actions += `<button class="btn btn-danger" onclick="cancelJob(${id})">❌ Cancel</button>`;
  }
  if (role === "freelancer" && status === 1) {
    actions += `<button class="btn btn-warn" onclick="raiseDispute(${id})">⚠️ Raise Dispute</button>`;
  }
  return `<div class="job-card">
    <h3>${job[4]}</h3>
    <span class="chip ${chipMap[status]}">${statusMap[status]}</span>
    <span style="font-size:0.72rem;color:#6b7280;margin-left:8px;">Job #${id} · ${created}</span>
    <div class="amount">$${amount} USDC</div>
    <div class="ngn">≈ ₦${ngn} NGN</div>
    <div class="addr">${role === "client" ? "To: " + job[2] : "From: " + job[1]}</div>
    ${actions ? `<div class="job-actions">${actions}</div>` : ""}
  </div>`;
}

async function releasePayment(id) {
  try {
    toast("Releasing payment…");
    const tx = await contract.releasePayment(id);
    await tx.wait();
    toast("Payment released ✓");
    await loadClientJobs(); await loadFreelancerJobs();
  } catch(e) { toast("Error: " + (e.reason || e.message), "err"); }
}

async function cancelJob(id) {
  try {
    toast("Cancelling…");
    const tx = await contract.cancelJob(id);
    await tx.wait();
    toast("Cancelled. USDC refunded ✓");
    await loadClientJobs();
  } catch(e) { toast("Error: " + (e.reason || e.message), "err"); }
}

async function raiseDispute(id) {
  try {
    toast("Raising dispute…");
    const tx = await contract.raiseDispute(id);
    await tx.wait();
    toast("Dispute raised ✓");
    await loadFreelancerJobs();
  } catch(e) { toast("Error: " + (e.reason || e.message), "err"); }
}

function listenToEvents() {
  const feed = document.getElementById("feedBox");
  contract.on("JobCreated", (jobId, client, freelancer, amount, title) => {
    const amt = ethers.formatUnits(amount, 6);
    addFeed(`Job #${jobId} created · "${title}" · $${amt} USDC locked`);
  });
  contract.on("PaymentReleased", (jobId, freelancer, amount) => {
    const amt = ethers.formatUnits(amount, 6);
    addFeed(`Job #${jobId} · $${amt} USDC released to ${shortAddr(freelancer)}`);
  });
  addFeed("Connected to Arc Testnet · Listening for events…");
}

function addFeed(msg) {
  const feed = document.getElementById("feedBox");
  const line = document.createElement("div");
  line.className = "feed-line";
  line.textContent = `[${new Date().toLocaleTimeString()}] ${msg}`;
  feed.prepend(line);
}

function switchTab(tab) {
  document.querySelectorAll(".tab").forEach((t,i) => t.classList.toggle("active", ["client","freelancer","feed"][i] === tab));
  document.querySelectorAll(".panel").forEach(p => p.classList.remove("active"));
  document.getElementById("panel-" + tab).classList.add("active");
}

function toggleTheme() {
  const isLight = document.body.classList.toggle("light");
  document.getElementById("darkToggle").textContent = isLight ? "🌙 Dark" : "☀️ Light";
}

function shortAddr(a) { return a.slice(0,6) + "…" + a.slice(-4); }

function toast(msg, type) {
  const el = document.getElementById("toast");
  el.textContent = msg;
  el.style.borderLeftColor = type === "err" ? "#ef4444" : type === "warn" ? "#f59e0b" : "#10b981";
  el.style.display = "block";
  setTimeout(() => el.style.display = "none", 4000);
}
</script>
</body>
</html>
