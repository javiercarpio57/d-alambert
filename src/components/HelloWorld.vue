<template>
  <div>
    <div class="pt-3 text-center">
      <span class="font-weight-thin display-2">Solución por d'alambert</span>
    </div>

    <div class="pt-5 text-center">
      <span class="font-weight-regular headline">Ingrese su función f(x):</span>

      <div class="d-flex justify-center flex-row pt-2 ml-10 mr-10 pl-10 pr-10">
        <div class="d-flex flex-column mt-2 pt-5">
          <div class="d-flex" style="width:600px;">
            <span class="font-weight-light headline mt-3">f(x) =</span>
            <v-text-field
              placeholder='Ingrese su función'
              class="ml-2 font-weight-light"
              solo
              clearable
              color="teal"
              v-model="funcionF"
            >
            </v-text-field>
          </div>
          <div class="d-flex" style="width:600px;">
            <span class="font-weight-light headline mt-3">g(x) =</span>
            <v-text-field
              placeholder='Ingrese su función'
              class="ml-2 font-weight-light"
              solo
              clearable
              color="teal"
              v-model="funcionG"
            >
            </v-text-field>
          </div>
        </div>
        <div class="pl-9 d-flex flex-row align-center">
          <div class="d-flex flex-column">
            <v-text-field
              label="Valor C"
              outlined
              v-model="C"
            ></v-text-field>

            <v-btn
            rounded
            color="success"
            dark
            class="ml-2 mt-1"
            v-on:click="f()"
            value="current"
            >
              Graficar
            </v-btn>
          </div>
        </div>
      </div>
    </div>

    <div>
      <line-chart
        :width="400"
        :height="100"
        :labels="xInterval"
        :datasets="displayedDatasets"
      ></line-chart>
    </div>
  </div>
</template>

<script>
import { evaluate, create, all } from 'mathjs';
import LineChart from './LineChart.vue';

const config = { };
const math = create(all, config);
math.import(require('mathjs-simple-integral'));

export default {

  name: 'HelloWorld',
  // datasets,
  components: {
    LineChart,
  },
  data: () => ({
    xInterval: [],
    data: [],
    funcion: '',
    funcionF: '',
    funcionG: '',
    fIzquierda: '',
    fDerecha: '',
    fCombinada: '',
    C: '',
    interval: '',
    contador: 10,
    selectedFunctions: [],
    datas: {
      'funcion f(x)': {
        label: '',
        borderColor: 'rgba(50, 115, 220, 0.5)',
        backgroundColor: 'rgba(50, 115, 220, 0.1)',
        data: [],
      },
      'funcion f(x - ct)': {
        label: '',
        borderColor: 'rgba(1, 1, 1, 0.5)',
        backgroundColor: 'rgba(1, 1, 1, 0.1)',
        data: [],
      },
      'funcion f(x + ct)': {
        label: '',
        borderColor: 'rgba(0, 255, 0, 0.5)',
        backgroundColor: 'rgba(0, 255, 0, 0.1)',
        data: [],
      },
      'funcion combinada': {
        label: '',
        borderColor: 'rgba(255, 255, 0, 0.5)',
        backgroundColor: 'rgba(255, 255, 0, 0.1)',
        data: [],
      },
    },
  }),
  methods: {
    f() {
      this.xInterval.length = 0;
      for (let i = 0; i <= 50; i += 1) {
        this.xInterval.push(i);
      }

      this.selectedFunctions.length = 0;
      this.selectedFunctions.push('funcion f(x)');

      this.data = this.xInterval.map(num => this.evaluateFunction(num, this.funcionF));
      this.datas['funcion f(x)'].data = this.data;
      this.datas['funcion f(x)'].label = this.funcionF;

      this.desplazamientoIzquierda();
      this.desplazamientoDerecha();

      this.graficarDesplazamientos();
      clearInterval(this.interval);
      this.interval = setInterval(this.graficarDesplazamientos, 1000);

    },

    integrateFunction(funcion, variable){
     return math.integral(funcion, variable).toString();
    },

    sumFunctions(foo, bar){
      
      let res = math.parse(foo.toString().concat("+").concat(bar.toString())) 
      let simple = math.simplify(res).toString()
      return simple
    },
    evaluateFunction(x, funcion) {
      const newF = funcion.replace(new RegExp('x', 'g'), x);
      return evaluate(newF);
    },
    desplazamientoIzquierda() {
      this.fIzquierda = this.funcionF.replace(new RegExp('x', 'g'), `(x + ${this.C} * t)`);
    },
    desplazamientoDerecha() {
      this.fDerecha = this.funcionF.replace(new RegExp('x', 'g'), `(x - ${this.C} * t)`);
    },
    graficarDesplazamientos() {
      const newIzquierda = this.fIzquierda.replace(new RegExp('t', 'g'), this.contador);
      const newDerecha = this.fDerecha.replace(new RegExp('t', 'g'), this.contador);

      const dataIzq = this.xInterval.map(num => 0.5 * this.evaluateFunction(num, newIzquierda));
      const dataDer = this.xInterval.map(num => 0.5 * this.evaluateFunction(num, newDerecha));

      const indexIzq = this.selectedFunctions.indexOf('funcion f(x - ct)');
      if (indexIzq > -1) {
        this.selectedFunctions.splice(indexIzq, 1);
      }

      const indexDer = this.selectedFunctions.indexOf('funcion f(x + ct)');
      if (indexDer > -1) {
        this.selectedFunctions.splice(indexDer, 1);
      }

      this.selectedFunctions.push('funcion f(x - ct)');
      this.selectedFunctions.push('funcion f(x + ct)');

      this.datas['funcion f(x - ct)'].data = dataIzq;
      this.datas['funcion f(x + ct)'].data = dataDer;
      this.datas['funcion f(x - ct)'].label = this.fIzquierda;
      this.datas['funcion f(x + ct)'].label = this.fDerecha;

      this.contador += 1;
      if (this.contador === 51) {
        this.contador = 0;
      }
    },
  },
  computed: {
    
    displayedDatasets() {
      return this.selectedFunctions.map(year => this.datas[year]);
    },
  },
  
};
</script>

<style scoped>
</style>
