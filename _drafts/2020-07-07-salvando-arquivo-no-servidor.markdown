

*----------------------------------------------------------------------*
*     Data Decalaration
*----------------------------------------------------------------------*
DATA: gt_spfli  TYPE TABLE OF spfli,
      gwa_spfli TYPE spfli.
DATA: gv_file   TYPE rlgrap-filename.

*----------------------------------------------------------------------*
*     START-OF-SELECTION
*----------------------------------------------------------------------*
PERFORM get_data.
IF NOT gt_spfli[] IS INITIAL.
  PERFORM save_file.
ELSE.
  MESSAGE 'No data found' TYPE 'I'.
ENDIF.
*&---------------------------------------------------------------------*
*&      Form  get_data
*&---------------------------------------------------------------------*
FORM get_data.
*Get data from table SPFLI
  SELECT * FROM spfli
         INTO TABLE gt_spfli.
ENDFORM.                    " get_data
*&---------------------------------------------------------------------*
*&      Form  save_file
*&---------------------------------------------------------------------*
FORM save_file.
  DATA: lv_data TYPE string.

*Move complete path to filename
  gv_file = 'spfli.txt'.

* Open the file in output mode
  OPEN DATASET gv_file FOR OUTPUT IN TEXT MODE ENCODING DEFAULT.
  IF sy-subrc NE 0.
    MESSAGE 'Unable to create file' TYPE 'I'.
    EXIT.
  ENDIF.

  LOOP AT gt_spfli INTO gwa_spfli.
    CONCATENATE gwa_spfli-carrid
                gwa_spfli-connid
                gwa_spfli-countryfr
                gwa_spfli-cityfrom
                gwa_spfli-airpfrom
                gwa_spfli-countryto
                gwa_spfli-cityto
                gwa_spfli-airpto
                gwa_spfli-arrtime
     INTO lv_data
     SEPARATED BY ','.
*TRANSFER moves the above fields from workarea to file  with comma
*delimited format
    TRANSFER lv_data TO gv_file.
    CLEAR: gwa_spfli.
  ENDLOOP.
* close the file
  CLOSE DATASET gv_file.
