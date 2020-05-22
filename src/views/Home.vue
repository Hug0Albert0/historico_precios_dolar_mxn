<template>
   <div>
      <v-layout class="d-flex justify-center">
         <v-flex xs12 md8 class="d-flex flex-column justify-center">
            <h1 class="my-5 text-center">Consulta de histórico. Valor del Peso Mexicano en USD</h1>
            <h4  class="text-center">Valor Oportuno BANXICO: ${{oportunoPrecio}} al día {{oportunoFecha}}</h4>
            <v-card class="my-3">
               <v-date-picker
                  v-model="fechaDolar"
                  full-width
                  full-height
                  :max="maximaFecha"
                  :min="minimaFecha"
                  @change="obtenerPreciosDolar(fechaDolar)"
                  :disabled="bloquearPeticion"
               >
               </v-date-picker>
            </v-card>
            <v-card :class="claseResultado" dark>
               <v-card-title class="text-center font-weight-bold justify-center display-1">
                  {{mensajeResultado}}
               </v-card-title>
            </v-card>
         </v-flex>
      </v-layout>
      <v-overlay :value="overlay">
         <v-progress-circular indeterminate size="64"></v-progress-circular>
      </v-overlay>
   </div>
</template>


<script>
   import axios from "axios"
   export default {
      data() {
         return {
            overlay: false,
            fechaDolar: null,
            maximaFecha: null,
            minimaFecha: "1992",
            precioDolar: null,
            fechaFormato: null,
            oportunoPrecio: null,
            oportunoFecha: null,
            bloquearPeticion: true,
            mensajeResultado: "",
            claseResultado: "teal",
         }
      },
      methods: {
         async obtenerPreciosDolar(fechaDolar) {
            try {
               this.bloquearPeticion = true;
               this.overlay = true;
               let datos = await axios.get(
                  `https://cors-anywhere.herokuapp.com/https://www.banxico.org.mx/SieAPIRest/service/v1/series/SF43718/datos/${fechaDolar}/${fechaDolar}`,
                  {
                        params:{},
                        headers: {
                           "Bmx-Token": "811be63f6bc4ea38d6970c1594f2560a34622e1a352133a2ad60104c430a6353",
                        }
                  }
               );
               this.precioDolar = await parseFloat(datos.data.bmx.series[0].datos[0].dato).toFixed(2).toString();
               this.fechaFormato = fechaDolar.split("-").reverse().join("/");
               this.claseResultado = "teal"
               this.mensajeResultado = `
                  Precio: $${this.precioDolar} MXN - Fecha: ${this.fechaFormato}
               `
            } catch (error) {
               this.fechaFormato = fechaDolar.split("-").reverse().join("/");
               this.claseResultado = "red"
               this.mensajeResultado = `No existen registros para la fecha ${this.fechaFormato}`
            }
            finally {
               this.overlay = false;
               this.bloquearPeticion = false;
            }
         },
         async cargarValorOportuno () {
            this.overlay = true;
            try {
               let datos = await axios.get(
                  "https://cors-anywhere.herokuapp.com/https://www.banxico.org.mx/SieAPIRest/service/v1/series/SF43718/datos/oportuno",
                  {
                        params:{},
                        headers: {
                           "Bmx-Token": "811be63f6bc4ea38d6970c1594f2560a34622e1a352133a2ad60104c430a6353",
                        }
                  }
               );
               this.oportunoPrecio = await parseFloat(datos.data.bmx.series[0].datos[0].dato).toFixed(2).toString();
               this.oportunoFecha = await datos.data.bmx.series[0].datos[0].fecha;
               this.maximaFecha = await datos.data.bmx.series[0].datos[0].fecha.split("/").reverse().join("-");
               this.fechaDolar = await datos.data.bmx.series[0].datos[0].fecha.split("/").reverse().join("-");
               this.obtenerPreciosDolar(this.fechaDolar);
            } catch (error) {
               console.log("No hay valores para éste día");
            }
         }
      },
      watch: {
         overlay (valor) {
            valor
         },
      },
      mounted() {
         this.cargarValorOportuno();
      }
   }
</script>
