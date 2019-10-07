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
            <div class="d-flex flex-row">
              <v-text-field
                label="Valor C"
                outlined
                v-model="C"
              ></v-text-field>

              <v-text-field
                label="Limite en eje X"
                outlined
                v-model="ejeX"
              ></v-text-field>
            </div>

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
        :height="180"
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
    ejeX: 50,
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
      for (let i = 0; i <= this.ejeX; i += 1) {
        this.xInterval.push(i);
      }

      this.selectedFunctions.length = 0;
      this.selectedFunctions.push('funcion f(x)');

      this.data = this.xInterval.map(num => this.evaluateFunction(num, this.funcionF));
      this.datas['funcion f(x)'].data = this.data;
      this.datas['funcion f(x)'].label = this.funcionF;

      this.desplazamientoIzquierda();
      this.desplazamientoDerecha();
      this.fCombinada = this.sumFunctions(this.fIzquierda, this.fDerecha);

      this.graficarDesplazamientos();
      clearInterval(this.interval);
      this.interval = setInterval(this.graficarDesplazamientos, 1000);
    },
    integrateFunction(funcion, variable) {
      return math.integral(funcion, variable).toString();
    },
    evaluateIntegral(funcion, limitInferior, limitSuperior) {
      const funcionIntegrada = this.integrateFunction(funcion, 'x');
      const superior = this.evaluateFunction(
        limitSuperior,
        funcionIntegrada,
      );
      const inferior = this.evaluateFunction(
        limitInferior,
        funcionIntegrada,
      );
      return (superior - inferior);
    },
    sumFunctions(foo, bar) {
      const res = math.parse(foo.toString().concat('+').concat(bar.toString()));
      const simple = math.simplify(res).toString();
      return simple;
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
      const newComb = this.fCombinada.replace(new RegExp('t', 'g'), this.contador);

      const dataIzq = this.xInterval.map(num => 0.5 * this.evaluateFunction(num, newIzquierda));
      const dataDer = this.xInterval.map(num => 0.5 * this.evaluateFunction(num, newDerecha));
      const dataComb = this.xInterval.map(
        num => 0.5 * this.evaluateFunction(num, newComb)
        + this.evaluateIntegral(
          this.funcionG,
          `((${(this.C * -1).toString()}) * ${this.contador})`,
          `(${this.C} * ${this.contador})`,
        ),
      );

      const indexIzq = this.selectedFunctions.indexOf('funcion f(x - ct)');
      if (indexIzq > -1) {
        this.selectedFunctions.splice(indexIzq, 1);
      }

      const indexDer = this.selectedFunctions.indexOf('funcion f(x + ct)');
      if (indexDer > -1) {
        this.selectedFunctions.splice(indexDer, 1);
      }

      const indexComb = this.selectedFunctions.indexOf('funcion combinada');
      if (indexComb > -1) {
        this.selectedFunctions.splice(indexComb, 1);
      }

      this.selectedFunctions.push('funcion f(x - ct)');
      this.selectedFunctions.push('funcion f(x + ct)');
      this.selectedFunctions.push('funcion combinada');

      this.datas['funcion f(x - ct)'].data = dataIzq;
      this.datas['funcion f(x + ct)'].data = dataDer;
      this.datas['funcion combinada'].data = dataComb;
      this.datas['funcion f(x - ct)'].label = this.fIzquierda;
      this.datas['funcion f(x + ct)'].label = this.fDerecha;
      this.datas['funcion combinada'].label = 'Resultado';

      this.contador += 1;
      console.log(Number(this.ejeX) + 1);
      if (this.contador === Number(this.ejeX) + 1) {
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
