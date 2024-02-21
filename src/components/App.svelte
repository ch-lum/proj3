<script>
    import Chart from '../components/Chart.svelte';
</script>

<main>
    <div id="wrapper" class="wrapper">
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

        <Chart class="chart"/>
    </div>
    

</main>

<style>
    /* main {
        text-align: center;
    } */

    .wrapper {
        position: relative;
        border: 2px solid blue;
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

</style>