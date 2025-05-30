# PL/SQL Procedure to Insert Lookups from DFF Contexts

This PL/SQL block iterates through specific Descriptive Flexfield (DFF) contexts from `FND_DESCR_FLEX_CONTEXTS_VL` and inserts them as lookup values into the `FND_LOOKUP_VALUES_PKG` table. It specifically targets DFF contexts whose codes start with 'XXGTN%' and are associated with 'Extra Person Info DDF' and are enabled.

```sql
DECLARE
    xrow rowid;
    j number;
    CURSOR C IS SELECT DESCRIPTIVE_FLEX_CONTEXT_CODE,
  DESCRIPTIVE_FLEX_CONTEXT_NAME
FROM FND_DESCR_FLEX_CONTEXTS_VL
WHERE (DESCRIPTIVE_FLEX_CONTEXT_CODE LIKE 'XXGTN%')
AND (DESCRIPTIVE_FLEXFIELD_NAME='Extra Person Info DDF')
and (ENABLED_FLAG ='Y')
ORDER BY DECODE(GLOBAL_FLAG, 'Y', 1, 2),
  descriptive_flex_context_code;
BEGIN
    FOR i IN c
    LOOP
    DBMS_OUTPUT.PUT_LINE(I.DESCRIPTIVE_FLEX_CONTEXT_CODE||'      '||I.DESCRIPTIVE_FLEX_CONTEXT_NAME);
-- Call the procedure
    FND_LOOKUP_VALUES_PKG.INSERT_ROW(X_ROWID               => XROW,
    x_lookup_type         => 'XXGTN_EIT_SEGMENTS_LKP',
    x_security_group_id   => 0,
    x_view_application_id => 3,
    x_lookup_code         => i.DESCRIPTIVE_FLEX_CONTEXT_CODE,
    x_tag                 => null,
    x_attribute_category  => null,
    x_attribute1          => null,
    x_attribute2          => null,
    x_attribute3          => null,
    X_ATTRIBUTE4          => NULL,
    x_enabled_flag        => 'Y',
    x_start_date_active   => null,
    x_end_date_active     => null,
    x_territory_code      => null,
    x_attribute5          => null,
    x_attribute6          => null,
    x_attribute7          => null,
    x_attribute8          => null,
    x_attribute9          => null,
    x_attribute10         => null,
    x_attribute11         => null,
    x_attribute12         => null,
    x_attribute13         => null,
    x_attribute14         => null,
    x_attribute15         => null,
    X_MEANING             => i.DESCRIPTIVE_FLEX_CONTEXT_NAME,
    x_description         => null,
    X_CREATION_DATE       => SYSDATE,
    x_created_by          => 1110,
    x_last_update_date    => sysdate,
    x_last_updated_by     => 1110,
    x_last_update_login   => 1110);
END loop;
    COMMIT;
    END;