<template>
  <div>
    <h1>{{ saludo}}</h1>
    <h2 :style="res>=200?'background-color:green;border:1px solid purple':'background-color:cyan;border-radius:30px'">{{ res }}</h2>

    <input type="text" placeholder="documento.." v-model="documento">
    <button @click="sumar()">Sumar</button>
    <input type="text" v-model="n1">
    <input type="text" v-model="n2">
    <!-- <img :src="ima" alt=""> -->

    <input type="radio" v-model="gender" value="masculino">Masculino
    <input type="radio" v-model="gender" value="femenino">Femenino
    <input type="radio" v-model="gender" value="otro">otro
    <p>Género seleccionado: {{ gender }}</p>

    <!-- listas desplegables
    seleccion unica -->
    <select v-model ="selectedfruit">
      <option disabled value="">Selecciona una fruta</option>
      <option value="manzana">Manzana</option>
      <option value="pera">Pera</option>
      <option value="naranja">Naranja</option>
    </select>
    <p>Fruta seleccionada: {{ selectedfruit }}</p>

     <!-- seleccion multiple -->
      <select v-model ="selectedfruits">
      <option disabled value="">Selecciona una fruta</option>
      <option value="manzana">Manzana</option>
      <option value="pera">Pera</option>
      <option value="naranja">Naranja</option>
    </select>
    <p>Fruta seleccionada: {{ selectedfruits }}</p>
  </div>
</template>

<script setup>
import {ref} from "vue"
let saludo = ref ("ADSO")
let dolar = 4000
dolar = 4500
let n1 = ref (0)
let n2 = ref (0)
let res = ref (0)
res.value = n1.value+n2.value



//directiva v-model
let documento = ref ("gabriel")
let gender = ref ("")
let selectedfruit = ref ("")
let selectedfruits = ref ([])

//directiva v-on abreviatura @
function sumar() {
  res.value=parseInt(n1.value) + parseInt(n2.value)
}

// directiva v-bind abreviatura :
//  let ima = ref("https://st1.uvnimg.com/f3/28/ef055a306f12b23083ee9bc23c45/los-simpson-homero-simpson-dona.jpg")
// </script>

<style scoped>

</style>