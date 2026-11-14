# UNFOLLOWERS-TRACKER
INSTAGRAM UNFOLLOWERS TRACKER APP
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Instagram Unfollowers Tracker — Cloud Edition</title>
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet" />
  <style>
    body { font-family: 'Inter', sans-serif; }
  </style>
</head>
<body class="bg-slate-50 text-slate-900 min-h-screen py-6 px-4 sm:px-6">
  <main class="max-w-4xl mx-auto space-y-5">
    <!-- Toast Notification -->
    <div id="toast" class="hidden fixed bottom-4 right-4 z-50 max-w-sm rounded-xl bg-slate-900 text-white px-4 py-3 text-sm shadow-lg"></div>

    <!-- Cloud Workspace Header -->
    <header class="bg-white rounded-2xl border border-slate-200 p-5 sm:p-6 shadow-sm space-y-4">
      <div class="flex flex-wrap items-start justify-between gap-4">
        <div>
          <div class="flex flex-wrap items-center gap-2">
            <span class="inline-block text-xs font-semibold uppercase tracking-wider text-rose-600 bg-rose-50 px-2.5 py-1 rounded-full">Cloud Edition</span>
            <span id="syncBadge" class="inline-flex items-center gap-1.5 text-xs font-medium text-emerald-700 bg-emerald-50 px-2.5 py-1 rounded-full">
              <span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span>
              Local Sync Ready
            </span>
          </div>
          <h1 class="text-2xl font-bold mt-2 text-slate-900">Instagram Unfollowers Tracker</h1>
          <p class="text-sm text-slate-600 mt-1">Track unfollowers over time with dated snapshots, trend charts, and cloud backup.</p>
        </div>

        <div class="flex flex-wrap items-center gap-2">
          <div class="flex items-center gap-1.5 bg-slate-100 rounded-xl px-3 py-1.5">
            <label for="workspaceInput" class="text-xs font-medium text-slate-500">Account:</label>
            <input
              id="workspaceInput"
              type="text"
              value="@demo_creator"
              class="w-28 bg-transparent text-xs sm:text-sm font-semibold text-slate-800 focus:outline-none"
              aria-label="Instagram workspace handle"
            />
          </div>
          <button
            id="saveSnapshotBtn"
            type="button"
            class="px-3.5 py-2 rounded-xl bg-rose-600 hover:bg-rose-700 text-white text-xs sm:text-sm font-semibold shadow-sm transition cursor-pointer"
          >
            Save Snapshot
          </button>
          <button
            id="openCloudModalBtn"
            type="button"
            class="px-3.5 py-2 rounded-xl bg-white hover:bg-slate-100 text-slate-700 border border-slate-300 text-xs sm:text-sm font-medium transition cursor-pointer"
          >
            Cloud Sync
          </button>
        </div>
      </div>

      <!-- Navigation Tabs -->
      <nav class="flex flex-wrap gap-2 border-t border-slate-100 pt-4" aria-label="Dashboard sections">
        <button type="button" data-tab="overview" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-semibold bg-slate-900 text-white cursor-pointer transition">
          Overview &amp; Trends
        </button>
        <button type="button" data-tab="unfollowers" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-medium bg-slate-100 text-slate-700 hover:bg-slate-200 cursor-pointer transition">
          Not Following Back
        </button>
        <button type="button" data-tab="upload" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-medium bg-slate-100 text-slate-700 hover:bg-slate-200 cursor-pointer transition">
          Upload / Edit Data
        </button>
        <button type="button" data-tab="history" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-medium bg-slate-100 text-slate-700 hover:bg-slate-200 cursor-pointer transition">
          Snapshot History (<span id="snapshotCountLabel">0</span>)
        </button>
      </nav>
    </header>

    <!-- KPI Summary Row (Always Visible) -->
    <section class="grid grid-cols-2 sm:grid-cols-4 gap-3">
      <div class="bg-white rounded-2xl p-4 border border-slate-200 shadow-sm text-center">
        <div id="followingCount" class="text-2xl font-bold text-slate-900">0</div>
        <div class="text-xs text-slate-500 mt-0.5">Following</div>
      </div>
      <div class="bg-white rounded-2xl p-4 border border-slate-200 shadow-sm text-center">
        <div id="followersCount" class="text-2xl font-bold text-slate-900">0</div>
        <div class="text-xs text-slate-500 mt-0.5">Followers</div>
      </div>
      <div class="bg-rose-50 rounded-2xl p-4 border border-rose-200 shadow-sm text-center">
        <div id="unfollowersCount" class="text-2xl font-bold text-rose-600">0</div>
        <div class="text-xs text-rose-700 mt-0.5">Not Following Back</div>
      </div>
      <div class="bg-amber-50 rounded-2xl p-4 border border-amber-200 shadow-sm text-center">
        <div id="newUnfollowersCount" class="text-2xl font-bold text-amber-700">0</div>
        <div class="text-xs text-amber-800 mt-0.5">New Unfollowers</div>
      </div>
    </section>

    <!-- TAB 1: Overview & Trends -->
    <section id="tab-overview" class="tab-panel space-y-5">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm">
          <h2 class="text-sm font-semibold text-slate-800">Lost Followers (Since Previous Snapshot)</h2>
          <p class="text-xs text-slate-500 mt-0.5 mb-3">Accounts that followed you before, but don't anymore.</p>
          <div id="lostFollowersContainer" class="text-sm text-slate-600"></div>
        </div>

        <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm">
          <h2 class="text-sm font-semibold text-slate-800">New Followers (Since Previous Snapshot)</h2>
          <p class="text-xs text-slate-500 mt-0.5 mb-3">Accounts that started following you recently.</p>
          <div id="gainedFollowersContainer" class="text-sm text-slate-600"></div>
        </div>
      </div>

      <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm">
        <div class="flex items-center justify-between gap-2 mb-3">
          <div>
            <h2 class="text-sm font-semibold text-slate-800">Follower &amp; Following Trend</h2>
            <p class="text-xs text-slate-500">Updates automatically as you save snapshots.</p>
          </div>
        </div>
        <div class="relative h-64 w-full">
          <canvas id="trendChart"></canvas>
        </div>
      </div>
    </section>

    <!-- TAB 2: Not Following Back -->
    <section id="tab-unfollowers" class="tab-panel hidden bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-4">
      <div class="flex flex-wrap items-center justify-between gap-3">
        <h2 class="text-sm font-semibold text-slate-800">Accounts Not Following You Back</h2>
        <div class="flex flex-wrap items-center gap-2 w-full sm:w-auto">
          <input
            id="searchInput"
            type="search"
            placeholder="Filter username..."
            aria-label="Filter unfollowers by username"
            class="flex-1 sm:w-48 rounded-lg border border-slate-300 px-3 py-1.5 text-xs sm:text-sm text-slate-800 focus:outline-none focus:ring-2 focus:ring-rose-500 focus:border-rose-500"
          />
          <button
            id="downloadCsvBtn"
            type="button"
            class="px-3 py-1.5 rounded-lg bg-slate-900 hover:bg-slate-800 disabled:bg-slate-300 disabled:cursor-not-allowed text-white text-xs sm:text-sm font-medium transition cursor-pointer"
          >
            Download CSV
          </button>
        </div>
      </div>

      <div id="resultsContainer" class="text-sm text-slate-600"></div>
    </section>

    <!-- TAB 3: Upload / Edit Data -->
    <section id="tab-upload" class="tab-panel hidden space-y-5">
      <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm">
        <details class="rounded-xl bg-slate-50 border border-slate-200 px-4 py-3 text-sm text-slate-700">
          <summary class="font-medium cursor-pointer text-slate-800 select-none">How to get your Instagram JSON files</summary>
          <ol class="list-decimal list-inside mt-2 space-y-1 text-xs sm:text-sm text-slate-600">
            <li>Open Instagram → <strong>Settings</strong> → <strong>Accounts Center</strong> → <strong>Your information and permissions</strong>.</li>
            <li>Select <strong>Download your information</strong> → <strong>Download or transfer information</strong> → <strong>Some of your information</strong>.</li>
            <li>Check <strong>Followers and following</strong>, choose <strong>Download to device</strong>, and set the format to <strong>JSON</strong>.</li>
            <li>Unzip the downloaded folder and upload <code>following.json</code> and <code>followers_1.json</code> below.</li>
          </ol>
        </details>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm flex flex-col gap-3">
          <label for="followingInput" class="text-sm font-semibold text-slate-800">
            Following <span class="font-normal text-slate-500">(one per line)</span>
          </label>
          <label class="flex flex-col items-center justify-center rounded-xl border-2 border-dashed border-slate-300 hover:border-rose-400 bg-slate-50 hover:bg-rose-50/40 px-3 py-3 text-center cursor-pointer transition">
            <span class="text-xs font-semibold text-rose-600">Upload following.json</span>
            <span id="followingFileStatus" class="text-xs text-slate-500 mt-0.5">Optional · .json file</span>
            <input id="followingFileInput" type="file" accept=".json,application/json" class="hidden" />
          </label>
          <textarea
            id="followingInput"
            rows="7"
            class="w-full rounded-xl border border-slate-300 p-3 text-sm font-mono text-slate-800 focus:outline-none focus:ring-2 focus:ring-rose-500 focus:border-rose-500"
            placeholder="alex_travels&#10;maya.codes&#10;foodie_daily"
          ></textarea>
        </div>

        <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm flex flex-col gap-3">
          <label for="followersInput" class="text-sm font-semibold text-slate-800">
            Followers <span class="font-normal text-slate-500">(one per line)</span>
          </label>
          <label class="flex flex-col items-center justify-center rounded-xl border-2 border-dashed border-slate-300 hover:border-rose-400 bg-slate-50 hover:bg-rose-50/40 px-3 py-3 text-center cursor-pointer transition">
            <span class="text-xs font-semibold text-rose-600">Upload followers_1.json</span>
            <span id="followersFileStatus" class="text-xs text-slate-500 mt-0.5">Optional · .json file</span>
            <input id="followersFileInput" type="file" accept=".json,application/json" class="hidden" />
          </label>
          <textarea
            id="followersInput"
            rows="7"
            class="w-full rounded-xl border border-slate-300 p-3 text-sm font-mono text-slate-800 focus:outline-none focus:ring-2 focus:ring-rose-500 focus:border-rose-500"
            placeholder="alex_travels&#10;foodie_daily"
          ></textarea>
        </div>
      </div>

      <div class="flex flex-wrap items-center gap-3">
        <button
          id="compareBtn"
          type="button"
          class="px-5 py-2.5 rounded-xl bg-rose-600 hover:bg-rose-700 text-white text-sm font-semibold shadow-sm transition cursor-pointer"
        >
          Find Unfollowers
        </button>
        <button
          id="sampleBtn"
          type="button"
          class="px-4 py-2.5 rounded-xl bg-white hover:bg-slate-100 text-slate-700 border border-slate-300 text-sm font-medium transition cursor-pointer"
        >
          Reset Sample Data
        </button>
        <button
          id="clearBtn"
          type="button"
          class="px-4 py-2.5 rounded-xl bg-white hover:bg-slate-100 text-slate-700 border border-slate-300 text-sm font-medium transition cursor-pointer"
        >
          Clear All
        </button>
      </div>
    </section>

    <!-- TAB 4: Snapshot History & Cloud Backup -->
    <section id="tab-history" class="tab-panel hidden bg-white rounded-2xl border border-slate-200 p-6 shadow-sm space-y-4">
      <div class="flex flex-wrap items-center justify-between gap-3">
        <div>
          <h2 class="text-sm font-semibold text-slate-800">Saved Snapshots</h2>
          <p class="text-xs text-slate-500">Compare past exports or move your history across devices.</p>
        </div>
        <div class="flex flex-wrap items-center gap-2">
          <button
            id="exportBackupBtn"
            type="button"
            class="px-3 py-1.5 rounded-lg bg-slate-900 hover:bg-slate-800 text-white text-xs sm:text-sm font-medium transition cursor-pointer"
          >
            Export Backup (.json)
          </button>
          <label class="px-3 py-1.5 rounded-lg bg-white hover:bg-slate-100 text-slate-700 border border-slate-300 text-xs sm:text-sm font-medium transition cursor-pointer">
            Restore Backup
            <input id="restoreBackupInput" type="file" accept=".json,application/json" class="hidden" />
          </label>
        </div>
      </div>

      <div id="historyListContainer" class="text-sm text-slate-600"></div>
    </section>

    <!-- Cloud Sync Settings Modal -->
    <div id="cloudModal" class="hidden fixed inset-0 z-40 bg-slate-900/50 flex items-center justify-center p-4">
      <div class="bg-white rounded-2xl border border-slate-200 max-w-md w-full p-6 shadow-xl space-y-4">
        <div class="flex items-center justify-between">
          <h3 class="text-base font-bold text-slate-900">Cloud Sync Settings</h3>
          <button id="closeCloudModalBtn" type="button" class="text-slate-400 hover:text-slate-700 text-lg font-bold cursor-pointer">&times;</button>
        </div>
        <p class="text-xs text-slate-600">
          By default, snapshots are saved in your browser and can be exported as a <code>.json</code> backup. To sync across devices automatically, connect a public CORS-enabled REST endpoint (such as Supabase).
        </p>
        <div class="space-y-3">
          <div>
            <label for="cloudEndpointInput" class="block text-xs font-semibold text-slate-700 mb-1">REST Endpoint URL</label>
            <input
              id="cloudEndpointInput"
              type="url"
              placeholder="https://your-project.supabase.co/rest/v1/snapshots"
              class="w-full rounded-lg border border-slate-300 px-3 py-2 text-xs text-slate-800 focus:outline-none focus:ring-2 focus:ring-rose-500"
            />
          </div>
          <div>
            <label for="cloudKeyInput" class="block text-xs font-semibold text-slate-700 mb-1">Anon / Public API Key (optional)</label>
            <input
              id="cloudKeyInput"
              type="password"
              placeholder="eyJhbGci..."
              class="w-full rounded-lg border border-slate-300 px-3 py-2 text-xs text-slate-800 focus:outline-none focus:ring-2 focus:ring-rose-500"
            />
          </div>
        </div>
        <div id="cloudStatusMsg" class="text-xs text-slate-500"></div>
        <div class="flex items-center justify-end gap-2 pt-2">
          <button id="testCloudBtn" type="button" class="px-3.5 py-2 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-800 text-xs font-semibold cursor-pointer">
            Test &amp; Push Latest
          </button>
          <button id="saveCloudConfigBtn" type="button" class="px-3.5 py-2 rounded-lg bg-rose-600 hover:bg-rose-700 text-white text-xs font-semibold cursor-pointer">
            Save Settings
          </button>
        </div>
      </div>
    </div>
  </main>

  <script>
    const STORAGE_KEY = 'ig_unfollowers_cloud_v1';
    const CLOUD_CFG_KEY = 'ig_unfollowers_cloud_cfg_v1';

    const workspaceInput = document.getElementById('workspaceInput');
    const syncBadge = document.getElementById('syncBadge');
    const saveSnapshotBtn = document.getElementById('saveSnapshotBtn');
    const openCloudModalBtn = document.getElementById('openCloudModalBtn');
    const closeCloudModalBtn = document.getElementById('closeCloudModalBtn');
    const cloudModal = document.getElementById('cloudModal');
    const cloudEndpointInput = document.getElementById('cloudEndpointInput');
    const cloudKeyInput = document.getElementById('cloudKeyInput');
    const cloudStatusMsg = document.getElementById('cloudStatusMsg');
    const testCloudBtn = document.getElementById('testCloudBtn');
    const saveCloudConfigBtn = document.getElementById('saveCloudConfigBtn');

    const followingInput = document.getElementById('followingInput');
    const followersInput = document.getElementById('followersInput');
    const followingFileInput = document.getElementById('followingFileInput');
    const followersFileInput = document.getElementById('followersFileInput');
    const followingFileStatus = document.getElementById('followingFileStatus');
    const followersFileStatus = document.getElementById('followersFileStatus');
    const compareBtn = document.getElementById('compareBtn');
    const sampleBtn = document.getElementById('sampleBtn');
    const clearBtn = document.getElementById('clearBtn');
    const searchInput = document.getElementById('searchInput');
    const downloadCsvBtn = document.getElementById('downloadCsvBtn');
    const exportBackupBtn = document.getElementById('exportBackupBtn');
    const restoreBackupInput = document.getElementById('restoreBackupInput');

    const followingCountEl = document.getElementById('followingCount');
    const followersCountEl = document.getElementById('followersCount');
    const unfollowersCountEl = document.getElementById('unfollowersCount');
    const newUnfollowersCountEl = document.getElementById('newUnfollowersCount');
    const snapshotCountLabel = document.getElementById('snapshotCountLabel');

    const resultsContainer = document.getElementById('resultsContainer');
    const lostFollowersContainer = document.getElementById('lostFollowersContainer');
    const gainedFollowersContainer = document.getElementById('gainedFollowersContainer');
    const historyListContainer = document.getElementById('historyListContainer');
    const toastEl = document.getElementById('toast');

    const SAMPLE_FOLLOWING = [
      'alex_travels',
      'maya.codes',
      'foodie_daily',
      'design.studio',
      'rohan_fitness',
      'minimal.desk'
    ];

    const SAMPLE_FOLLOWERS = [
      'alex_travels',
      'foodie_daily',
      'rohan_fitness',
      'new_fan_99'
    ];

    const DEFAULT_SNAPSHOTS = [
      {
        id: 'snap_1',
        createdAt: '2026-10-15T10:00:00Z',
        label: 'Oct 15',
        following: ['alex_travels', 'maya.codes', 'foodie_daily', 'design.studio', 'rohan_fitness'],
        followers: ['alex_travels', 'maya.codes', 'foodie_daily', 'rohan_fitness']
      },
      {
        id: 'snap_2',
        createdAt: '2026-11-01T10:00:00Z',
        label: 'Nov 1',
        following: ['alex_travels', 'maya.codes', 'foodie_daily', 'design.studio', 'rohan_fitness', 'minimal.desk'],
        followers: ['alex_travels', 'maya.codes', 'foodie_daily', 'rohan_fitness']
      },
      {
        id: 'snap_3',
        createdAt: '2026-11-14T10:00:00Z',
        label: 'Nov 14',
        following: [...SAMPLE_FOLLOWING],
        followers: [...SAMPLE_FOLLOWERS]
      }
    ];

    let snapshots = [];
    let currentUnfollowers = [];
    let currentFollowingCount = 0;
    let trendChartInstance = null;
    let toastTimer = null;

    function showToast(message) {
      toastEl.textContent = message;
      toastEl.classList.remove('hidden');
      if (toastTimer) clearTimeout(toastTimer);
      toastTimer = setTimeout(() => {
        toastEl.classList.add('hidden');
      }, 3200);
    }

    function loadSnapshotsFromStorage() {
      try {
        const raw = localStorage.getItem(STORAGE_KEY);
        if (raw) {
          const parsed = JSON.parse(raw);
          if (Array.isArray(parsed) && parsed.length > 0) {
            return parsed;
          }
        }
      } catch (e) {}
      return JSON.parse(JSON.stringify(DEFAULT_SNAPSHOTS));
    }

    function saveSnapshotsToStorage() {
      try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(snapshots));
      } catch (e) {}
    }

    function loadCloudConfig() {
      try {
        const raw = localStorage.getItem(CLOUD_CFG_KEY);
        if (raw) {
          const cfg = JSON.parse(raw);
          cloudEndpointInput.value = cfg.endpoint || '';
          cloudKeyInput.value = cfg.apiKey || '';
          updateSyncBadge(cfg.endpoint);
        }
      } catch (e) {}
    }

    function updateSyncBadge(endpoint) {
      if (endpoint && endpoint.startsWith('https://')) {
        syncBadge.innerHTML = '<span class="w-1.5 h-1.5 rounded-full bg-sky-500"></span>Cloud Endpoint Configured';
        syncBadge.className = 'inline-flex items-center gap-1.5 text-xs font-medium text-sky-700 bg-sky-50 px-2.5 py-1 rounded-full';
      } else {
        syncBadge.innerHTML = '<span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span>Local Sync Ready';
        syncBadge.className = 'inline-flex items-center gap-1.5 text-xs font-medium text-emerald-700 bg-emerald-50 px-2.5 py-1 rounded-full';
      }
    }

    function switchTab(tabName) {
      document.querySelectorAll('.tab-panel').forEach((panel) => {
        panel.classList.add('hidden');
      });
      const activePanel = document.getElementById('tab-' + tabName);
      if (activePanel) activePanel.classList.remove('hidden');

      document.querySelectorAll('.tab-btn').forEach((btn) => {
        const isActive = btn.dataset.tab === tabName;
        btn.className = isActive
          ? 'tab-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-semibold bg-slate-900 text-white cursor-pointer transition'
          : 'tab-btn px-3.5 py-1.5 rounded-lg text-xs sm:text-sm font-medium bg-slate-100 text-slate-700 hover:bg-slate-200 cursor-pointer transition';
      });

      if (tabName === 'overview') {
        renderChart();
      }
    }

    function parseUsernames(rawText) {
      const lines = rawText
        .split(/\r?\n/)
        .map((line) => line.trim().replace(/^@/, '').toLowerCase())
        .filter((line) => line.length > 0);
      return Array.from(new Set(lines));
    }

    function extractInstagramUsernames(data) {
      let entries = [];
      if (Array.isArray(data)) {
        entries = data;
      } else if (data && Array.isArray(data.relationships_following)) {
        entries = data.relationships_following;
      } else if (data && Array.isArray(data.relationships_followers)) {
        entries = data.relationships_followers;
      } else {
        throw new Error('Unrecognized Instagram JSON format.');
      }

      const usernames = [];
      entries.forEach((entry) => {
        const fromList = entry && Array.isArray(entry.string_list_data) && entry.string_list_data[0]
          ? entry.string_list_data[0].value
          : '';
        const fromTitle = entry && typeof entry.title === 'string' ? entry.title : '';
        const username = (fromList || fromTitle || '').trim().replace(/^@/, '').toLowerCase();
        if (username) usernames.push(username);
      });

      return Array.from(new Set(usernames));
    }

    function handleJsonUpload(fileInput, targetTextarea, statusEl) {
      const file = fileInput.files && fileInput.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (event) => {
        try {
          const parsed = JSON.parse(event.target.result);
          const usernames = extractInstagramUsernames(parsed);
          if (usernames.length === 0) {
            statusEl.textContent = 'No usernames found in this file.';
            statusEl.className = 'text-xs text-rose-600 mt-0.5 font-medium';
            return;
          }
          targetTextarea.value = usernames.join('\n');
          statusEl.textContent = `Loaded ${usernames.length} accounts from ${file.name}`;
          statusEl.className = 'text-xs text-emerald-700 mt-0.5 font-medium';
          findUnfollowers();
          showToast(`Loaded ${usernames.length} accounts from ${file.name}`);
        } catch (err) {
          statusEl.textContent = 'Invalid JSON file. Please upload an Instagram .json export.';
          statusEl.className = 'text-xs text-rose-600 mt-0.5 font-medium';
        }
      };
      reader.readAsText(file);
    }

    function renderMiniUserList(container, usernames, badgeText, badgeClass, emptyText) {
      container.innerHTML = '';
      if (usernames.length === 0) {
        container.innerHTML = `<p class="text-xs text-slate-500 italic">${emptyText}</p>`;
        return;
      }
      const ul = document.createElement('ul');
      ul.className = 'divide-y divide-slate-100 border border-slate-200 rounded-xl overflow-hidden';
      usernames.forEach((username) => {
        const li = document.createElement('li');
        li.className = 'flex items-center justify-between px-3 py-2 bg-white';
        const link = document.createElement('a');
        link.href = 'https://www.instagram.com/' + encodeURIComponent(username) + '/';
        link.target = '_blank';
        link.rel = 'noopener noreferrer';
        link.className = 'font-mono text-xs sm:text-sm text-slate-800 hover:text-rose-600 hover:underline';
        link.textContent = '@' + username;

        const badge = document.createElement('span');
        badge.className = `text-xs font-medium px-2 py-0.5 rounded-full ${badgeClass}`;
        badge.textContent = badgeText;

        li.appendChild(link);
        li.appendChild(badge);
        ul.appendChild(li);
      });
      container.appendChild(ul);
    }

    function renderDiffs(currentFollowers) {
      if (snapshots.length < 2) {
        newUnfollowersCountEl.textContent = '0';
        renderMiniUserList(lostFollowersContainer, [], '', '', 'Save at least 2 snapshots to track lost followers.');
        renderMiniUserList(gainedFollowersContainer, [], '', '', 'Save at least 2 snapshots to track new followers.');
        return;
      }

      const prevSnap = snapshots[snapshots.length - 2];
      const prevSet = new Set(prevSnap.followers);
      const currSet = new Set(currentFollowers);

      const lost = prevSnap.followers.filter((u) => !currSet.has(u));
      const gained = currentFollowers.filter((u) => !prevSet.has(u));

      newUnfollowersCountEl.textContent = String(lost.length);
      renderMiniUserList(lostFollowersContainer, lost, 'Unfollowed', 'text-amber-800 bg-amber-100', 'No lost followers since the previous snapshot.');
      renderMiniUserList(gainedFollowersContainer, gained, 'New follower', 'text-emerald-700 bg-emerald-100', 'No new followers since the previous snapshot.');
    }

    function renderChart() {
      const canvas = document.getElementById('trendChart');
      if (!canvas || typeof Chart === 'undefined') return;

      const labels = snapshots.map((s) => s.label || new Date(s.createdAt).toLocaleDateString());
      const followersSeries = snapshots.map((s) => s.followers.length);
      const followingSeries = snapshots.map((s) => s.following.length);

      if (trendChartInstance) {
        trendChartInstance.destroy();
      }

      trendChartInstance = new Chart(canvas, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              label: 'Followers',
              data: followersSeries,
              borderColor: '#e11d48',
              backgroundColor: 'rgba(225, 29, 72, 0.12)',
              tension: 0.3,
              fill: true
            },
            {
              label: 'Following',
              data: followingSeries,
              borderColor: '#0f172a',
              backgroundColor: 'rgba(15, 23, 42, 0.06)',
              tension: 0.3,
              fill: false
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: { position: 'top' }
          },
          scales: {
            y: { beginAtZero: true, ticks: { precision: 0 } }
          }
        }
      });
    }

    function renderHistory() {
      snapshotCountLabel.textContent = String(snapshots.length);
      historyListContainer.innerHTML = '';

      if (snapshots.length === 0) {
        historyListContainer.innerHTML = '<p class="text-slate-500 italic">No snapshots saved yet. Click "Save Snapshot" at the top to record one.</p>';
        return;
      }

      const ul = document.createElement('ul');
      ul.className = 'divide-y divide-slate-100 border border-slate-200 rounded-xl overflow-hidden';

      [...snapshots].reverse().forEach((snap) => {
        const li = document.createElement('li');
        li.className = 'flex flex-wrap items-center justify-between gap-3 px-4 py-3 bg-white hover:bg-slate-50';

        const meta = document.createElement('div');
        meta.innerHTML = `<div class="font-semibold text-slate-800 text-sm">${snap.label}</div>
          <div class="text-xs text-slate-500">${snap.following.length} following · ${snap.followers.length} followers</div>`;

        const actions = document.createElement('div');
        actions.className = 'flex items-center gap-2';

        const loadBtn = document.createElement('button');
        loadBtn.type = 'button';
        loadBtn.className = 'px-3 py-1 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-800 text-xs font-medium cursor-pointer';
        loadBtn.textContent = 'Load';
        loadBtn.addEventListener('click', () => {
          followingInput.value = snap.following.join('\n');
          followersInput.value = snap.followers.join('\n');
          findUnfollowers();
          showToast(`Loaded snapshot "${snap.label}"`);
        });

        const delBtn = document.createElement('button');
        delBtn.type = 'button';
        delBtn.className = 'px-3 py-1 rounded-lg bg-rose-50 hover:bg-rose-100 text-rose-700 text-xs font-medium cursor-pointer';
        delBtn.textContent = 'Delete';
        delBtn.addEventListener('click', () => {
          snapshots = snapshots.filter((item) => item.id !== snap.id);
          saveSnapshotsToStorage();
          renderHistory();
          renderDiffs(parseUsernames(followersInput.value));
          renderChart();
          showToast(`Deleted snapshot "${snap.label}"`);
        });

        actions.appendChild(loadBtn);
        actions.appendChild(delBtn);
        li.appendChild(meta);
        li.appendChild(actions);
        ul.appendChild(li);
      });

      historyListContainer.appendChild(ul);
    }

    function renderResults() {
      resultsContainer.innerHTML = '';
      downloadCsvBtn.disabled = currentUnfollowers.length === 0;

      if (currentFollowingCount === 0) {
        resultsContainer.innerHTML = '<p class="text-slate-500 italic">Upload a JSON file or add at least one username in "Following" and click Find Unfollowers.</p>';
        return;
      }

      if (currentUnfollowers.length === 0) {
        resultsContainer.innerHTML = '<p class="text-emerald-700 bg-emerald-50 border border-emerald-200 rounded-xl px-4 py-3 font-medium">Everyone you follow is following you back!</p>';
        return;
      }

      const query = searchInput.value.trim().replace(/^@/, '').toLowerCase();
      const visible = query
        ? currentUnfollowers.filter((username) => username.includes(query))
        : currentUnfollowers;

      if (visible.length === 0) {
        resultsContainer.innerHTML = '<p class="text-slate-500 italic">No matching accounts found for that filter.</p>';
        return;
      }

      const list = document.createElement('ul');
      list.className = 'divide-y divide-slate-100 border border-slate-200 rounded-xl overflow-hidden';

      visible.forEach((username) => {
        const li = document.createElement('li');
        li.className = 'flex items-center justify-between px-4 py-2.5 bg-white hover:bg-slate-50 gap-2';

        const link = document.createElement('a');
        link.href = 'https://www.instagram.com/' + encodeURIComponent(username) + '/';
        link.target = '_blank';
        link.rel = 'noopener noreferrer';
        link.className = 'font-mono text-sm text-slate-800 hover:text-rose-600 hover:underline';
        link.textContent = '@' + username;

        const badge = document.createElement('span');
        badge.className = 'text-xs font-medium text-rose-600 bg-rose-50 px-2 py-0.5 rounded-full';
        badge.textContent = 'Not following back';

        li.appendChild(link);
        li.appendChild(badge);
        list.appendChild(li);
      });

      resultsContainer.appendChild(list);
    }

    function findUnfollowers() {
      const following = parseUsernames(followingInput.value);
      const followers = parseUsernames(followersInput.value);
      const followerSet = new Set(followers);

      currentFollowingCount = following.length;
      currentUnfollowers = following.filter((user) => !followerSet.has(user));

      followingCountEl.textContent = String(following.length);
      followersCountEl.textContent = String(followers.length);
      unfollowersCountEl.textContent = String(currentUnfollowers.length);

      renderResults();
      renderDiffs(followers);
      renderHistory();
      renderChart();
    }

    function saveCurrentSnapshot() {
      const following = parseUsernames(followingInput.value);
      const followers = parseUsernames(followersInput.value);
      if (following.length === 0 && followers.length === 0) {
        showToast('Add following or follower usernames before saving a snapshot.');
        return;
      }

      const now = new Date();
      const label = now.toLocaleDateString(undefined, { month: 'short', day: 'numeric' }) +
        ' · ' + now.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' });

      const snap = {
        id: 'snap_' + Date.now(),
        createdAt: now.toISOString(),
        label,
        following,
        followers
      };

      snapshots.push(snap);
      saveSnapshotsToStorage();
      findUnfollowers();
      showToast(`Saved snapshot "${label}"`);
    }

    function downloadCsv() {
      if (currentUnfollowers.length === 0) return;
      const rows = ['username,profile_url'];
      currentUnfollowers.forEach((username) => {
        rows.push(`${username},https://www.instagram.com/${encodeURIComponent(username)}/`);
      });

      const blob = new Blob([rows.join('\n')], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'instagram_unfollowers.csv';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    }

    function exportCloudBackup() {
      const payload = {
        account: workspaceInput.value.trim() || '@demo_creator',
        exportedAt: new Date().toISOString(),
        snapshots
      };
      const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'ig_unfollowers_cloud_backup.json';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
      showToast('Exported cloud backup JSON.');
    }

    function restoreCloudBackup(fileInput) {
      const file = fileInput.files && fileInput.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (event) => {
        try {
          const parsed = JSON.parse(event.target.result);
          if (!parsed || !Array.isArray(parsed.snapshots)) {
            showToast('Invalid backup file.');
            return;
          }
          snapshots = parsed.snapshots;
          if (parsed.account) workspaceInput.value = parsed.account;
          saveSnapshotsToStorage();
          if (snapshots.length > 0) {
            const latest = snapshots[snapshots.length - 1];
            followingInput.value = latest.following.join('\n');
            followersInput.value = latest.followers.join('\n');
          }
          findUnfollowers();
          showToast(`Restored ${snapshots.length} snapshots from backup.`);
        } catch (e) {
          showToast('Could not read backup JSON.');
        }
      };
      reader.readAsText(file);
    }

    function loadSampleData() {
      followingInput.value = SAMPLE_FOLLOWING.join('\n');
      followersInput.value = SAMPLE_FOLLOWERS.join('\n');
      followingFileInput.value = '';
      followersFileInput.value = '';
      searchInput.value = '';
      followingFileStatus.textContent = 'Optional · .json file';
      followingFileStatus.className = 'text-xs text-slate-500 mt-0.5';
      followersFileStatus.textContent = 'Optional · .json file';
      followersFileStatus.className = 'text-xs text-slate-500 mt-0.5';
      findUnfollowers();
    }

    function clearAll() {
      followingInput.value = '';
      followersInput.value = '';
      followingFileInput.value = '';
      followersFileInput.value = '';
      searchInput.value = '';
      followingFileStatus.textContent = 'Optional · .json file';
      followingFileStatus.className = 'text-xs text-slate-500 mt-0.5';
      followersFileStatus.textContent = 'Optional · .json file';
      followersFileStatus.className = 'text-xs text-slate-500 mt-0.5';
      findUnfollowers();
    }

    document.querySelectorAll('.tab-btn').forEach((btn) => {
      btn.addEventListener('click', () => switchTab(btn.dataset.tab));
    });

    followingFileInput.addEventListener('change', () => {
      handleJsonUpload(followingFileInput, followingInput, followingFileStatus);
    });
    followersFileInput.addEventListener('change', () => {
      handleJsonUpload(followersFileInput, followersInput, followersFileStatus);
    });

    saveSnapshotBtn.addEventListener('click', saveCurrentSnapshot);
    searchInput.addEventListener('input', renderResults);
    downloadCsvBtn.addEventListener('click', downloadCsv);
    compareBtn.addEventListener('click', findUnfollowers);
    sampleBtn.addEventListener('click', loadSampleData);
    clearBtn.addEventListener('click', clearAll);
    exportBackupBtn.addEventListener('click', exportCloudBackup);
    restoreBackupInput.addEventListener('change', () => restoreCloudBackup(restoreBackupInput));

    openCloudModalBtn.addEventListener('click', () => {
      cloudStatusMsg.textContent = '';
      cloudModal.classList.remove('hidden');
    });
    closeCloudModalBtn.addEventListener('click', () => {
      cloudModal.classList.add('hidden');
    });
    saveCloudConfigBtn.addEventListener('click', () => {
      const endpoint = cloudEndpointInput.value.trim();
      const apiKey = cloudKeyInput.value.trim();
      try {
        localStorage.setItem(CLOUD_CFG_KEY, JSON.stringify({ endpoint, apiKey }));
      } catch (e) {}
      updateSyncBadge(endpoint);
      cloudModal.classList.add('hidden');
      showToast('Cloud sync settings saved.');
    });

    testCloudBtn.addEventListener('click', async () => {
      const endpoint = cloudEndpointInput.value.trim();
      const apiKey = cloudKeyInput.value.trim();
      if (!endpoint.startsWith('https://')) {
        cloudStatusMsg.textContent = 'Enter a valid https:// REST endpoint URL first.';
        cloudStatusMsg.className = 'text-xs text-rose-600';
        return;
      }
      cloudStatusMsg.textContent = 'Syncing latest snapshot...';
      cloudStatusMsg.className = 'text-xs text-slate-600';
      try {
        const headers = { 'Content-Type': 'application/json' };
        if (apiKey) {
          headers['apikey'] = apiKey;
          headers['Authorization'] = 'Bearer ' + apiKey;
        }
        const latest = snapshots[snapshots.length - 1] || null;
        const res = await fetch(endpoint, {
          method: 'POST',
          headers,
          body: JSON.stringify({
            account: workspaceInput.value.trim() || '@demo_creator',
            snapshot: latest
          })
        });
        if (!res.ok) throw new Error('HTTP ' + res.status);
        cloudStatusMsg.textContent = 'Synced successfully!';
        cloudStatusMsg.className = 'text-xs text-emerald-700 font-medium';
      } catch (err) {
        cloudStatusMsg.textContent = 'Could not reach endpoint (' + err.message + '). Local backup still works.';
        cloudStatusMsg.className = 'text-xs text-rose-600';
      }
    });

    snapshots = loadSnapshotsFromStorage();
    loadCloudConfig();
    loadSampleData();
  </script>
</body>
</html>
