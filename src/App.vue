<template>
<div id="app">
    <input type="text" 
        id="color-picker" 
        v-model="color" 
        auto-complete="off"
        @focus="openPicker"
        @blur="closePicker"
        :style="'background-color:'+color"
        >
    <colorPicker
        ref = "colorPicker"
        :color="color"
        :targetElem="'#color-picker'"
        @onChange="onChange"
        >
    </colorPicker>
</div>
</template>

<script>
import colorPicker from './components/vue-color-picker'
export default {
    components:{colorPicker},
    name: 'app',
    data(){
        return {
            color:"#DDDDDD"
        }
    },
    methods:{
        openPicker(event){
            this.$refs.colorPicker.openPicker();
            this.$refs.colorPicker.updateValue(event);
        },
        closePicker(){
            //this.$refs.colorPicker.closePicker();
        },
        onChange(color){
            this.color = color;
        },
    }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
