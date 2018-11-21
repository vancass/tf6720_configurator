<template>
  <div class="arburg">  
    <div v-for="arburgTag in arburgTags" :key="arburgTag.id">
      <ArburgTag :arburgData="arburgTag"></ArburgTag>
    </div>
    <button @click="addTag">Add tag</button>
    <br/>
    <button @click="generate">Generate</button>
    <button @click="saveConfig">Save</button>
    <input type="file" @change="readFile"/>
  </div>
</template>

<script>
import ArburgTag from "../components/ArburgTag.vue";

const header = `<?xml version="1.0" encoding="utf-8"?>
<Configuration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xsi:noNamespaceSchemaLocation="ConfigSchema_IoT_Version_1.6.xsd" Version="4.0.1.0">
  <Header>
    <Timestamp>2018-10-25T06:51:29Z</Timestamp>
  </Header>
  <GlobalConfig>
    <VisualInfo ScaleFactor="1" OffsetX="0" OffsetY="0" IsCollapsed="false" />
    <PwdInfos>
      <GlobalPassword>
        <PasswordInfos>
          <PasswordInfo Id="a532de08-6b6e-463b-a402-3c1057a39bc7" UserName="" pwd="" privKeyPwd="" pskId="" pskSecret="" />
          <PasswordInfo Id="b1f613f7-b5ba-4a54-8b67-ccb319b59162" UserName="" pwd="" privKeyPwd="" pskId="" pskSecret="" />
        </PasswordInfos>
      </GlobalPassword>
    </PwdInfos>
    <Logging FilePath="[BootDir]\\TcIotDataAgent.log" Flags="Critical,Error,Info" Switch="Off" />
    <ServerList />
    <Impersonate>0</Impersonate>
  </GlobalConfig>
  <BaseConfig>
    <FormatterInstances />
    <FormatterTypes />
    <Gates>
      <Gate Id="1" ReferenceTo="0" Switch="ON" Type="OpcUaDevice" Name="Screen01">
        <Url>opc.tcp://172.20.114.201:4880/Arburg</Url>
        <SecuritySettings MessageSecurityMode="None" SecurityPolicy="http://opcfoundation.org/UA/SecurityPolicy#None" />
        <AuthenticationSettings PwdInfoId="a821229b-3418-4020-ac9a-fd5e14f43cdc" Type="None" />
        <Channels>
          <Channel Id="1" ReferenceTo="0" Name="Arburg_Sub" Switch="ON" Role="Subscriber" SamplingMode="onChange" CycleTime="-1" Timeout="-1" Retain="false" MappingId="1" QoS="0" InhibitTime="0" BufferSize="0" OnChangePartialMode="true">
	        <Formatter EnableSymbolTimestamp="true" ProgId="{EC125306-74D0-46E4-9518-9258DB043679},TcDataAgent.Formatter_Binary.1" Name="Raw Binary" Type="InOut" />`;

export default {
  name: "Arburg",
  components: {
    ArburgTag
  },
  data: function() {
    return {
      arburgTags: [],
      finalText: ""
    };
  },
  methods: {
    addTag() {
      this.arburgTags.push({
        id: this.arburgTags.length + 1,
        desc: "Name to be shown in MQTT",
        identifier: "123456",
        samplingMode: "",
        samplingRate: ""
      });
    },

    generate() {
      this.processData();
      this.download(this.finalText, "arburg.xml");
    },
    readFile(ev) {
      const file = ev.target.files[0];

      const reader = new FileReader();
      reader.readAsText(file);

      reader.onload = event => {
        var obj = JSON.parse(event.target.result);
        this.arburgTags = obj.tags;
        console.log(obj.tags);
      };
    },
    saveConfig() {
      var tags = this.arburgTags;
      var json = JSON.stringify({ tags });
      this.download(json, "saved.json");
    },

    storeFile(index, file) {
      console.log("Storing file");
      this.opcSubscribers[index].file = file;
    },

    download(data, filename) {
      var file = new Blob([data], { type: "text/plain" });
      if (window.navigator.msSaveOrOpenBlob)
        // IE10+
        window.navigator.msSaveOrOpenBlob(file, filename);
      else {
        // Others
        var a = document.createElement("a"),
          url = URL.createObjectURL(file);
        a.href = url;
        a.download = filename;
        document.body.appendChild(a);
        a.click();
        setTimeout(function() {
          document.body.removeChild(a);
          window.URL.revokeObjectURL(url);
        }, 0);
      }
    },
    processData() {
      let finalText = header;
      let symbolIndex = 1;

      this.arburgTags.forEach(tag => {
        let symbol =
          `            
            <Symbol Id="` +
          symbolIndex +
          `" ReferenceTo="0" Switch="ON" DataType="REAL" DataTypeGuid="{18071995-0000-0000-0000-00000000000D}" Urn="` +
          tag.desc +
          `" UaIdentifier="ns=4;s=` +
          tag.identifier +
          `" UaNsName="http://www.arburg.com/" UaAttributeId="13" Conversion="Lossless">
            </Symbol>
          `;

        finalText += symbol;
        symbolIndex++;
      });
    }
  }
};
</script>

<style scoped>
.arburg {
  margin-top: 50px;
  outline: solid 1px black;
}
</style>