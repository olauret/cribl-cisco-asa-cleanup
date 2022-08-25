# Cisco ASA Clean-up
----

This pack uses lookup files to help drop events, suppress events, and extract fields from surviving events. Using lookup files provides a much cleaner management of potentially hundreds, or thousands, of event types. 

* **For Splunk Delivery**: You can either leave (some of) the fields as index time useful for tstats, accelerated data models, etc. Or you you can re-write _raw as a JSON object with the newly extracted fields. Splunk delivery is set-up in `prep_for_splunk`, a pipeline chained from the `cisco_asa_cleanup` pipeline by default.

* **For Elastic Delivery**: A lookup is used to translate field naming convention from Splunk CIM to Elastic ECS. This lookup is called from the `prep_for_ECS` pipeline, which is optionally chained from the `cisco_asa_cleanup` pipeline.


## Requirements Section

Before you activate the Pack on live data:
* The **asa_drops.csv** file will need to be updated with ASA codes that you do not want
    * Format is `asa_code`,`"comment or memo"`
    * There are 29 codes included by default based on previous experience. YMMV.
* The **asa_parsing.csv** file will need to have codes + regex extractions for the fields you require
    * Format is `asa_code`,`"regex_with_named_capture_group(s)"`
    * The field `_no_matches` will be added to each event with a regex defined
        * If no matches are found, it will be `true` meaning the regex failed
    * If a regex uses the same named group twice (eg, `(?<group>foo)|(?<group>bar)`) you'll need to name the second with _ALT. Eg: `group_ALT`. This will be undone after extraction automatically.
    * There are 179 codes included by default based on previous experience. YMMV.
    * We attempted to maintain compliance with the Splunk CIM in naming fields
* The **asa_suppress.csv** file contains ASA codes that should only be allowed 1 event per 30 seconds per worker process
    * There are 2 codes included by default based on previous experience. YMMV.
* The **splunk2elastic.csv** file contains Splunk CIM to ECS field name mappings
    * Not every field name in either model is covered
    * This lookup is used in the **prep_for_ECS** pipeline
    * The `prep_for_ECS` pipeline will create nested objects for names with periods in them

You're encouraged to add to the included CSVs and submit a pull request!

## Extracted fields

Fields extracted are placed at the top level of the event (eg, metadata or index time field). You can choose either to:
    * Reserialize them, replacing _raw with a JSON payload of selected events
    * Leave _raw alone and keep some fields (as index time data)
    * Use the `prep_for_ECS` pipeline to translate them to ECS naming conventions and format

## Using The Pack

1. Install (Packs -> Add New -> Add from Dispensary)
2. Inspect the optional pipeline rules and select accordingly
    - In particular, **if you're sending to Elastic** you'll want to disable rule 17 and activate rule 18 (in the `cisco_asa_cleanup` pipeline)
3. Download and install a GeoIP db if GeoIP enhancement is desired (see [maxmind.com](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data?lang=en))
4. Modify the lookup files as required for your needs (provided entries may or may not meet your needs)
    * We recommend you download the lookup files to your local system and manage versioning there. Re-upload the files when modified.
5. Point your Cisco ASA log stream to the Pack and an appropriate destination
    * If your ASA logs come in on a dedicated source, you can apply the pack as a pre-processing pipeline



## Release Notes

### Version 1.1.4 - 2022-08-25
    - Readme file was munged up. No functional changes.

### Version 1.1.3 - 2022-08-01
    - Fixed bad regex for 106100, 106102 -- missing backslash before right bracket in last group

### Version 1.1.2 - 2022-02-22
    - The people responsible for the sacking have now been sacked

### Version 1.1.1 - 2022-02-22
    - Dummy release to fix bad packaging

### Version 1.1.0 - 2022-01-19
    - Added many codes to parsing lookup from the Elastic ASA package
    - Added a pipeline to translate Splunk CIM fields to ECS fields
    - Adjusted main pipeline to Chain either the pre_for_splunk or prep_for_ECS pipeline

### Version 1.0.1 - 2021-12-14
    - Added 199015,199016,199017,199018,313001,313004 to drops
    - Added 111007,111008,111010,113005,315011,414003,414004,606001,606002,606003,606004,711004 to parsing

### Version 1.0.0 - 2021-12-06
Replaced wonky aggregation function with straight suppression. So much cleaner. Version bumped to 1.0.

### Version 0.5.2 - 2021-08-11
fixed bad extract for a few parse lookups

### Version 0.5.1 - 2021-08-04
Cleaned up and added some new codes to drop and supress lookups; fixed bad extract for a few parse lookups; added optional aggregation based suppression; updated this doc

### Version 0.3.2 - 2021-07-21
Fixed bad asa_code extract; added option to keep _raw as is; cleaned up docs

### Version 0.3.0 - 2021-07-19
Serialize option added, suppression added with one sample code

### Version 0.2.0 - 2021-07-08
Added readme, and image

### Version 0.1.0 - 2021-07-08
1st release


## Contributing to the Pack
To contribute to the Pack, contact Jon Rust <jrust@cribl.io>


## Contact
To contact us please email <jrust@cribl.io>.


## License
This Pack uses the following license: [`Apache 2.0`](https://github.com/criblio/appscope/blob/master/LICENSE).
