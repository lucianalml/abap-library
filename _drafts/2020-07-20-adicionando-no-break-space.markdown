
DATA:
  lv_string   TYPE string,
  pt_conteudo TYPE TABLE OF solisti1.

lv_string = '1111 22222333  44 4555'.

CALL FUNCTION 'CONVERT_STRING_TO_TAB'
  EXPORTING
    i_string         = lv_string
    i_tabline_length = '5'
  TABLES
    et_table         = pt_conteudo[].

DATA: lv_linha_tam5 TYPE c LENGTH 5,
      lv_final      TYPE string.

LOOP AT pt_conteudo INTO DATA(ls_conteudo).
  lv_linha_tam5 = ls_conteudo.

* The NO-BREAK space character is different – you can type it by pressing ALT+255 –
* this character will look the same like normal space but in unicode it will have number U+00A0.

  IF lv_linha_tam5+4(1) = space.
    lv_linha_tam5+4(1) = cl_abap_conv_in_ce=>uccp( '00A0' ).  " NO BREAK space
  ENDIF.

  CONCATENATE lv_final lv_linha_tam5 INTO lv_final.

ENDLOOP.

WRITE lv_final.