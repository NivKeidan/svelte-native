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
    import { PAGES, NO_DATE, LOCAL_DATA_STRING, LOCAL_TIMESTAMP_STRING } from "./constants";
    import { getString, setString } from "tns-core-modules/application-settings";

    const delayBetweenServerReachAttempts = 3000;
    let currentPage = PAGES.TODAY;
    let offline = true;
    let todayEntries = [];
    let upcomingEntries = [];
    let futureEntries = [];
    let generalEntries = [];
    let highestId = 0;
    let latestTimestamp = 0;
    let headers = null;

    onMount( async () => {
        await loadDataFromServer();
    });

    function loadLocalData() {
        const d = getString(LOCAL_DATA_STRING);
        const ts = getString(LOCAL_TIMESTAMP_STRING);
        if (ts > latestTimestamp) {
            populatePages(JSON.parse(d)).then(() => {
                latestTimestamp = ts;
            })
        }
    }

    function saveLocalData(d, ts) {
        setString(LOCAL_DATA_STRING, JSON.stringify(d));
        setString(LOCAL_TIMESTAMP_STRING, ts);
    }

    function validateHeaders() {
        if (headers === null) {
            headers = new Headers();
            headers.append('Content-Type', 'application/json');
            headers.append('Authorization', 'Basic ' + basicAuthString);
        }
    }

    function propsToStr(obj) {
        let str = '';
        for (let prop in obj) {
            str += prop + "=" + obj[prop] + ",";
        }
        return str;
    }

    function loadDataFromServer() {
        validateHeaders();
        fetch(SERVER_ADDRESS, {
            method: 'GET',
            headers: headers,
        }).then(r => {
            if (r.ok)
                return r.json();
            else
                throw r;
        }).then(data => {
            offline = false;

            if (data.timestamp > latestTimestamp) {
                populatePages(data.data).then( () => {
                    latestTimestamp = data.timestamp;
                    saveLocalData(data.data, data.timestamp);
                })
            }

            setTimeout(loadDataFromServer, delayBetweenServerReachAttempts);
        }).catch(e => {
            let propertyNames = Object.getOwnPropertyNames(e);
            propertyNames.forEach(function(property) {
                let descriptor = Object.getOwnPropertyDescriptor(e, property);
                console.log(property + ":" + e[property] + ":" + propsToStr(descriptor));
            });
            console.log("failed loading data: ", e);
            offline = true;
            loadLocalData();
            setTimeout(loadDataFromServer, delayBetweenServerReachAttempts);
        });
    }

    function clearEntries() {
        todayEntries = [];
        futureEntries = [];
        upcomingEntries = [];
        generalEntries = [];
    }

    async function populatePages(data) {
        clearEntries();
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
        else
            return null;
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

<page actionBarHidden="true">
    <stackLayout >
        <activityIndicator busy={offline}/>
        <frame id="main_frame" on:swipe={handleSwipe}>
            <Today bind:entries={todayEntries}/>
        </frame>
    </stackLayout>
</page>

<style>
</style>
