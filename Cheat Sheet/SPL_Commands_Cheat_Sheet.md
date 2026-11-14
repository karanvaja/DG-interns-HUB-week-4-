# SPL Command Cheat Sheet

| SPL Command | Meaning | Example |
|---|---|---|
| **search** | Base search to filter data | `index=lab EventCode=4625` |
| **stats** | Aggregate data (count, sum, avg) | `... \| stats count by src_ip` |
| **table** | Display results in a specific column format | `... \| table _time, user, src_ip` |
| **eval** | Create or modify a field | `... \| eval status = if(count > 5, "High", "Low")` |
| **where** | Filter results using calculated conditions | `... \| where count > 10` |
| **rex** | Extract fields using Regular Expressions | `... \| rex field=_raw "from (?<ip>\d+\.\d+\.\d+\.\d+)"` |
| **timechart** | Create a time-series graph/chart | `... \| timechart span=5m count by user` |
| **sort** | Sort results by a specific field | `... \| sort - count` |
| **dedup** | Remove duplicate results | `... \| dedup user` |
| **top** | Find the most common values | `... \| top limit=5 Image` |
| **rare** | Find the least common values | `... \| rare limit=5 CommandLine` |
| **rename** | Change the name of a field | `... \| rename src_ip as Source_IP` |
| **fields** | Include or exclude specific fields | `... \| fields - _raw` |
| **head / tail** | Show first N or last N results | `... \| head 10` |
| **transaction** | Group related events together | `... \| transaction src_ip maxspan=5m` |
| **iplocation** | Add geographic location data based on IP | `... \| iplocation src_ip` |