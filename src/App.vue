<template>
  <div class="container">
    <div class="row">
      <div class="col-md-12">
        <legend>
          <h2><span class="mega-octicon octicon-mark-github"></span>&nbsp;GitHub 文件浏览器</h2>
        </legend>

       <!--  prevent 阻止表单提交
        methods 只有纯粹的数据逻辑，而不是去处理 DOM 事件细节 -->
        <form @submit.prevent="getRepo" class="form-inline">
          <div class="form-group">
            <label for="Input username">Input username: </label>
            <input v-model.trim="username" type="text" class="form-control" placeholder="hanzichi">
          </div>
          <button class="btn btn-primary" type="submit">search</button>
        </form>
        <br/>

        <ol class="breadcrumb">
          <li><strong>当前路径:</strong></li>
          <li v-for="(item, index) of nav">
            <a v-if="index !== nav.length - 1" href="javascript:;" @click="changeNav($event, index)">{{ item }}</a>
            <template v-else class="active">{{ item }}</template>
          </li>
        </ol>

        <!-- loading -->
        <div class="loader loader-default" :class="{'is-active': isLoading}"></div>

        <!-- show files, or code -->
        <transition name="fade">
        <table class="table table-hover" v-if="files.length || !code" key="keya">
          <!-- 必须要有标签（tag），且会在 dom 中生成 -->
          <transition-group name="fade" tag="tbody">
            <!-- 要有 key -->
            <tr v-for="item of files" :key="item">
              <td v-if="item.type === 'dir' || !item.type">
                <span class="octicon octicon-file-directory"></span>
                <a href="javascript:;" @click="changePath">{{ item.name }}</a>
              </td>
              <td v-else>
                <span class="octicon octicon-file-text"></span>
                <a href="javascript:;" @click="changePath">{{ item.name }}</a>
              </td>
              <td class="text-right">
                <a download :href="item.download_url" v-if="item.type === 'file'">
                  <span class="octicon octicon-cloud-download"></span>
                </a>
              </td>
            </tr>
          </transition-group>
        </table>
        <pre v-else key="keyb">{{ code }}</pre>
        <transition>
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
      code: '',
      isLoading: false,
      show: false
    }
  },

  created() {

  },

  methods: {
    // 输入框
    getRepo() {
      let username = this.username;

      if (username === '')
        this.openAlertBox('请输入用户名');
      else {
        this.nav = [];  // reset
        this.nav.push(username);
      }
    },

    // 导航栏
    changeNav(e, index) {
      this.nav = this.nav.slice(0, index + 1);
    },

    // list
    changePath(e) { // 不需要传入 e 参数，默认
      let val = e.target.innerHTML;
      this.nav.push(val);
    },

    sortFiles() {
      if (!('type' in this.files[0]))
        return;

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
    },

    openAlertBox(msg) {
      alert(msg);
    }
  },

  watch: {
    nav(newVal, oldVal) {
      // console.log(newVal, oldVal);

      this.isLoading = true;

      let api;

      if (this.nav.length === 1)
        api = 'https://api.github.com/users/' + this.nav[0] + '/repos';
      else {
        let a = this.nav.slice(0, 2);
        let b = this.nav.slice(2);
        api = 'https://api.github.com/repos/' + a.join('/') + '/contents/' + b.join('/');
      }

      axios.get(api)
        .then((res) => {
          if (res.data.content) {  // 是具体的文件
            this.files = [];
            this.code = decodeURIComponent(escape(window.atob(res.data.content)));
            // console.log(this.code)
          } else {  // dir
            this.files = res.data;
            this.sortFiles();
            // console.log(this.files)
          }
          this.isLoading = false;
        })
        .catch((error) => {
          console.log(error.message);
          let msg = error.message;
          // alert(msg)
          if (msg.indexOf('404') !== -1)
            this.openAlertBox('请输入正确的用户名');
          else if (msg.indexOf('403') !== -1)
            this.openAlertBox('请求太频繁受限了！')
          this.isLoading = false;
        });
    }
  }
}
</script>

<style>
@import '../node_modules/pure-css-loader/dist/css-loader.css';

.fade-enter-active {
  transition: opacity .5s
}

.fade-leave-active {
  transition: opacity 0s
}

.fade-enter, .fade-leave-active {
  opacity: 0
}

.table>tbody tr>td {
  padding: 8px;
  line-height: 1.42857143;
  vertical-align: top;
  border-top: 1px solid #ddd;
}
</style>
