<template>
    <div class="autocomplete">
        <input 
            v-model="query"
            @input="inputChange" 
            type="text" 
            :placeholder="placeholder"
             @keyup.down="keyDown" @keyup.up="keyUp" @keyup.enter="keyEnter"
        >
        <ul class="autocomplete-options" v-show="isOpen">
            <li v-for="(option, i) in options" :key="option.id" @click="selectOption(option)" class="autocomplete-option" :class="{ 'is-active': i === cursor }">
                <slot name="item" :data="option"><span v-html="defaultValue(option)"></span></slot>
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
            options: [],
            query: '',
            isOpen: false,
            cursor: 0
        };
    },
    created: function() {
        this.debouncedLookup = _.debounce(this.lookup, this.wait);
    },
    methods: {
        defaultValue(option) {
            if (typeof option === 'string') {
                return option;
            } else if (typeof option === 'object') {
                if (option.value) {
                    return this.highlight(option.value);
                } else {
                    return String(option[Object.keys(option)[0]]);
                }
            } else {
                return '';
            }
        },
        async inputChange() {
            let q = this.query.trim();
            if (q.length >= this.minLen) {
                await this.debouncedLookup();
            } else {
                if (this.isOpen) {
                    this.closeOptions();
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
                this.options = this.extractOptions(response.data);
                this.isOpen = this.options.length > 0;
            }
        },
        extractOptions(options) {
            let res = options.replace('suggestCallBack(', '');
            res = res.substring(0, res.length - 1);
            let o = JSON.parse(res)[1];

            var suggestions = [];

            for (let x of o) {
                suggestions.push({ value: x[0] });
            }
            return suggestions;
            //return _.map(options, x => ({ id: x.id, value: x[this.labelField] }));
        },
        selectOption(option) {
            this.$emit('selected', option.value);
            if (this.query != option.value) {
                this.query = option.value;
            }
            this.closeOptions();
        },
        closeOptions() {
            this.isOpen = false;
            this.cursor = -1;
            this.options = [];
        },
        keyDown(ev) {
            ev.preventDefault();
            if (this.cursor < this.options.length - 1) {
                this.cursor++;
                this.itemView(this.$el.getElementsByClassName('autocomplete-option')[this.cursor]);
            }
        },
        keyUp(ev) {
            ev.preventDefault();
            if (this.cursor > 0) {
                this.cursor--;
                this.itemView(this.$el.getElementsByClassName('autocomplete-option')[this.cursor]);
            }
        },
        itemView(item) {
            if (item && item.scrollIntoView) {
                item.scrollIntoView(false);
            }
        },
        keyEnter() {
            if (this.cursor >= 0 && this.options[this.cursor]) {
                this.selectOption(this.options[this.cursor]);
            } else {
                this.selectOption({ value: this.query });
            }
        },
        handleClickOutside(evt) {
            if (!this.$el.contains(evt.target)) {
                this.closeOptions();
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

.autocomplete-options {
    padding: 0;
    margin: 0;
    border: 1px solid #eeeeee;
    max-height: 300px;
    overflow: auto;
    width: 100%;
    position: absolute;
    background-color: #ffffff;
}

.autocomplete-option {
    list-style: none;
    text-align: left;
    padding: 2px 6px;
    cursor: pointer;
    font-weight: bold;
    color: #222;
}

.autocomplete-option.is-active,
.autocomplete-option:hover {
    background-color: #f0f0f0;
}
</style>
