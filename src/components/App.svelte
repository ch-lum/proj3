<script>
    import Chart from '../components/Chart.svelte';
    import { inputValue } from '../store.js';

    function handleInputChange(event) {
        inputValue.set(event.target.value);
    }
</script>

<main>
    <div id="wrapper" class="wrapper">
        <div class="titles">
            <h1>When our words are used</h1>
            <h2>Hover, click, and search through the vocabulary of 1360 Facebook comments on a POTUS post announcing the creation of 14 million jobs. Only includes words that appear in 10 or more comments.</h2>
        </div>
        <input type="text" id="substring-input" class="search" bind:value={$inputValue} on:input={handleInputChange} placeholder="Search here">

        <div id="tooltip" class="tooltip">
            <div class="tooltip-range" id="range"></div>
        </div>

        <div id="comments-neg" class="comments-neg">
            <div class="neg-header" id="neg-header"></div>
            <div id="neg-body" class="neg-body"></div>
        </div>

        <div id="comments-pos" class="comments-pos">
            <div class="pos-header" id="pos-header"></div>
            <div id="pos-body" class="pos-body"></div>
        </div>

        <Chart class="chart" inputValue={$inputValue}/>
    </div>
    

</main>

<style>
    /* main {
        text-align: center;
    } */

    .wrapper {
        position: relative;
        /* border: 2px solid blue; */
        padding: 0px;
        justify-content: center;
        align-items: center;
        text-align: center;
    }

    .tooltip {
        opacity: 0;
        width: 150px;
        height: 50px;
        border: 2px solid darkslategray;
        position: absolute;
        font-family: georgia;
        font-size: calc(1vw+ 1vh);
        text-align: center;
        text-decoration: underline;
        font-weight: bold;
        white-space: pre-line;
        background-color: #262c2c;
    }

    .comments-neg {
        opacity: 0;
        width: 250px;
        height: 250px;
        border: 2px solid darkslategray;
        position: absolute;
        background-color: rgba(255, 230, 230, 0.7);
        padding: 10px;
        overflow-y: scroll;
        overflow-x: hidden;
        font-weight: bold;
    }
    
    .neg-header {
        opacity: 1;
        font-family: helvetica;
        text-align: center;
    }

    .neg-body {
        opacity: 1;
        font-family: courier;
        font-size: 14px;
        text-align: left;
        white-space: pre-line;
    }

    .comments-pos {
        opacity: 0;
        width: 250px;
        height: 250px;
        border: 2px solid darkslategray;
        position: absolute;
        background-color: rgba(230, 255, 230, 0.7);
        padding: 10px;
        overflow-y: scroll;
        overflow-x: hidden;
        font-weight: bold;
    }
    
    .pos-header {
        opacity: 1;
        font-family: helvetica;
        text-align: center;
    }

    .pos-body {
        opacity: 1;
        font-family: courier;
        font-size: 14px;
        text-align: left;
        white-space: pre-line;
    }

    .search {
        font-family: Helvetica;
        font-size: 24px;
        font-style: italic;
        position: absolute;
        top: 60px;
        left: 850px;
        z-index: 1;
        text-align: center;
        color: gray;
        background-color: rgba(174, 185, 207, 0.452);
        border: none;
        border-bottom: 3px solid gray;
        border-radius: 6px
    }

    .titles {
        position: absolute;
        left: 100px;
        width: 700px;
        text-align: left;
        /* border: 2px solid red; */
        z-index: -1;
    }

    h1 {
        font-family: Georgia;

    }
    
    h2 {
        font-family: Helvetica;
        font-weight: normal;
        font-size: 100%;
        color: darkslategray;
    }
</style>