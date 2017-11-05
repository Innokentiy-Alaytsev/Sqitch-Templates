# PostgreSQL #

## Template for generating CREATE SCHEMA change code ##

The template pg-create-schema can be used for generating of boilerplate code for
CREATE SCHEMA project changes.

The template may be used as follows:
```sh
sqitch add schema_change \
	   --requires dependency_change \
	   -m 'test template' \
	   --template pg-create-schema \
	   -s schema=schema_name -s comment="Schema change"
```

The code above will generate CREATE SCHEMA SQL query:
```sql
BEGIN;

CREATE SCHEMA schema_name;

COMMENT ON SCHEMA schema_name
	IS 'Schema change';

COMMIT;
```

The basic tests for the generated query are generated as well:
```sql
BEGIN;
SELECT no_plan ();


SELECT has_schema ('schema_name');

SELECT tables_are (
	'schema_name',
	ARRAY[]::NAME[]);


SELECT finish ();
ROLLBACK;
```
