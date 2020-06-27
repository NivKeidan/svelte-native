<script>
    import { Template } from 'svelte-native/components';
    import { getDisplayDate } from './date';
    import { handleSwipe } from './App.svelte';

    export let entries = [];
    let datedEntries = {};

    $: {
        datedEntries = {};
        entries.forEach( e => {
            if (!datedEntries[e.date])
                datedEntries[e.date] = [];
            datedEntries[e.date] = [...datedEntries[e.date], e];
        });
        sortDatedEntries();
        datedEntries = datedEntries;
    }

    function sortDatedEntries() {
        Object.keys(datedEntries).forEach( date => {
            datedEntries[date].sort( (a,b) => { return a.time - b.time});
        });
    }
</script>

<page>
    <actionBar title="Future"/>
    <stackLayout>
        <listView items="{Object.keys(datedEntries)}" on:swipe={handleSwipe} height="100%">
            <Template let:item>
                <label height="15%">
                    <span text="{getDisplayDate(item)}"/>
                    <span text=" - "/>
                    <span text="{datedEntries[item].map((e) => e.text).join(', ')}" textWrap="true"></span>
                </label>
            </Template>
        </listView>
    </stackLayout>
</page>


<style>
</style>
