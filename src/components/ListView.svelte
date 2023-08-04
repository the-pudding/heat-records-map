<script>
    import {groups, timeFormat} from "d3";
    export let recordsData;
    export let dataType;
    export let stationIds;


    


    const formatTime = timeFormat("%m/%d/%y");
    const formatTimeNoYear = timeFormat("%b. %e");
    let dataForChart;

    function formatDate(dataToMake){
        let dateNoSuffix = new Date(dataToMake.slice(0,4), +dataToMake.slice(5,7) -1, dataToMake.slice(8,10));
        return formatTime(dateNoSuffix);
    }

    function ordinal_suffix_of(i) {
        var j = i % 10,
            k = i % 100;
        if (j == 1 && k != 11) {
            return "st";
        }
        if (j == 2 && k != 12) {
            return "nd";
        }
        if (j == 3 && k != 13) {
            return "rd";
        }
        return "th";
    }

    function formatDateText(dataToMake){

        let dateNoSuffix = new Date(dataToMake.slice(0,4), +dataToMake.slice(5,7) -1, dataToMake.slice(8,10));
        let text = formatTimeNoYear(dateNoSuffix);

        let suffix = ordinal_suffix_of(dataToMake.slice(8,10));

        return text + suffix;
    }

    const resortData = () => {

        let yourDate = new Date()
        // let dateNoSuffix = new Date(dataToMake.slice(0,4), +dataToMake.slice(5,7) -1, dataToMake.slice(8,10));
        // yourDate = yourDate.toISOString().split('T')[0];
        recordsData.forEach(d => {
            let dateToSubtract = new Date(d["date"].slice(0,4), d["date"].slice(5,7) -1, d["date"].slice(8,10));
            d["days_since"] = Math.floor((yourDate - dateToSubtract) / (1000*60*60*24)); //+yourDate.replace("-","").replace("-","") - +d["date"].replace("-","").replace("-","")
            d["name"] = stationIds.get(d.station)[0]["name"];
            d["state_abbr"] = stationIds.get(d.station)[0]["state_abbr"];
        })


        dataForChart = groups(recordsData.filter(d => d["record"] == dataType), d => d["days_since"]);
    }


    $: dataType, resortData();
    $: console.log(dataForChart)

</script>

{#if dataForChart}
    {#if dataForChart.length == 0}
        <p style="" class="day-right-top no-data">No all-time heat records were set in the past 14 days.</p>
    {/if}
    {#each dataForChart as day}
        <div class="day">
            <div class="day-left">
                <p class="ago">{day[0]} {day[0] < 2 ? " day" : " days"} ago</p>
                <p class="full">{formatDate(day[1][0]["date"])}</p>
            </div>
            <div class="day-right">
                {#each groups(day[1], d => d["record"]).sort((a,b) => a[0].localeCompare(b[0])) as recordType}
                    {#each groups(recordType[1], d => d["tie"]).sort((a,b) => a[0].localeCompare(b[0])) as recordTie}
                        <div class="record-group">
                            <div class="day-record-type">
                                {#if recordType[0] == "daily"}
                                    {#if recordTie[0] == "true"}
                                        <p style="" data-tie={recordTie[0]} class="day-right-top">Tied the record for hottest {formatDateText(recordType[1][0]["date"])}:</p>
                                    {:else}
                                        <p data-tie={recordTie[0]} class="day-right-top">Hottest {formatDateText(recordType[1][0]["date"])} on record in:</p>
                                    {/if}
                                {:else}
                                    {#if recordTie[0] == false}
                                    <p data-tie={recordTie[0]} class="day-right-top">Hottest temperature ever recorded in:</p>
                                    {:else}
                                        <p class="day-right-top">Tied the record for the hottest temperature ever in:</p>
                                    {/if}
                                {/if}
                            </div>
                            {#each recordTie[1].sort((a,b) => a.name.localeCompare(b.name)) as record}
                                <div class="record">
                                    <p class="city"><span>{record.name.replace(" Area","")}, {record["state_abbr"]},</span> {record["temp"]}°F</p>
                                </div>
                            {/each}
                        </div>
                    {/each}
                    
                {/each}
                
            </div>

        </div>
        
        
        
    {/each}
{/if}
<!-- {#each data as row}
    <p>{row["name"].replace(" Area","")} - {row["days_since_daily"]} days - {row["days_since_val"]}°F on {row["days_since_daily_date"]}</p>
{/each} -->

<style>
    .day-record-type {
        margin-bottom: 5px;
    }
    .record-group {
        margin-bottom: 25px;
    }
    .day-left {
        margin-right: 20px;
        width: 120px;
    }

    .day-right-top {
        font-size: 16px;
        opacity: .8;
        margin-bottom: 5px;
        line-height: 1.2;
        margin-top: 2px;
    }

    .city span {
        font-weight: 600;
    }
    .day {
        display: flex;
        margin-bottom: 40px;
    }

    p {
        margin: 0;
    }
    .ago {
        font-weight: 600;
        margin: 0;
        font-size: 18px;
        line-height: 1;
    }

    .full {
        opacity: .7;
        font-size: 14px;
        margin: 0;
    }

    .no-data {
        font-size: 24px;
    }

    @media only screen and (max-width: 600px) {
        .city {
            font-size: 14px;
        }
        .ago {
            margin-bottom: 8px;
        }
    }
</style>
