---
layout: post
title:  "Enviando email no ABAP"
date:   2020-02-25 09:34:03 -0300
categories: email
---
Hoje vou mostrar pra vocês como enviar email via código.

{% highlight abap %}
REPORT  zteste.

TYPES: BEGIN OF tp_anexo,
  file_type TYPE char03,
  file_name TYPE so_obj_des,
  attachx   TYPE solix_tab,
  END OF tp_anexo.

DATA gt_attach          TYPE STANDARD TABLE OF tp_anexo.

SELECTION-SCREEN BEGIN OF BLOCK 1 WITH FRAME TITLE text-001.
PARAMETERS:
  p_email  TYPE ad_smtpadr OBLIGATORY,
  p_anexo  TYPE rlgrap-filename.
SELECTION-SCREEN END OF BLOCK 1.


AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_anexo.
  PERFORM select_file USING p_anexo.

START-OF-SELECTION.
  PERFORM process_file USING p_anexo.
  PERFORM envia_email.

*&---------------------------------------------------------------------*
*&      Form  ENVIA_EMAIL
*&---------------------------------------------------------------------*
FORM envia_email .
  DATA: cl_message      TYPE REF TO cl_bcs,
        lo_recipient    TYPE REF TO if_recipient_bcs,
        lo_document     TYPE REF TO cl_document_bcs,
        lo_sender       TYPE REF TO if_sender_bcs,
        lt_text         TYPE bcsy_text,
        lv_req_status   TYPE bcs_rqst VALUE 'N',
        lv_status_mail  TYPE bcs_stml VALUE 'N',
        ls_attach       LIKE LINE OF gt_attach.

  FIELD-SYMBOLS:
      <lfs_line> LIKE LINE OF lt_text.

  TRY.
* Cria mensagem
      cl_message = cl_bcs=>create_persistent( ).

* Adiciona o remetente
      lo_sender = cl_sapuser_bcs=>create( sy-uname ).
      cl_message->set_sender( lo_sender ).

* Adiciona destinatarios

      CALL METHOD cl_cam_address_bcs=>create_internet_address
        EXPORTING
          i_address_string = p_email
        RECEIVING
          result           = lo_recipient.

      CALL METHOD cl_message->add_recipient
        EXPORTING
          i_recipient = lo_recipient.

* Adiciona o corpo do email

      APPEND INITIAL LINE TO lt_text ASSIGNING <lfs_line>.
      <lfs_line>-line = 'Corpo do e-mail'.

*      lo_document = cl_document_bcs=>create_from_text(
*        i_text = lt_text
*        i_subject = 'Assunto' ).

* Para enviar com anexo precisa setar i_type
      lo_document = cl_document_bcs=>create_document(
                          i_type    = 'RAW'
                          i_text    = lt_text
                          i_subject = 'Assunto' ).

* Adiciona anexos
      IF p_anexo IS NOT INITIAL.
        LOOP AT gt_attach INTO ls_attach.
          TRY.
              lo_document->add_attachment(
              EXPORTING
              i_attachment_type = ls_attach-file_type
              i_attachment_subject = ls_attach-file_name
              i_att_content_hex = ls_attach-attachx ).
            CATCH cx_document_bcs.
          ENDTRY.
        ENDLOOP.

      ENDIF.

      cl_message->set_document( lo_document ).

* Envia a mensagem
      CALL METHOD cl_message->set_status_attributes
        EXPORTING
          i_requested_status = lv_req_status
          i_status_mail      = lv_status_mail.

      cl_message->set_send_immediately( 'X' ).
      CALL METHOD cl_message->send( ).
      COMMIT WORK.

    CATCH cx_bcs.
      MESSAGE i368(00) WITH 'Erro ao enviar email'.
  ENDTRY.
  IF sy-subrc = 0.
    WRITE 'E-mail enviado com sucesso!'.
  ENDIF.
ENDFORM.                    " ENVIA_EMAIL

*&---------------------------------------------------------------------*
*&      Form  SELECT_FILE
*&---------------------------------------------------------------------*
FORM select_file  USING   p_anexo.
  DATA :
    lv_subrc  LIKE sy-subrc,
    lt_it_tab TYPE filetable.

  "Mostrando caixa de dialogo para seleção do arquivo
  CALL METHOD cl_gui_frontend_services=>file_open_dialog
    EXPORTING
      window_title     = 'Select File'
*      default_filename = '*'
      multiselection   = ' '
    CHANGING
      file_table       = lt_it_tab
      rc               = lv_subrc.

  " Escrevendo o arquivo na area
  LOOP AT lt_it_tab INTO p_anexo.
  ENDLOOP.
ENDFORM.                    " SELECT_FILE

*&---------------------------------------------------------------------*
*&      Form  process_file
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->FILENAME   text
*----------------------------------------------------------------------*
FORM process_file USING filename.

  DATA lv_filename        TYPE string.
  DATA lt_data            TYPE solix_tab.
  DATA lv_filelength      TYPE i.
  DATA lv_str_filename    TYPE string.
  DATA lv_junk            TYPE string.
  DATA lv_filetype        TYPE string.
  DATA ls_attach          LIKE LINE OF gt_attach.

  MOVE filename TO lv_filename.
* Upload file to read data as binary
  CALL METHOD cl_gui_frontend_services=>gui_upload
    EXPORTING
      filename                = lv_filename
      filetype                = 'BIN' "Binary Mode
    IMPORTING
      filelength              = lv_filelength
    CHANGING
      data_tab                = lt_data
    EXCEPTIONS
      file_open_error         = 1
      file_read_error         = 2
      no_batch                = 3
      gui_refuse_filetransfer = 4
      invalid_type            = 5
      no_authority            = 6
      unknown_error           = 7
      bad_data_format         = 8
      header_not_allowed      = 9
      separator_not_allowed   = 10
      header_too_long         = 11
      unknown_dp_error        = 12
      access_denied           = 13
      dp_out_of_memory        = 14
      disk_full               = 15
      dp_timeout              = 16
      not_supported_by_gui    = 17
      error_no_gui            = 18
      OTHERS                  = 19.
  IF lt_data[] IS NOT INITIAL.
* Get Stripped file name
    CALL FUNCTION 'SO_SPLIT_FILE_AND_PATH'
      EXPORTING
        full_name     = lv_filename
      IMPORTING
        stripped_name = lv_str_filename
      EXCEPTIONS
        x_error       = 1
        OTHERS        = 2.
    SPLIT lv_str_filename AT '.' INTO lv_junk lv_filetype.
    ls_attach-file_type = lv_filetype.
    ls_attach-file_name = lv_str_filename.
    ls_attach-attachx[] = lt_data[].
    APPEND ls_attach TO gt_attach.
    CLEAR ls_attach.

  ENDIF.
ENDFORM.                    "process_file
{% endhighlight %}
