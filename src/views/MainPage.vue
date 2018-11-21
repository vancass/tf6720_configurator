<template>
  <div class="main-page">  
    <div class="pubsub">
      <div class="subscriber">
        <div v-for="(opcSub, index) in opcSubscribers" :key="opcSub.id">
          <Opc :opcData="opcSub" @storeFile="storeFile(index, $event)"></Opc>
        </div>
        <button @click="addSubscriber">Add subscriber</button>
      </div>
      <div class="publisher">
        <Mqtt :mqttData="mqttData"/>
      </div>
    </div>
    <br/>
    <br/>
    <button @click="generate">Generate</button>
    <br/>
    <br/>
<!--     <td><input type="file" @change="handleFile"/></td>
    <br/>
    <button @click="testDownload">Download</button> -->

    <div id="progress_bar"><div class="percent">0%</div></div>
  </div>
</template>

<script>
import Opc from "../components/Opc.vue";
import Mqtt from "../components/Mqtt.vue";

var progress = document.querySelector(".percent");

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
    <Gates>`;

export default {
  name: "MainPage",
  components: {
    Opc,
    Mqtt
  },
  data: function() {
    return {
      opcSubscribers: [],

      file: null,
      mqttData: { broker: "127.0.0.1", topicName: "testTopic" },
      allTags: [],
      allTopics: {},
      dataType: {
        BOOL: "{18071995-0000-0000-0000-000000000030}",
        INT: "{18071995-0000-0000-0000-000000000006}",
        DATETIME: "{34318025-32E2-43ED-9E4D-D6AF280C2F5E}",
        "STRING(255)": "{18071995-0000-0000-0000-0001000000FF}"
      },

      finalText: ""
    };
  },

  computed: {},

  methods: {
    addSubscriber() {
      this.opcSubscribers.push({
        id: this.opcSubscribers.length + 1,
        opcUrl: "opc.tcp://192.168.1.1:4840",
        opcNamespace: "http://www.unifiedautomation.com/DemoServer/",
        file: ""
      });
    },

    storeFile(index, file) {
      console.log("Storing file");
      this.opcSubscribers[index].file = file;
      const reader = new FileReader();
      reader.readAsText(file);

      reader.onload = event => {
        const data = event.target.result.split("\n");

        data.forEach(tag => {
          tag = tag.replace(/\s+/, "");

          const topic = tag
            .split(".")
            .slice(0, -1)
            .join("/");
          let type = "";

          if (tag.endsWith("_b")) type = "BOOL";
          else if (tag.endsWith("_int")) type = "INT";
          else if (tag.endsWith("_dt")) type = "DATETIME";
          else if (tag.endsWith("_str")) type = "STRING(255)";
          else type = "BOOL";

          if (!this.allTags[index]) {
            //create new
            this.allTags[index] = [{ name: tag, topic: topic, type: type }];
          } else {
            //append
            this.allTags[index].push({
              name: tag,
              topic: topic,
              type: type
            });
          }

          if (!this.allTopics[topic]) {
            this.allTopics[topic] = [{ name: tag, type: type }];
          } else {
            this.allTopics[topic].push({ name: tag, type: type });
          }
        });
      };
    },

    generate() {
      let valid = true;
      this.opcSubscribers.forEach(opcSub => {
        if (!opcSub.opcUrl || !opcSub.opcNamespace) {
          alert("Please Fill All Required Field");
          valid = false;
        }
      });

      if (valid) {
        this.processData();

        this.download(this.finalText, "test.xml");
      }
    },

    processData() {
      let finalText = header;
      let symbolIndex = 1;

      //OPC UA
      this.allTags.forEach((allTag, index) => {
        let gateOpc =
          `
        <Gate Id="` +
          (index + 1) +
          `" ReferenceTo="0" Switch="ON" Type="OpcUaDevice" Name="Screen01">
        <Url>` +
          this.opcSubscribers[index].opcUrl +
          `</Url>
        <SecuritySettings MessageSecurityMode="None" SecurityPolicy="http://opcfoundation.org/UA/SecurityPolicy#None" />
        <AuthenticationSettings PwdInfoId="a821229b-3418-4020-ac9a-fd5e14f43cdc" Type="None" />
        <Channels>
        `;

        finalText += gateOpc;

        let channelOpc =
          `
          <Channel Id="` +
          (index + 1) +
          `" ReferenceTo="0" Name="Subscriber_` +
          (index + 1) +
          `" Switch="ON" Role="Subscriber" SamplingMode="onChange" CycleTime="-1" Timeout="-1" Retain="false" MappingId="1" QoS="0" InhibitTime="0" BufferSize="0" OnChangePartialMode="true">
	        <Formatter EnableSymbolTimestamp="true" ProgId="{EC125306-74D0-46E4-9518-9258DB043679},TcDataAgent.Formatter_Binary.1" Name="Raw Binary" Type="InOut" />
          `;
        finalText += channelOpc;

        allTag.forEach((tag, idx) => {
          let symbolOpc =
            `            
            <Symbol Id="` +
            symbolIndex +
            `" ReferenceTo="0" Switch="ON" DataType="` +
            tag.type +
            `" DataTypeGuid="` +
            this.dataType[tag.type] +
            `" Urn="Objects.PLC1.` +
            tag.name +
            `" UaIdentifier="ns=4;s=` +
            tag.name +
            `" UaNsName="` +
            this.opcSubscribers[index].opcNamespace +
            `" UaAttributeId="13" Conversion="Lossless">
            </Symbol>
          `;

          finalText += symbolOpc;
          symbolIndex++;
        });

        finalText += `
          </Channel>
          </Channels>
        </Gate>
          `;
      });

      //MQTT
      let gateMqtt =
        `
        <Gate Id="` +
        (this.opcSubscribers.length + 1) +
        `" ReferenceTo="0" Switch="ON" Type="Mqtt" Name="newMQTTGate_2" ClientId="newMQTTGate_2">
        <Url>` +
        this.mqttData.broker +
        `</Url>
        <Port>1883</Port>
        <AuthenticationSettings PwdInfoId="d7e978e1-b1b5-481f-b6a5-cb58033386cd" Type="None" TLSType="None" />
        <Channels>
        `;

      finalText += gateMqtt;

      let idx = this.opcSubscribers.length + 1;
      const lastVarCount = symbolIndex - 1;

      for (var key in this.allTopics) {
        let channelMqtt =
          `
          <Channel Id="` +
          idx +
          `" ReferenceTo="0" Name="Publisher" Switch="ON" Role="Publisher" SamplingMode="onChange" CycleTime="0" Timeout="-1" ServiceName="` +
          key +
          `" Retain="false" SendStateInfo="" MappingId="-1" QoS="1" InhibitTime="0" BufferSize="0" OnChangePartialMode="true">
            <Formatter EnableSymbolTimestamp="false" ProgId="{C817C03C-0815-4AD1-BD7A-EC02321575BA},TcDataAgent.Formatter_JsonSimple.1" Name="Simple JSON" Type="InOut" />
          `;

        finalText += channelMqtt;

        this.allTopics[key].forEach((value, index) => {
          let symbolMqtt =
            `
            <Symbol Id="` +
            symbolIndex +
            `" ReferenceTo="0" Switch="ON" DataType="` +
            value.type +
            `" DataTypeGuid="` +
            this.dataType[value.type] +
            `" Urn="` +
            value.name +
            `" Conversion="Lossless">
            </Symbol>
            `;

          finalText += symbolMqtt;
          symbolIndex++;
        });

        finalText += `
          </Channel>
          `;

        idx++;
      }

      let mid = `
        </Channels>
        </Gate>
        </Gates>
      <Mappings>
        <Mapping Id="1" Switch="ON" Name="Mapping_Subscriber" Update="single">
      `;

      finalText += mid;

      let n = 1;

      this.allTags.forEach((tags, index) => {
        tags.forEach((tag, idx) => {
          finalText +=
            `
            <Link Switch="ON" SrcId="` +
            n +
            `" DstId="` +
            (lastVarCount + n) +
            `" />`;
          n++;
        });
      });

      let footer = `
        </Mapping>
      </Mappings>
  </BaseConfig>
</Configuration>
      `;

      finalText += footer;

      this.finalText = finalText;
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
    }
  }
};
</script>

<style scoped>
.main-page {
  margin-top: 50px;
  padding: 0px 20px;
  outline: solid 1px black;
}

.pubsub {
  display: flex;
  max-width: 750px;
  margin: 0px auto;
  /*outline: solid red 1px;*/
}

.subscriber {
  /*outline: solid red 1px;*/
}
.publisher {
  flex: 1;
  /*outline: solid red 1px;*/
}

table {
  width: 100%;
}

input {
  width: 100%;
}

#progress_bar {
  margin: 10px 0;
  padding: 3px;
  border: 1px solid #000;
  font-size: 14px;
  clear: both;
  opacity: 0;
  -moz-transition: opacity 1s linear;
  -o-transition: opacity 1s linear;
  -webkit-transition: opacity 1s linear;
}
#progress_bar.loading {
  opacity: 1;
}
#progress_bar .percent {
  background-color: #99ccff;
  height: auto;
  width: 0;
}
</style>
