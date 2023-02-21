<template>
    <div class="backdrop">
        <div class="modal">
            <div class="close">
                <span @click="closeModal">x</span>
            </div>
            <div class="header">
                <img v-if="coinHttp" class="item" :src="coinHttp.replace('large','small')">
                <p class="item">{{coinId.toUpperCase()}}</p>
                <p class="item">{{coinSymbol.toUpperCase()}} / USDT </p>
                <!--                    <a v-if="coinData.links.homepage[0]" class="item" :href="coinData.links.homepage[0]" target="_blank">{{coinData.links.homepage[0].replace('http://','')}}</a>-->
            </div>
            <hr>
            <div class="header">
                <div class="item">
                    <button v-on:click="select($event)" class="btn" type="button" value="d">1d</button>
                </div>
                <div class="item">
                    <button v-on:click="select($event)" class="btn" type="button" value="w">1w</button>
                </div>
                <div class="item">
                    <button v-on:click="select($event)" class="btn" type="button" value="m">1m</button>
                </div>
                <div class="item">
                    <button v-on:click="select($event)" class="btn" type="button" value="y">1y</button>
                </div>
            </div>

            <div class="chart" ref="chartdiv">

            </div>
        </div>
    </div>
</template>

<script>
    import * as am4core from "@amcharts/amcharts4/core";
    import * as am4charts from "@amcharts/amcharts4/charts";
    import am4themes_animated from "@amcharts/amcharts4/themes/animated";

    am4core.useTheme(am4themes_animated);

    export default {
        name: "CoinDetailsModal",
        props: ['coinId', 'coinSymbol', 'coinHttp'],
        data() {
            return {
                coinData: {},

            }
        },
        methods: {
            closeModal() {
                this.$emit('close');
            },
            async getCoinDetails() {
                // await fetch("https://api.coingecko.com/api/v3/coins/" + this.symbol.toLowerCase()).then  (
                //      res =>  res.json()).then(data => {
                //     this.coinData = data;
                // }).catch(e => {
                //     alert(e)
                // });
                const response = await fetch("https://api.coingecko.com/api/v3/coins/" + this.coinId.toLowerCase());
                const data = await response.json();
                this.coinData = data;
            },
            async getCoinHistoricalData(interval) {
                let dateNow = new Date().getTime();
                let coinHistory = [];
                await fetch("https://api.binance.com/api/v3/klines?symbol=" + this.coinSymbol.toUpperCase() + interval).then(res => res.json()).then(data => {
                    for (let i = 0; i < data.length; i++) {
                        coinHistory.push({date: data[i][0], name: "", value: data[i][4]})
                    }
                }).catch((e) => {
                    alert(e)
                });
                this.createChart(coinHistory);
            },
            select(e) {
                this.chart.dispose();
                let endTime = new Date().getTime();
                let startTime = null;
                let btnValue = e.target.value;
                let interval = "";
                switch (btnValue) {
                    case "d":
                        startTime = new Date(new Date().setDate(new Date().getDate() - 1)).getTime();
                        interval = "USDT&startTime=" + startTime + "&endTime=" + endTime + "&interval=15m";
                        this.getCoinHistoricalData(interval);
                        break;
                    case "w":
                        startTime = new Date(new Date().setDate(new Date().getDate() - 7)).getTime();
                        interval = "USDT&startTime=" + startTime + "&endTime=" + endTime + "&interval=1h";
                        this.getCoinHistoricalData(interval);
                        break;
                    case "m":
                        startTime = new Date(new Date().setDate(new Date().getDate() - 30)).getTime();
                        interval = "USDT&startTime=" + startTime + "&endTime=" + endTime + "&interval=8h";
                        this.getCoinHistoricalData(interval);
                        break;
                    case "y":
                        startTime = new Date(new Date().setDate(new Date().getDate() - 365)).getTime();
                        interval = "USDT&startTime=" + startTime + "&endTime=" + endTime + "&interval=1w";
                        this.getCoinHistoricalData(interval);
                        break;
                }
            },

            createChart(coinHistory) {
                let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);
                // chart.colors.list = [
                //     am4core.color("#F9F871")
                // ];
                chart.paddingRight = 20;

                let data = [];
                let visits = 10;
                for (let i = 1; i < coinHistory.length; i++) {
                    visits += Math.round((Math.random() < 0.5 ? 1 : -1) * Math.random() * 10000);
                    // data.push({date: new Date(2022, 0, i), name: "name" + i, value: visits});
                    data.push({
                        date: coinHistory[i]['date'],
                        name: "name" + i,
                        value: coinHistory[i]['value']
                    });
                }

                chart.data = data;

                let dateAxis = chart.xAxes.push(new am4charts.DateAxis());
                dateAxis.renderer.grid.template.location = 0;
                dateAxis.renderer.labels.template.fill = am4core.color("#ffffff");

                let valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
                valueAxis.tooltip.disabled = true;
                valueAxis.renderer.minWidth = 35;
                valueAxis.renderer.labels.template.fill = am4core.color("#ffffff");

                let series = chart.series.push(new am4charts.LineSeries());
                series.dataFields.dateX = "date";
                series.dataFields.valueY = "value";
                series.tooltipText = "{valueY.value}";
                chart.cursor = new am4charts.XYCursor();

                let scrollbarX = new am4charts.XYChartScrollbar();
                scrollbarX.series.push(series);
                chart.scrollbarX = scrollbarX;
                chart.scrollbarX.parent = chart.bottomAxesContainer;

                this.chart = chart;
            }
        },
        computed() {
        },
        created() {
            // this.getCoinDetails();
        },

        mounted() {

            let startTime = new Date(new Date().setDate(new Date().getDate() - 1)).getTime();
            let endTime = new Date().getTime();
           let  interval = "USDT&startTime=" + startTime + "&endTime=" + endTime + "&interval=15m";
            this.getCoinHistoricalData(interval);
        },
        unmounted() {
            if (this.chart) {
                this.chart.dispose();
            }
        }
    }
</script>

<style scoped>
    #container {
        text-align: center;
    }

    p {
        color: white;
    }

    .backdrop {
        top: 0;
        position: fixed;
        background: rgba(0, 0, 0, 0.5);
        width: 100%;
        height: 100%;
        animation: fadeOut 1s;
    }

    .modal {
        width: 50%;
        padding: 20px;
        margin: 100px auto;
        background: #303655;
        border-radius: 10px;
        color: white;
        animation: fadeIn 0.5s;
    }

    @keyframes fadeIn {
        0% {
            opacity: 0;
        }
        100% {
            opacity: 1;
        }
    }

    @keyframes fadeOut {
        100% {
            opacity: 1
        }
        0% {
            opacity: 0
        }
    }


    .close {
        text-align: right;
        color: yellow;
        cursor: pointer;
        font-size: 20px;
        transition-duration: 0.5s;
    }

    .close:hover {
        color: yellowgreen;
    }

    .header {
        display: flex;
    }

    .item {
        margin: auto; /* Magic! */
    }

    a {
        text-decoration: none;
        color: white;
        transition-duration: 0.5s;
    }

    a:hover {
        color: yellowgreen;
    }

    .chart {
        width: 100%;
        height: 500px;
    }

    .btn {
        padding: 5px 10px 5px 10px;
        background: #00003B;
        border: solid 1px yellow;
        border-radius: 5px;
        color: yellow;
    }

    .btn:focus {
        color: black;
        background: white;
    }

</style>