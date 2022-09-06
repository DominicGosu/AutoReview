<template>
  <div class="content">
  <div class="navbar">
    <div class="logo">AutoReview</div>
  </div>
  <div class="drop-zone" @drop.prevent="onDropFile"
  >
      <div class="title-drop">Kéo và thả file vào đây</div>
      <div class="title-or">hoặc</div>
      <label class="drop-zone-button" data-message="selectFiles">
        Chọn file
        <input type="file" @change="inputFile" class="file-input" ref="inputfile"/>
      </label>
</div>
<div class="button-navbar">
  <button class="btn button-primary" @click="getCode">
    Review
  </button>
  <button class=" btn button-primary">Clear</button>
  <button class=" btn button-primary" @click="viewDataParse">Xem kết quả đọc file</button>
</div>
  <div class="content-editor">
    <prism-editor class="my-editor" v-model="code" :highlight="highlighter" line-numbers></prism-editor>
    <div class="parse-code" v-if="showParseData">
      <pre>
        <code>{{primaData}}</code>
      </pre>
    </div>
  </div>
  <div class="message-view">
    <div class="" v-for="(item,index) in errorList" :key="index">
      <div :class="{'error-message':item.type=='error','warning-message':item.type=='warning'}">
        {{item.message}}
      </div>
    </div>
  </div>
</div>
</template>

<script>
  // import Prism Editor
  import { PrismEditor } from 'vue-prism-editor';
  import 'vue-prism-editor/dist/prismeditor.min.css'; // import the styles somewhere
  // eslint-disable-next-line
  import _ from 'lodash';
  // import highlighting library (you can use any library you want just return html string)
  import { highlight, languages } from 'prismjs/components/prism-core';
  import 'prismjs/components/prism-clike';
  import 'prismjs/components/prism-javascript';
  import 'prismjs/themes/prism-tomorrow.css'; // import syntax highlighting styles
var esprima = require('esprima');
  export default {
    components: {
      PrismEditor,
    },
    mounted()
    {
      let me = this;
      const events = ['dragenter', 'dragover', 'dragleave', 'drop']
      events.forEach((eventName) => {
        document.body.addEventListener(eventName, me.preventDefaults);
    });
    },
    unmounted()
    {
      let me = this;
      const events = ['dragenter', 'dragover', 'dragleave', 'drop']
      events.forEach((eventName) => {
        document.body.removeEventListener(eventName, me.preventDefaults);
    });
    },
    data: () => ({
        code: '',
        fileinput:null,
        primaData:null,
        showParseData:false,
        errorList:[],
      }
     ),

    methods: {
      viewDataParse()
      {
        let me = this;
        me.showParseData = !me.showParseData;
        me.getCode();
      },
      preventDefaults(e) {
          e.preventDefault()
      },
      onDropFile(event)
      {
        let me = this;
        let files = event.dataTransfer.files;
        if(files)
        {
          me.readFile(files[0]);
        }
      },
      highlighter(code) {
        return highlight(code, languages.js); // languages.<insert language> to return html with markup
      },
      inputFile(event)
      {
        let me = this;
        let files = event.target.files;
        if(files)
        {
          me.readFile(files[0]);
        }
      },
      readFile(file) {
      let me = this;
      const reader = new FileReader();
        reader.onload = (res) => {
          me.code = res.target.result;
        };
        reader.onerror = (err) => console.log(err);
        reader.readAsText(file);
    },
      getCode()
      {
        let me = this;
        let source = me.code;
        let options = {"attachComment":false,
        "range":false,
        "loc":true,
        "sourceType":"module"};
        me.primaData = esprima.parse(source,options);
        if(me.primaData)
        {
          me.reviewCode();
        }
      },
      reviewCode()
      {
        let me = this;
          me.errorList = [];
          var functions = [];
          var variables = [];
          var IfStatements = [];
          me.getObjectByType(me.primaData,"type","FunctionExpression",functions);
          if(functions.length > 0)
          {
            for(let i =0 ;i< functions.length; i++)
            {
              me.getObjectByType(functions[i],"type","VariableDeclarator",variables);
              me.getObjectByType(functions[i],"type","IfStatement",IfStatements);
              if(functions[i].params && functions[i].params.length > 0)
              {
                functions[i].params.forEach(item=>{
                  let clone = _.clone(item);
                  clone.id = {};
                  clone.id.name = item.name;
                  variables.push(clone);
                });
              }
            }
          }
          if(variables.length > 0)
          {
              let IfStatementsName = [];
              if(IfStatements.length > 0)
              {
                for(let i =0;i<IfStatements.length;i++)
                {
                  me.getObjectByType(IfStatements[i],"name",null,IfStatementsName,true);
                }
                IfStatementsName = IfStatementsName.map(item=>item.name);
              }
                let test = variables.filter(variable =>
                {
                    return !IfStatementsName.includes(variable.id.name);
                });
                if(test.length > 0)
                {
                  for(let i = 0 ;i< test.length;i++)
                  {
                    this.errorList.push({
                      type:"warning",
                      message:`Dòng ${test[i].loc.start.line} : Giá trị ${test[i].id.name} chưa kiểm tra khác null`,
                    });
                  }
                }
          }
        
      },
      getObjectByType(theObject,key,value,output,hasOwnProperty = false) {
        var result = null;
        if(theObject instanceof Array) {
            for(var i = 0; i < theObject.length; i++) {
                result = this.getObjectByType(theObject[i],key,value,output,hasOwnProperty);
            }
        }
        else
        {
            for(var prop in theObject) {
                // console.log(prop + ': ' + theObject[prop]);
                if(prop == key) {
                    if(theObject[prop] == value) {
                        if(!hasOwnProperty)
                        {
                          output.push(theObject);
                        }
                        return theObject;
                    }
                    if(hasOwnProperty)
                    {
                      output.push(theObject);
                    }
                }
                if(theObject[prop] instanceof Object || theObject[prop] instanceof Array)
                    result = this.getObjectByType(theObject[prop],key,value,output,hasOwnProperty);
            }
        }
        return result;
      }
    },
  };
</script>

<style>
  .navbar{
    background-color: #2148b1;
    height: 60px;
  }
  .my-editor {
    background: #2d2d2d;
    color: #ccc;
    /* you must provide font-family font-size line-height. Example: */
    font-family: Fira code, Fira Mono, Consolas, Menlo, Courier, monospace;
    font-size: 14px;
    line-height: 1.5;
    padding: 5px;
    max-height: 500px;
    height: 500px;
    border: 1px solid #e5e8ef;
    flex:1;
  }

  /* optional class for removing the outline */
  .prism-editor__textarea:focus {
    outline: none;
  }
  .file-input
  {
    display:none;
  }
  .drop-zone{
    border: 1px dashed #d1e1f1;
    background-color: #ecf1f6;
    border-radius: 4px;
    padding: 29px 10px;
    transition: background-color .1s linear .1s;
    text-align: center;
    font-size:20px;
    margin-bottom:20px;
    margin-top:20px;
    
  }
  .title-drop,.title-or
  {
    margin-bottom: 4px;
  }
  .drop-zone-button
  {
    box-sizing: border-box;
    background-color: transparent;
    border: 1px solid #5ba3ff;
    border-radius: 5px;
    color: #5ba3ff;
    cursor: pointer;
    display: inline-block;
    font-size: 14px;
    font-weight: normal;
    margin: 0;
    padding: 6px 14px;
    vertical-align: baseline;
    white-space: nowrap;
    
  }
  .button-navbar
  {
    display:flex;
    justify-content: center;
    margin-bottom: 20px;
  }
  .button-primary
  {
    outline: 0;
    background-color: #00c5be;
    border: 1px #fff solid;
    color: #fff;
    border: 1px #cecece solid;
    padding: 4px 16px;
    border-radius: 5px;
    font-size: 20px;
    margin-right: 10px;
  }
  .logo{
    line-height: 40px;
    font-size: 36px;
    color: white;
  }
  .navbar{
    display: flex;
    align-items: center;
    padding:0 20px;
  }
  .content-editor
  {
    display:flex;
  }
  .parse-code
  {
    width:50%;
    margin-left: 10px;
    padding:0 20px;
    background: #2d2d2d;
    color: #ccc;
    max-height: 510px;
    overflow: auto;
  }
  .warning-message
  {
    background: #FFCC00;
    height: 30px;
    display: flex;
    align-items: center;
    padding: 0 20px;
    margin-top: 10px;
  }
</style>