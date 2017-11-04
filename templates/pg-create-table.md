# PostgreSQL #

## Template for generating CREATE TABLE change code ##

The template pg-create-table can be used for generation of boilerplate code for
CREATE TABLE project changes. The generated tests may be redundant.

The template may be used as follows:
```sh
sqitch add test_template \
	   --requires ac_role \
	   --requires ac_user \
	   -m 'test template' \
	   --template pg-create-table \
	   -s schema=access_control -s table=test_template_tbl \
	   -s column=pk -s type=UUID -s null="NOT NULL" -s unique=true -s default="" \
	   -s column=tt_name -s type=NAME -s null="" -s unique=false -s default="CAST (gen_random_uuid () AS TEXT)"\
	   -s pkey=pk \
	   -s pkey=tt_name \
	   -s fkey=pk -s refschm=access_control -s reftbl=ac_user -s refattr=pk -s match=FULL -s onupd=CASCADE -s ondel=RESTRICT \
	   -s fkey=tt_name -s refschm=access_control -s reftbl=ac_role -s
	   refattr=role_name -s match=FULL -s onupd=CASCADE -s ondel=CASCADE
```

The code above will generate CREATE TABLE SQL query:
```sql
CREATE TABLE test_template_tbl (
	pk UUID NOT NULL,
	tt_name NAME DEFAULT CAST (gen_random_uuid () AS TEXT),

	CONSTRAINT PK_test_template_tbl_pk_tt_name
		PRIMARY KEY (pk, tt_name),

	CONSTRAINT FK_test_template_tbl_pk
		FOREIGN KEY (pk)
		REFERENCES ac_user (pk)
		MATCH FULL
		ON UPDATE CASCADE
		ON DELETE RESTRICT,

	CONSTRAINT FK_test_template_tbl_tt_name
		FOREIGN KEY (tt_name)
		REFERENCES ac_role (role_name)
		MATCH FULL
		ON UPDATE CASCADE
		ON DELETE CASCADE
)
WITH (
	OIDS=FALSE
);
```

The basic tests for the generated query are generated as well, including test
for table existance, columns existance, foreign and primary keys existance and
validity:
```sql
-- Test test_template
SET client_min_messages TO warning;
CREATE EXTENSION IF NOT EXISTS pgtap;
RESET client_min_messages;

BEGIN;
SELECT no_plan();
-- SELECT plan(1);

SET search_path TO access_control, public;


SELECT has_table ('test_template_tbl');
SELECT has_pk    ('test_template_tbl');


-- test_template_tbl.pk
SELECT has_column        ( 'test_template_tbl', 'pk' );
SELECT col_type_is       ( 'test_template_tbl', 'pk', 'uuid' );
SELECT col_not_null      ( 'test_template_tbl', 'pk' );
SELECT col_hasnt_default ( 'test_template_tbl', 'pk' );

-- test_template_tbl.tt_name
SELECT has_column        ( 'test_template_tbl', 'tt_name' );
SELECT col_type_is       ( 'test_template_tbl', 'tt_name', 'name' );
-- No NULL constraints for tt_name
SELECT col_has_default   ( 'test_template_tbl', 'tt_name' );

-- test_template_tbl FOREIGN KEY pk
SELECT col_is_fk ('test_template_tbl', 'pk');
SELECT fk_ok ('test_template_tbl', 'pk', 'ac_user', 'pk');

-- test_template_tbl FOREIGN KEY tt_name
SELECT col_is_fk ('test_template_tbl', 'tt_name');
SELECT fk_ok ('test_template_tbl', 'tt_name', 'ac_role', 'role_name');

-- test_template_tbl PRIMARY KEY tt_name
SELECT col_is_pk ('test_template_tbl', ARRAY[ 'pk', 'tt_name' ]);


PREPARE insert_with_defaults AS
	INSERT INTO access_control.ac_creditials
		DEFAULT VALUES;

SELECT throws_ok ('insert_with_defaults');



PREPARE insert_with_nonexistent_fk AS
	INSERT
		INTO access_control.test_template_tbl (pk, tt_name)
		VALUES (gen_random_uuid ()::UUID, gen_random_uuid ()::NAME);

SELECT throws_ok ('insert_with_nonexistent_fk');


SELECT finish();
ROLLBACK;
```
