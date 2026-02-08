<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from 'vue';
import Player from './Player.vue';
const year = ref('');
const leagueId = ref('');
const userTeamId = ref('');
const auth = ref('');

// Cookie management functions
function setCookie(name, value, days = 7) {
  const date = new Date();
  date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
  const expires = "expires=" + date.toUTCString();
  document.cookie = name + "=" + value + ";" + expires + ";path=/";
}

function getCookie(name) {
  const nameEQ = name + "=";
  const cookies = document.cookie.split(';');
  for (let cookie of cookies) {
    cookie = cookie.trim();
    if (cookie.indexOf(nameEQ) === 0) {
      return cookie.substring(nameEQ.length);
    }
  }
  return null;
}

// Load from cookies on mount
function loadFromCookies() {
  const savedYear = getCookie('year') || (new Date()).getFullYear();
  const savedLeagueId = getCookie('leagueId');
  const savedUserTeamId = getCookie('userTeamId');
  const savedAuth = getCookie('auth');
  
  if (savedYear) year.value = savedYear;
  if (savedLeagueId) leagueId.value = savedLeagueId;
  if (savedUserTeamId) userTeamId.value = savedUserTeamId;
  if (savedAuth) auth.value = savedAuth;
}

const liveDraftData = ref(null);
const liveDraftRecap = ref(null);
const liveDraftOrder = ref(null);
const liveDraftPlayers = ref(null);
const fetchStatus = ref('paused')
const fetchInterval = ref(5)
const initialfetchRequired = ref(true);
const debug = ref(false);
const options = ref(false);
let intervalId = null;
const isFetching = ref(false);

const teamshape = [
  [
    ['HOK',1],
    ['FRF',1],
    ['2RF',1],
    ['HFB',1],
    ['5/8',1],
    ['CTW',1],
    ['FLB',1]
  ],
  [
    ['HOK',1],
    ['FRF',1],
    ['FRF',2],
    ['2RF',1],
    ['2RF',2],
    ['HFB',1],
    ['5/8',1],
    ['CTW',1],
    ['CTW',2],
    ['FLB',1]
  ],
  [
    ['HOK',1],
    ['FRF',1],
    ['FRF',2],
    ['2RF',1],
    ['2RF',2],
    ['HFB',1],
    ['5/8',1],
    ['CTW',1],
    ['CTW',2],
    ['FLB',1]
  ],
  [
    ['HOK',1],
    ['FRF',1],
    ['FRF',2],
    ['2RF',1],
    ['2RF',2],
    ['2RF',3],
    ['HFB',1],
    ['5/8',1],
    ['CTW',1],
    ['CTW',2],
    ['CTW',3],
    ['FLB',1]
  ]
]

const benchsize = [
  5,4,7,4
]

// Reactive variable for member count
const teams = computed(() => {
  let t = liveDraftData.value?.members ?? [];
  //sort into draft order
  t.sort((a, b) => liveDraftOrder.value?.default?.findIndex(team => team.user_team_id === a.id) - liveDraftOrder.value?.default?.findIndex(team => team.user_team_id === b.id));
  return t;
});
const memberCount = computed(() => {
  return liveDraftData.value?.league?.member_count ?? null;
});
const draftPick = computed(() => {
  return Math.floor((liveDraftData.value?.league?.draft_round - 1) * (memberCount.value ?? 1)) + liveDraftData.value?.league?.current_turn;
});
const draftPickTeam = computed(() => {
  return liveDraftOrder.value?.default?.find(team => team.pick === draftPick.value)?.user_team_id;
});

const squad = computed(() => {
  let t = teamshape[liveDraftData.value?.league?.options?.field_layout] || [];
  //flex
  if (liveDraftData.value?.league?.options?.flex_pos) {
    t = t.concat([['FLX',1]]);
  }
  //bench
  let bsize = benchsize[liveDraftData.value?.league?.options?.field_layout] || 0;
  let b = [...Array(bsize)].map((_,i) => ['BENCH',i+1])
  t = t.concat(b);
  return t;
});

function player(player_id) {
  if (!liveDraftPlayers.value || !player_id) return null;
  return liveDraftPlayers.value.find(p => p.id == player_id);
}

function team(team_id) {
  if (!teams.value || !team_id) return null;
  return teams.value.find(t => t.id == team_id);
}

const lastDraftPlayer = computed(() => {
  const lastPick = liveDraftRecap.value?.[0];
  return player(lastPick?.player_id);
});
const lastDraftTeam = computed(() => {
  const lastPick = liveDraftRecap.value?.[0];
  return team(lastPick?.user_team_id);
});

watch([year, leagueId, userTeamId, auth], () => {
  initialfetchRequired.value = true;
  setCookie('year', year.value);
  setCookie('leagueId', leagueId.value);
  setCookie('userTeamId', userTeamId.value);
  setCookie('auth', auth.value);
});

function buildUrl() {
  return `https://www.supercoach.com.au/${year.value}/api/nrl/draft/v1/leagues/${leagueId.value}/userteam/${userTeamId.value}/livedraft`;
}

async function initialData() {
  genericFetch(buildUrl() + 'Order', liveDraftOrder);
  genericFetch('https://www.supercoach.com.au/2026/api/nrl/draft/v1/players-cf?embed=positions', liveDraftPlayers);
}


async function fetchLiveDraft() {
  fetchStatus.value = 'fetching'
  if (initialfetchRequired.value) {
    initialData();
  }
  await genericFetch(buildUrl(), liveDraftData);
  console.log('Fetched live draft data:', liveDraftData.value);
  
  await genericFetch(`https://www.supercoach.com.au/${year.value}/api/nrl/draft/v1/leagues/${leagueId.value}/recap`, liveDraftRecap);

  liveDraftRecap.value = liveDraftRecap.value.reverse(); // Show most recent picks first
  
  if (liveDraftData.value && !liveDraftData.value.error) {
    fetchStatus.value = 'ok';
  } else {
    fetchStatus.value = 'error';
  }
}

async function genericFetch(url, dataRef) {
  if (!auth.value) {
    dataRef.value = { error: 'No auth token provided' };
    return;   
  }
  try {
    const response = await fetch(url, {
      headers: {
        'Authorization': `Bearer ${auth.value}`
      }
    });
    if (!response.ok) throw new Error('Network response was not ok');
    let json = await response.json();
    dataRef.value = json;
  } catch (error) {
    dataRef.value = {error: error.message};
  }
}

function startInterval() {
  if (intervalId) clearInterval(intervalId);
  if (isFetching.value) {
    fetchLiveDraft();
    intervalId = setInterval(fetchLiveDraft, fetchInterval.value * 1000);
  }
}

function stopInterval() {
  if (intervalId) clearInterval(intervalId);
  intervalId = null;
  fetchStatus.value = 'paused'
}

function toggleFetching() {
  isFetching.value = !isFetching.value;
  if (isFetching.value) {
    startInterval();
  } else {
    stopInterval();
  }
}



onMounted(() => {
  startInterval();
});

onUnmounted(() => {
  stopInterval();
});

if (isFetching.value) {
  startInterval();
}

loadFromCookies();
</script>

<template>
  <div class="app-options" v-if="options">
    <h1>Live Draft</h1>

    <form @submit.prevent style="margin: 1em 0;">
      <label>
        Year:
        <input v-model="year" />
      </label>
      <label>
        League ID:
        <input v-model="leagueId" />
      </label>
      <label>
        User Team ID:
        <input v-model="userTeamId" />
      </label>
      <label>
        Auth Token:
        <input v-model="auth" />
      </label>
      <label>
        Fetch Interval (seconds):
        <input v-model="fetchInterval" type="number" />
      </label>
    </form>
    <p><strong>URL:</strong> {{ buildUrl() }}</p>
    <p><strong>Status:</strong> {{ isFetching ? 'Fetching' : 'Paused' }}</p>
    <p>{{ fetchStatus }}</p>

    membercount: {{ memberCount }}
    draftpick: {{ draftPick }} (draftPickTeam: {{ draftPickTeam }})
  </div>
  <div class="statusbar">
    <div>{{ liveDraftData?.league?.name }}</div>
    <div v-if="draftPick > 0">pick {{ draftPick }}</div>
    <div v-if="lastDraftPlayer">last pick: {{ lastDraftPlayer?.first_name }} {{ lastDraftPlayer?.last_name }} ({{ lastDraftTeam?.teamname }})</div>
    <div class="controls">
      <button @click="() => options = !options" title="Toggle options panel">
        🛠
      </button>
      <button @click="() => debug = !debug" title="Toggle debug panel">
        🪲
      </button>
      <button @click="toggleFetching">
        {{ isFetching ? '⏸' : '▶' }}
      </button>
      <div class="status" :class="fetchStatus" :title="fetchStatus"></div>
    </div>
  </div>
 <div v-if="liveDraftData" class="teams-grid">
  <div v-for="(team, index) in teams" :key="index" class="team-container" :class="{ 'current-pick': team.id === draftPickTeam }">
    <div class="team-header" :data-team-id="team.id">
      <div class="name">{{ team.teamname }}</div>
      <div class="user">{{ team.user.first_name }}</div>
    </div>
    <div class="team-players">
      <div v-for="(pos,i) in squad.filter(p => p[0] !== 'BENCH')" :key="i">
        <div v-if="i == 0 || squad[i-1][0] !== pos[0]" class="position-header">
          {{ pos[0] }}
        </div>
        <Player
          :player_id="team.players.filter(p => p.position === pos[0] && p.picked == 'true')?.[pos[1]-1]?.player_id || null"
          :players="liveDraftPlayers"
          :position="pos[0]"
          :recently-picked="(liveDraftRecap || []).findIndex(pick => pick.player_id === team.players.filter(p => p.position === pos[0] && p.picked == 'true')?.[pos[1]-1]?.player_id)"
        />
      </div>
      <div class="position-header">
        BENCH
      </div>
      <div v-for="(pos,i) in squad.filter(p => p[0] == 'BENCH')" :key="i" class="player bench">
        <Player :player_id="team?.players?.filter(p => p.picked == 'false')?.[i]?.player_id || null" :players="liveDraftPlayers" position="bench" :recently-picked="(liveDraftRecap || []).findIndex(pick => pick.player_id === team?.players?.filter(p => p.picked == 'false')?.[i]?.player_id)" />
      </div>
    </div>
  </div> 
</div> 

<div class="debug-panel" v-if="debug">
  <div>
    <h5>Draft Data</h5>
    <pre v-if="liveDraftData">{{ JSON.stringify(liveDraftData, null, 2) }}</pre>
    <p v-else>Loading or error...</p>
  </div>

  <div>
    <h5>Draft Recap</h5>
    <pre v-if="liveDraftRecap">{{ JSON.stringify(liveDraftRecap, null, 2) }}</pre>
    <p v-else>Loading or error...</p>
  </div>

  <div>
    <h5>Draft Order</h5>
    <pre v-if="liveDraftOrder">{{ JSON.stringify(liveDraftOrder, null, 2) }}</pre>
    <p v-else>Loading or error...</p>
  </div>

  <div>
    <h5>Players</h5>
    <pre v-if="liveDraftPlayers">{{ JSON.stringify(liveDraftPlayers, null, 2) }}</pre>
    <p v-else>Loading or error...</p>
  </div>
</div>

</template>


<style scoped>

.app-options {
  padding:2em;

  label {
    display: block;
    margin-bottom: 0.5em;
  }
}

.statusbar {
  background: #ddd;
  margin-bottom: 0em;
  padding: 0.2em 0.5em;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1em;
  font-weight: bold;

  .controls {
    display: flex;
    gap: 0.5em;
    align-items: center;
    button { margin:0 !important; }
  }
  .status {
    height: 18px;
    width: 18px;
    border-radius: 50%;
    &.fetching {
      background: green;
    }
    &.ok {
      background: limegreen;
    }
    &.error {
      background: red;
    }
    &.paused {
      background: orange;
    }
  }
}

.teams-grid {
  display: flex;
  gap: 0.5em;
  padding:0.5em;

  .team-container {
    border: 1px solid gray;
    border-radius: 1em;
    flex: 1;
    background-image: linear-gradient(to bottom,#b7e66e,#5bacb1);
  
    &.current-pick {
      border-color: red;
      box-shadow: 0 0 10px red;
    }

    .team-header {
      border-radius: 1em 1em 0 0;
      color: wheat;
      background: black;
      padding: 3px;
      text-align: center;
      white-space: nowrap;

      .name {
        font-size: 1.1em;
        font-weight: bold;
      }
      .user {
        font-size: 0.9em;
        font-style: italic;
      }
    }

    .team-players {
      padding: 5px;
    }

    .position-header {
      background: green;
      text-align: center;
      color: white;
      width: 5rem;
      margin: 5px auto 0;
      font-size: 1em;
      border-radius: 10px 10px 0 0;
      font-weight: bold;
    }

  }
}

.app-panel, .teams-grid {
  font-family: system-ui, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  font-size:0.8em;
}

.debug-panel {
  display: flex;

  > * {
    flex: 1 1 0;
    min-width: 0;
    width: 25%;
    margin: 0.5em;
    padding: 0.5em;
    background: #FAF4DD;
    border: 1px solid gray;
    box-sizing: border-box;
    overflow-x: auto;
  }
}
</style>
