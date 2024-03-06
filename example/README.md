
# SAMPLE CODE FOR VALIDATING SCHEMA AND DATA

This is the sample code for testing the schema and json data.

To test the schema and json data file compatbility execute

`python3 example/test_avro.py example/test.avsc example/test.json`

`test_avro.py`: main driver program
`test.avsc`: Schema file
`test.json`: Json data file

## Changes

This is the fork of leocalm/avro_validator

1. Updated float and double array type to accept both integer `123` and decimal `123.45` numbers
2. Changed the code for record type to accept `logicalType` as parameter. Additional code needs to added to check type matching the logical type.
3. Updated the record type to accept `version` as paramter. (Note: Version field should be removed as per latest specification. We can use namespace instead of version.)

### Example1

```
RGOUDA-M-38P0:avro_validator rgouda$ python3 example/test_avro.py example/schema/fmc-6.6.0.avsc example/json/fmc-6.6.0/922712ac-0002-437c-9eda-13f33adcd318/c8934c23-c917-4484-ba08-429a3b035b45_1631876426566627481_418 
The fields from value [{'SSLStats', 'fmc', 'snortRestart', 'theme', 'recordedAt', 'analysis', 'recordVersion', 'managedDevices', 'recordType'}] differs from the fields of the record type [{'SSLStats', 'deploymentData', 'fmc', 'snortRestart', 'theme', 'recordedAt', 'analysis', 'recordVersion', 'managedDevices', 'recordType'}]. The following fields are required, but not present: [{'deploymentData'}].
```

`deploymentData` is missing in the json files. It is required as per the schema.

### Example2

```
RGOUDA-M-38P0:avro_validator rgouda$ python3 example/test_avro.py example/schema/fdm-1.2.avsc example/json/fdm-1.2/137764f9-02ba-4c8c-8445-c9d272493804/24f37f62-a624-4fbc-aadc-c126a27a6c16_1631840278049042475_897 
JSON {'type': 'array', 'name': 'user_agent_summary_array', 'doc': 'This will contain userAgentStats array.', 'items': {'type': 'record', 'name': 'userAgentStats', 'doc': 'List of all user agents and respective usage statistics.', 'fields': [{'name': 'userAgent', 'type': 'string', 'doc': 'User Agent.'}, {'name': 'urlFrequencyMap', 'type': {'type': 'array', 'items': {'type': 'record', 'name': 'urlFrequencyMaps', 'fields': [{'name': 'url', 'type': 'string'}, {'name': 'count', 'type': 'int'}]}}}, {'name': 'userAgentFrequency', 'type': 'int', 'doc': 'The total number of times user agent is present in apache log files.'}]}}
KEYS in schema:  dict_keys(['type', 'name', 'doc', 'items'])
KEYS allowed:  {'type', 'items'}
Error parsing the schema. Problem found:
 Error parsing the field [userAgentStats]: The ArrayType can only contains {'type', 'items'} keys
 ```

`userAgentStats` field in schema has below structure.

```
                "type": "array",
                "name": "user_agent_summary_array",
                "doc": "This will contain userAgentStats array.",
                "items": {
```

As per the latest specification <https://avro.apache.org/docs/current/spec.html#Arrays>

```
Arrays use the type name "array" and support a single attribute:

items: the schema of the array's items.
For example, an array of strings is declared with:

{
  "type": "array",
  "items" : "string",
  "default": []
}
```

so we need to remove `name` and `doc` fields and `userAgentStats` structure should be like

```
                "type": "array",
                "items": {
```

### Example 3

```
RGOUDA-M-38P0:avro_validator rgouda$ python3 example/test_avro.py example/schema/fdm-1.2.avsc example/json/fdm-1.2/137764f9-02ba-4c8c-8445-c9d272493804/24f37f62-a624-4fbc-aadc-c126a27a6c16_1631840278049042475_897 
The fields from value [{'vpnProfileStats', 'recordedAt', 'licenseActivated', 'systemHealth', 'userAgentStats', 'snort3RuntimeStatistics', 'versions', 'identityUsageStats', 'haUsageStats', 'sslRulesStats', 'recordVersion', 'deviceInfo', 'recordType', 'intrusionUsageStats', 'accessControlStats'}] differs from the fields of the record type [{'recordedAt', 'licenseActivated', 'systemHealth', 'userAgentStats', 'versions', 'deviceInfo', 'recordType', 'recordVersion'}]. The following fields are not in the schema, but are present: [set()].
```

if we take the diff of the above 2 record list
`'vpnProfileStats', 'recordedAt', 'licenseActivated', 'systemHealth', 'userAgentStats', 'snort3RuntimeStatistics', 'versions', 'identityUsageStats', 'haUsageStats', 'sslRulesStats', 'recordVersion', 'deviceInfo', 'recordType', 'intrusionUsageStats', 'accessControlStats'`

`'recordedAt', 'licenseActivated', 'systemHealth', 'userAgentStats', 'versions', 'deviceInfo', 'recordType', 'recordVersion'`

The fields are missing in the schema `fdm-1.2.avsc` that needs to be added for device with id `24f37f62-a624-4fbc-aadc-c126a27a6c16` of org `137764f9-02ba-4c8c-8445-c9d272493804`

1. vpnProfileStats
2. snort3RuntimeStatistics
3. identityUsageStats
4. haUsageStats
5. sslRulesStats
6. intrusionUsageStats
7. accessControlStats
