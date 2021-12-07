# Splunk Cheatsheet


## Stats:
```
source="apache.access_combined.log"
|  stats sum(bytes) as total_bytes
|  eval total_bytes=tostring(total_bytes, "commas")
```

```
source="apache.access_combined.log"
| top limit=4 file 
| stats sum(count) as "Total"
```

## Tables:
```
source="apache.access_combined.log"
|  table clientip, user, action
|  where isnotnull(action)
|  rename action as ACTION, user as USER, clientip as CLIENTIP
|  fields - USER
```

## Duplicate Entries:
```
source="apache.access_combined.log"
| table clientip
| dedup clientip
```
```
| stats count by clientip
| sort count
```

## Time:
```
source="apache.access_combined.log"
| eval unix_time = strptime(_time, "%s")
| eval readable_time = strftime(unix_time, "%d-%m-%Y %H:%M")
| table _time, unix_time, readable_time
```

## If/Else OR Case:
```
source="apache.access_combined.log"
| rex field=_raw "^(?:[^\"]*\"){4}\s(?P<response_code>\d+)"
| table clientip, response_code
| eval status_code = if ((response_code == 200), "success", "failure")
| where response_code=401
```

```
source="apache.access_combined.log"
| rex field=_raw "^(?:[^\"]*\"){4}\s(?P<response_code>\d+)"
| eval status_code = CASE ((response_code == 200), "success", 
    (response_code == 401), "unauthorized", (response_code == 404), "not found")
| stats count by response_code, status_code
```

## Like Conditions:
```
source="apache.access_combined.log"
| where like (clientip, "10%")
| table clientip, user
| dedup clientip
| search user="tcarroll12"
```

## Lookups:
- Create lookup file http_status.csv
```
| inputlookup http_status.csv | where status = 100
```

- Create definetion file as statusinfo

```
source="apache.access_combined.log"
| rex field=_raw "^(?:[^\"]*\"){4}\s(?P<status>\d+)"
| table clientip, status
| lookup statusinfo, status OUTPUT status_description
| stats count by status, status_description
```