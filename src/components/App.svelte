<script>
    import Chart from '../components/Chart.svelte';

    export let name = "World";
</script>

<main>
    <h1>Hello {name}!</h1>
    <p>Visit <a href="https://svelte.dev/tutorial"> Svelte tutorial</a> to learn how to build Svelte apps.</p>

    <Chart/>
</main>

<style>
    main {
        text-align: center;
    }
</style>