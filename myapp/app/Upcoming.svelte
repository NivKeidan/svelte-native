<script>
    import { Template } from 'svelte-native/components';
    import { getDaysFromToday, getDayName } from './date';
    import { timeCompareFn, getDisplayString } from './time';
    import { handleSwipe } from './App.svelte';

    export let entries = [];
    const displayDaysNum = 7;
    let datedEntries = {};

    $: {
        datedEntries = getBaseDatedEntriesObject();
        entries.forEach( e => {
            datedEntries[e.date] = [...datedEntries[e.date], e];
        });
        sortDatedEntries();
        datedEntries = datedEntries;
    }

    function sortDatedEntries() {
        Object.keys(datedEntries).forEach( date => {
            datedEntries[date].sort( (a,b) => { return timeCompareFn(a.time, b.time)});
        });
    }

    function getBaseDatedEntriesObject() {
        let o = {};
        for (let i = 1; i <= displayDaysNum; i++) {
            o[getDaysFromToday(i)] = [];
        }
        return o;
    }


</script>

<page>
    <stackLayout>
        <listView items="{Object.keys(datedEntries)}" on:swipe={handleSwipe}>
            <Template let:item height="15%">
                <label textWrap="{true}">
                    <span text="{getDayName(item)}"/>
                    <span text=" - "/>
                    <span text="{datedEntries[item].map((e) => {
                        let t = getDisplayString(e.time);
                        if (t !== '')
                            return t + ' ' + e.text;
                        else
                            return e.text;
                    }).join(' | ')}"></span>
                </label>
            </Template>
        </listView>
    </stackLayout>
</page>


<style>
</style>
