
-- assume the accountadmin role
USE ROLE accountadmin;

-- create the tb_doc_ai database
CREATE OR REPLACE DATABASE tb_doc_ai;

-- create the raw_doc schema
CREATE OR REPLACE SCHEMA tb_doc_ai.raw_doc;

-- create the doc_ai stage
CREATE OR REPLACE STAGE tb_doc_ai.raw_doc.doc_ai
    DIRECTORY = (ENABLE = TRUE)
    ENCRYPTION =  (TYPE = 'SNOWFLAKE_SSE');

-- create the inspection_reports stage
CREATE OR REPLACE STAGE tb_doc_ai.raw_doc.inspection_reports
    DIRECTORY = (ENABLE = TRUE)
    ENCRYPTION =  (TYPE = 'SNOWFLAKE_SSE');

-- create the doc_ai warehouse
CREATE OR REPLACE WAREHOUSE doc_ai
    WAREHOUSE_SIZE = 'small'
    WAREHOUSE_TYPE = 'standard'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    INITIALLY_SUSPENDED = TRUE
COMMENT = 'document ai warehouse';

-- create the tb_doc_ai role
CREATE OR REPLACE ROLE tb_doc_ai;

-- grant document ai privileges
GRANT DATABASE ROLE SNOWFLAKE.DOCUMENT_INTELLIGENCE_CREATOR TO ROLE tb_doc_ai;

-- grant doc_ai warehouse privileges
GRANT USAGE, OPERATE ON WAREHOUSE doc_ai TO ROLE tb_doc_ai;

-- grant tb_doc_ai database privileges
GRANT ALL ON DATABASE tb_doc_ai TO ROLE tb_doc_ai;
GRANT ALL ON SCHEMA tb_doc_ai.raw_doc TO ROLE tb_doc_ai;
GRANT CREATE STAGE ON SCHEMA tb_doc_ai.raw_doc TO ROLE tb_doc_ai;
GRANT CREATE SNOWFLAKE.ML.DOCUMENT_INTELLIGENCE ON SCHEMA tb_doc_ai.raw_doc TO ROLE tb_doc_ai;
GRANT ALL ON ALL STAGES IN SCHEMA tb_doc_ai.raw_doc TO ROLE tb_doc_ai;

-- set my_user_var variable to equal the logged-in user
SET my_user_var = (SELECT  '"' || CURRENT_USER() || '"' );

-- grant the logged in user the doc_ai_role
GRANT ROLE tb_doc_ai TO USER identifier($my_user_var);
