<template>
    <!--MAIN DIALOG -->
    <div style="text-align:left">
    <h1>Title à mettre ici !!</h1>
    <span v-html="config.config.freetext">
      </span>
    </div>
</template>

<script>
import axios from "axios";

export default {
  name: "FreeText",
  data: () => ({
    
  }),
  props: {
    config: {
      type: Object
    }
  },
  methods: {
    
    
  },
  created: function() {    
  },

  mounted: function() {},
  beforeDestroy: function() {
    clearInterval(this.timer);
  }
};
</script>

<style>

</style>

