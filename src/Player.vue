<script setup>
import { computed } from 'vue';
const props = defineProps({
  player_id: [String, Number],
  players: Array,
  position: String,
  recentlyPicked: Number
});

const player = computed(() => {
  if (!props.players || !props.player_id) return null;
  return props.players.find(p => p.id == props.player_id);
});
</script>

<template>
  <div v-if="player" :class="`player recently-picked-${recentlyPicked}`">
    <div class="name">{{ player.first_name[0] }} {{ player.last_name }}</div>
    <div class="info">
        <span>{{ player.positions.filter(p => p.position !== props.position).map(p => p.position).join('&thinsp;|&thinsp;') }}</span>
        <span class="team" :class="player.team.abbrev">{{ player.team.abbrev }}</span>
    </div>
  </div>
  <div v-else class="player empty">
    <div class="name">&nbsp;</div>
  </div>
</template>

<style scoped>
.player {
  margin: 0 auto 3px;
  background: #ffffffd3;
  height: 42px;
  width: 120px;
  border-radius: 8px;
  overflow: clip;
  box-shadow: 0 2px 5px #00000055;

  &.empty {
    background: #ddd;
    border-color: transparent;
  }

  &.recently-picked-0 {
    background: #FFCC00;
  }
  &.recently-picked-1 {
    background: #FED11C;
  }
  &.recently-picked-2 {
    background: #FED637;
  }
  &.recently-picked-3 {
    background: #FDDB53;
  }
  &.recently-picked-4 {
    background: #FCE06F;
  }
  &.recently-picked-5 {
    background: #FCE58A;
  }
  &.recently-picked-6 {
    background: #FBEAA6;
  }
  &.recently-picked-7 {
    background: #FAEFC2;
  }
  &.recently-picked-8 {
    background: #FAF4DD;
  }
}

.name {
    font-size: 1.1em;
    line-height: 1em;
    padding: 4px;
    font-weight: bold;
    white-space: nowrap;
    overflow: hidden;

}

.info {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 0.9em;
    padding-left: 4px;

    .team { font-weight: bold;}
}

.team {

    width: 48px;
    text-align: left;
    padding-left: 8px;
    height: 20px;
    line-height: 20px;
    display: inline-block;

    &.CBR {
        background: linear-gradient(135deg, limegreen 0%, 
            limegreen 70%,
            yellow 70%,
            yellow 80%,
            blue 80%,
            blue 100%);    
    }

    &.SHA {
        background: linear-gradient(135deg, skyblue 0%, 
            skyblue 70%,
            white 70%,
            white 80%,
            black 80%,
            black 100%);          
    }

    &.SYD {
        color:white;
        background: linear-gradient(135deg, #00305e 0%, 
            #00305e 70%,
            white 70%,
            white 80%,
            #e82c2e 80%,
            #e82c2e 100%);               
    } 

    &.MEL {
        color:white;
        background: linear-gradient(135deg, #632390 0%, 
            #632390 70%,
            gold 70%,
            gold 80%,
            #00305e 80%,
            #00305e 100%);        
    }

    &.BRO {
        color:white;
        background: linear-gradient(135deg, maroon 0%, 
            maroon 70%,
            gold 70%,
            gold 80%,
            maroon 80%,
            maroon 100%);           
    }

    &.PTH {
        color:white;
        background: linear-gradient(135deg, black 0%, 
            black 70%,
            red 70%,
            red 75%,
            yellow 75%,
            yellow 80%,
            green 80%,
            green 85%,
            black 85%,
            black 100%);
    }

    &.MNL {
        color:white;
        background: linear-gradient(135deg, maroon 0%, 
            maroon 70%,
            white 70%,
            white 80%,
            maroon 80%,
            maroon 100%);   
    }

    &.WST {
        background: linear-gradient(135deg, #f68b1f 0%, 
            #f68b1f 70%,
            white 70%,
            white 80%,
            black 80%,
            black 100%);        
    }

    &.NEW {
        color:white;
        background: linear-gradient(135deg, #00539f 0%, 
            #00539f 70%,
            #ee3524 70%,
            #ee3524 80%,
            #00539f 80%,
            #00539f 100%);
    }

    &.GCT {
        color:white;
        background: linear-gradient(135deg, #007ac1 0%, 
            #007ac1 70%,
            #eee 70%,
            #eee 80%,
            gold 80%,
            gold 100%);        
    }

    &.BUL {
        color:white;
        background: linear-gradient(135deg, blue 0%, 
            blue 70%,
            white 70%,
            white 80%,
            blue 80%,
            blue 100%);          
    }

    &.STG {
        background: linear-gradient(135deg, white 0%, 
            white 70%,
            red 70%,
            red 80%,
            white 80%,
            white 100%);         
    }

    &.NQC {
        color:white;
        background: linear-gradient(135deg, #002b5c 0%, 
            #002b5c 70%,
            #ffdd00 70%,
            #ffdd00 80%,
            #002b5c 80%,
            #002b5c 100%);          
    }

    &.NZL {
        color:white;
        background: linear-gradient(135deg, #262C68 0%, 
            #262C68 70%,
            white 70%,
            white 80%,
            #008445 80%,
            #008445 100%);         
    }

    &.DOL {
        color:white;
        background: linear-gradient(135deg, red 0%, 
            red 70%,
            white 70%,
            white 80%,
            rgb(238, 204, 140) 80%,
            rgb(238, 204, 140) 100%);          
    }

    &.PAR {
        background: linear-gradient(135deg, gold 0%, 
            gold 70%,
            #003087 70%,
            #003087 80%,
            gold 80%,
            gold 100%);            
    }

    &.STH {
        color:white;
        background: linear-gradient(135deg, green 0%, 
            green 70%,
            red 70%,
            red 80%,
            green 80%,
            green 100%);         
    }
}
</style>
