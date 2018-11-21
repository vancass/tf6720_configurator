<template>
  <div class="opc-input">

    <table>
      <tr>
        <td>Broker URL:</td>
        <td><input type="text" v-model="mqttData.broker"/></td>
      </tr>
      <tr>
        <td>Topic name:</td>
        <td><input type="text" v-model="mqttData.topicName"/></td>
      </tr>

    </table>

  </div>
</template>

<script>
export default {
  name: 'Mqtt',
  data: function(){
    return{
      
    }
  },

  props: {
    mqttData: {}
  },


  methods: {
  }
}
</script>