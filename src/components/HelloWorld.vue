<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
  <div>
    <el-select 
      class="select"
      v-model="enigma.selectedApp" 
      placeholder="Please select an app" 
      no-match-text="Nothing found"
      no-data-text="There are no apps available or there is an error :'("
      :disabled="selectDisabled" 
      filterable
      size="large"
      @change="selectApp">
      <el-option
        v-for="app in enigma.apps"
        :key="app.qDocId"
        :label="app.qTitle"
        :value="app.qDocId">
      </el-option>
    </el-select>
    <i class="el-icon-loading" v-bind:class="{ hidden: !selectDisabled }"></i>
  </div>
  <el-button 
    type="primary" 
    plain
    v-bind:class="{ hidden: !enigma.selectedApp }"
    v-on:click="serialize">Serialize</el-button>

  </div>
</template>

<script>
// const WebSocket = require('ws');
import app2json from "serializeapp";
import FileSaver from "filesaver.js";
import moment from "moment";
const enigma = require("enigma.js");
const schema = require("enigma.js/schemas/12.20.0.json");

var sessionConfig = {
  schema,
  url: "ws://localhost:4848/app/",
  createSocket: url => new WebSocket(url),
  unsecure: true
};

// console.log(session);

// session.on("opened", () => console.log("Session: Opened"));
// session.on("closed", () => console.log("Session: Closed"));
// session.on('notification:OnConnected', (data) => console.log(data));
// session.on('traffic:sent', data => console.log('sent:', data));
// session.on('traffic:received', data => console.log('received:', data));

export default {
  name: "HelloWorld",
  data() {
    return {
      msg: "Welcome to Your Vue.js App",
      selectDisabled: true,
      enigma: {
        session: {},
        global: {},
        app: {},
        apps: {},
        selectedApp: ""
      }
    };
  },
  mounted: function() {
    var _this = this;

    openSession(_this, sessionConfig)
      .then(function(session) {
        return session.open();
      })
      .then(function(global) {
        console.log("Session open 1");
        _this.enigma.global = global;
        // console.log(global)
        return global.qvVersion();
      })
      .then(function(version) {
        _this.msg = version;
        return _this.enigma.global.getDocList();
      })
      .then(function(docLists) {
        _this.selectDisabled = false;
        _this.enigma.apps = docLists;
      });
  },
  methods: {
    selectApp: function(appId) {
      var _this = this;
      _this.selectDisabled = true;
      openSession(_this, sessionConfig, appId)
        .then(function(session) {
          return session.open();
        })
        .then(function(global) {
          console.log("Session open 2");
          _this.enigma.global = global;
          // console.log(global);
          return _this.enigma.global.openDoc(appId);
        })
        .then(function(enigmaApp) {
          _this.enigma.app = enigmaApp;
          _this.selectDisabled = false;
          return _this.enigma.app.getAppProperties();
        })
        .then(function(props) {
          console.log(props);
        });
    },
    serialize: function() {
      var _this = this;
      app2json(_this.enigma.app).then(function(serialized) {
        var stamp = moment().format('YYYY-MM-DD_hh-mm-ss');
        var fileName = `${_this.enigma.app.id}_${stamp}.json`
        serialized = JSON.stringify(serialized, null, 4)
        
        var blob = new Blob([serialized], {
          type: "application/json;charset=utf-8"
        });
        FileSaver.saveAs(blob, fileName);
      });
    }
  }
};

function openSession(_this, sessionConfig, appId) {
  if (_this.enigma.app.id != null) {
    _this.enigma.global = {};

    return new Promise(function(res, rej) {
      res(_this.enigma.app.session.close());
    }).then(function() {
      var localConfig = sessionConfig;
      localConfig.url = localConfig.url + "" + appId;
      return enigma.create(sessionConfig);
    });
  } else {
    return new Promise(function(res, rej) {
      console.log("nothing to close");
      res(enigma.create(sessionConfig));
    });
  }
}

function closeSession(_this, session, appId) {}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.hidden {
  display: none;
}

.select {
  width: 900px;
}
</style>
