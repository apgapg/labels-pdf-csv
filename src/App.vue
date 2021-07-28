<template>
  <div id="app">
    <h4>Convert csv to Address Labels easily</h4>
    <div style="width: 400px" class="text-center p-4 mx-auto">
      <div class="text-danger">*Upload .csv file only</div>
      <input class="form-control mt-3" type="file" id="formFile" accept="text/csv" @change="onFileChange"/>

      <div class="mt-4"></div>
      <div class="form-check" v-for="item of columns" :key="item">
        <input class="form-check-input" type="checkbox" value="" id="flexCheckChecked"
               :checked="selectedColumns.includes(item)" @change="handleChange(item)">
        <label class="form-check-label" for="flexCheckChecked">
          {{ item }}
        </label>
      </div>
      <div v-if="selectedColumns.length>0" class="btn btn-success mt-4" @click="onDownloadClick">Download as PDF</div>

    </div>
  </div>
</template>

<script>
import {jsPDF} from "jspdf";

export default {
  name: "App",

  components: {},
  data() {
    return {
      file: null,
      fileinput: null,
      json: null,
      columns: [],
      selectedColumns: [],
    }
  },
  methods: {
    onFileChange(e) {
      const file = e.target.files[0];
      const reader = new FileReader();
      reader.onload = e => {
        console.log(e.target.result);
        this.json = this.csvJSON(e.target.result);
        console.log(this.json)
        if (this.json && this.json.length > 1) {
          this.columns = Object.keys(this.json[0])
        }
      }
      reader.readAsText(file);

    },
    csvJSON(csv) {

      var lines = csv.split("\n");

      var result = [];

      // NOTE: If your columns contain commas in their values, you'll need
      // to deal with those before doing the next step
      // (you might convert them to &&& or something, then covert them back later)
      // jsfiddle showing the issue https://jsfiddle.net/
      var headers = lines[0].split(",");

      for (var i = 1; i < lines.length; i++) {

        var obj = {};
        var currentline = lines[i].split(",");

        for (var j = 0; j < headers.length; j++) {
          obj[headers[j]] = currentline[j];
        }

        result.push(obj);

      }

      //return result; //JavaScript object
      return result //JSON
    },
    onDownloadClick() {

      try {
        const doc = new jsPDF();
        doc.setFontSize(12)

        const startPoints = 10
        const nextColStartPoints = 110
        const labelPerPage = 8
        let currentRowPoints = 20
        const incrementRowPoints=6

        for (let i = 0; i < this.json.length; i=i+2) {
          const item1 = this.json[i]
          const item2 = this.json[i+1]
          for (let j = 0; j < this.selectedColumns.length; j = j + 1) {
            const key = this.selectedColumns[j]
            doc.text(`${item1[key]}`, startPoints, currentRowPoints);
            if (item2) {
              doc.text(`${item2[key]}`, nextColStartPoints, currentRowPoints);
            }
            currentRowPoints = currentRowPoints + incrementRowPoints
          }
          doc.text("",startPoints,currentRowPoints)
          currentRowPoints = currentRowPoints + (incrementRowPoints*2)
          console.log(i)

          if ((i+2) % (labelPerPage) === 0) {
            console.log("Break:",i)
            doc.addPage("a4", "0");
            currentRowPoints=20
          }
        }


        //doc.addPage("a4", "0");
        //doc.text("Do you like that?", 20, 20);
        doc.save(`labels-${new Date().toLocaleDateString()}.pdf`)
      } catch (e) {
        console.error(e)
        alert('Some error occurred');
      }
    },
    handleChange(item) {
      if (this.selectedColumns.includes(item)) {

        const index = this.selectedColumns.indexOf(item);
        if (index > -1) {
          this.selectedColumns.splice(index, 1);
        }

      } else {
        this.selectedColumns.push(item)
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
