
<template>
  <div :class="(className ? className + '-wrapper ' : '') + 'autocomplete-wrapper'">
    <input  type="text"
            :aria-label="name"
            :id="id"
            :name="name"
            :class="(className ? className + '-input ' : '') + 'autocomplete-input'"
            :placeholder="placeholder"
            v-model="type"
            @input="input(type)"
            @dblclick="showAll"
            @blur="hideAll"
            @keydown="keydown"
            @focus="focus"
            autocomplete="off" />

    <div :class="(className ? className + '-list ' : '') + 'autocomplete transition autocomplete-list'" v-show="showList">
      <ul>
        <li v-for="(data, i) in json"
            transition="showAll"
            :class="activeClass(i)">

          <a  href="#"
              @click.prevent="selectList(data)"
              @mousemove="mousemove(i)">
            <b>{{ data[anchor] }}</b>
            <span>{{ data[label] }}</span>
          </a>

        </li>
      </ul>
    </div>
  </div>
</template>


<script>

  /*! Copyright (c) 2016 Naufal Rabbani (http://github.com/BosNaufal)
  * Licensed Under MIT (http://opensource.org/licenses/MIT)
  *
  * Vue 2 Autocomplete @ Version 0.0.1
  *
  */

  /*!
  *  javascript-debounce 1.0.0
  *
  *  A lightweight, dependency-free JavaScript module for debouncing functions based on David Walsh's debounce function.
  *
  *  Source code available at: https://github.com/jgarber623/javascript-debounce
  *
  *  (c) 2015-present Jason Garber (http://sixtwothree.org)
  *
  *  javascript-debounce may be freely distributed under the MIT license.
  */

  var debounce = function(callback, delay) {
    var timeout;
    return function() {
      var context = this, args = arguments;
      clearTimeout(timeout);
      timeout = setTimeout(function() {
        callback.apply(context, args);
      }, delay);
    };
  };

  export default {

    props: {
      id: String,
      name: String,
      className: String,
      placeholder: String,

      // Intial Value
      initValue: {
        type: String,
        default: ""
      },

      // Anchor of list
      anchor: {
        type: String,
        required: true
      },

      // Label of list
      label: String,

      // Debounce time
      debounce: Number,

      // ajax URL will be fetched
      url: {
        type: String,
        required: true
      },

      // query param
      param: {
        type: String,
        default: 'q'
      },

      // Custom Params
      customParams: Object,

      // minimum length
      min: {
        type: Number,
        default: 0
      },

      // Process the result before retrieveng the result array.
      process: Function,

      // Callback
      onInput: Function,
      onShow: Function,
      onBlur: Function,
      onHide: Function,
      onFocus: Function,
      onSelect: Function,
      onBeforeAjax: Function,
      onAjaxProgress: Function,
      onAjaxLoaded: Function,

    },

    data() {
      return {
        showList: false,
        type: "",
        json: [],
        focusList: ""
      };
    },


    methods: {

      // Netralize Autocomplete
      clearInput() {
        this.showList = false
        this.type = ""
        this.json = []
        this.focusList = ""
      },

      // Get the original data
      cleanUp(data){
        return JSON.parse(JSON.stringify(data));
      },

      input(val){

        // Callback Event
        this.onInput ? this.onInput(val) : null

        // Debounce the "getData" method.
        if(!this.debouncedGetData || this.debounce !== this.oldDebounce) {
          this.oldDebounce = this.debounce;
          this.debouncedGetData = this.debounce ? debounce(this.getData.bind(this), this.debounce) : this.getData;
        }

        // Get The Data
        this.debouncedGetData(val)
      },

      showAll(){
        this.json = [];

        this.getData("")

        // Callback Event
        this.onShow ? this.onShow() : null

        this.showList = true;
      },

      hideAll(e){
        // Callback Event
        this.onBlur ? this.onBlur(e) : null

        setTimeout(() => {

          // Callback Event
          this.onHide ? this.onHide() : null

          this.showList = false;
        },250);
      },

      focus(e){
        this.focusList = 0;

        // If they click on our field, act as if they typed something to show the list / aaw - 2017.02.23
        this.input(this.type);

        // Callback Event
        this.onFocus ? this.onFocus(e) : null
      },

      mousemove(i){
        this.focusList = i;
      },

      keydown(e){
        let key = e.keyCode;

        // Disable when list isn't showing up
        if(!this.showList) return;

        switch (key) {
          case 40: //down
            this.focusList++;
          break;
          case 38: //up
            this.focusList--;
          break;
          case 13: //enter
            this.selectList(this.json[this.focusList])
            this.showList = false;
          break;
          case 27: //esc
            this.showList = false;
          break;
        }

        // When cursor out of range
        let listLength = this.json.length - 1;
        this.focusList = this.focusList > listLength ? 0 : this.focusList < 0 ? listLength : this.focusList;

      },

      activeClass(i){
        return {
          'focus-list' : i == this.focusList
        };
      },

      selectList(data){
        let clean = this.cleanUp(data);

        // Put the selected data to type (model)
        this.type = clean[this.anchor];

        this.showList = false;

        /**
        * Callback Event
        * Deep clone of the original object
        */
        this.onSelect ? this.onSelect(clean) : null
      },

      getData(val){

        let self = this;

        // Hide the list if the value isn't minimum length / aaw - 2017.02.20
        if ((!val) || (val.length < this.min)) {
          this.showList = false;
          return;
        }
        if(this.url != null){

          // Callback Event
          this.onBeforeAjax ? this.onBeforeAjax(val) : null

          let ajax = new XMLHttpRequest();

          let params = ""
          if(this.customParams) {
            Object.keys(this.customParams).forEach((key) => {
              params += `&${key}=${this.customParams[key]}`
            })
          }

          ajax.open('GET', `${this.url}?${this.param}=${val}${params}`, true);
          ajax.send();

          ajax.addEventListener('progress', function (data) {
            if(data.lengthComputable){

              // Callback Event
              // Should be self. not this. / aaw - 2017.02.20
              self.onAjaxProgress ? self.onAjaxProgress(data) : null
            }
          });

          ajax.addEventListener('loadend', function (data) {
            if (this.responseText !== undefined && this.responseText && this.responseText.length > 0) {
              let json = JSON.parse(this.responseText);

              // Callback Event
              // Should be self. not this. / aaw - 2017.02.20
              self.onAjaxLoaded ? self.onAjaxLoaded(json) : null

              self.json = self.process ? self.process(json) : json;
              // Only show the list if there is data to display / aaw - 2017.02.20
              self.showList = (self.json !== undefined && self.json && self.json.length > 0);
            }
          });

        }
      },

      setValue(val) {
        this.type = val
      }
    },

    created(){
      // Sync parent model with initValue Props
      this.type = this.initValue ? this.initValue : null
    }

  }
</script>
