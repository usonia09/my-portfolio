<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";

    let data = [];

    onMount(async () => {
        data = await d3.csv("loc.csv", row => ({
            ...row,
            line: Number(row.line), // or just +row.line
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));

        let commits = [];
        commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
                    let first = lines[0];
                    let {author, date, time, timezone, datetime} = first;
                    let ret = {
                        id: commit,
                        url: "https://github.com/vis-society/lab-7/commit/" + commit,
                        author, date, time, timezone, datetime,
                        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                        totalLines: lines.length
                    };

                    // Like ret.lines = lines
                    // but non-enumerable so it doesn’t show up in JSON.stringify
                    Object.defineProperty(ret, "lines", {
                        value: lines,
                        configurable: true,
                        writable: true,
                        enumerable: false,
                    });

                    return ret;
                });

    });

</script>

<svelte:head>
    <title>Meta</title>
</svelte:head>

<section>
    <h2>Summary Stats</h2>
    <dl class="stats">
        <dt>Total <abbr title="Lines of code">LOC</abbr></dt>
        <dd>{data.length}</dd>
        <!-- <dt>Total <abbr title="Total number of Commits">COMS</abbr></dt>
        <dd>{commits.length}</dd> -->
    </dl>
    
</section>
