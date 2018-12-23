<template>
  <el-row>  
    <el-col :span="12">
        <div>
            <div id='main' style="width: 100%;height: 500px"></div>
            <div id='main3D' style="width: 100%;height: 500px"></div>
        </div>
    </el-col>
    <el-col :span="12">
        <div class="formBox">
            <el-form ref="form" :model="search" label-width="120px" size="mini" label-position="left" >
                <el-form-item label="食品&检测项" class="formItem">
                    <el-select v-model="search.searchKeys" multiple placeholder="请选择">
                        <el-option-group
                        v-for="group in selectOptions"
                        :key="group.label"
                        :label="group.label">
                        <el-option
                            v-for="item in group.options"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value">
                        </el-option>
                        </el-option-group>
                    </el-select>
                </el-form-item>
                <el-form-item label="抽检不合格频次" class="formItem">
                    <el-slider
                    v-model="search.frequency"
                    range
                    show-stops
                    :max="frequencyMax">
                    </el-slider>
                </el-form-item>
                <el-form-item label="检测日期" class="formItem"> 
                    <el-date-picker
                    v-model="search.date"
                    type="daterange"
                    align="right"
                    unlink-panels
                    range-separator="至"
                    start-placeholder="开始日期"
                    end-placeholder="结束日期"
                    :picker-options="pickerOptions2">
                    </el-date-picker>
                </el-form-item>
                <el-form-item size="small" class="formItem">
                    <el-button type="primary" @click="onSubmit">生成图表</el-button>
                    <el-button>取消</el-button>
                </el-form-item>
            </el-form>
        </div>
        <div id='mainMap' style="width: 100%;height: 500px"></div>
    </el-col>
  </el-row>
</template>
<script>
//                            _ooOoo_
//                           o8888888o
//                           88" . "88
//                           (| -_- |)
//                           O\  =  /O
//                        ____/`---'\____
//                      .'  \\|     |//  `.
//                     /  \\|||  :  |||//  \
//                    /  _||||| -:- |||||-  \
//                    |   | \\\  -  /// |   |
//                    | \_|  ''\---/''  |   |
//                    \  .-\__  `-`  ___/-. /
//                  ___`. .'  /--.--\  `. . __
//               ."" '<  `.___\_<|>_/___.'  >'"".
//              | | :  `- \`.;`\ _ /`;.`/ - ` : | |
//              \  \ `-.   \_ __\ /__ _/   .-` /  /
//         ======`-.____`-.___\_____/___.-`____.-'======
//                            `=---='
//        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//                      Buddha Bless, No Bug !
import echarts from 'echarts'
import 'echarts-gl'
import $ from 'jquery'
export default {
    name: 'echart',
    data () {
        return {
            myChart:"",  
            myChart3D:"",  
            myChartMap:"", 
            graph: {},
            categories: [1,2],
            option: {
                title: {
                    text: '食品检测图',
                    subtext: "全部数据",
                    top: 'top',
                    left: 'left'
                },
                tooltip: {
                    formatter : "",
                    textStyle:{
                    　　align:'left'
                    }
                },
                legend: [{
                    // selectedMode: 'single',
                    show: false,
                    data: []
                }],
                animationDurationUpdate: 1500,
                animationEasingUpdate: 'quinticInOut',
                series : [
                    {
                        name: '食品检测图',
                        type: 'graph',
                        layout: 'circular',
                        circular: {
                            rotateLabel: true
                        },
                        data: [],
                        links: [],
                        categories: [],
                        roam: true,
                        focusNodeAdjacency:true,
                        label: {
                            normal: {
                                position: 'right',
                                formatter: '{b}'
                            }
                        },
                        lineStyle: {
                            normal: {
                                color: 'source',
                                curveness: 0.3
                            }
                        }
                    }
                ]
            },
            selectOptions:[{
                label: '食品',
                options:[]
            },{
                label: '检测项',
                options:[]
            }],
            checkItems: [],
            foods: [],
            search: {
                frequency: [0,0],
                searchKeys: [],
                data: []
            },
            pickerOptions2: "",
            frequencyMax : 0,
            option3D:  {
                tooltip: {
                    formatter : "",
                    textStyle:{
                    　　align:'left'
                    }
                },
                visualMap: {
                    max: 20,
                    inRange: {
                        color: ['#313695', '#4575b4', '#74add1', '#abd9e9', '#e0f3f8', '#ffffbf', '#fee090', '#fdae61', '#f46d43', '#d73027', '#a50026']
                    }
                },
                xAxis3D: {
                    // name: "检测项",
                    type: 'category',
                    data: []
                },
                yAxis3D: {
                    // name: "食品",
                    type: 'category',
                    data: []
                },
                zAxis3D: {
                    // name: "检测不合格频次",
                    type: 'value'
                },
                grid3D: {
                    boxWidth: 200,
                    boxDepth: 80,
                    light: {
                        main: {
                            intensity: 1.2
                        },
                        ambient: {
                            intensity: 0.3
                        }
                    }
                },
                series: [{
                    type: 'bar3D',
                    data: {},
                    shading: 'color',

                    label: {
                        show: false,
                        textStyle: {
                            fontSize: 16,
                            borderWidth: 1
                        }
                    },
                    
                    itemStyle: {
                        opacity: 0.4
                    },

                    emphasis: {
                        label: {
                            textStyle: {
                                fontSize: 20,
                                color: '#900'
                            }
                        },
                        itemStyle: {
                            color: '#900'
                        }
                    }
                }]
            },

            optionMap: {
                backgroundColor: '#ffffff',
                title: {
                    text: '全国主要城市食品安全抽检',
                    //subtext: 'data from PM25.in',
                    //sublink: 'http://www.pm25.in',
                    x:'center',
                    textStyle: {
                        color: '#000000'
                    }
                },
                tooltip: {
                    // trigger: 'item',
                    formatter: function (params) {
                        
                        return params.name + ' : ' + params.value;
                    }
                },
                legend: {
                    orient: 'vertical',
                    y: 'bottom',
                    x:'right',
                    data:['pm2.5'],
                    textStyle: {
                        color: '#fff'
                    }
                },
                visualMap: {
                    min: 0,
                    max: 200,
                    // calculable: true,
                    color: ['#d94e5d','#eac736','#50a3ba'],
                    // textStyle: {
                    //     color: '#fff'
                    // }
                },
                geo: {
                    map: 'china',
                    label: {
                        emphasis: {
                            show: false
                        }
                    },
                    itemStyle: {
                        normal: {
                            areaColor: '#84aab1',
                            borderColor: '#111'
                        },
                        emphasis: {
                            areaColor: '#2a333d'
                        }
                    }
                },
                series: [
                    {
                        name: 'categoryA',
                        type: 'map',
                        geoIndex: 0,
                        // tooltip: {show: false},
                        data:[
                            {name: '北京', value: 0},
                            {name: '天津', value: 0},
                            {name: '上海', value: 0},
                            {name: '重庆', value: 0},
                            {name: '河北', value: 0},
                            {name: '河南', value: 0},
                            {name: '云南', value: 0},
                            {name: '辽宁', value: 0},
                            {name: '黑龙江', value: 0},
                            {name: '湖南', value: 0},
                            {name: '安徽', value: 0},
                            {name: '山东', value: 0},
                            {name: '新疆', value: 0},
                            {name: '江苏', value: 0},
                            {name: '浙江', value: 0},
                            {name: '江西', value: 0},
                            {name: '湖北', value: 0},
                            {name: '广西', value: 0},
                            {name: '甘肃', value: 0},
                            {name: '山西', value: 0},
                            {name: '内蒙古', value: 0},
                            {name: '陕西', value: 0},
                            {name: '吉林', value: 0},
                            {name: '福建', value: 0},
                            {name: '贵州', value: 0},
                            {name: '广东', value: 0},
                            {name: '青海', value: 0},
                            {name: '西藏', value: 0},
                            {name: '四川', value: 0},
                            {name: '宁夏', value: 0},
                            {name: '海南', value: 0},
                            {name: '台湾', value: 0},
                            {name: '香港', value: 0},
                            {name: '澳门', value: 0},
                            {name: '南海诸岛', value: 0}
                        ]
                    }
                ]
            },
        }
    },
    mounted () {
        var _this = this;
        // 基于准备好的dom，初始化echarts实例
        _this.myChart = echarts.init(document.getElementById('main'));
        _this.myChart3D = echarts.init(document.getElementById('main3D'));
        _this.myChartMap = echarts.init(document.getElementById('mainMap'));
        //获取分类
        $.getJSON("static/data/categories.json",function(data){
            _this.categories = data;
            _this.option.legend[0].data = data.map(function (a) {
                return a.name;
            });
            _this.option.series[0].categories = data;
        });
        //获取图表数据
        $.get('static/data/test_small.gexf', function (xml) {
            
            console.log(echarts.dataTool);
            _this.graph = echarts.dataTool.gexf.parse(xml);
            _this.initNodes(_this.graph.nodes);
            _this.initLinks(_this.graph.links);
            var max = 0;
            debugger
            _this.graph.links.forEach(function(e){
                if(e.frequency > max){
                    max = e.frequency;
                }
            });
            _this.frequencyMax = parseInt(max);
            _this.search.frequency[1] = parseInt(max);
            _this.optionMap.visualMap.max = parseInt(max);
            _this.initEchart(_this.graph.nodes,_this.graph.links);
        }, 'xml');
        //渲染下拉列表
        $.getJSON("static/data/selectData.json",function(data){
            data.forEach(function(e,index){
                if(e.type=="food"){
                    _this.selectOptions[0].options.push({
                        value: e.name,
                        label: e.name
                    });
                    _this.foods.push({
                        value: e.name,
                        label: e.name
                    });
                }
            });
            data.forEach(function(e){
                if(e.type=="testItem"){
                    _this.selectOptions[1].options.push({
                        value: e.name,
                        label: e.name
                    });
                    _this.checkItems.push({
                        value: e.name,
                        label: e.name
                    });
                }
            });
        });
        _this.option.tooltip.formatter = function(params){
            
            if(params.data.source){//判断是node还是link
                var sourceName = "";
                var targetName = "";
                _this.graph.nodes.forEach(function(node){
                    if(params.data.source == node.id){
                        sourceName = node.name;
                    }
                    else if(params.data.target == node.id){
                        targetName = node.name;
                    }
                });
                var textArea = params.marker+sourceName+">"+targetName+"</br>"+"抽检不合格频次 :  "+params.data.frequency+"</br>";
                var index = 0;
                params.data.options.forEach(function(option){
                    if(option.label=="-"){
                        textArea +=  "</br>"
                        index = 0;
                    }else{
                        if(index%3 || index==0){
                            textArea +=  " "+option.label+" : "+option.value
                        }else{
                            textArea +=  " "+option.label+" : "+option.value+"</br>"
                        }
                        index++;
                    }
                    
                });
                return textArea;
            }else{
                return params.marker+" "+params.data.name
            }
        }
    },
    methods: {
        //init node
        initNodes : function(nodes){
            nodes.forEach(function (node) {
                node.itemStyle = null;
                // node.value = node.symbolSize;
                //node.symbolSize /= 1.5;
                node.label = {
                    normal: {
                        show: node.symbolSize > 8
                    }
                };
                node.category = node.attributes.modularity_class;
            });
        },

        //init link
        initLinks: function(links){
            links.forEach(function (link) {
                var width = link.lineStyle.width ? link.lineStyle.width : "1.0";
                link.lineStyle = {
                    width : parseInt(width)/2
                };
                link.value = width;
                link.frequency = parseInt(width);
            });
        },

        initEchart: function(nodes,links,subText,frequency){
            if(nodes.length < 30){
                nodes.forEach(function(node){
                    node.label.normal.show = true;
                });
            }
            this.option.series[0].data = nodes;
            this.option.series[0].links = links;
            this.option.title.subtext = subText ? "与"+subText+"有关且抽检不合格频次"+frequency+"的数据" : "全部数据",
            // myChart.showLoading();
            this.initEchart3D(nodes,links);
            this.initEchartMap(nodes,links);
            this.myChart.setOption(this.option);
            //预留分类点击事件
            // myChart.on("legendselectchanged", function (param) {
            //     
            //     option.title.text = "1111"
            //     this.setOption(option);
            // });
        },

        initEchart3D: function(nodes,links){
            
            var xLabel = [];
            var xId = [];
            var yLabel = [];
            var yId = [];
            var value = [];
            nodes.forEach(function(node){
                if(node.symbolSize<10){
                    xLabel.push(node.name);
                    xId.push(node.id);
                }else{
                    yLabel.push(node.name);
                    yId.push(node.id);
                }
            });
            var max = 0;
            links.forEach(function(link){
                value.push([xId.indexOf(link.target),yId.indexOf(link.source),parseInt(link.frequency)]);
                if(parseInt(link.frequency) > max){
                    max = parseInt(link.frequency);
                }
            });
            this.option3D.visualMap.max = max;
            this.option3D.xAxis3D.data = xLabel;
            this.option3D.yAxis3D.data = yLabel;
            this.option3D.series[0].data = value.map(function (item) {
                return {
                    value: [item[0], item[1], item[2]]
                }
            });
            this.option3D.tooltip.formatter = function(param){
                
                return param.marker+" 食品检测信息表<br>-食品 : "+yLabel[param.data.value[1]]+"<br>-检测项 : "+xLabel[param.data.value[0]]+"<br>-检测不合格频次 : "+param.data.value[2]
            }
            this.myChart3D.setOption(this.option3D);
        },

        initEchartMap: function(nodes,links){
            debugger
            var _this = this;
            var idList = [];
            _this.optionMap.series[0].data.forEach(function(e){
                e.value = 0;
            });
            nodes.forEach(function(node){
                if( _this.search.searchKeys.indexOf(node.name)>=0 || !_this.search.searchKeys.length){
                    idList.push(node.id);
                }
            });
            links.forEach(function(link){
                
                if(idList.indexOf(link.source)>=0 && idList.indexOf(link.target)>=0){
                    
                    link.options.forEach(function(option){
                        
                        _this.optionMap.series[0].data.forEach(function(e){
                            if(e.name == option.label){
                                e.value += parseInt(option.value);
                            }
                        });
                    })
                }
            });
            this.myChartMap.setOption(this.optionMap);
        },

        //搜索框功能
        searchNode: function(keys,frequency){
            var ids = [];
            var idList = [];
            var nodesList = [];
            var linksList = [];
            this.graph.nodes.forEach(function(node){
                if(keys.indexOf(node.name)>=0){
                    ids.push(node.id);
                }
            });
            this.graph.links.forEach(function(link){
                if((link.frequency >= frequency[0] && link.frequency <= frequency[1])  && ( ids.indexOf(link.source )>=0 || ids.indexOf(link.target )>=0  || !(keys.length>0) )){
                    linksList.push(link);
                    if(idList.indexOf(link.source)<0){
                    idList.push(link.source);
                    }
                    if(idList.indexOf(link.target)<0){
                        idList.push(link.target);
                    }
                }
            });

            this.graph.nodes.forEach(function(node){
                if(idList.indexOf(node.id)>=0){
                    nodesList.push(node);
                }
            });
            var keytext = keys[0];
            keys.forEach(function(e,index){
                if(index > 0){
                    keytext += "、"+e
                }
            });
            var frequencyText = "介于"+frequency[0]+"-"+frequency[1]+"之间"
            this.initEchart(nodesList,linksList,keytext,frequencyText);
        },
        onSubmit : function(){
            
            this.searchNode(this.search.searchKeys,this.search.frequency);
        }
    }
}

</Script>

<style>
.formBox {
    height: 500px;
    width: 50%;
    text-align: center;
    margin: auto;
}
.formItem {
    text-align: left;
}
</style>