<script>
    import '../app.css';
    import Header from '$lib/Header.svelte';

</script>


    <div class="bg-light text-dark dark:bg-dark dark:text-light">
        <Header />
        <main class="pb-12">
            <slot />
        </main>
    </div>
