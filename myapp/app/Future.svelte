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
    <stackLayout>
        <listView items="{Object.keys(datedEntries)}" on:swipe={handleSwipe}>
            <Template let:item height="15%">
                <label>
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
