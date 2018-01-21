<template>
  <div>
    <v-header :seller="seller"></v-header>

    <nav class="border-1px">
      <div class="tab-item"><router-link to="/goods">商品</router-link></div>
      <div class="tab-item"><router-link to="/ratings">评论</router-link></div>
      <div class="tab-item"><router-link to="/seller">商家</router-link></div>
    </nav>
    <keep-alive>
      <router-view :seller='seller'></router-view>
    </keep-alive>
  </div>
</template>

<script>
  import header from './components/header/header'
  import axios from 'axios'

  const ERR_OK = 0
  export default {
    data () {
      return {
        seller: {}
      }
    },
    created () {
      let _this = this
      axios.get('/api/seller').then((res) => {
        if (res.data.errno === ERR_OK) {
          _this.seller = res.data.data
        }
      })
    },
    components: {
      'v-header': header
    }
  }
</script>

<style lang="stylus" scoped>
  @import "./common/style/mixin.styl"
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }
  nav {
    display: flex;
    width: 100%;
    height: 40px;
    line-height: 40px;
    border-1px(rgba(7, 17, 27, 0.1));
    div.tab-item {
      flex: 1;
      text-align: center;
      &>a{
        display: block;
        font-size: 14px;
        color: rgb(77, 85, 93);
        &.active {
          color: rgb(240, 20, 20);
        }
      }
    }
  }
</style>
