<template>
  <div id="app">
    <div class="table">
      <div class="row">
        <div class="left"></div>
        <h2>Blockchain1</h2>
        <div v-for="(row, indRow) in rows"
             :key="'r_' + indRow"
             class="row">
          <div v-for="(column, indCol) in numCols(row)"
               :key="'c_' + indCol"
               v-bind:class="{ cell_left: column % 2 == 1, cell_right: column % 2 == 0 }">
            <house-blockchain v-on:get-hash="getNewHash($event, $eventData)"
                              :index="getIndex(row, column)"
                              :houseNo="getIndex(row, column)"
                              :active="active[getIndex(row, column)]"
                              :hash="hash[getIndex(row, column)]"
                              :eventData="eventData[getIndex(row, column)]"
                              :blockNo="blockNo + getIndex(row, column)"
                              :previousHash="previousHash[getIndex(row, column)]"
                              :originalHash="originalHash[getIndex(row, column)]"
                              :blocks="blocks"
                              :nonce="nonce[getIndex(row, column)]">
            </house-blockchain>
          </div>
        </div>
        <div class="right"></div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import VueRouter from 'vue-router';
import sha256 from 'sha256';
import HouseBlockchain from './components/HouseBlockchain.vue';
Vue.use(VueRouter);

var mockEventData = "Date|Time|EventID|Appl.ID |ActivePWR |ReactivePwr|Voltage|Intensity |HASH\u003e\u003e\u003e 10Mar2015 0:00:00 1201210 9b75a5178a 2.58 0.136 241.97 10.6\u003e\u003e\u003e 10Mar2015 0:01:00 1201211 9b75a5178f 2.552 0.1 241.75 10.4\u003e\u003e\u003e 10Mar2015 0:02:00 1201212 9b75a51791 2.55 0.1 241.64 10.4\u003e\u003e\u003e 10Mar2015 0:03:00 1201213 9b75a5178d 2.55 0.1 241.71 10.4\u003e\u003e\u003e 10Mar2015 0:04:00 1201214 9b75a5178b 2.554 0.1 241.98 10.4\u003e\u003e\u003e 10Mar2015 0:05:00 1201215 9b75a5178f 2.55 0.1 241.83 10.4\u003e\u003e\u003e 10Mar2015 0:06:00 1201216 9b75a51790 2.534 0.09 241.07 10.4\u003e\u003e\u003e 10Mar2015 0:07:00 1201217 9b75a51791 2.484 0.21 241.29 10.2\u003e\u003e\u003e 10Mar2015 0:08:00 1201218 9b75a5178a 2.468 0.32 241.23 10.2\u003e\u003e\u003e 10Mar2015 0:09:00 1201219 9b75a51793 2.48 0.54 242.28 10.2";

const MAX_BLOCKS = 100;

var valid_hash = function(hash_code) {
   if (hash_code !== undefined) {
     return hash_code.startsWith("000");
   }
   return false;
};

var router = new VueRouter({
  mode: 'history',
  routes: []
});

export default {
  router,
  data: function() {
    return {
      blockNo: 432,
      hash: new Array(MAX_BLOCKS).fill(''),
      previousHash: new Array(MAX_BLOCKS).fill(''),
      nonce: new Array(MAX_BLOCKS).fill(0),
      eventData: new Array(MAX_BLOCKS).fill(mockEventData),
      originalHash: new Array(MAX_BLOCKS).fill(''),
      active: new Array(MAX_BLOCKS).fill(false),
      blocks: 1,
      rows: 1
    }
  },
  components: {
    houseBlockchain: HouseBlockchain
  },
  methods: {
    encode: function(id) {
      var nonce = -1;
      var hash_code = undefined;
      while (!valid_hash(hash_code)) {
        nonce++;
        var text_to_encode = this.previousHash[id] + this.eventData[id] + nonce;
        hash_code = sha256(text_to_encode);
      }
      return {"nonce": nonce, "hash": hash_code};
    },
    getNewHash: function(data) {
      this.$set(this.eventData, data.id, data.data);
      this.getHash(data.id);
    },
    getHash: function(id) {
      var data = this.encode(id);
      this.$set(this.nonce, id, data.nonce);
      this.$set(this.hash, id, data.hash);
      if (this.originalHash[id] === '') {
        this.$set(this.originalHash, id, this.hash[id]);
      } else {
        this.$set(this.active, id, (this.originalHash[id] != this.hash[id]));
      }
      if (id < this.blocks - 1) {
        this.$set(this.previousHash, id + 1, this.hash[id]);
        this.getHash(id + 1);
      }
    },
    numCols: function(rows) {
      if (2 * rows > this.blocks) {
        return 1;
      }
      return 2;
    },
    getIndex: function(row, column) {
      return 2 * row + column - 3; 
    },
    setInitialHash: function() {
      this.$set(this.previousHash, 0, "0005100308e7e0bea95a3e88e4e406c37133f0929c80866bda04bc0bce53a14");
    },
  },
  mounted: function() {
    this.blocks = this.$route.query.blocks !== undefined ? Math.min(this.$route.query.blocks, MAX_BLOCKS) : 1;
    this.rows = Math.ceil(this.blocks / 2); 
    this.setInitialHash();
    this.getHash(0);
  },
  listeners: {
    'get-hash': function(houseId) {
      this.getHash(houseId);
    }
  }
}
</script>

<style>
/**
 * Labels and titles elements
 */
h2 {
    font-family: Calibri, Arial, Sans-serif;
}

/**
 * DIV elements that create the page structure
 */
div.table {
  display: table;
}

div.row {
    display: table-row;
}

div.left {
    width: 20%;
    display: table-cell;
}

div.right {
    width: 20%;
    display: table-cell;
}

div.cell_left {
    display: table-cell;
    padding-right: 50px;
}

div.cell_right {
    display: table-cell;
    padding-left: 50px;
}

/**
 * Miscelaneous
 */

.bg_red {
    background-color: red;
}
</style>
