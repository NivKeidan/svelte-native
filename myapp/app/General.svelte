<script>
    import { Template } from 'svelte-native/components';
    import { handleSwipe } from './App.svelte';

    export let entries = [];

</script>

<page>
    <actionBar title="General"/>
    <stackLayout>
        <listView items="{entries}" on:swipe={handleSwipe} height="100%">
            <Template let:item>
                <label text="{item.text}" textWrap="true"/>
            </Template>
        </listView>
    </stackLayout>
</page>


<style>
</style>
