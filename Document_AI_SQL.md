USE ROLE tb_doc_ai;
USE WAREHOUSE doc_ai;
USE DATABASE tb_doc_ai;
USE SCHEMA raw_doc;

LIST @inspection_reports;

SELECT inspection_report_extraction!PREDICT(GET_PRESIGNED_URL(@inspection_reports, '02.13.2022.5.pdf'));

CREATE OR REPLACE TABLE ir_raw
COMMENT = '{"origin":"sf_sit-is", "name":"voc", "version":{"major":1, "minor":0}, "attributes":{"is_quickstart":1, "source":"sql", "vignette":"docai"}}'
AS
SELECT inspection_report_extraction!PREDICT(GET_PRESIGNED_URL(@inspection_reports, RELATIVE_PATH)) AS ir_object
FROM DIRECTORY(@inspection_reports);

SELECT * FROM ir_raw;

SELECT
    TO_DATE(REPLACE(ir_object:"DATE"[0].value::varchar,'-','/')) AS date,
    ir_object:"TRUCK_ID"[0].value::varchar AS truck_id, 
    CASE
        WHEN ir_object:"PIC_PRESENT"[0].value::varchar = 'Y' THEN 'Pass' -- convert Y to Pass
        WHEN ir_object:"PIC_PRESENT"[0].value::varchar = 'N' THEN 'Fail' -- convert N to Fail
        WHEN ir_object:"PIC_PRESENT"[0].value::varchar = 'X' THEN 'Not Observed' -- convert X to Not Observed
        ELSE 'Not Observed'
    END AS person_in_charge_present,
    CASE
        WHEN ir_object:"FOOD_PROPER_TEMP"[0].value::varchar = 'Y' THEN 'Pass' -- convert Y to Pass
        WHEN ir_object:"FOOD_PROPER_TEMP"[0].value::varchar = 'N' THEN 'Fail' -- convert N to Fail
        WHEN ir_object:"FOOD_PROPER_TEMP"[0].value::varchar = 'X' THEN 'Not Observed' -- convert X to Not Observed
        ELSE 'Not Observed'
    END AS food_proper_temp
FROM ir_raw
ORDER BY truck_id;
