<script>
    import { Template } from 'svelte-native/components';
    import { getDisplayString } from './time';
    import { handleSwipe } from './App.svelte';

    export let entries = [];

</script>

<page>
    <actionBar title="Today"/>
    <stackLayout>
        <listView items="{entries}" on:swipe={handleSwipe} height="100%">
            <Template let:item>
                <gridLayout columns="50, *" rows="*">
                    <label row="0" col="0" text="{getDisplayString(item.time)}"/>
                    <label row="0" col="1" text="{item.text}" textWrap="true" />
                </gridLayout>
            </Template>
        </listView>
    </stackLayout>
</page>


<style>
</style>
