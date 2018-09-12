<template>
  <div class="hello">
    <div  class="columns medium-4">
    <typeahead
            source="https://clients1.google.com/complete/search?hl=en&client=youtube&ds=yt&jsonp=suggestCallBack&q="
            placeholder="Type something here..."
            :minLen="3"
            labelField="title"
            @selected="optionSelected">
            <template slot="item" slot-scope="option">
                <strong>{{ option.data.value }}</strong>
            </template>
    </typeahead>
    {{query}}
    </div>
     <vuetable ref="vuetable"
    api-url="https://vuetable.ratiw.net/api/users"
    :fields="['name', 'nickname', 'email', 'gender']"
    data-path="data"
    pagination-path=""
  ></vuetable>
  <div style="background-color: #f9f9f9">
  <vue-json-pretty :data="results"></vue-json-pretty>
  </div>
    <div class="columns medium-4" v-for="(result, index) of results" :key="index">
    <div class="card">
      <div class="card-section">
        <p> {{ index }} </p>
      </div>
      <div class="card-divider">
        <p>$ {{ result.USD }}</p>
      </div>
      <div class="card-section">
        <p> &#8364; {{ result.EUR }}</p>
      </div>
    </div>
    </div>
  </div>

</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
import Vuetable from 'vuetable-2';
import Typeahead from './Typeahead.vue';

export default {
    name: 'HelloWorld',
    components: {
        VueJsonPretty,
        Vuetable,
        Typeahead
    },
    props: {
        msg: String
    },
    data: function() {
        return {
            results: [],
            query: ''
        };
    },
    mounted() {
        this.axios.get('https://min-api.cryptocompare.com/data/pricemulti?fsyms=BTC,ETH,ELF&tsyms=USD,EUR').then(response => {
            this.results = response.data;
        });
    },
    methods: {
        optionSelected(result) {
            this.query = result;
        }
    }
};
</script>

<style scoped lang="scss">
h3 {
    margin: 40px 0 0;
}
ul {
    list-style-type: none;
    padding: 0;
}
li {
    display: inline-block;
    margin: 0 10px;
}
a {
    color: #42b983;
}
</style>
