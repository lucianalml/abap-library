



    DATA:
      lr_tabdescr TYPE REF TO cl_abap_structdescr,
      lr_data     TYPE REF TO data,
      lt_dfies    TYPE ddfields,
      ls_dfies    TYPE dfies,
      lv_header   TYPE string.


    CREATE DATA lr_data LIKE LINE OF gt_data.

    lr_tabdescr ?= cl_abap_structdescr=>describe_by_data_ref( lr_data ).

    lt_dfies = cl_salv_data_descr=>read_structdescr( lr_tabdescr ).

* Monta header com descricos das colunas
    LOOP AT lt_dfies INTO ls_dfies.

      IF ls_dfies-fieldname = 'DT_FALEC'.
        ls_dfies-fieldtext = 'Data de falecimento'.
      ENDIF.

      lv_header = lv_header && '|' && ls_dfies-fieldtext.
    ENDLOOP.

    SHIFT lv_header LEFT DELETING LEADING '|' .

    INSERT lv_header INTO ct_string_tab INDEX 1.