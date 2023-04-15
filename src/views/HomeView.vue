<script setup>

import { useRoute } from 'vue-router'
import { onMounted, ref } from 'vue'
import axios from 'axios'

const route = useRoute()
const idPartie = ref()
const idPlayer = ref()
const partie = ref()
const mot = ref('')
const nbmots = ref('')

onMounted(() => {
  idPartie.value = route.params.idgame
  idPlayer.value = route.params.idplayer

  setInterval(async () => {
    partie.value = await fetchApi()
  }, 1500
)

})


async function fetchApi() {
  let response = await axios.get(`http://localhost:8088/sae401/index.php/api/parties/${idPartie.value}`)
  console.log(response.data)
  return response.data
}
async function ajouter(){
  await axios.post('http://localhost:8088/sae401/index.php/api/indices', {
    partie: '/api/parties/' + idPartie.value,
    mot: mot.value,
    nbmots: nbmots.value,
    user: '/api/users/' + idPlayer.value,
    
    
  }
  
  )
  console.log(ajouter)
}

</script>


<template>
    <div class="contenu">
      <div v-if="partie" class="parent1 grille">
    <div v-for="mot in partie.motparties" :key="mot.id">
      <div v-if="idPlayer == 1" class="petitegrille" :class="mot.couleurJ1">
      </div>
      <div v-else class="petitegrille" :class="mot.couleurJ2">
      </div>

    </div>
        </div>
    <div class="jeu">
        <div class="statut">
            <p>
                action en cours
            </p>
        </div>
    <div v-if="partie" class="parent">
      <div v-if="idPlayer == 1 && partie.quijoue == '2'" class="notyourturn"></div>
      <div v-if="idPlayer == 2 && partie.quijoue == '1'" class="notyourturn"></div>
    <div  v-for="mot in partie.motparties" @click="cartetrigger(index)" :key="mot.id">
      <button v-if="idPlayer == 1 || partie.etat == 'decouvert' || partie.etat == 'jeton' " :class="[mot.couleurJ1] ">
            <div class="card" >
                <div class="hautcard">
                    <div class="left"></div>
                    <div class="right">
                        <img src="1\src\assets\imgcard.png" alt="">
                    </div>
                </div>
                <div class="bascard">
                    <p>{{ mot.mot.fr }}</p>
                </div>                
            </div>
        </button>


        <button v-if="idPlayer == 2 || partie.etat == 'decouvert' || partie.etat == 'jeton' " :class="[mot.couleurJ2] ">
            <div class="card" >
                <div class="hautcard">
                    <div class="left"></div>
                    <div class="right">
                        <img src="1\src\assets\imgcard.png" alt="">
                    </div>
                </div>
                <div class="bascard">
                    <p>{{ mot.mot.fr }}</p>
                </div>                
            </div>
        </button>

 </div>

        </div>
        
        
        <div>

    <div v-if="partie">
    <div v-if="idPlayer == 1 && partie.quijoue == '1'" class="notyourturn1" >
      <h2>Ajouter un indice</h2>
      <div>
        <label for="nom">Nom :</label>
        <input type="text" id="mot" v-model="mot" />
      </div>
      <div>
        <label for="valeur">Valeur :</label>
        <input type="text" id="nbmots" v-model="nbmots" />
      </div>
      <input type="hidden" name="valeur" value="2" />
      <button @click="ajouter">Ajouter</button>
    </div>
    <div v-if="idPlayer == 2 && partie.quijoue == '2'" class="notyourturn1">
      <h2>Ajouter un indice</h2>
      <div>
        <label for="nom">Nom :</label>
        <input type="text" id="mot" v-model="mot" />
      </div>
      <div>
        <label for="valeur">Valeur :</label>
        <input type="text" id="nbmots" v-model="nbmots" />
      </div>
      <button @click="ajouter">Ajouter</button>
    </div>
  </div>

  <div v-if="partie">
    <form v-if="idPlayer == 1 && partie.quijoue == '2'" class="notyourturn1">
     indice nbindice
    </form>
    <form v-if="idPlayer == 2 && partie.quijoue == '1'" class="notyourturn1">
     indice nbindice
    </form>
  </div>
  </div>
    </div>
    <div v-if="partie" class="droit">
        <div class="side">
            <div class="theirside">
              <p>{{ partie.user2.pseudo }}</p>
            </div>
            <div class="nbcard">
                <div class="theircard">
                    <p>10</p>
                </div>
                <div class="yourcard">
                    <p>9</p>
                </div>
            </div>
            <div class="yourside">
              <p>{{ partie.user1.pseudo }}</p>
                <p>pseudo</p>
            </div>
        </div>
    <div class="chat">
        <div class="chat-container">
            <div class="chat-body">
                <div class="title">
                    game log
                </div>
                <div class="boxmessage">
              <div v-for="indice in partie.indices" :key="indice.id" class="chat-message">

              <p><span class="pseudo">{{indice.user.pseudo}}</span> <span class="action">action</span> <span class="indice">{{indice.mot}}</span> <span class="nbindice">{{indice.nbmots}}</span></p>
            </div>
            </div>
            </div>
          </div>
          
    </div>
</div>

</div>

</template>

<style scoped>
.notyourturn {
  position: absolute;
    background-color: #0000004d;
    width: 100%;
    height: 100%;
    pointer-events: none;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
.parent1 {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    grid-template-rows: repeat(5, 1fr);
      }
.petitegrille {
  height: 2rem;
  width: 3rem;
  border: solid 1px black;
}
.parent {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    grid-template-rows: repeat(5, 1fr);
    grid-column-gap: 10px;
    grid-row-gap: 10px;
    border: solid black 1px;
    padding: 1rem;
    position: relative;
    }
.parent > div > button { 
  width: 7rem;
    height: 5rem;
    border: solid rgb(255, 255, 255) 1px;
    background-color: #f7e0c4;
    border-radius: 10px;
    padding: 5%;
    
}

.jeu {
  display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
}
.card {
  background-color: #f6d9b6;
  border:#9d8064 solid 2px;
  border-radius: 10px;
    display: flex;
flex-direction: column;
height: 100%;
  }
input[type="number"] {
    width: 3rem;}
form {
    margin-top: 1rem;
}

.side {
    background-color: green;
    border: 1px solid darkgreen;
    border-radius: 10px;
    padding: 0.5rem;
    margin-bottom: 1rem;
}

    .chat-container {
        width: 15rem;
        height: 25rem;
        background-color: #ffffff;
        border-radius: 10px;
        overflow: hidden;
        border: black solid 1px;
      }
      
      .chat-body {
        height: 100%;
        padding: 10px;
        box-sizing: border-box;
        overflow-y: auto;
      }
      
      .chat-message {
        background-color: #028500;
        border-radius: 5px;
        padding: 10px;
        margin-bottom: 10px;
      }

      .contenu {
        display: flex;
        justify-content: space-around;
        align-items: center;
      }
      .title {
        border-bottom: black solid 1px;
        margin-bottom: 0.5rem;
        text-align: center;
        
      }
      .pseudo {
        font-weight: bold;
      }
      .hautcard {
        height: 50%;
        display: flex;
        gap: 5%;
        padding-left: 5%;
      }
      .left {
        height: 100%;
    width: 70%;
    border-bottom: solid 2px black;
    opacity: 30%;
      }
      .right {
        height: 100%;
      }
      .right > img {
        height: 100%;
        width: 100%;
        padding-top: 0.2rem;
        opacity: 40%;
        padding-right: 0.2rem;
      }
      .bascard {
        height: 50%;
        padding: 3%;

      }
      .bascard > p {
        background-color: white;
    padding: 0%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 0px 0px 5px 5px;
    text-transform: uppercase;
    font-weight: 900;
      }

      .statut {
        width: 12rem;
    padding: 0.5rem;
    border: black 1px solid;
      }
      .statut > p {
        text-align: center;
      }

.Vert {
    background-color:  #c3e1b3!important;
}
.Vert > .card {
  background-color: #49a818!important;
  border: #439e17 solid 2px!important;
}
.Noir {
  background-color: #757575!important;
}
.Noir > .card {
background-color: #525151;
border: #343434 solid 2px;
}

.Neutre {
  background-color: #f7e0c4;
}
</style>