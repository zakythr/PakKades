<template>
  <div>
    <div id="app" ref="spreadsheet"></div>
    <h1 class="text-center">Data Warga Desa Puncung</h1>
    <div>
        <input class="btn btn-primary tambah" type="button" value="Add New Row" @click="() => spreadsheet.insertRow()" />
        <input class="btn btn-primary tambah" type="button" value="Delete Selected Row" @click="() => spreadsheet.deleteRow()" />
    </div>
    <input type="text" v-model="search" placeholder="search blogs" />
    <div v-for="blog in blogs" class="single-blog">
      <h2>{{blog.title | to-uppercase}}</h2>
      <article>{{blog.body | snippet}}</article>
    </div>
  </div>
</template>

<script>
import jexcel from 'jexcel'
import 'jexcel/dist/jexcel.css'
import axios from 'axios'
// var host = 'http://10.199.14.46:8018/'
var host = 'http://localhost:8026/'
export default {
  // name: 'App',
  data() {
    return {
      dataDesa: [],
      search: ''
    }
  },
  mounted() {
    this.load()
  },
  methods: {
    load() {
      axios.get(host + 'api/datadesa/').then(res => {
        console.log(res.data)
        var jexcelOptions = {
          data: res.data,
          allowToolbar: true,
          onchange: this.updateRow,
          oninsertrow: this.newRow,
          ondeleterow: this.deleteRow,
          responsive: true,
          columns: [
            { type: 'hidden', title: 'ID', width: '10px' },
            { type: 'text', title: 'Nama', width: '140px' },
            { type: 'text', title: 'NIK', width: '110px' },
            { type: 'text', title: 'Nomer KK', width: '110px' },
            { type: 'calendar', title: 'Tgl Lahir', width: '120px' },
            { type: 'dropdown', title: 'Status', source: [ 'Menikah', 'Belum Menikah' ], width: '120px' },
            { type: 'dropdown', title: 'Pendidikan', source: [ 'SD', 'SMP', 'SMA', 'S1', 'S2', 'S3' ], width: '120px' },
            { type: 'text', title: 'Pekerjaan', width: '120px' },
            { type: 'text', title: 'Alamat', width: '220px' }
          ]
        }
        let spreadsheet = jexcel(this.$el, jexcelOptions)
        Object.assign(this, { spreadsheet })
      })
    },
    newRow() {
      axios.post(host + 'api/datadesa/', this.form).then(res => {
        console.log(res.data)
      })
    },
    updateRow(instance, cell, columns, row, value) {
      axios.get(host + 'api/datadesa/').then(res => {
        var index = Object.values(res.data[row])
        index[columns] = value
        console.log(index)
        axios.put(host + 'api/datadesa/' + index[0], {
          id: index[0],
          nama: index[1],
          nik: index[2],
          no_kk: index[3],
          tgl_lahir: index[4],
          setatus: index[5],
          pendidikan: index[6],
          pekerjaan: index[7],
          alamat: index[8]
        }).then(res => {
          console.log(res.data)
        })
      })
    },
    deleteRow(instance, row) {
      axios.get(host + 'api/datadesa/').then(res => {
        var index = Object.values(res.data[row])
        // console.log(index)
        console.log(row)
        axios.delete(host + 'api/datadesa/' + index[0])
      })
    }
  },
  computed: {
    filteredBlogs: function() {
      return this.blogs.filter((blog) => {
        return blog.title.match(this.search)
      })
    }
  }
}
</script>
<style>
  .tambah {
    margin-top: 10pt;
    margin-bottom: 10pt;
    margin-left: 10pt;
    }
</style>
