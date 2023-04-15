<script setup>

</script>

<template>
        <button>
            <div class="card">
                <div class="hautcard">
                    <div class="left"></div>
                    <div class="right">
                        <img src="src\assets\imgcard.png" alt="">
                    </div>
                </div>
                <div class="bascard">
                    <p>{{ mot.mot.fr }}</p>
                </div>                
            </div>
        </button>
</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
.div2 {
    background-color:  #c3e1b3!important;
}
.div2 > .card {
  background-color: #49a818!important;
  border: #439e17 solid 2px!important;
}
.div3 {
  background-color:  #ffb87e!important;
}
.div3 > .card {
background-color: #e03c08;
border: #c0482d solid 2px;
}
.div4 {
  background-color: #757575!important;
}
.div4 > .card {
background-color: #525151;
border: #343434 solid 2px;
}
.card {
  background-color: #f6d9b6;
  border:#9d8064 solid 2px;
  border-radius: 10px;
    display: flex;
flex-direction: column;
height: 100%;
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

</style>
