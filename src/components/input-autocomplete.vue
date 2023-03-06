<template>
    <div>
        <input v-model="input" @input="handleInput" placeholder="Enter the name of the place">
        <div class="bordered" v-if="input" v-show="!closeDiv">
            <ul>
                <li v-for="(item, i) in cityGeoApiName" :key="i" @click="setInput(item, i)">{{ item.cityNameOriginal + ', ' + item.countryName}}</li>
            </ul>
        </div>
    </div>
</template>

<script>
    export default{
        props: ['cityGeoApiName'],
        data() {
            return {
                input: null,
                closeDiv: false,
                index: 1
            }
        },
        methods: {
            handleInput(e) {
                this.closeDiv = false
                this.$emit('input', e.target.value)
                this.$emit('getStartGeoApi', {currentInputValue: this.input})
                console.log('INPUT VALUE AUCTOCOMPLETE' + this.input)
            },
            setInput(value, i) {
                this.input = value.cityNameOriginal
                this.$emit('input', value.cityNameOriginal)
                this.closeDiv = true
                this.index = i
                console.log('INNNNDSEEEEEEX: ')
                console.log(i)
                console.log(value)
                
                this.$emit('getRequestDataInput', { finalRequestValue: value.cityNameOriginal, currentLat: value.lat , currentLon: value.lon, indexEl: this.index})
                //this.setFinalInput(value)
            },
            /*setFinalInput(value){
                //console.log(value.target.value)
                //this.$emit('getRequestDataInput', { finalRequestValue: value.target.value, indexEl: this.index})
            }*/
        }
    }
</script>

<style scoped>
    .bordered {
        border: 1px solid black;
        display: block;
        background-color: aliceblue;
        border-radius: 15px;
    }

    ul{
        text-decoration: none;
        text-align: left;
    }

    ul li {
        cursor: pointer;
        list-style-type: none;
        font-size: 1.1em;
    }

    ul li:hover {
        background: rgba(0, 0, 0, 0.3)
    }


    div {
        width: 450px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        margin-left: auto;
        margin-right: auto;
    }

    div input {
        height: 29px;
        border-radius: 15px;
        font-size: 1.2em;
    }
</style>