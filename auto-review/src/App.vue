<template>
  <div class="flex">
    <div class="content-left">
            <Codereview v-model = "content"
            @change="changeHanlder"
            ></Codereview>
          </div>
          <div class="content-right">
            {{result}}
          </div>
    </div>
</template>

<script>
  import Codereview from './components/CodeReview.vue'
var esprima = require('esprima');
export default {
  name: 'App',
  components: {
    Codereview
  },
  mounted()
  {
    let me = this;
    me.result = esprima.parseScript(me.content);
  }, 
  data()
  {
    return {
      program:'const answer = 42',
      result:null,
      content: 'test',
    }
  },
  methods:{
    changeHanlder(value)
    {
      let me = this;
      console.log(value);
      me.result = esprima.parseScript(value);
    }
  }
}
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
<style scoped>
  .flex
  {
    display:flex;
  }
.content-left
{
  width: 50%;
}
.content-right
{
  flex:1;
}
</style>
