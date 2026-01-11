# Multiscrape for Propane Price Info
I have this in my multiscrape.yaml file referenced from configuration.yaml to pull in the latest price in my area.
## Multiscrape.yaml
```yaml
- resource: https://www.eia.gov/dnav/pet/pet_pri_wfr_dcus_snc_w.htm
  name: NC Residential Propane Price Info
  scan_interval: 86400
  headers:
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.9
    Connection: keep-alive
  timeout: 60
  verify_ssl: true
  
  sensor:
    - name: Current NC Propane Price
      select: "tr.DataRow:nth-of-type(3) td.Current2"
      value_template: "{{ value if value else None }}"
      unit_of_measurement: "$/gal"
      attributes:
        - name: week
          select: "th.Series5:nth-child(7)"
```
