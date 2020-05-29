<script>
    import { Template } from 'svelte-native/components';
    import { getDisplayString } from './time';
    import { handleSwipe } from './App.svelte';

    export let entries = [];

    export let handleSwipeFunc;

</script>

<page>
    <stackLayout>
        <listView items="{entries}" on:swipe={handleSwipe}>
            <Template let:item height="10%">
                <label>
                    <span text="{getDisplayString(item.time)}"/>
                    <span text="{item.text}" textWrap="true" />
                </label>
            </Template>
        </listView>
    </stackLayout>
</page>


<style>
</style>
