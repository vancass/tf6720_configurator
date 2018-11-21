<template>
  <div class="arburg-tag">
    <div class="tag">
      <div class="row">
        <label>Description:</label>
        <input type="text" v-model="arburgData.desc"/>
      </div>
      <div class="row">
        <label>OPC UA Identifier:</label>
        <input type="text" v-model="arburgData.identifier"/>
      </div>
      <div class="row">
        <label>Sampling mode:</label>
        <select v-model="arburgData.samplingMode">
          <option>Cyclic</option>
          <option>On Change</option>
        </select>
      </div>
      <div class="row">
        <label>Sampling rate:</label>
        <input type="text" v-model="arburgData.samplingRate" :disabled="!(arburgData.samplingMode=='Cyclic')"/>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ArburgTag",
  data: function() {
    return {};
  },

  props: {
    arburgData: Object
  },

  methods: {}
};
</script>

<style scoped>
.arburg-tag {
  padding-bottom: 50px;
}

.row {
  max-width: 400px;
  padding-bottom: 10px;
}

label {
  display: inline-block;
  width: 150px;
}

input[type="select"] {
  min-width: 200px;
}
.tag {
  max-width: 400px;
}
</style>