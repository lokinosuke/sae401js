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

// Define computed properties or methods here
const getuserun = () => {
  return '/api/users/' + partie.value.user1.id;
}

const getuserdeux = () => {
  return '/api/users/' + partie.value.user2.id;
}
async function gameover() {
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      resultat : "terminé"
    })
    if (partie.value.cartej1 < partie.value.cartej2) {
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
        victoire : parseInt(partie.value.user1.victoire) + 1,
        trophee : parseInt(partie.value.user1.trophee) + 6
      })
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
        defaite : parseInt(partie.value.user2.defaite) + 1,
        trophee : parseInt(partie.value.user2.trophee) + 6
      })
    }
    else {
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
        victoire : parseInt(partie.value.user2.victoire) + 1,
        trophee : parseInt(partie.value.user2.trophee) + 6
      })
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
        defaite : parseInt(partie.value.user1.defaite) + 1,
        trophee : parseInt(partie.value.user1.trophee) + 3
      })
    }
    if (partie.value.cartej1 == partie.value.cartej2) {
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
        defaite : parseInt(partie.value.user1.defaite) + 1,
        trophee : parseInt(partie.value.user1.trophee) - 3
      })
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
        defaite : parseInt(partie.value.user2.defaite) + 1,
        trophee : parseInt(partie.value.user2.trophee) - 3
      })
    }
  }
async function fetchApi() {
  let response = await axios.get(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`)
  console.log(response.data)
  console.log(idPlayer)
  return response.data
}
async function ajouter(){
  console.log('ajouter')
  await axios.post('http://mmi21h01.sae401.ovh/api/indices', {
    partie: '/api/parties/' + idPartie.value, 
    mot: mot.value,
    nbmots: nbmots.value,
    action: 'donne',
    user: idPlayer.value == 1 ? getuserun() : getuserdeux(), 
  })
  if (idPlayer.value == 1) {
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      quijoue: 2,
      quidonne: 0
    })
  } else {
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      quijoue: 1,
      quidonne: 0
    })
  }
}
async function endguessing(){
  if (partie.value.tour < 8) {
  if (idPlayer.value == 1) {
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      quidonne: 1,
      quijoue: 0,
      tour: parseInt(partie.value.tour) + 1
    })
  } else {
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      quijoue: 0,
      quidonne: 2,
      tour: parseInt(partie.value.tour) + 1
      
    })
  }
} else {
  gameover()
  await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
    tour: parseInt(partie.value.tour) + 1
  })
   }
}


async function cartetrigger(index) {
    if (idPlayer.value == 1 && (partie.value.quijoue == '1' || partie.value.quijoue == '0') && partie.value.tour < 8) {
      if (partie.value.motparties[index].couleurJ2 == 'Vert') {
    console.log('good pour le J1 :)');
    partie.value.motparties[index].etat = 'decouvert1';
    await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
      etat: 'decouvert1'
      });
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      cartej1 : parseInt(partie.value.cartej1) - 1,
      cartetotal : parseInt(partie.value.cartetotal) - 1,
    })
    }

  if (partie.value.motparties[index].couleurJ2 == 'Neutre') {
    console.log('good pour le J1 :)')
    partie.value.motparties[index].etat = 'jeton1';
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      quijoue: 0,
      quidonne: 1,
      jeton: parseInt(partie.value.jeton) - 1,
      tour: parseInt(partie.value.tour) + 1
    }, {
      headers: {
        'Content-Type': 'application/ld+json'
      }
    });
    await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
      etat: 'jeton1'
    })
    }

    if (partie.value.motparties[index].couleurJ2 == 'Noir') {
    console.log('good pour le J1 :)');
    partie.value.motparties[index].etat = 'perdu';
    await axios.patch(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
      etat: 'perdu'
    });
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      resultat : "terminé",
    })
    await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
      victoire : parseInt(partie.value.user2.victoire) + 1,
      trophee : parseInt(partie.value.user2.trophee) + 12,
    })
    await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
      defaite : parseInt(partie.value.user1.defaite) + 1,
      trophee : parseInt(partie.value.user1.trophee) - 6,
    })
    }
  }
  if (idPlayer.value == 2 && partie.value.quijoue == '2' && partie.value.tour < 8) {
    if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
  await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
    cartej2 : parseInt(partie.value.cartej2) - 1,
    })}

    if (partie.value.motparties[index].couleurJ1 == 'Noir') {
    console.log('good pour le J2 :)');
    partie.value.motparties[index].etat = 'perdu';
    await axios.patch(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
      etat: 'perdu'
    });
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      resultat : "terminé"
    })
    await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
      victoire : parseInt(partie.value.user1.victoire) + 1,
      trophee : parseInt(partie.value.user1.trophee) + 12,
    })
    await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
      defaite : parseInt(partie.value.user2.defaite) + 1,
      trophee : parseInt(partie.value.user2.trophee) - 6
    })
    }
    if (partie.value.motparties[index].couleurJ1 == 'Vert') {
    console.log('good pour le J2 :)');
    partie.value.motparties[index].etat = 'decouvert2';
    await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
      etat: 'decouvert2'
    })
      await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      cartej2 : parseInt(partie.value.cartej2) - 1,
      cartej1 : parseInt(partie.value.cartej1) - 1,
      cartetotal : parseInt(partie.value.cartetotal) - 1,
    })
    }
    if (partie.value.motparties[index].couleurJ1 == 'Neutre') {
    console.log('good pour le J2 :)');
    partie.value.motparties[index].etat = 'jeton2';
    await axios.patch(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
      etat: 'jeton2'
    }, {
      headers: {
        'Content-Type': 'application/merge-patch+json'
      }
    });
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
    quijoue: 0,
    quidonne: 2,
    jeton: parseInt(partie.value.jeton) - 1,
    tour: parseInt(partie.value.tour) + 1
    })

    }
}
if (idPlayer.value == 1 && partie.value.tour == 8) {
  if (partie.value.motparties[index].couleurJ1 == 'Vert') {
  await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
    cartej1 : parseInt(partie.value.cartej2) - 1,
    });
}
if (partie.value.motparties[index].couleurJ1 == 'Neutre') {
  await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
    jeton : parseInt(partie.value.jeton) - 1,
    });
  await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
    tour: parseInt(partie.value.tour) + 1,
    statut: 'terminé',
    resultat: 'perdu'
    });
  }}}


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

        <div v-if='partie' class="statut">
          <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour < 7">
      <p>{{ partie.user2.pseudo }} est en train de donner un indice</p> 
    </div>

    <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour < 8">
      <p>Vous êtes en train de donner un indice</p> 
    </div>
    <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour < 8">
      <p>Vous êtes en train de donner un indice</p> 
    </div>
    <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour < 8">
      <p>{{ partie.user1.pseudo }} est en train de donner un indice</p> 
    </div>
    <div v-if="idPlayer == 2 && partie.quijoue =='1' && partie.quidonne =='0' && partie.tour < 8">
    <p>{{ partie.user1.pseudo }} est en train de deviner</p>
    </div>
    <div v-if="idPlayer == 1 && partie.quijoue =='1' && partie.quidonne =='0' && partie.tour < 8">
    <p>Vous êtes en train de deviner</p>
    </div>
    <div v-if="idPlayer == 2 && partie.quijoue =='2' && partie.quidonne =='0' && partie.tour < 8">
    <p>Vous êtes en train de deviner</p>
    </div>
    <div v-if="idPlayer == 1 && partie.quijoue =='2' && partie.quidonne =='0' && partie.tour < 8">
    <p>{{ partie.user2.pseudo }} est en train de deviner</p>
    </div>


    <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour == 8">
    <p>Vous êtes en mort subite</p>
    </div>
    <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour == 8">
    <p>Vous êtes en mort subite</p>
    </div>
    <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour == 8">
    <p>Vous êtes en mort subite</p>
    </div>
    <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour == 8">
    <p>Vous êtes en mort subite</p>
    </div>
        </div>
    <div v-if="partie" class="parent">
      <div v-if="partie.resultat == 'terminé'">
    finito
      </div>
      <div v-if="partie.tour > 8" class="gameover"> 
        <h3>Game Over</H3>
      <div>
          <div>
            <img :src="'/src/assets/' + partie.user1.avatar" alt="">
            <p>{{ partie.user1.pseudo }}</p>
          </div>
          <div>
            <img :src="'/src/assets/' + partie.user2.avatar"  alt="">
            <p>{{ partie.user2.pseudo }}</p>
          </div>
          
      </div>
      <a href="http://mmi21h01.sae401.ovh">quit</a>
      </div>
      <div v-if="idPlayer == 1 && (partie.quijoue == '2' || partie.quijoue == '0') && partie.tour < 8 || partie.tour == 9" class="notyourturn"></div>
      <div v-if="idPlayer == 2 && (partie.quijoue == '1' || partie.quijoue == '0') && partie.tour < 8 || partie.tour == 9" class="notyourturn"></div>
      
      <div v-for="(mot, index) in partie.motparties" @click="cartetrigger(index)" :key="mot.id">
        <button v-if="idPlayer == 1 || partie.etat == 'decouvert' || partie.etat == 'jeton' " :class="[mot.couleurJ1]">
          <div v-if="idPlayer == 1 && mot.etat == 'decouvert1'" class="decouvert1"></div>
          <div v-if="idPlayer == 1 && mot.etat == 'decouvert2'" class="decouvert2" style="transform: rotate(180deg);"></div>
          <div v-if="idPlayer == 1 && mot.etat == 'jeton2'" class="jeton2" style="transform: rotate(180deg);bottom: 0;right: 0px;"></div>
          <div v-if="idPlayer == 1 && mot.etat == 'jeton1'" class="jeton1"></div>

            <div class="card" >
                <div class="hautcard">
                    <div class="left"></div>

                </div>
                <div class="bascard">
                    <p>{{ mot.mot.fr }}</p>
                </div>                
            </div>
        </button>


        <button v-if="idPlayer == 2 || partie.etat == 'decouvert' || partie.etat == 'jeton' " :class="[mot.couleurJ2] ">
          <div v-if="idPlayer == 2 && mot.etat == 'decouvert1'" class="decouvert1" style="transform: rotate(180deg);"></div>
          <div v-if="idPlayer == 2 && mot.etat == 'decouvert2'" class="decouvert2"></div>
          <div v-if="idPlayer == 2 && mot.etat == 'jeton2'" class="jeton2"></div>
          <div v-if="idPlayer == 2 && mot.etat == 'jeton1'" class="jeton1" style="transform: rotate(180deg); bottom: 0;right: 0px;"></div>



            <div class="card" >
                <div class="hautcard">
                    <div class="left"></div>
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
    <div v-if="idPlayer == 1 && partie.quidonne == '1' && partie.tour < 8" class="notyourturn1 " >
      <h2>Ajouter un indice</h2>
      <div>
        <label for="nom">Nom :</label>
        <input type="text" id="mot" v-model="mot" />
      </div>
      <div>
        <label for="valeur">Valeur :</label>
        <input type="number" min="1" max="9" id="nbmots" v-model.number="nbmots" />
      </div>
      <input type="hidden" name="valeur" :value="getuserun()"/>
      <button @click="ajouter">Ajouter</button>
    </div>
    <div v-if="idPlayer == 2 && partie.quidonne == '2' && partie.tour < 8" class="notyourturn1">
      <h2>Ajouter un indice</h2>
      <div>
        <label for="nom">Nom :</label>
        <input type="text" id="mot" v-model="mot" />
      </div>
      <div>
        <label for="valeur">Valeur :</label>
        <input type="number" min="1" max="9" id="nbmots" v-model.number="nbmots" />
      </div>
      <button @click="ajouter">Ajouter</button>
    </div>
  </div>

  <div v-if="partie">

    <div v-if="idPlayer == 1 && partie.quijoue == '1'" class="notyourturn1">
     <div v-for="(indice, index) in partie.indices">
      <div v-if="index === partie.indices.length - 1" class="indice">
    <div class="motindice">{{indice.mot}}</div>
    <div class="nbindice">{{indice.nbmots}}</div>
  </div>
</div>
<button @click="endguessing">end guessing</button>
    </div>

    <div v-if="idPlayer == 2 && partie.quijoue == '2'" class="notyourturn1">
      <div v-for="(indice, index) in partie.indices" >
  <div v-if="index === partie.indices.length - 1" class="indice">
    <div class="motindice">{{indice.mot}}</div>
    <div class="nbindice">{{indice.nbmots}}</div>
  </div>
</div>
<button @click="endguessing">end guessing</button>

    </div>
  </div>
  </div>
    </div>
    <div v-if="partie" class="droit">
        <div class="side">
            <div class="theirside">
              <img :src="'/src/assets/' + partie.user2.avatar" class="profil" alt="">
              <p>{{ partie.user2.pseudo }}</p>
            </div>
            <div class="nbcard">
                <div class="carte">
                    <p>{{ partie.cartetotal }}</p>
                </div>
                <div class="jeton">
                    <p>{{ partie.jeton }}</p>
                </div>
            </div>
            <div class="yourside">
              <img :src="'/src/assets/' + partie.user1.avatar" class="profil" alt="" style="background-color: {{ partie.user1.couleur }};">
              <p>{{ partie.user1.pseudo }}</p>
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

              <p>{{indice.user.pseudo}} {{indice.action}} {{indice.mot}} {{indice.nbmots}}</p>
            </div>
            </div>
            </div>
          </div>
          
    </div>
</div>

</div>

</template>

<style scoped>
.gameover > div {
  display: flex;
  justify-content: space-around;
}
.gameover > div > div > img {
  height: 5rem;
}
.gameover > div > div > p {
  text-align: center;
}
.gameover {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 2;
    background-color: white;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 20rem;
    height: 20rem;
}
.indice {
  display: flex;
    gap: 1rem;
}
.indice > div {
  color: white;
    font-size: 24px;
    text-transform: uppercase;
}
.theirside, .yourside {
  display: flex;
    align-items: center;
}
.statut > div {
  background-color: white;
    width: fit-content;
}
.profil {
  height: 2rem;
}
.contenu {
  margin: 0;
    padding: 0;
    background-image: url(../assets/fond_1.svg);
    height: 100vh;
    background-repeat: no-repeat;
    background-size: cover;
}
.right {
  background-image: url(../assets/imgcard.png);
  height: 100%;
    width: 30%;
    background-size: contain;
    margin-right: 0.2rem;
    margin-top: 0.1rem;
    border-radius: 0% 30% 0% 0%;
}
.decouvert1 {
  pointer-events: none;
  position: absolute;
    width: 90%;
    background-size: contain;
    background-image: url(../assets/imthedanger.jpg) ;
    height: 85%;
}
.jeton1 {
  pointer-events: none;
  position: absolute;
    width: 40%;
    background-size: contain;
    background-image: url(../assets/jeton.png) ;
    height: 60%;
}
.jeton2 {
  pointer-events: none;
  position: absolute;
    width: 40%;
    background-size: contain;
    background-image: url(../assets/jeton.png) ;
    height: 60%;

}
.decouvert2 {
  pointer-events: none;
  position: absolute;
    width: 90%;
    background-size: contain;
    background-image: url(../assets/imthedanger.jpg) ;
    height: 85%;
}
.notyourturn {
  position: absolute;
    background-color: #0000004d;
    width: 100%;
    height: 100%;
    z-index: 1;
    pointer-events: none;
}
.notyourturn1 {
    display: flex;
    flex-direction: column;
    row-gap: 0.5rem;
}

.notyourturn1 > button {
  cursor: pointer;
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
    padding: 1rem;
    position: relative;
    backdrop-filter: blur(5px);
    border-radius: 10px;
    }
.parent > div > button { 
  width: 7rem;
    height: 5rem;
    border: solid #92268d 1px;
    background-color: #C932C2;
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
  background-color: #ffa9fb;
  border:#a82ba1 solid 2px;
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
    backdrop-filter: blur(5px);
    border-radius: 10px;
    padding: 0.5rem;
    margin-bottom: 1rem;
}

    .chat-container {
        width: 15rem;
        height: 25rem;
        backdrop-filter: blur(5px);
        border-radius: 10px;
        overflow: hidden;
      }
      
      .chat-body {
        height: 100%;
        padding: 10px;
        box-sizing: border-box;
        overflow-y: auto;
      }
      
      .chat-message {
        background-color: #292a2e;
        border-radius: 5px;
        padding: 10px;
        margin-bottom: 10px;
      }
.chat-message > p {
  color: white;
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
        color: white;
        
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
    width: 95%;
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
    border: black 1px solid;
    width: fit-content;
      }
      .statut > p {
        padding: 0.5rem;
        text-align: center;
      }

.Vert {
    background-color:  #ff8036!important;
    position: relative;
}
.Vert > .card {
  background-color: #FD5F02!important;
  border: #ff6309 solid 2px!important;
}
.Noir {
  background-color: #757575!important;
  position: relative;
}
.Noir > .card {
background-color: #525151;
border: #343434 solid 2px;
}

.Neutre {
  background-color: #C932C2;
  position: relative;
}
</style>