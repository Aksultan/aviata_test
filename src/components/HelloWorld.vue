<template>
  <div class="hello">
    <div class="filters">
      <div class="filter tarif_option">
        <div class="filter_header">Опции тарифа
          <label class="reset_container">
            <input type="checkbox" checked="checked" v-if="filters.options.length" @change="resetFilterOptions()">
            <span class="reset_checkmark"></span>
          </label>
          <span class="modal">Сбросить выбор</span>
        </div>
        <label class="container">Только прямые
          <input type="checkbox" id="straight_only" v-model="filters.options" :value=0>
          <span class="checkmark"></span>
        </label>
        <label class="container">Только с багажом
          <input type="checkbox" id="baggage_only" v-model="filters.options" :value=1>
          <span class="checkmark"></span>
        </label>
        <label class="container">Только возвратные
          <input type="checkbox" id="refundable_only" v-model="filters.options" :value=2>
          <span class="checkmark"></span>
        </label>
      </div>
      <div>
      </div>
      <div class="filter company">
        <div class="filter_header">Авиакомпании
          <label class="reset_container">
            <input type="checkbox" checked="checked" v-if="filters.airlines.length" @change="resetFilterAirlines()">
            <span class="reset_checkmark"></span>
          </label>
          <span class="modal">Сбросить выбор</span>
        </div>
      <div class="scroller">
        <label class="container">Все
          <input type="checkbox" :disabled="filters.airlines.length==0 || filters.airlines.length" :checked="filters.airlines.length==0">
          <span class="checkmark"></span>
        </label>
        <label class="container" v-for="(item, index) in data.airlines" :key="index">{{item}}
          <input type="checkbox" v-model="filters.airlines" :value="item">
          <span class="checkmark"></span>
        </label>
      </div>
      </div>
      <a href="#" @click="resetFilterAirlines(), resetFilterOptions()">Сбросить все фильтры</a>
    </div>
    <div class="tickets">
      <div class="ticket" v-for="(ticket, index) in filterData" :key="index">
        <div class="ticket_left"> 
           <div class="ticket_info">
          <img width="40" height="40" :src="'https://aviata.kz/static/airline-logos/80x80/'+ticket.validating_carrier+'.png'" class="company_logo">
          <span class="company_name">{{ ticket.itineraries[0][0].carrier_name }}</span>
          <div class="dep_info">
            <span class="dep_date date">
              <span class="dep_day day">{{ ticket.itineraries[0][0].segments[0].dep_time_iso | date }}</span>
              <span class="dep_time time">{{ ticket.itineraries[0][0].segments[0].dep_time_iso | time }}</span>
            </span>
          </div>
          <div class="tr_map">
            <span class="small_text dep_from">{{ ticket.itineraries[0][0].segments[0].origin_code }}</span>
            <span class="small_text arr_to">{{ ticket.itineraries[0][0].segments[0].dest_code }}</span>
            <span class="total_time">{{ totalTime( ticket.itineraries[0][0].segments[0].dep_time_iso, ticket.itineraries[0][0].segments[0].arr_time_iso) }}</span>
          </div>
          <div class="arr_info">
            <span class="arr_date date">
              <span class="arr_day day">{{ ticket.itineraries[0][0].segments[0].arr_time_iso | date }}</span>
              <span class="arr_time time">{{ ticket.itineraries[0][0].segments[0].arr_time_iso | time }}</span>
              <span class="extra">+1</span>
            </span>
          </div>
        </div>
        <div class="options">
          <a href="#">Детали перелета</a>
          <a href="#">Условия тарифа</a>
          <span class="no_refund" v-if="!ticket.itineraries[0][0].refundable">
            <img src="../assets/icons/noref.png" alt="" >
            невозвратный
          </span>
        </div>
        </div>
        <div class="price_info">
          <span class="price">{{ ticket.itineraries[0][0].price.amount+" ₸"}}</span>
          <button class="btn">Выбрать</button>
          <span class="total_price">Цена за всех пасажиров</span>
          <div class="baggage">
            <span>Нет багажа</span>
            <button v-if="ticket.itineraries[0][0].segments[0].baggage_options[0].value">+ Добавить багаж</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import json from '../assets/results.json';
import moment from 'moment';
import momentDurationFormatSetup from "moment-duration-format";
momentDurationFormatSetup(moment);
moment.locale('ru')

export default {
  name: 'HelloWorld',
  data(){
    return{
      data:json,
      flights:json.flights,
      filters:{
        options:[],
        airlines:[]
      }
    }    
  },
  methods:{

    moment: () => {
      return moment();
    },

    totalTime: (dep_time, arr_time) => {
      let x = moment(arr_time).diff(dep_time);
      return moment.duration(x).format("h [ч] m [м]");
    },

    resetFilterOptions() {
      this.filters.options = [];
    },

    resetFilterAirlines() {
      this.filters.airlines = [];
    }

  },
  filters:{

    date: (date) => {
      return moment(date).format('DD MMM, ddd');
    },

    time: (time) => {
      return moment(time).format('h:mm');
    },
    
  },
  computed:{
    filterData(){
      let filters = this.filters; 
      return this.flights.filter(x=>{

        if(  filters.options.indexOf(0) != -1 && filters.options.length != 0 )
          return  x.itineraries[0][0].segments[0].stops == 0  
        else{
          return true;
        }

      }).filter(x=>{

        if( filters.options.indexOf(1) != -1 && filters.options.length != 0 )
          return x.itineraries[0][0].segments[0].baggage_options[0].value;
        else
          return true;    

      }).filter(x=>{

        if( filters.options.indexOf(2) != -1 && filters.options.length != 0 )
          return x.itineraries[0][0].refundable;
        else
          return true;
     
     }).filter(x=>{
       
       if( filters.airlines.length )
          return filters.airlines.includes(x.itineraries[0][0].segments[0].carrier_name);
        else
          return true;  
      
      })
    }
    
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
body{
  background-color: #d7d7d7;
}

.hello{
  display: flex;
  justify-content: center;
}

.filters{
  font-family: Arial;
  color:#202123;
  max-width: 240px;
  float: left;
  margin-right: 25px;
}

.filter{
  background-color: #F5F5F5;
  text-align: left;
  border-radius:4px;
  display: flex;
  flex-direction: column;
  min-width: 240px;
  margin-top: 12px;
}

.filter_header{
  position: relative;
  font-weight: bold;
  font-size:14px;
  margin:12px;
}

.reset_container {
  position: relative;
  cursor: pointer;
  float: right;
  height: 20px;
  padding-left: 10px;
  padding-right: 10px;
  user-select: none;

}

.reset_container input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}


.reset_checkmark {
  position: absolute;
  top: 0;
  left: 0;
  height: 20px;
  width: 20px;
}

.reset_container input:checked ~ .reset_checkmark {
  background: url("../assets/icons/fA.png") no-repeat;
}

.reset_container:hover input ~ .reset_checkmark {
  background: url("../assets/icons/fH.png") no-repeat;
}

.reset_checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

.reset_container input:checked ~ .reset_checkmark:after {
  display: block;
}

.reset_container:hover>.modal{
  display: block;
}

.modal{
  position: absolute;
  display: none;
}

.container {
  display: flex;
  flex-direction: row-reverse;
  justify-content: flex-end;
  align-items: center;
  position: relative;
  height: 32px;
  cursor: pointer;
  font-size: 12px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  transition: .3s;
  padding: 10px 12px;
}

.container input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

.checkmark {
  display: block;
  height: 12px;
  width: 12px;
  background: url("../assets/icons/chN.png") no-repeat;
  transition: 0.3s;
  margin:0 12px 0 0;
}

.container:hover input ~ .checkmark {
  background: url("../assets/icons/chH.png") no-repeat;
}

.container:hover{
  background-color:#EBEBEB;
}

.container input:checked ~ .checkmark {
  background: url("../assets/icons/chA.png") no-repeat;
}

.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

.container input:checked ~ .checkmark:after {
  display: block;
}

.filter_option{
  list-style: none;
  margin: 5px 0 0 0;
  padding: 0;
}

.filter_option li{
  margin-top: 20px;
}

a{
  border-bottom: 1px dashed #7284E4;
  color:#7284E4;
  text-decoration: none;
  float: left;
  margin-top: 13px;
}

.scroller{
  max-height: 462px;
  overflow-y: scroll;
  scroll-padding-right: 5px;
}

.scroller::-webkit-scrollbar{
  width:8px;
}

.scroller::-webkit-scrollbar-thumb{
  width: 3px;
  background-color: #E1E1E1;
  border-radius: 2px;
  border-right: 3px solid #F5F5F5;
}

.tickets{
  display: flex;
  flex-direction: column;
}

.ticket{
  display: flex;
  justify-content: space-between;
  max-width: 925px;
  background-color: #fff;
  margin-top: 12px;
  overflow: hidden;
  border-radius:4px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.15);
}

.ticket_info{
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px 0 20px 40px;
}

.company_name{
  font-size: 14px;
  font-weight: 600;
  margin-right:25px;
  min-width: 131px;
}

.dep_info{
  display: flex;
  margin-right: 25px;
}

.date{
  display:flex;
  flex-direction: column;
  min-width: 62px;
}

.arr_date{
  position: relative;
}

.day{
   font-size:12px;
}

.time{
  font-size: 24px;
  font-weight: 600;
}

.tr_map{
  position: relative;
  width:164px;
  height:1px;
  background-color: #B9B9B9;
  margin-right: 25px;
}

.tr_map::after{
  content:"";
  position: absolute;
  top:-2px;
  left:-6px;
  padding: 2px;
  border-radius:4px;
  border:1px solid #B9B9B9;
}

.tr_map::before{
  content:"";
  position: absolute;
  top:-2px;
  right:-6px;
  padding: 2px;
  border-radius:4px;
  border:1px solid #B9B9B9;
}

.small_text{
  position: absolute;
  top:-15px;
  font-size: 10px;
  color:#B9B9B9;
}

.dep_from{
  left:-6px;
}

.arr_to{
  right:-6px;
}

.total_time{
  position: absolute;
  top: -17px;
  left: 60px;
  font-size: 12px;
}

.total_time::after{
  content:"";
  position: absolute;
  top: 15px;
  left: 19px;
  padding: 2px;
  border-radius:4px;
  border:1px solid #B9B9B9;
  background-color: #fff;
}

.extra{
  position: absolute;
  top: 2px;
  right: -11px;
  color:rgba(255, 55, 36, 0.8);
  font-size: 10px;
}

.options{
  display:flex;
  justify-content: flex-start;
  align-items: center;
  padding:0 0 25px 45px;
}

.options a{
  margin-top: 0;
}

.options a:nth-child(2){
  margin-left:25px;
}

.no_refund{
  display: flex;
  align-items: center;
  margin-left:50px;
  font-size: 12px;
  text-align: center;
  color: #707276;
}

.no_refund img{
    margin-right:7px;
}

.price_info{
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
  background-color: #F5F5F5;
  min-width: 240px;
  margin-left:90px;
  padding: 15px;
}

.price{
  font-family: Arial;
  font-size: 24px;
}

button{
   border:none;
}

button:hover{
  cursor: pointer;
}

.btn{
  background-color:#55BB06;
  font-weight: bold;
  color:#fff;
  font-size: 18px;
  padding: 10px 50px;
  border-radius:4px;
}

.total_price{
  font-size: 12px;
  color:#707276;  
}

.baggage span{
  font-size: 12px;
}

.baggage button{
  margin-left: 6px;
  font-size: 12px;
  color:#5763B3;
  background-color: #EAF0FA;
  padding:2px 4px;
  border-radius: 2px;
}
</style>
