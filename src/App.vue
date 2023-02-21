<template>
    <CoinDetailsModal v-if="showCoinModal" :coinId="coinId" :coinSymbol="coinSymbol" :coinHttp="coinHttp"  @close="closeCoinDetailsModal">
    </CoinDetailsModal>

    <h1>Markets</h1>
    <input v-model="searchCoin" placeholder="Search..">
    <br>
    <table>
        <thead>
        <tr>
            <th style="width: 5%">#</th>
            <th style="width: 5%"></th>
            <th style="width: 10%"></th>
            <th style="width: 10%">Name</th>
            <th>Current Price</th>
            <th>Price Change</th>
            <th>Market Cap</th>
            <th></th>
        </tr>
        </thead>
        <tr v-for="(currency,index) in filteredList" :key="index">
            <td>{{index}}</td>
            <td><img class="coinImg" :src="currency.image"></td>
            <td>{{currency.symbol.toUpperCase()}}</td>
            <td>{{currency.name}}</td>
            <td>{{currency.current_price}}</td>
            <td :class="priceChangeColor(currency.price_change_24h)">{{currency.price_change_24h}}</td>
            <td>{{currency.market_cap}}</td>
            <td>
                <button @click="showDetails(currency.id, currency.symbol, currency.image)">Detail</button>
            </td>
        </tr>
    </table>
</template>

<script>

    import CoinDetailsModal from "./components/CoinDetailsModal";

    export default {
        name: 'App',
        components: {CoinDetailsModal},
        data() {
            return {
                currencyList: [],
                searchCoin: "",
                input: null,
                coinId: null,
                coinSymbol: null,
                coinHttp: null,
                showCoinModal: false
            }
        },
        methods: {
            async getCoins() {
                fetch("https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=100&page=1&sparkline=false")
                    .then(res => res.json())
                    .then(data => {
                        this.currencyList = data;
                    }).catch(e => {
                    alert(e)
                });
            },
            showDetails(coinId, coinSymbol, coinHttp) {
                this.coinId = coinId;
                this.coinSymbol = coinSymbol;
                this.coinHttp = coinHttp;
                this.showCoinModal = true;
            },
            priceChangeColor(price) {
                return price > 0 ? 'price-up' : 'price-down'
            },

            closeCoinDetailsModal() {
                this.showCoinModal = false;
            }
        }
        ,
        computed: {
            filteredList() {
                return this.currencyList.filter(f => f.name.toUpperCase().includes(this.searchCoin.toUpperCase()) || f.symbol.toUpperCase().includes(this.searchCoin.toUpperCase()));
            }
        },
        mounted() {
            this.getCoins();
        },
        created() {
        }
    }
</script>

<style>
    body {
        background: #00003B;
    }

    #app {
        /*font-family: Avenir, Helvetica, Arial, sans-serif;*/
        font-family: "Segoe UI", serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        /*text-align: center;*/
        /*margin-top: 100px;*/
        background: #00003B;
    }
</style>

<style scoped>
    table {
        color: white;
        border-collapse: collapse;
        width: 60%;
        margin-left: auto;
        margin-right: auto;
    }

    td, th {
        border-bottom: solid black 1px;
        text-align: left;
        padding-top: 5px;
        padding-bottom: 5px;
    }

    .coinImg {
        height: 30px;
    }

    /*thead {*/
    /*    font-size: 20px;*/
    /*}*/

    h1 {
        color: white;
        margin-bottom: 20px;
        text-align: center;
    }

    input {
        display: block;
        color: white;
        margin-left: auto;
        margin-right: auto;
        width: 60%;
        background: #303655;
        border: 1px solid black;
        border-radius: 3px;
        padding: 10px;
    }

    .price-down {
        color: red;
    }

    .price-up {
        color: greenyellow;
    }

    ::placeholder {
        color: white;
    }

    button {
        padding: 5px 10px 5px 10px;
        background: #00003B;
        border: none;
        color: yellow;
    }

    button:hover {
        color: greenyellow;
    }


</style>
