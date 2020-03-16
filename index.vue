<template>
    <div class="calendar-main">
        <div class="choose_year">
            <!-- <div class="icon" @click="chooseYears(-1)"><i class="el-icon-d-arrow-left"></i></div> -->
            <div class="icon" @click="chooseMonth(-1)"><i class="el-icon-arrow-left"></i></div>
            <div class="date">{{year}}年{{month.toString().padStart(2, '0')}}月</div>
            <div class="icon" @click="chooseMonth(1)"><i class="el-icon-arrow-right"></i></div>
            <!-- <div class="icon" @click="chooseYears(1)"><i class="el-icon-d-arrow-right"></i></div> -->
        </div>
        <!-- <div class="lunar-detail-date">
            {{lunarDetail.lunarYear.substring(0,2)}} -【{{lunarDetail.lunarYear.substring(2,3)}}】年{{lunarDetail.lunarMonth}}月{{lunarDetail.lunarDay}}
        </div> -->

        <div class="days_area">
            <div class="days_area_title">
                <div class="day week" v-for="week in weeks" :key="week">{{week}}</div>
            </div>
            <div class="days_area_content">
                <div class="day day-list" @click="chooseThisDay(day.id,day.status)" v-for="(day, index) in days" :key="index" >
                    <div class="day-list_box " 
                        :class="{ 'choose_day':day.id === today,'lastMonth':day.lastMonth,already_schedule:day.status === true }"
                        >
                        <p>{{day.gregorian}}</p>
                        <span :class="{'Holiday':day.Holiday === true}">{{day.lunar}}</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- <div class="bottom-btn">
            <span @click="close">取消</span>
            <span @click="confirm(3,2)">确定</span>
        </div> -->

    </div>
</template>

<script>
    import {sloarToLunar,getYearMonthDay} from "./calendar.js"
    import { HolidayFind } from '@/utils/validate'
    export default {
        name: 'LunarCalendar',
        props:['currentDate'],
        data () {
            return {
                year: 0,
                month: 0,
                today: 0,
                days: [],
                weeks: ['一', '二', '三', '四', '五', '六','日'],
                todayDate:0,
                lastMonth:[], //上个月数据
                HolidayFind:null
            }
        },
        mounted () {
            this.HolidayFind = new HolidayFind();
            
            this.todayDate = new Date().getDate();
            if(this.currentDate){
                const now = new Date(this.currentDate);
                this.year = now.getFullYear();
                this.month = now.getMonth() + 1;
                this.today = now.getDate();
            }else{
                const now = new Date();
                this.year = now.getFullYear();
                this.month = now.getMonth() + 1;
                this.today = now.getDate();
            }
            this.getDays()
        },
        computed:{
            selectDate(){
                return `${this.year}-${this.month.toString().padStart(2, '0')}-${this.today}`
            },
            lunarDetail(){
               return sloarToLunar(this.year,this.month.toString().padStart(2, '0'),this.today);
            }
        },
        methods: {
            //上个月数据
            lastMonthFun(){
                this.lastMonth = [];
                const time = new Date();
                time.setFullYear(this.year, this.month -1, 0);
                for (let i = 1; i <= time.getDate(); i++) {
                    this.lastMonth.push({
                        gregorian: i, 
                        lunar: getYearMonthDay(this.year, this.month -1, i),
                        // id:Math.random().toString(36).substr(-8),
                        lastMonth : true
                    })
                    // this.lastMonth.push({
                    //     gregorian: '', 
                    //     lunar:'',
                    //     id:'',
                    //     lastMonth : true
                    // })
                }
            },
            //获取1号属于星期几
            getWeek(){
                const time = new Date();
                time.setFullYear(this.year, this.month, 0);
                time.setDate(1);
                return time.getDay() === 0?6: time.getDay() -1;
            },
            // 获取当前月份所有公历日期及其农历日期
            getDays () {
                this.lastMonthFun();    //获取上个月数据
                let monthData = [];
                const time = new Date();
                time.setFullYear(this.year, this.month, 0);
                let numbers = this.getWeek();
                //计算不在一周的剔除到下个月
                // let monthDataSum = (time.getDate() + numbers)  - (parseInt((numbers + time.getDate()) /7) * 7);
                let Nowadays = new Date().getDate()  //今天日期
                let monthFrequency = time.getDate() //这个月有几天
                for (let i = 1; i <= monthFrequency; i++) {
                    let Holiday = this.HolidayFind.getHoliday(this.month,i);
                    monthData.push({
                        status: i < Nowadays?true:false, //是否是已经排完班的
                        gregorian: i, 
                        lunar:Holiday?Holiday.name:getYearMonthDay(this.year, this.month, i),
                        Holiday:Holiday?true:false,
                        id:Math.random().toString(36).substr(-8)
                    })
                }
                if(this.getWeek()){
                    let lastMonth = this.lastMonth.slice(-numbers);
                    this.days = monthData;
                    this.days = [
                        ...lastMonth, //上个月数据
                        ...monthData //这个月数据
                    ]
                }else{
                    this.days = monthData; //一号是星期一就不用在添加上月数据了
                }
            },
            // 改变年份
            chooseYears (state) {
                this.year += state;
                if(new Date().getFullYear()===this.year&&(new Date().getMonth()+1)===this.month){
                    this.today = this.todayDate;
                }else{
                    this.today = 0;
                }
                this.getDays()
            },
            // 改变月份
            chooseMonth (state) {
                this.month += state;
                if(new Date().getFullYear()===this.year&&(new Date().getMonth()+1)===this.month){
                    this.today = this.todayDate;
                }else{
                    this.today = 0;
                }
                if (this.month < 1) {
                    this.month = 12;
                    this.chooseYears(-1)
                } else if (this.month > 12) {
                    this.month = 1;
                    this.chooseYears(1) 
                } else {
                    this.getDays()
                }
            },
            chooseThisDay (id,status) {
                if(id && !status){
                    this.today = id;
                }
            },
            getBit (m, n) {
                return 29 + ((m >> n) & 1)
            },
            close(){
                this.$emit('close')
            },
            confirm(month,day){
            }
        }
    }
</script>

<style lang="less" scoped>
    .calendar-main{
        -moz-user-select:none; /* Firefox私有属性 */
        -webkit-user-select:none; /* WebKit内核私有属性 */
        -ms-user-select:none; /* IE私有属性(IE10及以后) */
        -khtml-user-select:none; /* KHTML内核私有属性 */
        -o-user-select:none; /* Opera私有属性 */
        user-select:none; /* CSS3属性 */
        width:428px;
        height: 479px;    
        background-color: #fff;
        box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.2);
        -webkit-border-radius: 5px;
        -moz-border-radius: 5px;
        border-radius: 5px;
        .choose_year{
            display: flex; justify-content: center; align-items: center;
            height: 52px; background: #1ca75f ; color:#fff; font-size: 14px;
            .icon{
                width: 10%;
                height: 35px;
                line-height: 35px;
                text-align: center;
                font-size: 13px;
                cursor: pointer;
                // &:hover{
                //     color: dodgerblue;
                // }
            }
            .date{
                // width: 60%;
                width:319px;
                text-align: center;
                height: 35px;
                line-height: 35px;
                font-size: 14px;
            }
        }
        .lunar-detail-date{
            line-height: 15px;
            text-align: center;
            color: #FF6800;
        }
        .days_area{
            padding: 0 10px;padding-bottom: 13px; padding-right:5px;
            .days_area_title{
                display: flex; 
            }
            .days_area_content{display: flex;flex-wrap: wrap; }
            .day{
                width:58px;
                height: 49px;
                line-height: 27px;
                text-align: center;
                cursor: pointer;
                display: flex; 
                align-items: center; 
                justify-content: center; flex-wrap: wrap;align-content:flex-start;
                p{
                    margin: 0;
                    font-size: 13px;
                    width: 100%;
                    display: inline-block;
                    line-height: 20px;
                }
                span{
                    font-size: 13px;
                    line-height: 20px;
                }
            }
            .day-list{
                margin-top:1px;
                display: flex; justify-content: center; align-items: center;
                height: 59px;
                .day-list_box{
                    overflow: hidden;
                    width: 49px; height: 49px; display: flex; flex-wrap: wrap; justify-content: center;
                    padding:5px; box-sizing: border-box;color:#555555; position: relative; 
                    // &:hover{
                    //     color:#fff;
                    //     background-color:#f197a2;
                    //     -webkit-border-radius: 50%;
                    //     -moz-border-radius: 50%;
                    //     border-radius: 50%;
                    //     opacity: 0.6;
                    // }
                    .Holiday{
                        color: #57a7f6;
                        overflow: hidden;text-overflow:ellipsis;white-space: nowrap;
                    }
                }
                .already_schedule::after{
                    width: 0;
                    height: 0;
                    height: 0px;
                    border-color:   #4cbf83 transparent transparent #4cbf83;
                    border-width: 4px 4px 4px 4px;
                    border-style: solid;
                    content:''; 
                    position: absolute;
                    left:5px;
                    top:5px;
                }
                .lastMonth{
                    color:#c5c2c2;
                }
                .lastMonth::after{
                    width: 0;
                    height: 0;
                    border-left: 4px solid transparent;
                    border-right: 4px solid transparent;
                    border-bottom: 6px solid #c5c2c2;
                    transform:rotate(-49deg);
                    content:''; 
                    position: absolute;
                    left:5px;
                    top:5px;
                }
            }
            .week{
                background-color: #fff;
                color: #19233c;
                font-weight: bold;
                height: 49px;
                line-height: 49px;
                margin: 0;
                font-size: 14px;
                width: 58px;
                color:#1ca75f;
            }
            .choose_day{
                background-color: #f197a2;
                color: #fff;
                font-weight: bold;
                -webkit-border-radius: 50%;
                -moz-border-radius: 50%;
                border-radius: 50%;
                width:58px; height:58px;
                span{
                    color: #fff;
                }
                p{
                    color: #fff;
                }
            }
        }
        .bottom-btn{
            text-align: right;
            span{
                padding: 4px 8px;
                margin-right: 5px;
                cursor: pointer;
                border: 1px solid #999;
                -webkit-border-radius: 5px;
                -moz-border-radius: 5px;
                border-radius: 5px;
                &:hover{
                    color: dodgerblue;
                    border: 1px solid dodgerblue;
                }
            }
        }
    }
</style>
