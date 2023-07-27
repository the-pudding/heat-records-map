<script>
    export let data;
    export let dataType;


    const resortData = () => {
        data = data.sort((a,b) => {
            return +a.days_since_daily - +b.days_since_daily
        })
    }


    $: dataType, resortData();

</script>

{#each data as row}
    <p>{row["name"].replace(" Area","")} - {row["days_since_daily"]} days - {row["days_since_val"]}°F on {row["days_since_daily_date"]}</p>
{/each}
