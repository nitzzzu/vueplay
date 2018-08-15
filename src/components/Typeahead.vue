// https://stackoverflow.com/questions/50927481/scroll-with-arrow-keys-on-dropdown-vuejs
// https://github.com/paliari/v-autocomplete/blob/master/src/Autocomplete.vue

<template>
    <div class="autocomplete">
        <input 
            v-model="query"
            @input="onChange" 
            type="text" 
            :placeholder="placeholder"
             @keyup.down="keyDown" @keyup.up="keyUp" @keyup.enter="keyEnter"
        >
        <ul class="autocomplete-results" v-show="isOpen">
            <li v-for="(result, i) in results" :key="result.id" @click="setResult(result)" class="autocomplete-result" :class="{ 'is-active': i === cursor }" v-html="highlight(result.value)">
            </li>
        </ul>
    </div>
</template>

<script>
import _ from 'lodash';

export default {
    name: 'typeahead',
    props: {
        source: { type: String },
        minLen: { type: Number },
        placeholder: { type: String },
        labelField: { type: String },
        valueField: { type: String },
        wait: { type: Number, default: 150 }
    },
    data() {
        return {
            results: [],
            query: '',
            isOpen: false,
            cursor: 0
        };
    },
    created: function() {
        this.debouncedLookup = _.debounce(this.lookup, this.wait);
    },
    methods: {
        async onChange() {
            let q = this.query.trim();
            if (q.length >= this.minLen) {
                await this.debouncedLookup();
            } else {
                if (this.isOpen) {
                    this.closeResults();
                }
            }
        },
        async lookup() {
            let response = await this.axios.get(
                'https://cors-anywhere.herokuapp.com/http://suggestqueries.google.com/complete/search?hl=en&ds=yt&xhr=t&client=youtube&jsonp=suggestCallBack&q=' +
                    this.query.trim()
            );
            let q = this.query.trim();
            if (q.length >= this.minLen) {
                this.cursor = -1;
                this.results = this.extractResults(response.data);
                this.isOpen = this.results.length > 0;
            }
        },
        extractResults(results) {
            let res = results.replace('suggestCallBack(', '');
            res = res.substring(0, res.length - 1);
            let o = JSON.parse(res)[1];

            var suggestions = [];

            for (let x of o) {
                suggestions.push({ value: x[0] });
            }
            return suggestions;
            //return _.map(results, x => ({ id: x.id, value: x[this.labelField] }));
        },
        setResult(result) {
            this.$emit('selected', result.value);
            if (this.query != result.value) {
                this.query = result.value;
            }
            this.closeResults();
        },
        closeResults() {
            this.isOpen = false;
            this.cursor = -1;
            this.results = [];
        },
        keyDown(ev) {
            ev.preventDefault();
            if (this.cursor < this.results.length - 1) {
                this.cursor++;
                this.itemView(this.$el.getElementsByClassName('autocomplete-result')[this.cursor]);
            }
        },
        keyUp(ev) {
            ev.preventDefault();
            if (this.cursor > 0) {
                this.cursor--;
                this.itemView(this.$el.getElementsByClassName('autocomplete-result')[this.cursor]);
            }
        },
        itemView(item) {
            if (item && item.scrollIntoView) {
                item.scrollIntoView(false);
            }
        },
        keyEnter() {
            if (this.cursor >= 0 && this.results[this.cursor]) {
                this.setResult(this.results[this.cursor]);
            } else {
                this.setResult({ value: this.query });
            }
        },
        handleClickOutside(evt) {
            if (!this.$el.contains(evt.target)) {
                this.closeResults();
            }
        },
        highlight(text) {
            return text.replace(new RegExp(this.query, 'gi'), '<span style="font-weight: normal">$&</span>');
        }
    },
    mounted() {
        document.addEventListener('click', this.handleClickOutside);
    },
    destroyed() {
        document.removeEventListener('click', this.handleClickOutside);
    }
};
</script>

<style scoped lang="scss">
.autocomplete {
    position: relative;
    width: 400px;
}

.autocomplete-results {
    padding: 0;
    margin: 0;
    border: 1px solid #eeeeee;
    max-height: 300px;
    overflow: auto;
    width: 100%;
    position: absolute;
    background-color: #ffffff;
}

.autocomplete-result {
    list-style: none;
    text-align: left;
    padding: 2px 2px;
    cursor: pointer;
    font-weight: bold;
    color: #222;
}

.autocomplete-result.is-active,
.autocomplete-result:hover {
    background-color: #f0f0f0;
}
</style>
