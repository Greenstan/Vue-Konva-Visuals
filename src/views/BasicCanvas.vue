

<template>
    <h1>This is a basic page with Text</h1>
    <div class="col">
    <div class="row">
        <div class="col">
            <div class="row w-100" v-for=" (v, k) in values ">
                <div class="col"> Value {{ k + 1 }}  </div>
                <div class="col"> <input type="text" v-model="values[k]"> </div>
            </div>
        </div>

        
    </div>
    
    <div class="row">
        <div class="m-4">
            <v-stage :config="configKonva" >
                    <v-layer>
        
                        <v-text :config="configValTitleText(0)" />
                        <v-rect :config = "configCell(0, 0)" />
        
                        <div v-for="column in 6">
                            <v-rect  :config = "configCell(0, column)" />
                            <v-text :config= "configValTitleText(column)" />
                        </div>
                    </v-layer>
    
                    <v-layer>
                        <div v-for="(v, k) in values" >
                            <div v-for="column in 6" >
                                <v-rect :config = "configCell(k+1, column)" />
                            </div>
                            <v-rect  :config = "configCell(k+1, 0)" />                                
                        </div>
                    </v-layer>
                    <v-layer>      
                        <div v-for="(v, k) in values">
                            <v-text :config="configText(valueNames[k], k+1)" />
                            <v-line v-if="k + 1 != values.length+1" :config="configLine(k+1)" />
                            <v-circle :config="configCircle(k+1, v)" />
                        </div>
                    </v-layer>
            </v-stage>
       
        </div>
    </div>

    <div class="row">
        <v-stage :config = "configKonva2">
            <v-layer>
                <v-text :config="configScoreResultCategory(currentCategory)" />
            </v-layer>
            <v-layer>
                <v-text v-for="percentage in [0,50,100,120]" :config="configScoreResultScaleText(percentage)" />
                <v-text :config="configScoreResultNotchLines()" />
            </v-layer>
            <v-layer>
                <v-rect v-for="section in [1,2,3,4]" :config="configScoreBarSection(section)"  />
               
                <v-line :config="configScoreBarLine(currentScore)" />
            </v-layer>
            <v-layer>
                <v-rect :config="configScoreResultRect(currentScore)" />
                <v-text :config="configScoreResultText(currentScore)" />
            </v-layer>
        </v-stage>
    </div>
    


    <br>
    <div class="row h-25">
        <div class="col">
            <div class="row ">
                <div class="col text-center">
                    Proficiency Scale
                </div>
            </div>
            <div  class="row">
                <Line 
                :options="chartOptions"
                :data="chartData(proportionOfValues)"
                style="height:600px;width:800px"
                />
            </div>
           
        </div>
    </div>
  
</div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import {  Line} from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, PointElement , Filler, LineElement,CategoryScale, LinearScale } from 'chart.js'


var values = ref([1, 1, 6, 1,3, 2])
let proportionOfValues = computed( ()=> {
    return values.value.map( ( _ ,key) => {
        let numberOfCurrentValue =  values.value.filter(currentVal => currentVal == key+1).length
        return (numberOfCurrentValue/values.value.length) *100
    })
})
const scoreWeight = { 1: -2 , 2: -1, 3:3, 4:4, 5:5, 6:6 }
var valueNames = ref(["Discipline","Courage","Agility","Evolvement","Relationships", "Empowerment"])


ChartJS.register(  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  Filler)




// Profficiency area chart 
let chartData  = (valuesData: number[]) =>  {
    return { 
    labels: ['|','||','|||','||||','|||||','||||||'], 
    datasets: [ {
        label: false,
        data: valuesData,  
        fill:true,
        borderColor: '#053D60',
        backgroundColor:"#053D60",
        pointStyle: false,
    }],
    }
}

const chartOptions = {
    responsive: true, 
    maintainAspectRatio: false,
    plugins:{
        legend:{
            display:false
        },
        tooltip:{
            enabled:false,
        }
    },

    scales:{
        y:{
            suggestedMin:0,
            suggestedMax:100,
            ticks: {
                min: 0,
                max: 100,
                stepSize: 10,
                callback: function (value:number) {
                     return (value ).toFixed(0) + '%'; // convert it to percentage
                },
                format: {
                    style: 'percent'
                }
            }
        }
    }
}




// Calculate score
let score = (values:number[], weights: { [ num:number] : number } ) : number => {
        let weightScoreSum = 0;
        Object.keys(weights).forEach( v =>{

           let numberOfCurrentScore =  values.filter(value => value == Number(v) ).length            
           let currentWeightedScore = numberOfCurrentScore * weights[Number(v)]
           weightScoreSum += currentWeightedScore
        })

        let percentScoreCalculation = Math.round( (weightScoreSum/ (values.length * 5)) * 100) 
        return percentScoreCalculation
    }

let currentScore = computed( () => score(values.value ,scoreWeight ) ) 


// Calculate category based off of score
let category = (score:number): string =>{
        if( score <= 20){
            return "Unfit"
        } else if (score > 20 && score <= 40 ){
            return "Weak"
        } else if (score > 40 && score <= 60){
            return "Fit"
        }else if( score > 60 && score <= 80){
            return "Talent"
        }else if (score > 80 && score <= 100){
            return "Star"
        }else{
            return "Leader"
        }

    }
let currentCategory = computed( ()=>category(currentScore.value) )

const gridPos = 90;
const rectSide = gridPos/2;
const shift = (gridPos/4)
const fontSize = gridPos/6 - 3

const configKonva = {
    width: 800,
    height: 800,
    showWarnings: false,
}


let configText = (text:String, circleNumber:number) => {
    return {
        text: text,
        x: shift - fontSize,
        y: circleNumber*gridPos +shift,
        fontSize: fontSize
    }
}

let configValTitleText = (value: number) => {

    let text = "|".repeat(value);

    return {
        text: text,
        fill: 'white',
        x: value*gridPos + shift,
        y: rectSide-shift,
        fontSize: 25
    }
}

let configCell = (circleNumber: number, column: number) => {

    const cellColor =  circleNumber == 0  ? "#053D60" :(circleNumber%2 == 0  ? "grey" : "white")

    return {
        x:gridPos * column,
        y:gridPos * circleNumber,
        width: gridPos,
        height: gridPos,
        stroke: 'black',
        strokeWidth: 1, 
        fill:cellColor,
    }
}


let configCircle = (circleNumber: number, value: number) => {
    return {
        x: gridPos * value + 2 * shift,
        y: gridPos * circleNumber + 2 * shift,
        radius: rectSide/2,
        fill: '#053D60',

    }
}

let configLine = (circleNumber: number) => {

    let currentVal:number = values.value[circleNumber-1]
    let nextVal:number = values.value[circleNumber]
    const rectPosition:number = rectSide/2 + shift
    return {
        points: [rectPosition + currentVal * gridPos, // x1
                 rectPosition + (circleNumber) * gridPos, // y1
                 rectPosition + nextVal * gridPos, //x2
                 rectPosition + (circleNumber +1) * gridPos], //y2
        stroke: '#29A6F2',
        lineCap: 'round',
        lineJoin: 'round',
        strokeWidth: 4,
        dash: [33, 10], // for dashed line
    }
}


let configKonva2 ={
    width:800,
    height:300,
    scale:{x:1,y:1},
}


let configScoreBarSection = (part:number) =>{
    let color:string; 
    switch(part){
        case(1):
        color = '#EEEEEE'
        break;
        case(2):
        color = '#A9D3F4'
        break; 
        case(3):
        color = '#FCEDB5'
        break; 
        case(4):
        color = '#8B9EAE'
        break; 
        default:
        color = 'black'
        
    }    
    return {
        x: 100 *part,
        y: 150,
        width:100,
        height:30,
        fill:color,
        strokeWidth: 1, 
    }
}

let configScoreBarLine = (score: number)=>{
    if (score<0){
        score = 0 
    }
    return {
        points: [101,165, 99 + 400 * (score/120) ,165],
        strokeWidth: 6,
        stroke: '#29A6F2',
        lineCap: 'round',
        lineJoin: 'round',
    }
}

let configScoreResultRect = (score: number)=>{
    return { 
        x:250,
        y:200,
        width:110,
        height:70,
        fill:"#FFCF03",
        cornerRadius: 10,
        strokeWidth: 1,
    }
}
let configScoreResultText = (score: number) => {

    let shift: number
    if (score > 10 && score < 100){
        shift = 10        
    }else if (score >= 100){
        shift = 0
    }else{
        shift = 20
    }
    return {
        text: `${score}%`,
        x:250 + 15 + shift,
        y:200 + 25,
        fontSize: 30,
        fill:"#043D60"
    }

}

let configScoreResultNotchLines = ()=>{
    return {
        text: "|".repeat(110),
        x: 100,
        y: 125, 
        fontSize:14,
        fill:'black',
    }
}

let configScoreResultScaleText = (val:number)=>{
    return {
        text: `${val}%`,
        x: 95 + (val/120)*399 ,
        y: 110, 
        fontSize:16,
        fill:"#043D60",
    }
}

let configScoreResultCategory = (categoryText:string)=>{
    return {
        text: categoryText,
        x: 270,
        y: 25, 
        fill:"#043D60",
        fontSize:40,
        horizontalAlign:'middle'
    }
}

</script>

<style>

table, th, td {
    border: 1px solid black;
  }

</style>