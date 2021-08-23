<template>
  <div>
    <div v-bind:ref="containerName" v-bind:id="containerName"></div>
  </div>
</template>

<script>
export default {
  props: {
    isMicroFrontend: {
      type: Boolean,
      default: false
    },
    isWebComponent: {
      type: Boolean,
      default: false
    },
    name: {
      type: String,
      default: ""
    },
    "host-main-js": {
      type: String,
      default: ""
    },
    "host-main-html": {
      type: String,
      default: ""
    },
    "host-polyfill-js":{
      type: String,
      default: ""
    },
    "host-css": {
      type: String,
      default: ""
    },
    "container-name": {
      type: String,
      default: ""
    },
    tags: {
      type: String,
      default: ""
    }
  },
  data: () => {
    return {
      // main js data
      scriptMainId:'',
      scriptMain:'',

      // polyfill data
      scriptPolyfillId:'',
      scriptPolyfill:'',

      // css data
      cssId:'',
      appCss:'',

      hasPolyfillLoaded: false,
      hasMainLoaded: false
    }
  },
  computed:{
    containerNameStr() {
      return this.containerName.toString()
    },
    functionName(){
      return `render-${this.name}`
    }
  },
  mounted() {
    this.updateCustomAppContainer();
  },

  methods: {
    loadWebComponent: function(){
      if (this.hasPolyfillLoaded && this.hasMainLoaded) {
        const parsedTags = JSON.parse(this.tags);
        parsedTags.map((tag) => {
          const webComponent = document.createElement(tag.tagName);
          if (tag && tag.inputs && tag.inputs.length > 0){
            tag.inputs.map((attribute) => {
              if (attribute.inputValue !== null){
                webComponent.setAttribute(attribute.inputName, attribute.inputValue);
              }
            });
          }
          if (tag && tag.outputs && tag.outputs.length > 0){
            tag.outputs.map((output) => {
              if (output.eventCb !== null) {
                webComponent.addEventListener(output.eventName, output.eventCb);
              }
            });
          }
          if (tag.toBody){
            document.body.appendChild(webComponent);
          }else{
            const webComponentContainer = this.$refs[this.containerName];
            if(webComponentContainer){
              webComponentContainer.appendChild(webComponent);
            }
          }
        });
      }
    },
    renderMF : function(){
      if (this.hasPolyfillLoaded && this.hasMainLoaded) {
        window["render-dash-board-debitore"]('ciao');
        let tags = JSON.parse(this.tags) || []
        if(window[this.functionName]){

          window[this.functionName](`${this.containerName}-container`, tags);
        }else{
          console.log('ancora non esiste')
          this.renderMF()
        }
      }
    },
    updateCustomAppContainer() {
      const webComponentContainer = this.$refs[this.containerName];
      if(webComponentContainer){
        console.log('this.isMicroFrontend',this.isMicroFrontend)
        webComponentContainer.innerHTML = `<object style="width: 100%; height:80vh;" data="${this.hostMainHtml}?isMicro=${this.isMicroFrontend}&tag=${encodeURIComponent(this.tags)}"></object>`;
      }
    },
  }
};
</script>
