<template>
  <div>
    <h1 class="text-center">Graficas</h1>
    <v-card v-if="mostrarGrafica">
      <GChart :type="tipoGrafica" :data="chartData" :options="chartOptions" />
      <v-container>
        <v-row>
          <v-col class="d-flex justify-center">
            <v-btn class="secondary" @click="tipoGrafica = 'ColumnChart'">Grafica de barras</v-btn>
            <v-spacer></v-spacer>
            <v-btn class="accent" @click="tipoGrafica = 'ScatterChart'">Grafica de puntos</v-btn>
            <v-spacer></v-spacer>
            <v-btn class="secondary" @click="tipoGrafica = 'Table'">Tabla</v-btn>
             <v-spacer></v-spacer>
            <v-btn class="accent" @click="tipoGrafica = 'LineChart'">Grafica de linea</v-btn>
            
          </v-col>
        </v-row>
      </v-container>
    </v-card>
  </div>
</template>

<script>
import axios from "axios";
import { GChart } from "vue-google-charts";

export default {
  components: {
    GChart
  },
  data: () => ({
    tipoGrafica: 'ColumnChart',
    mostrarGrafica: false,
    datosAPI: null,
    chartData: [["Año", "Promedio", "Precio mas alto", "Precio mas bajo"]],
    chartOptions: {
      title: "Promedio anual paridad peso-dolar",
      subtitle: "Sales, Expenses, and Profit: 2014-2017",
      height: 500,
      width: "100%",
      bar: { groupWidth: "80%" },
      legend: {
        position: "top"
      },
      animation:{
        duration: 1000,
        easing: 'in',
        startup: true
      }
    }
  }),
  methods: {
    async obtenerTodo() {
      let headers = {
        "Bmx-Token":
          "811be63f6bc4ea38d6970c1594f2560a34622e1a352133a2ad60104c430a6353"
      };

      try {
        let response = await axios.get(
          "https://cors-anywhere.herokuapp.com/https://www.banxico.org.mx/SieAPIRest/service/v1/series/SF43718/datos",
          { headers }
        );
        this.datosAPI = response.data.bmx.series[0].datos;
        console.log(response.data.bmx.series[0].datos);
        this.sacarPromedioAnual();
      } catch (error) {
        console.warn(error);
      }
    },
    async sacarPromedioAnual() {
      let resultado = [];
      let anioInicio = parseInt(this.datosAPI[0].fecha.split("/")[2]);
      let anioFin = parseInt(
        this.datosAPI[this.datosAPI.length - 1].fecha.split("/")[2]
      );

      for (anioInicio; anioInicio <= anioFin; anioInicio++) {
        let suma = 0;
        let divisor = 0;
        let promedio = 0;
        let mayor = 0;
        let menor = 0;
        this.datosAPI.forEach(e => {
          let anio = parseInt(e.fecha.split("/")[2]);

          if (anio == anioInicio) {
            suma += parseFloat(e.dato);
            divisor++;
            if (divisor == 1) {
              mayor = parseFloat(e.dato);
              menor = parseFloat(e.dato);
            } else {
              mayor = parseFloat(e.dato) >= mayor ? parseFloat(e.dato) : mayor;
              menor = parseFloat(e.dato) <= menor ? parseFloat(e.dato) : menor;
            }
          }
        });
        promedio = suma / divisor;

        resultado.push({
          fecha: anioInicio,
          promedio,
          mayor,
          menor
          //suma,
          //dias: divisor
        });
      }

      this.datosAPI = resultado;
      this.llenarChart(resultado);
    },
    async llenarChart(datos) {
      datos.forEach(e => {
        this.chartData.push([e.fecha.toString(), e.promedio, e.mayor, e.menor]);
      });
      this.mostrarGrafica = true;
    }
  },
  created() {
    this.obtenerTodo();
  }
};
</script>

<style lang="scss" scoped>
</style>