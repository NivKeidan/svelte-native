<script>
    import Upcoming from './Upcoming.svelte';
    import Today from './Today.svelte';
    import Future from './Future.svelte';
    import General from './General.svelte';
    import { navigate } from 'svelte-native';
    import { onMount } from 'svelte';
    import { timeCompareFn} from './time';
    import { getDaysFromToday } from './date';
    import { SwipeDirection } from "tns-core-modules/ui/gestures";
    import { PAGES, NO_DATE } from "./constants";

    const orderedPageNames = [PAGES.GENERAL, PAGES.TODAY, PAGES.UPCOMING, PAGES.FUTURE];
    const delayBetweenServerReachAttempts = 10000;
    let currentPage = PAGES.TODAY;
    let offline = true;
    let afterInitialLoad = false;
    let todayEntries = [];
    let upcomingEntries = [];
    let futureEntries = [];
    let generalEntries = [];
    let highestId = 0;

    onMount( async () => {
        await loadDataFromServer();
    });

    function loadDataFromServer() {
        fetch("http://1f00bb37f0ef.ngrok.io").then(r => r.json()).then(data => {
            clearEntries();
            populatePages(data);
            offline = false;
            afterInitialLoad = true;
            setTimeout(loadDataFromServer, delayBetweenServerReachAttempts);
        }).catch(e => {
            console.log("failed loading data", e);
            offline = true;
            setTimeout(loadDataFromServer, delayBetweenServerReachAttempts);
        });
    }

    function clearEntries() {
        todayEntries = [];
        futureEntries = [];
        upcomingEntries = [];
        generalEntries = [];
    }

    function populatePages(data) {
        data.forEach(e => insertEntry(e));
    }

    function insertEntry(entry) {
        let page = PAGES.FUTURE;
        if (entry.date <= getDaysFromToday(7))
            page = PAGES.UPCOMING;
        if (entry.date <= getDaysFromToday(0))
            page = PAGES.TODAY;
        if (entry.date === NO_DATE) {
            page = PAGES.GENERAL;
        }
        insertEntryToPage(page, entry);
    }

    function insertEntryToPage(page, entry) {
        let entriesObject = getEntriesArray(page);
        entriesObject.push(entry);
        sortPage(page);
        updatePage(page);
    }

    async function updatePage(page) {
        // Weird Svelte hacks to make the dom reflect changes
        switch (page) {
            case PAGES.TODAY:
                todayEntries = [...todayEntries];
                break;
            case PAGES.UPCOMING:
                upcomingEntries = [...upcomingEntries];
                break;
            case PAGES.FUTURE:
                futureEntries = [...futureEntries];
                break;
            case PAGES.GENERAL:
                generalEntries = [...generalEntries];
                break;
            default:
                console.log("could not get entries object from input: ", page);
                return null;
        }
    }

    function sortPage(page) {
        let entriesArr = getEntriesArray(page);
        entriesArr.sort((a,b) => {
            if (a.date === b.date)
                return timeCompareFn(a.time,b.time);
            else
                return a.date - b.date;
        });
    }

    function getNextPageName() {
        switch (currentPage) {
            case PAGES.GENERAL:
                return PAGES.TODAY;
            case PAGES.TODAY:
                return PAGES.UPCOMING;
            case PAGES.UPCOMING:
                return PAGES.FUTURE;
        }
        return null;
    }

    function getPrevPageName() {
        switch (currentPage) {
            case PAGES.TODAY:
                return PAGES.GENERAL;
            case PAGES.UPCOMING:
                return PAGES.TODAY;
            case PAGES.FUTURE:
                return PAGES.UPCOMING;
        }
        return null;
    }

    function getPageObject(n) {
        switch (n) {
            case PAGES.GENERAL:
                return General;
            case PAGES.TODAY:
                return Today;
            case PAGES.UPCOMING:
                return Upcoming;
            case PAGES.FUTURE:
                return Future;
        }
    }

    function getEntriesArray(n) {
        switch(n) {
            case PAGES.GENERAL:
                return generalEntries;
            case PAGES.TODAY:
                return todayEntries;
            case PAGES.UPCOMING:
                return upcomingEntries;
            case PAGES.FUTURE:
                return futureEntries;
        }
    }

    export function handleSwipe(args) {
        let nextPage;

        if (args.direction === SwipeDirection.left)
            nextPage = getNextPageName();
        else if (args.direction === SwipeDirection.right)
            nextPage = getPrevPageName();
        if (nextPage === null) {
            return;
        }
        currentPage = nextPage;
        navigate({
            page: getPageObject(nextPage),
            frame: "main_frame",
            props: {entries: getEntriesArray(nextPage)}
        });
    }

</script>

<page>
    <stackLayout>
            <activityIndicator top="0" left="0" busy={offline}/>
        <frame top="0" left="0" height="100%" width="100%" id="main_frame" on:swipe={handleSwipe}>
            <Today bind:entries={todayEntries}/>
        </frame>
    </stackLayout>
</page>

<style>
</style>
