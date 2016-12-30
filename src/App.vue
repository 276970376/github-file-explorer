<template>
  <div class="container">
    <div class="row">
      <div class="col-md-12">
        <legend>
          <h2><span class="mega-octicon octicon-mark-github"></span>&nbsp;GitHub 文件浏览器</h2>
        </legend>

        <!-- prevent 阻止表单提交 -->
        <form @submit.prevent="getRepo" class="form-inline">
          <div class="form-group">
            <label for="Input username">Input username: </label>
            <input v-model="username" type="text" class="form-control" placeholder="hanzichi">
          </div>
          <button class="btn btn-primary" type="submit">search</button>
        </form>
        <br/>

        <ol class="breadcrumb">
          <li><strong>当前路径:</strong></li>
          <li v-for="(item, index) in nav">
            <a v-if="index !== nav.length - 1" href="javascript:;" @click="changeNav($event, index)">{{ item }}</a>
            <template v-else class="active">{{ item }}</template>
          </li>
        </ol>

        <table class="table table-hover" v-if="files.length || !code">
          <tbody>
            <tr v-for="item in files">
              <td v-if="item.type === 'dir' || !item.type">
                <span class="octicon octicon-file-directory"></span>
                <a href="javascript:;" @click="changePath($event)">{{ item.name }}</a>
              </td>
              <td v-else>
                <span class="octicon octicon-file-text"></span>
                <a href="javascript:;" @click="changePath($event)">{{ item.name }}</a>
              </td>
              <td class="text-right">
                <a download :href="item.download_url" v-if="item.type === 'file'">
                  <span class="octicon octicon-cloud-download"></span>
                </a>
              </td>
            </tr>
          </tbody>
        </table>
        <pre v-else>{{ code }}</pre>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      files: [],
      nav: [],
      username: '',
      code: ''
    }
  },

  methods: {
    // 输入框
    getRepo() {
      this.nav = [];  // reset
      this.nav.push(this.username);
    },

    // 导航栏
    changeNav(e, index) {
      this.nav = this.nav.slice(0, index + 1);
    },

    // list
    changePath(e) {
      let val = e.target.innerHTML;
      this.nav.push(val);
    },

    sortFiles() {
      if (!('type' in this.files[0]))
        return;

      // console.log(this.files)

      this.files.sort((a, b) => {
        // file, dir
        // sort 有坑，注意
        if (a.type !== b.type) {
          if (a.type > b.type)
            return 1;
          else
            return -1;
        } else {
          if (a.name > b.name)
            return 1;
          else
            return -1;
        }
      });

      // console.log(this.files)
    }
  },

  watch: {
    nav() {
      let api;

      if (this.nav.length === 1)
        api = 'https://api.github.com/users/' + this.nav[0] + '/repos';
      else {
        let a = this.nav.slice(0, 2);
        let b = this.nav.slice(2);
        api = 'https://api.github.com/repos/' + a.join('/') + '/contents/' + b.join('/');
      }

      axios.get(api).then((res) => {
        if (res.data.content) {
          this.files = [];
          this.code = decodeURIComponent(escape(window.atob(res.data.content)));
          // console.log(this.code)
        } else {
          this.files = res.data;
          this.sortFiles();
          // console.log(this.files)
        }
      });
    }
  }
}
</script>
