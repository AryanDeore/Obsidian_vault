
### fct_pings cleanup:
- Rejection reasons
	- no_match and No Match → No Match 
	- verification_exists and Verification Exists → Verification 
	- dnc_scrub = ["dnc_scrub"] → DNC Scrub
- Tra`ffic category`
	- 2 external

```csv
"REJECTION_REASON","REJECTION_REASON_COUNT"
,31002420
"no_match",10188418
"No Match",3480998
"p10",1360741
"verification_exists",1196670
"internal_dnc",705301
"Verfication Exists",535087
"FPG: CLIENTS",230596
"blacklist",191938
"[""internal_dnc""]",144656
"DNC",107600
"tcpa_litigator_list",30132
"[""tcpa_litigator_list""]",22153
"tld",17543
"[""dnc_scrub""]",2786
"[""bla""]",1823
"[""webrecon""]",984
"FPG: BLOCK",765
"[""bla"",""tcpa_litigator_list""]",687
"[""webrecon"",""tcpa_litigator_list""]",629
"BLA",551
"[""webrecon"",""tcpa_litigator_list"",""dnc_scrub""]",440
"NUM",321
"[""tcpa_litigator_list"",""dnc_scrub""]",289
"[""bla"",""tcpa_litigator_list"",""dnc_scrub""]",167
"[""bla"",""webrecon"",""tcpa_litigator_list""]",88
"[""bla"",""webrecon"",""tcpa_litigator_list"",""dnc_scrub""]",80
"[""bla"",""dnc_scrub""]",76
"FPG: BLOCK,CLIENTS",63
"blacklist_alliance",53
"[""webrecon"",""dnc_scrub""]",52
"dnc_scrub",36
"webrecon",23
"[""bla"",""webrecon""]",9
"[""bla"",""webrecon"",""dnc_scrub""]",8
"Internal_DNC",4

```





```csv
"TRAFFIC_CATEGORY","TRAFFIC_CATEGORY_COUNT"
,36022420
"External",12239419
"Internal",510250
"No Match",425420
"Blacklist",14809
"RateQuote",9730
"Verfication Exists",5122
"External ",84
"Priority Spanish",36

```