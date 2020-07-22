<template>
  <div id="app">
    <div class="table">
      <div class="row">
        <div class="left"></div>
        <h2>Blockchain1</h2>
        <div v-for="row in rows" class="row">
          <div v-for="column in numCols(row)" v-bind:class="{ cell_left: column % 2 == 1, cell_right: column % 2 == 0 }">
            <form>
              <div>
                <div class="block_no">Block <span v-html="lpad((getIndex(row, column) + 1))" /></div>
                <img class="roof" src="./assets/roof.png">
              </div>
              <table v-bind:class="{bg_red: active[getIndex(row, column)] && blocks > 1 }">
                <tr>
                  <th colspan="4">Consumption Data registered in a Blockchain Database</th>
                </tr>
                <tr>
                  <td class="label">Block No</td>
                  <td class="text">
                    {{ block_no + getIndex(row, column) | blockNoFormat }}
                  </td>
                  <td class="label dark_blue">Nonce</td>
                  <td class="text">
                    <input disabled="true" type="text" v-model="nonce[getIndex(row, column)]" />
                  </td>
                </tr>
                <tr>
                  <td class="label purple">Previous&nbsp;Hash</td>
                  <td class="text" colspan="3">
                    <input class="red" disabled="true" type="text" v-model="previous_hash[getIndex(row, column)]" />
                  </td>
                </tr>
                <tr>
                  <td class="label big_font">Event<br/>Data</td>
                  <td class="text" colspan="3">
                    <textarea v-on:keyup="getHash(row - 1)" v-model="eventData[getIndex(row, column)]"></textarea>
                  </td>
                </tr>
                <tr>
                  <td class="label">Block&nbsp;Hash</td>
                  <td class="text" colspan="3">
                    <input class="hash" disabled="true" type="text" v-model="hash[getIndex(row, column)]" />
                  </td>
                 </tr>
               </table>
               <br />
               <table class="no_border">
                 <tr>
                   <td class="label narrow">SHA&nbsp;256</td>
                   <td class="text" colspan="3"> 
                     <input class="hash"  v-bind:class="{bg_red: active[getIndex(row, column)]}" disabled="true" type="text" v-model="original_hash[getIndex(row, column)]" />
                   </td>
                 </tr>
               </table>
             </form>
           </div>
         </div>
        </div>
        <div class="right"></div>
      </div>
    </div>
  </div>
</template>

<script>
import VueRouter from 'vue-router';
// import sha256 from 'sha256';

var mockEventData = "Date|Time|EventID|Appl.ID |ActivePWR |ReactivePwr|Voltage|Intensity |HASH\u003e\u003e\u003e 10Mar2015 0:00:00 1201210 9b75a5178a 2.58 0.136 241.97 10.6\u003e\u003e\u003e 10Mar2015 0:01:00 1201211 9b75a5178f 2.552 0.1 241.75 10.4\u003e\u003e\u003e 10Mar2015 0:02:00 1201212 9b75a51791 2.55 0.1 241.64 10.4\u003e\u003e\u003e 10Mar2015 0:03:00 1201213 9b75a5178d 2.55 0.1 241.71 10.4\u003e\u003e\u003e 10Mar2015 0:04:00 1201214 9b75a5178b 2.554 0.1 241.98 10.4\u003e\u003e\u003e 10Mar2015 0:05:00 1201215 9b75a5178f 2.55 0.1 241.83 10.4\u003e\u003e\u003e 10Mar2015 0:06:00 1201216 9b75a51790 2.534 0.09 241.07 10.4\u003e\u003e\u003e 10Mar2015 0:07:00 1201217 9b75a51791 2.484 0.21 241.29 10.2\u003e\u003e\u003e 10Mar2015 0:08:00 1201218 9b75a5178a 2.468 0.32 241.23 10.2\u003e\u003e\u003e 10Mar2015 0:09:00 1201219 9b75a51793 2.48 0.54 242.28 10.2";

const MAX_BLOCKS = 100;

var valid_hash = function(hash_code) {
   if (hash_code !== undefined) {
     return hash_code.startsWith("000");
   }
     return false;
};

var encode = function(id) {
  var nonce = -1;
  var hash_code = undefined;
  while (!valid_hash(hash_code)) {
    nonce++;
    var text_to_encode = app.previous_hash[id] + app.eventData[id] + nonce;
//    hash_code = sha256(text_to_encode);
  }
  return {"nonce": nonce, "hash": hash_code};
};

var router = new VueRouter({
  mode: 'history',
  routes: []
});

export default {
  router,
  data: function() {
    return {
      block_no: 432,
      hash: new Array(MAX_BLOCKS).fill(''),
      previous_hash: new Array(MAX_BLOCKS).fill(''),
      nonce: new Array(MAX_BLOCKS).fill(''),
      eventData: new Array(MAX_BLOCKS).fill(mockEventData),
      original_hash: new Array(MAX_BLOCKS).fill(''),
      active: new Array(MAX_BLOCKS).fill(''),
      blocks: 1,
      rows: 1
    }
  },
  methods: {
    getHash: function(id) {
      var data = encode(id);
      this.$set(this.nonce, id, data.nonce);
      this.$set(this.hash, id, data.hash);
      if (this.original_hash[id] === '') {
        this.$set(this.original_hash, id, this.hash[id]);
      } else {
        this.$set(this.active, id, (this.original_hash[id] != this.hash[id]));
      }
      if (id < this.blocks - 1) {
        this.$set(this.previous_hash, id + 1, this.hash[id]);
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
      this.$set(this.previous_hash, 0, "0005100308e7e0bea95a3e88e4e406c37133f0929c80866bda04bc0bce53a14");
    },
    lpad: function(num) {
      if (num < 10) {
        return "&nbsp;&nbsp;" + num;
      } else if (num < 100) {
        return "&nbsp;" + num;
      } else {
        return num;
      }
    }
  },
  filters: {
    blockNoFormat: function(num) {
      return ("000" + num).slice(-6);
    }
  },
  created: function() {
    this.setInitialHash();
    this.getHash(0);
  },
  mounted: function() {
    this.blocks = this.$route.query.blocks !== undefined ? Math.min(this.$route.query.blocks, MAX_BLOCKS) : 1;
    this.rows = Math.ceil(this.blocks / 2); 
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
 * Table elements
 */
table {
    border: solid 2px grey; 
    border-spacing: 5px;
    background-color: #e2f0d9;
    width: 428px;
}

table.no_border {
    border: none;
    background-color: white;
}

th {
    text-align: left;
    font-size: 10pt;
    font-family: Calibri, Arial, Sans-serif;
}

td {
    border: solid 1pt #325490; 
    margin: 0;
    font-size: 8pt;
    background-color: #729fcf;
    font-family: Calibri, Arial, "Arial Narrow";
}

td.text {
    background-color: #e7e6e6; 
}

td.label {
    text-align: center;
    color: white;
    border-radius: 5px;
    font-weight: bold;
}

td.label.narrow {
    width: 16%;
}

td.big_font {
    font-size: 16pt;
    text-align: center;
}

td.dark_blue {
    background-color: #002060;
}

td.purple {
    background-color: #7030a0;
}

td.red {
    color: red;
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
 * Graphical elements
 */

.roof {
    width: 100%;
    margin-bottom: 5px;
    object-fit: contain;
}

.block_no {
    position: relative;
    left: 168px;
    top: 80px;
    font-size: 16pt;
    font-weight: bold;
    font-family: Cablibri, Arial, Sans-serif;
}

/**
 * Input elements
 */
input[type=text] {
    border: none;
    font-size: 8pt;
    background-color: #e7e6e6;
    color: black;
    width: 80%;
    font-family: Calibri, Arial, Sans-serif;
    font-weight: bold;
    font-size: 8pt;
}

input[type=text].red {
    color: red;
    font-weight: normal;
    width: 100%;
}

input[type=text].bg_red {
    background-color: red;
}

input[type=text].hash {
    font-weight: normal;
    width: 99%;
}

textarea {
    font-family: Calibri, Arial, Sans-serif;
    font-size: 8pt;
    background-color: #e7e6e6;
    color: black;
    border: medium none;
    width: 340px;
    display: block;
    overflow: hidden;
    resize: none;
    height: 150px;
}

/**
 * Miscelaneous
 */

.bg_red {
    background-color: red;
}
</style>
