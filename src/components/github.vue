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

        <!-- 子组件 -->
        <list :nav="nav"></list>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import list from './list.vue';

export default {
  data() {
    return {
      nav: [],
      username: ''
    }
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
    }
  },

  components: {
    list
  }
}
</script>
