<script>
    import { Template } from 'svelte-native/components';
    import { handleSwipe } from './App.svelte';

    export let entries = [];

</script>

<page>
    <stackLayout>
        <listView items="{entries}" on:swipe={handleSwipe}>
            <Template let:item>
                <label height="25%" text="{item.text}" textWrap="true" />
            </Template>
        </listView>
    </stackLayout>
</page>


<style>
</style>
