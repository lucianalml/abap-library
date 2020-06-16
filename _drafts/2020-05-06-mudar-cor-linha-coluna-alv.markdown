

TYPES:
      BEGIN OF ty_resultado,
        racct            TYPE faglflexa-racct,
        empreendimento   TYPE string,
        total_faturado   TYPE bseg-dmbtr,
        total_arrecadado TYPE bseg-dmbtr,
        total_transitado TYPE bseg-dmbtr,
        resultado        TYPE bseg-dmbtr,
        media_meses      TYPE bseg-dmbtr,
        percentual       TYPE p LENGTH 3 DECIMALS 2,
        meses            TYPE i,
        color            TYPE lvc_t_scol,
      END OF ty_resultado.


    DATA:
      lo_grid             TYPE REF TO cl_salv_table,
      lo_columns          TYPE REF TO cl_salv_columns_table,
      lo_column           TYPE REF TO cl_salv_column,


    lo_columns = lo_grid->get_columns( ).
    lo_columns->set_optimize( abap_true ).


    TRY.
        CALL METHOD lo_columns->set_color_column
          EXPORTING
            value = 'COLOR'.
      CATCH cx_salv_data_error.
    ENDTRY.




  METHOD: set_color.

    DATA:
          ls_color LIKE LINE OF resultado-color.

    IF resultado-resultado < 0.
      ls_color-fname     = 'RESULTADO'.
      ls_color-color-col = 6.
      ls_color-color-int = 0.
      ls_color-color-inv = 0.
      APPEND ls_color TO resultado-color.
    ENDIF.

    IF resultado-total_arrecadado < ( resultado-media_meses * ( resultado-percentual / 100 ) ) .

      ls_color-fname     = 'TOTAL_ARRECADADO'.
      ls_color-color-col = 7.
      ls_color-color-int = 0.
      ls_color-color-inv = 0.
      APPEND ls_color TO resultado-color.
    ENDIF.

  ENDMETHOD.

