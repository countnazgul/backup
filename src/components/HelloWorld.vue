<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <el-select v-model="enigma.selectedApp" placeholder="Select" filterable @change="selectApp">
      <el-option
        v-for="app in enigma.apps"
        :key="app.qDocName"
        :label="app.qTitle"
        :value="app.qDocId">
      </el-option>
    </el-select>


  </div>
</template>

<script>
// const WebSocket = require('ws');
const enigma = require("enigma.js");
const schema = require("enigma.js/schemas/12.20.0.json");

const session = enigma.create({
  schema,
  url: "ws://localhost:4848/app/engineData",
  createSocket: url => new WebSocket(url),
  unsecure: true
});

// console.log(session);

// session.on('notification:OnConnected', (data) => console.log(data));
// session.on('traffic:sent', data => console.log('sent:', data));
// session.on('traffic:received', data => console.log('received:', data));

export default {
  name: "HelloWorld",
  data() {
    return {
      msg: "Welcome to Your Vue.js App",
      enigma: {
        global: {},
        app: {},
        apps: {},
        selectedApp: ""
      }
    };
  },
  mounted: function() {
    var _this = this;

    openSession(_this)
      .then(function(global) {
        console.log("Session open 1");
        _this.enigma.global = global;
        console.log(global)
        return global.qvVersion();
      })
      .then(function(version) {
        return (_this.msg = version);
      })
      .then(function() {
        return _this.enigma.global.getDocList();
      })
      .then(function(docLists) {
        return (_this.enigma.apps = docLists);
      });
  },
  methods: {
    selectApp: function(appId) {
      var _this = this;

      openSession(_this)
        .then(function(global) {
          console.log("Session open 2");
          _this.enigma.global = global;
          console.log(global)
          return global.qvVersion();
        })
        .then(function(version) {
          return (_this.msg = version);
        })
        .then(function() {
          return _this.enigma.global.getDocList();
        })
        .then(function(docLists) {
          return (_this.enigma.apps = docLists);
        })
        .then(function() {
          return _this.enigma.global.openDoc(appId);
        })
        .then(function(enigmaApp) {
          _this.enigma.app = enigmaApp;
          return _this.enigma.app.getAppProperties();
        })
        .then(function(props) {
          console.log(props);
        });
    }
  }
};

function openSession(_this) {
  return closeSession(_this).then(function(test) {
    return session.open();
  });
}

function closeSession(_this) {
  if (_this.enigma.global.id) {
    return session.close().then(function() {
      return new Promise(function(res, rej) {
        return res("aaa");
      });
    });
  } else {
    return new Promise(function(res, rej) {
      console.log("nothing to close");
      res("aaa");
    });
  }
}
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
</style>
