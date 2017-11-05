# PostgreSQL #

## Template for generating CREATE EXTENSION change code ##

The template pg-create-extension can be used for generating of boilerplate code for
CREATE EXTENSION project changes.

The template may be used as follows:
```sh
sqitch add "extension_change" \
	   -m "extension_change comment" \
	   --template pg-create-extension \
	   -s extension="extension_name" \
	   -s schema="extension_schema" \
	   -s version="1.0"
```

The code above will generate CREATE EXTENSION SQL query:
```sql
BEGIN;

CREATE EXTENSION IF NOT EXISTS 'extension_name'
	SCHEMA extension_schema
	VERSION '1.0';

COMMIT;
```

The basic tests for the generated query are generated as well:
```sql
BEGIN;
SELECT plan (1);


SELECT has_extension ('extension_name');


SELECT finish ();
ROLLBACK;
```
