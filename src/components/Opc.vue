<template>
  <div class="opc-input">

    <table>
      <tr>
        <td>URL:</td>
        <td><input type="text" v-model="opcData.opcUrl"/></td>
      </tr>
      <tr>
        <td>Namespace:</td>
        <td><input type="text" v-model="opcData.opcNamespace"/></td>
      </tr>
      <tr>
        <td>Taglist:</td>
        <td><input type="file" @change="handleFile"/></td>
      </tr>

    </table>

  </div>
</template>

<script>
export default {
  name: 'Opc',
  data: function(){
    return{
      
    }
  },

  props: {
    opcData: {}
  },


  methods: {
    handleFile(ev){
      const file = ev.target.files[0];

      this.$emit('storeFile', file);
    }
  }
}
</script>

<style scoped>
.opc-input{
  padding-bottom: 20px;
}
</style>