---
layout: post
title:  "SM30 Events"
date:   2020-02-20 09:34:03 -0300
categories: sm30
---

Olhe e aprenda como fazer um batch input.


{% highlight abap %}
*&---------------------------------------------------------------------*
*&  Include           ZEHSC007_TOP
*&---------------------------------------------------------------------*


*-----------------------------------------------------------------------
* Variaveis globais
*-----------------------------------------------------------------------
DATA:
 gt_bdcdata     TYPE TABLE OF bdcdata,
 gt_messtab     TYPE TABLE OF bdcmsgcoll.
 
 
 
 
 
*&---------------------------------------------------------------------*
*&  Include           ZEHSC007_F01
*&---------------------------------------------------------------------*

*&---------------------------------------------------------------------*
*&      Form  SHDB_CRIAR_MEDICO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM shdb_criar_medico  USING  is_medico TYPE lcl_medico=>typ_medico.

 DATA:
   ls_bdcdata TYPE bdcdata,
   ls_messtab TYPE bdcmsgcoll.

 PERFORM f_shdb_clear.

*  PERFORM f_bdc_dynpro USING  'SAPLCBIH_WA00'           '1010'.
*  PERFORM f_bdc_field  USING: 'BDC_CURSOR'              'CCIHS_WASEL-WAID' ,
*                              'BDC_OKCODE'              '=GRSL',
*                              'CCIHS_WASEL-WAID'        it_wa_ghe-waid,
*                              'CCIHS_WASEL-WATYPE'      '  ',
*                              'CCIHS_WASEL-VALDAT'      vg_data.


ENDFORM.

*&---------------------------------------------------------------------*
*&      Form  F_SHDB_CLEAR
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_shdb_clear.
 CLEAR:
   gt_bdcdata,
   gt_messtab.
ENDFORM.                    " F_SHDB_CLEAR


*&---------------------------------------------------------------------*
*&      Form  F_BDC_DYNPRO
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_bdc_dynpro USING program dynpro.

 DATA:
   ls_bdcdata     TYPE bdcdata.

 ls_bdcdata-program  = program.
 ls_bdcdata-dynpro   = dynpro.
 ls_bdcdata-dynbegin = 'X'.
 APPEND ls_bdcdata TO gt_bdcdata.

ENDFORM.                    "f_bdc_dynpro

*&---------------------------------------------------------------------*
*&      Form  F_BDC_FIELD
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_bdc_field USING fnam fval.

 DATA:
   ls_bdcdata     TYPE bdcdata.

 ls_bdcdata-fnam = fnam.
 ls_bdcdata-fval = fval.
 APPEND ls_bdcdata TO gt_bdcdata.

ENDFORM.                    "f_bdc_field

*&---------------------------------------------------------------------*
*&      Form  F_BDC_TRANSACTION
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM f_bdc_transaction USING tcode .

 DATA:
   ls_opt LIKE ctu_params.

 CLEAR gt_messtab.

 ls_opt-dismode  = 'N'.
*  it_opt-dismode  = 'A'.  " exibir telas
 ls_opt-updmode  = 'S'.
 ls_opt-defsize  = 'X'.
 ls_opt-racommit  = 'X'.

 CALL TRANSACTION tcode
   USING gt_bdcdata
   OPTIONS FROM ls_opt
   MESSAGES INTO gt_messtab.

*  COMMIT WORK.

ENDFORM.                    "f_bdc_transaction

{% endhighlight %}
