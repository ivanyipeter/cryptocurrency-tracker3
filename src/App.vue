<template>
    <CoinDetailsModal v-if="showCoinModal" :coinId="coinId" :coinSymbol="coinSymbol" :coinHttp="coinHttp"
                      @close="closeCoinDetailsModal">
    </CoinDetailsModal>

    <div class="header">
        <h1>MARKETS</h1>
        <a href="https://github.com/ivanyipeter/cryptocurrency-tracker3" target="_blank">
            <div class="git-flash top-right"><img class="github-logo" src="./assets/GitHub-Mark.png" alt="alt-text">
            </div>
        </a>
    </div>
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
            <th>Price Change
                <div v-on:click="sortByPriceChange" :class="this.priceChangeSortAsc ? 'up-arrow' : 'down-arrow' "></div>
            </th>
            <th>Market Cap
                <div v-on:click="sortByMarketCap" :class="this.marketCapSortAsc ? 'up-arrow' : 'down-arrow' "></div>
            </th>
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
                showCoinModal: false,
                marketCapSortAsc: false,
                priceChangeSortAsc: false
            }
        },
        methods: {
            getCoins() {
                fetch("https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=100&page=1&sparkline=false")
                    .then(res => res.json())
                    .then(data => {
                        this.currencyList = data;
                    }).catch(e => {
                    alert(e)
                });
            },
            sortByMarketCap() {
                this.marketCapSortAsc = !this.marketCapSortAsc;
                if (this.marketCapSortAsc) {
                    this.getCoinsByMarketCapAsc();
                } else {
                    this.getCoinsByMarketCapDesc();
                }
            },
            getCoinsByMarketCapAsc() {
                this.currencyList = this.currencyList.sort((a, b) => (a.market_cap > b.market_cap) ? 1 : (a.market_cap < b.market_cap) ? -1 : 0);
            },
            getCoinsByMarketCapDesc() {
                this.currencyList = this.currencyList.sort((a, b) => (a.market_cap < b.market_cap) ? 1 : (a.market_cap > b.market_cap) ? -1 : 0);
            },
            sortByPriceChange() {
                this.priceChangeSortAsc = !this.priceChangeSortAsc;
                if (this.priceChangeSortAsc) {
                    this.getCoinsByPriceChangeAsc();
                } else {
                    this.getCoinsByPriceChangeDesc();
                }
            },
            getCoinsByPriceChangeAsc() {
                this.currencyList = this.currencyList.sort((a, b) => (a.price_change_24h > b.price_change_24h) ? 1 : (a.price_change_24h < b.price_change_24h) ? -1 : 0);
            },
            getCoinsByPriceChangeDesc() {
                this.currencyList = this.currencyList.sort((a, b) => (a.price_change_24h < b.price_change_24h) ? 1 : (a.price_change_24h > b.price_change_24h) ? -1 : 0);
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
            },
        },
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
        font-family: "Segoe UI", serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
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

    .header {
        color: white;
        margin-bottom: 25px;
        text-align: center;
        font-size: 25px;
    }

    h1 {
        margin: 0px;
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
        background: #303655;
        color: yellow;
        border-radius: 5px;
        transition-duration: 0.5s;
    }

    button:hover {
        background: white;
        color: black;
    }

    .up-arrow {
        width: 0;
        height: 0;
        border: solid 5px transparent;
        background: #00003B;
        border-bottom: solid 7px white;
        border-top-width: 0;
        cursor: pointer;
        display: inline-block;
    }

    .down-arrow {
        width: 0;
        height: 0;
        border: solid 5px transparent;
        background: #00003B;
        border-top: solid 7px white;
        border-bottom-width: 0;
        margin-top: 1px;
        cursor: pointer;
        display: inline-block;
    }

    .top-right {
        /*position: absolute;*/
        /*top: 60px;*/
        /*left: 104rem;*/
    }

    .github-logo {
        padding: 0;
        height: 30px;
        opacity: 20%;
    }

    .git-flash {
        animation: blinker 5s infinite;
    }

    @keyframes blinker {
        30% {
            opacity: 60%;
        }
    }
</style>
