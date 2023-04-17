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
    await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
      tour: parseInt(partie.value.tour) + 1
    })
  }
}


async function cartetrigger(index) {
  if (idPlayer.value == 1 && (partie.value.quijoue == '1') && partie.value.tour < 9) {
    if (partie.value.cartetotal > 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
        })}}
    if (partie.value.cartetotal == 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
          statut: 'terminé',
          resultat : 'victoire'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}
    if (partie.value.cartetotal > 1) {
      if (partie.value.motparties[index].couleurJ2 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ1 == 'Neutre') {
        console.log('good pour le J1 :)');
        partie.value.motparties[index].etat = 'decouvert1';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
        })
      }}
    if (partie.value.cartetotal = 1) {
      if (partie.value.motparties[index].couleurJ2 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ1 == 'Neutre') {
        console.log('good pour le J1 :)');
        partie.value.motparties[index].etat = 'decouvert1';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
          statut: 'terminé',
          resultat : 'victoire'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}

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
      await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
        etat: 'perdu'
      });
      await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
        resultat : "défaite",
        statut : "terminé",
      })
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
        defaite : parseInt(partie.value.user2.victoire) + 1,
        trophee : parseInt(partie.value.user2.trophee) - 12,
      })
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
        defaite : parseInt(partie.value.user1.defaite) + 1,
        trophee : parseInt(partie.value.user1.trophee) - 12 ,
      })
    }
  }
  if (idPlayer.value == 2 && partie.value.quijoue == '2' && partie.value.tour < 9) {
    if (partie.value.cartetotal > 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
        })

      }}
    if (partie.value.cartetotal == 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
          statut : "terminé",
          resultat : "victoire"
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}


    if (partie.value.motparties[index].couleurJ1 == 'Noir') {
      console.log('good pour le J2 :)');
      partie.value.motparties[index].etat = 'perdu';
      await axios.patch(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
        etat: 'perdu'
      });
      await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
        resultat : "défaite",
        statut : "terminé",
      }, {
        headers: {
          'Content-Type': 'application/ld+json'
        }
      });
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
            defaite : parseInt(partie.value.user1.defaite) + 1,
            trophee : parseInt(partie.value.user1.trophee) - 12,
          },
          {

            headers: {
              'Content-Type': 'application/ld+json'
            }

          });
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
            defaite : parseInt(partie.value.user2.defaite) + 1,
            trophee : parseInt(partie.value.user2.trophee) - 12
          },
          {

            headers: {

              'Content-Type': 'application/ld+json'
            }
          });
    }
    if (partie.value.cartetotal > 1) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ2 == 'Neutre') {
        console.log('good pour le J2 :)');
        partie.value.motparties[index].etat = 'decouvert2';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
        })
      }}
    if (partie.value.cartetotal == 1) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ2 == 'Neutre') {
        console.log('good pour le J2 :)');
        partie.value.motparties[index].etat = 'decouvert2';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}
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
  if (idPlayer.value == 1 && partie.value.tour == 9) {
    if (partie.value.cartetotal > 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
        })}}
    if (partie.value.cartetotal == 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
          statut: 'terminé',
          resultat : 'victoire'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}
    if (partie.value.cartetotal > 1) {
      if (partie.value.motparties[index].couleurJ2 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ1 == 'Neutre') {
        console.log('good pour le J1 :)');
        partie.value.motparties[index].etat = 'decouvert1';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
        })
      }}
    if (partie.value.cartetotal = 1) {
      if (partie.value.motparties[index].couleurJ2 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ1 == 'Neutre') {
        console.log('good pour le J1 :)');
        partie.value.motparties[index].etat = 'decouvert1';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert1'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
          statut: 'terminé',
          resultat : 'victoire'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}
    if (partie.value.motparties[index].couleurJ2 == 'Neutre') {
      await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
        jeton : parseInt(partie.value.jeton) - 1,
      });
      await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
        tour: parseInt(partie.value.tour) + 1,
        statut: 'terminé',
        resultat: 'perdu'
      });
    }
  if (partie.value.motparties[index].couleurJ2 == 'Noir') {
await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
  etat: 'perdu' });
  await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
    statut: 'terminé',
    resultat: 'défaite'
  });
  await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
            defaite : parseInt(partie.value.user1.defaite) + 1,
            trophee : parseInt(partie.value.user1.trophee) - 12,
          },
          {

            headers: {
              'Content-Type': 'application/ld+json'
            }

          });
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
            defaite : parseInt(partie.value.user2.defaite) + 1,
            trophee : parseInt(partie.value.user2.trophee) - 12
          },
          {

            headers: {

              'Content-Type': 'application/ld+json'
            }
          });

  }
  }
  if (idPlayer.value == 2 && partie.value.tour == 9) {
    if (partie.value.cartetotal > 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
        })

      }}
    if (partie.value.cartetotal == 2) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].couleurJ2 == 'Vert') {
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        });
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej2 : parseInt(partie.value.cartej2) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 2,
          statut : "terminé",
          resultat : "victoire"
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}
      if (partie.value.cartetotal > 1) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ2 == 'Neutre') {
        console.log('good pour le J2 :)');
        partie.value.motparties[index].etat = 'decouvert2';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
        })
      }}
    if (partie.value.cartetotal == 1) {
      if (partie.value.motparties[index].couleurJ1 == 'Vert' && partie.value.motparties[index].etat == 'libre' && partie.value.motparties[index].couleurJ2 == 'Neutre') {
        console.log('good pour le J2 :)');
        partie.value.motparties[index].etat = 'decouvert2';
        await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
          etat: 'decouvert2'
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
          cartej1 : parseInt(partie.value.cartej1) - 1,
          cartetotal : parseInt(partie.value.cartetotal) - 1,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
        await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
          victoire : parseInt(partie.value.user2.victoire) + 1,
          trophee : parseInt(partie.value.user2.trophee) + 12,
        })
      }}
    if (partie.value.motparties[index].couleurJ1 == 'Neutre') {
      await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
        jeton : parseInt(partie.value.jeton) - 1,
      });
      await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
        tour: parseInt(partie.value.tour) + 1,
        statut: 'terminé',
        resultat: 'perdu'
      });
    }
  if (partie.value.motparties[index].couleurJ1 == 'Noir') {
await axios.put(`http://mmi21h01.sae401.ovh/api/motparties/${partie.value.motparties[index].id}`, {
  etat: 'perdu' });
  await axios.put(`http://mmi21h01.sae401.ovh/api/parties/${idPartie.value}`, {
    statut: 'terminé',
    resultat: 'défaite'
  });
  await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user1.id}`, {
            defaite : parseInt(partie.value.user1.defaite) + 1,
            trophee : parseInt(partie.value.user1.trophee) - 12,
          },
          {

            headers: {
              'Content-Type': 'application/ld+json'
            }

          });
      await axios.put(`http://mmi21h01.sae401.ovh/api/users/${partie.value.user2.id}`, {
            defaite : parseInt(partie.value.user2.defaite) + 1,
            trophee : parseInt(partie.value.user2.trophee) - 12
          },
          {

            headers: {

              'Content-Type': 'application/ld+json'
            }
          });

  }
  }}


</script>


<template>
  <div class="contenu">
    <div v-if="partie" class="parent1 grille ml-10">
      <div v-for="mot in partie.motparties" :key="mot.id">
        <div v-if="idPlayer == 1" class="petitegrille" :class="mot.couleurJ1">
        </div>
        <div v-else class="petitegrille" :class="mot.couleurJ2">
        </div>

      </div>
    </div>
    <div class="jeu h-full flex flex-col justify-between">

      <div v-if='partie' class="statut h-14 w-full blury rounded-b-xl p-3 grid place-content-center">
        <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour < 9">
          <p>{{ partie.user2.pseudo }} est en train de donner un indice</p>
        </div>

        <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour < 9">
          <p>Vous êtes en train de donner un indice</p>
        </div>
        <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour < 9 && partie.statut == 'en cours'">
          <p>Vous êtes en train de donner un indice</p>
        </div>
        <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour < 9 && partie.statut == 'en cours'">
          <p>{{ partie.user1.pseudo }} est en train de donner un indice</p>
        </div>
        <div v-if="idPlayer == 2 && partie.quijoue =='1' && partie.quidonne =='0' && partie.tour < 9">
          <p>{{ partie.user1.pseudo }} est en train de deviner</p>
        </div>
        <div v-if="idPlayer == 1 && partie.quijoue =='1' && partie.quidonne =='0' && partie.tour < 9 && partie.statut == 'en cours'">
          <p>Vous êtes en train de deviner</p>
        </div>
        <div v-if="idPlayer == 2 && partie.quijoue =='2' && partie.quidonne =='0' && partie.tour < 9 && partie.statut == 'en cours'">
          <p>Vous êtes en train de deviner</p>
        </div>
        <div v-if="idPlayer == 1 && partie.quijoue =='2' && partie.quidonne =='0' && partie.tour < 9 && partie.statut == 'en cours'">
          <p>{{ partie.user2.pseudo }} est en train de deviner</p>
        </div>


        <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour == 9">
          <p>Vous êtes en mort subite</p>
        </div>
        <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='1' && partie.tour == 9">
          <p>Vous êtes en mort subite</p>
        </div>
        <div v-if="idPlayer == 2 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour == 9">
          <p>Vous êtes en mort subite</p>
        </div>
        <div v-if="idPlayer == 1 && partie.quijoue =='0' && partie.quidonne =='2' && partie.tour == 9">
          <p>Vous êtes en mort subite</p>
        </div>
      </div>
      <div v-if="partie" class="parent blury">
        <div class="terminé" v-if="partie.statut == 'terminé' && partie.resultat == 'défaite'">
          <h2>Vous avez perdu</h2>
          <h3>Partie terminée</H3>
          <div>
            <div>
              <img :src="'/src/assets/' + partie.user1.avatar" alt="" :style=" {backgroundColor: partie.user1.couleur}">
              <p>{{ partie.user1.pseudo }}</p>
            </div>
            <div>
              <img :src="'/src/assets/' + partie.user2.avatar"  alt="" :style=" {backgroundColor: partie.user2.couleur}">
              <p>{{ partie.user2.pseudo }}</p>
            </div>

          </div>

        </div>
        <div class="terminé" v-if="partie.statut == 'terminé' && partie.resultat == 'victoire'">


          <div class="fixed top-0 left-0 right-0 z-50 w-full p-4 overflow-x-hidden overflow-y-auto md:inset-0 h-[calc(100%-1rem)] md:h-full">
            <div class="relative w-full h-full max-w-2xl md:h-auto ">
              <!-- Modal content -->
              <div class="relative rounded-lg shadow second-black  border-2 border-green-400/30 ">
                <!-- Modal header -->
                <div class="flex items-start justify-between p-4 border-b rounded-t border-zinc-700">
                  <p class="text-l font-medium px-6 py-1 bg-green-400/10 text-green-500 rounded">Victoire</p>
                </div>
                <!-- Modal body -->
                <div class="p-4 space-y-6">
                  <div class=" py-3 flex justify-between items-center">
                    <div class=" flex items-center rounded-l-xl justify-between">
                      <div class="flex items-center">
                        <div class="mr-3 w-16 h-16 rounded-xl bg-cover bg-center shadow-lg shadow-orange-600/10 "
                             :style="{backgroundImage: `url('/src/assets/${partie.user1.avatar}')`, backgroundColor: partie.user1.couleur}">
                        </div>
                        <div>
                          <p class="self-center text-xl font-semibold whitespace-nowrap leading-none">
                            {{  partie.user1.pseudo }}</p>
                          <p class=" leading-none mt-1 text-l text-zinc-500">{{  partie.user1.biographie }}</p>
                        </div>
                      </div>
                      <p
                          class="ml-6 text-2xl font-semibold flex items-center gap-2 text-gray-500 main-gradient-text leading-none">
                        {{  partie.user1.trophee }}
                        <i class="fa-sharp fa-solid fa-trophy fa-xs"></i>
                      </p>
                    </div>
                    <div class="flex items-center rounded-l-xl justify-between">
                      <p
                          class="mr-6 text-2xl font-semibold flex items-center gap-2 text-gray-500 main-gradient-text leading-none">
                        <i class="fa-sharp fa-solid fa-trophy fa-xs"></i>
                        {{ partie.user2.trophee }}
                      </p>
                      <div class="flex items-center">

                        <div>
                          <p class="self-center text-xl font-semibold whitespace-nowrap leading-none text-right">
                            {{ partie.user2.pseudo }}</p>
                          <p class=" leading-none mt-1 text-l text-zinc-500 text-right">{{ partie.user2.biographie }}</p>
                        </div>
                        <div class="ml-3 w-16 h-16 rounded-xl bg-cover bg-center shadow-lg shadow-orange-600/10 "
                             :style="{backgroundImage: `url('/src/assets/${partie.user2.avatar}')`, backgroundColor: partie.user2.couleur}">
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="w-full flex p-4 gap-4">
                  <div class="flex-1 aspect-video main-black rounded-lg grid place-content-center "><p class="text-xl text-center">{{ partie.nom }}</p><p class="text-zinc-500 text-center">Nom</p></div>
                  <div class="flex-1 aspect-video main-black rounded-lg grid place-content-center "><p class="text-3xl text-center">{{  partie.tour }}/9</p><p class="text-zinc-500 text-center">Tour</p></div>
                  <div class="flex-1 aspect-video main-black rounded-lg grid place-content-center"><p class="text-3xl text-center flex items-center gap-1 main-gradient-text">
                      
                    - {{ partie.user1.trophee }}
                    <i class="fa-sharp fa-solid fa-trophy fa-xs"></i></p><p class="text-zinc-500 text-center">Trophés</p></div>
                </div>
              </div>
            </div>
          </div>


          <h2>Vous avez gagné</h2>
          <h3>Partie terminée</H3>
          <div>
            <div>
              <img :src="'/src/assets/' + partie.user1.avatar" alt="" :style=" {backgroundColor: partie.user1.couleur}">
              <p>{{ partie.user1.pseudo }}</p>
              <p>+12</p>
            </div>
            <div>
              <img :src="'/src/assets/' + partie.user2.avatar"  alt="" :style=" {backgroundColor: partie.user2.couleur}">
              <p>{{ partie.user2.pseudo }}</p>
              <p>+12</p>
            </div>

          </div>

        </div>
        <div v-if="partie.tour > 9" class="gameover">
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
        <div v-if="idPlayer == 1 && (partie.quijoue == '2' || partie.quijoue == '0') && partie.tour < 9 || partie.tour == 10 || partie.statut == 'terminé'" class="notyourturn"></div>
        <div v-if="idPlayer == 2 && (partie.quijoue == '1' || partie.quijoue == '0') && partie.tour < 9 || partie.tour == 10 || partie.statut == 'terminé'" class="notyourturn"></div>

        <div v-for="(mot, index) in partie.motparties" @click="cartetrigger(index)" :key="mot.id" >
          <button v-if="idPlayer == 1 || partie.etat == 'jeton' " :class="{'Vert': mot.etat == 'decouvert1' || mot.etat == 'decouvert2'}" style="position: relative;">
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


          <button v-if="idPlayer == 2 || partie.etat == 'jeton' " :class="{'Vert': mot.etat == 'decouvert1' || mot.etat == 'decouvert2'}" style="position: relative;">
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


      <div class="eheoh h-32 w-full blury rounded-t-xl p-3">


        <div v-if="idPlayer == 1 && partie.quidonne == '1' && partie.tour < 8" class="notyourturn1 flex h-full justify-between" >
          <div class="h-full grid place-content-center">
            <h2 class="title text-xl  flex items-center justify-between text-zinc-300 text-center ">Ajouter un indice</h2>
          </div>
          <div class="flex gap-4 w-fit justify-end">
            <div class=" h-full quantity ">
              <input placeholder="voiture" type="text" id="mot" v-model="mot" class="minipl main-black w-full h-full main-black rounded-md" />
            </div>
            <div class="aspect-square h-full quantity">
              <input class="aspect-square main-black w-full h-full main-black rounded-md" type="number" :placeholder="1" min="1" max="9" id="nbmots" v-model.number="nbmots" />
            </div>
            <input type="hidden" name="valeur" :value="getuserun()"/>
            <button class="h-full aspect-square w-fit ease-in duration-100 text-white focus:ring-4 shadow-lg shadow-orange-600/20 font-medium rounded-md text-sm text-center main-gradient-b cursor-pointer" @click="ajouter">Ajouter</button>
          </div>
        </div>
        <div v-if="idPlayer == 2 && partie.quidonne == '2' && partie.tour < 8" class="notyourturn1 flex h-full justify-between">
          <div class="h-full grid place-content-center">
            <h2 class="title text-xl  flex items-center justify-between text-zinc-300 text-center ">Ajouter un indice</h2>
          </div>
          <div class="flex gap-4 w-fit justify-end">
            <div class=" h-full quantity ">
              <input placeholder="voiture" type="text" id="mot" v-model="mot" class="minipl main-black w-full h-full main-black rounded-md" />
            </div>
            <div class="aspect-square h-full quantity">
              <input class="aspect-square main-black w-full h-full main-black rounded-md" type="number" :placeholder="1" min="1" max="9" id="nbmots" v-model.number="nbmots" />
            </div>
            <input type="hidden" name="valeur" :value="getuserun()"/>
            <button class="h-full aspect-square w-fit ease-in duration-100 text-white focus:ring-4 shadow-lg shadow-orange-600/20 font-medium rounded-md text-sm text-center main-gradient-b cursor-pointer" @click="ajouter">Ajouter</button>
          </div>
        </div>


        <div v-if="partie" class="h-full">

          <div v-if="idPlayer == 1 && partie.quijoue == '1' && partie.statut == 'en cours'" class="notyourturn1  h-full flex justify-center gap-4">
            <div v-for="(indice, index) in partie.indices" class="h-full grid place-content-center">
              <div v-if="index === partie.indices.length - 1" class="indice">
                <div class="motindice">{{indice.mot}}</div>
                <div class="nbindice">{{indice.nbmots}}</div>
              </div>
            </div>
            <button class="h-full aspect-square w-fit ease-in duration-100 text-white focus:ring-4 shadow-lg shadow-orange-600/20 font-medium rounded-md text-sm text-center main-gradient-b cursor-pointer" @click="endguessing">Finir de deviner</button>
          </div>

          <div v-if="idPlayer == 2 && partie.quijoue == '2' && partie.statut == 'en cours'" class="notyourturn1 h-full flex justify-center gap-4">
            <div v-for="(indice, index) in partie.indices" class="h-full grid place-content-center">
              <div v-if="index === partie.indices.length - 1" class="indice">
                <div class="motindice">{{indice.mot}}</div>
                <div class="nbindice">{{indice.nbmots}}</div>
              </div>
            </div>
            <button class="h-full aspect-square w-fit ease-in duration-100 text-white focus:ring-4 shadow-lg shadow-orange-600/20 font-medium rounded-md text-sm text-center main-gradient-b cursor-pointer" @click="endguessing">Finir de deviner</button>

          </div>
        </div>
      </div>
    </div>
    <div v-if="partie" class="droit blury h-full rounded-l-xl px-3 py-3 w-80 overflow-hidden">
      <div class="side flex gap-4">
        <div class="theirside flex flex-col items-center gap-4 p-4 second-black rounded-xl flex-1">
          <div class="aspect-square w-20 rounded-xl bg-cover bg-center shadow-lg shadow-orange-600/10" :style="{backgroundImage: `url('/src/assets/${partie.user2.avatar}')`, backgroundColor: partie.user2.couleur}">
          </div>
          <p class="self-center text-xl font-semibold whitespace-nowrap leading-none text-zinc-300">{{ partie.user2.pseudo }}</p>
        </div>
        <div class="theirside flex flex-col items-center gap-4 p-4 second-black rounded-xl flex-1">
          <div class="aspect-square w-20 rounded-xl bg-cover bg-center shadow-lg shadow-orange-600/10" :style="{backgroundImage: `url('/src/assets/${partie.user1.avatar}')`, backgroundColor: partie.user1.couleur}">
          </div>
          <p class="self-center text-xl font-semibold whitespace-nowrap leading-none text-zinc-300">{{ partie.user1.pseudo }}</p>
        </div>
      </div>
      <div class="nbcard flex gap-2">
        <div class="tour flex flex-col items-center p-3 second-black rounded-lg flex-1 grid place-content-center">
          <p class="text-xl text-center"> {{ partie.tour }}/9</p>
          <p class="text-zinc-500 text-center">Tour</p>
        </div>
        <div class="carte flex flex-col items-center p-3 second-black rounded-lg flex-1 grid place-content-center">
          <p class="text-xl text-center flex gap-1 items-center">{{ partie.cartetotal }}<svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><path d="M20,4H4C2.897,4,2,4.897,2,6v12c0,1.103,0.897,2,2,2h16c1.103,0,2-0.897,2-2V6C22,4.897,21.103,4,20,4z M4,18V6h16 l0.001,12H4z"></path><path d="M6.5 11h3c.276 0 .5-.224.5-.5v-2C10 8.224 9.776 8 9.5 8h-3C6.224 8 6 8.224 6 8.5v2C6 10.776 6.224 11 6.5 11zM6 14H12V16.001H6zM13 14H18V16.001H13z"></path></svg></p>
          <p class="text-zinc-500 text-center">Carte</p>
        </div>
        <div class="jeton flex flex-col items-center p-3 second-black rounded-lg flex-1 grid place-content-center">
          <p class="text-xl text-center flex gap-1 items-center">{{ partie.jeton }}<svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><path d="M12,6C7.03,6,2,7.546,2,10.5v4C2,17.454,7.03,19,12,19s10-1.546,10-4.5v-4C22,7.546,16.97,6,12,6z M4,14.5v-1.197 c0.576,0.335,1.251,0.623,2,0.86v1.881C4.688,15.53,4,14.918,4,14.5z M16,14.648v1.971c-0.867,0.179-1.867,0.31-3,0.358v-2 C14.028,14.935,15.041,14.823,16,14.648z M11,16.978c-1.133-0.048-2.133-0.179-3-0.358v-1.971c0.959,0.174,1.972,0.287,3,0.33 V16.978z M18,16.044v-1.881c0.749-0.237,1.424-0.524,2-0.86V14.5C20,14.918,19.313,15.53,18,16.044z M12,13c-5.177,0-8-1.651-8-2.5 S6.823,8,12,8s8,1.651,8,2.5S17.177,13,12,13z"></path></svg></p>
          <p class="text-zinc-500 text-center">Jeton</p>
        </div>
      </div>
      <div class="chat mt-6 border-t-2 border-zinc-500/10 h-full">
        <div class="chat-container h-full">
          <div class="title text-xl font-semibold flex items-center justify-between text-zinc-300 mb-3 mt-1">
            game log
          </div>
          <div class="chat-body h-full">

            <div class="boxmessage rounded-lg text-zinc-500">
              <div v-for="indice in partie.indices" :key="indice.id" class="chat-message blury rounded-lg">
                <div class="flex items-center justify-between">

                  <div class="flex items-center gap-2">
                    <div class="aspect-square w-8 rounded-xl bg-cover bg-center shadow-lg shadow-orange-600/10" :style="{backgroundImage: `url('/src/assets/${indice.user.avatar}')`, backgroundColor: indice.user.couleur}"></div>
                    <p class="text-zinc-400 font-semibold ">
                      {{indice.user.pseudo}}
                    </p>
                  </div>
                  <p class="text-zinc-300">
                    {{indice.action}} {{indice.mot}} {{indice.nbmots}}
                  </p>
                </div>
              </div>
            </div>
          </div>
        </div>

      </div>
    </div>

  </div>

</template>

<style scoped>

.boxmessage > .chat-message:nth-child(odd) > div{
  flex-direction: row-reverse;
}
.boxmessage > .chat-message:nth-child(odd) > div > div{
  flex-direction: row-reverse;
}


.gameover,.terminé > div {
  display: flex;
  justify-content: space-around;
}
.gameover,.terminé > div > div > img {
  height: 5rem;
}
.gameover, .terminé > div > div > p {
  text-align: center;
}
.gameover, .terminé {
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
  padding: 15px;
}
.statut > div {
  color: #7c7c7c;
  font-size: 1.4rem;
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
.decouvert1, .decouvert2 {
  pointer-events: none;
  position: absolute;
  width: 90%;
  background-size: contain;
  background-image: url(../assets/detective.jpeg) ;
  height: 85%;
}
.jeton1, .jeton2 {
  pointer-events: none;
  position: absolute;
  width: 40%;
  background-size: contain;
  background-image: url(../assets/jeton.png) ;
  height: 60%;
}
.notyourturn {
  position: absolute;
  background-color: #000000a1;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;
}
.notyourturn1 {
  display: flex;
  row-gap: 0.5rem;
}

.notyourturn1 > button {
  cursor: pointer;
}

* {
  box-sizing: border-box;

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
  border: solid #ffffff 1px;
  background-color: #ffffff;
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
  background-color: #ebc597;
  border:#eec28b solid 2px;
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
  border-radius: 10px;
  margin-bottom: 1rem;
}

.chat-container {

}

.chat-body {
  height: 100%;
  box-sizing: border-box;
  overflow-y: auto;
}

.chat-message {
  padding: 10px;
  margin-bottom: 10px;
}
.chat-message > p {
  color: white;
}
.contenu {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.title {
  font-size: 1.4rem;
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
  font-size: smaller;
  font-weight: 500;
}

.statut {
  width: 100%;

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
  background-color: #f6dab7;
  position: relative;
}




.quantity {
  position: relative;
}

input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type=number] {
  -moz-appearance: textfield;
}

.quantity input {
  line-height: 1.65;
  float: left;
  display: block;
  padding: 0;
  margin: 0;
  padding-left: 45%;
  border: none;
  box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.08);
  font-size: 1rem;
}

.quantity input:focus {
  outline: 0;
}

.quantity-nav {
  float: left;
  position: relative;
  height: 42px;
}

.quantity-button {
  position: relative;
  cursor: pointer;
  border: none;
  border-left: 1px solid rgba(0, 0, 0, 0.08);
  width: 21px;
  text-align: center;
  color: #333;
  font-size: 13px;
  font-family: "FontAwesome" !important;
  line-height: 1.5;
  padding: 0;
  background: #FAFAFA;
  -webkit-transform: translateX(-100%);
  transform: translateX(-100%);
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  -o-user-select: none;
  user-select: none;
}

.quantity-button:active {
  background: #EAEAEA;
}

#nbmots {
  width: 100%;
}
.minipl {
  padding-left: 15px !important;
}
</style>