---
layout: post
title:  "Fazendo uma ALV CL SALV"
date:   2020-02-28 09:34:03 -0300
categories: alv
---
Hoje vou mostrar pra vocês como fazer uma linda `ALV` facil facil! Existem vários jeitos de se criar uma ALV, os mais conhecidos são ALV OO `cl_gui_alv_grid`, `cl_salv_table` e `alv reuse`.

Olha aqui como eu consigo fazer um [link maneiro][link-legal] sucesso das galaxias.

Olha como sua ALV vai ficar prontinha:

![Screenshot2]({{site.baseurl}}/assets/imagem-alv2.jpg)

<more>

[link-legal]: https://www.google.com/search?q=legal

{% highlight abap %}
*--------------------------------------------------------------------*
* Variáveis Globais
*--------------------------------------------------------------------*
DATA:
  og_salv_table           TYPE REF TO cl_salv_table.
 
FORM zf_exibe_dados  USING    p_t_relatorio TYPE y_t_relatorio.

  DATA:
    ol_salv_columns_table   TYPE REF TO cl_salv_columns_table,
    ol_salv_column_table    TYPE REF TO cl_salv_column_table,
    ol_functions            TYPE REF TO cl_salv_functions_list.

  IF p_t_relatorio IS INITIAL.
    MESSAGE e089(zlpp01).
*   Não existem dados para a seleção.
    RETURN.
  ENDIF.

*   Gerar a ALV
  TRY.
      CALL METHOD cl_salv_table=>factory
        IMPORTING
          r_salv_table = og_salv_table
        CHANGING
          t_table      = p_t_relatorio.
    CATCH cx_salv_msg.
      RETURN.
  ENDTRY.

* Definir funções da toolbar
  ol_functions = og_salv_table->get_functions( ).
  ol_functions->set_all( abap_true ).

*   Manter dados das colunas da ALV
  ol_salv_columns_table = og_salv_table->get_columns( ).
  ol_salv_columns_table->set_optimize( abap_true ).

  TRY .

      ol_salv_column_table ?= ol_salv_columns_table->get_column( 'LTXA1_SUB' ).
      ol_salv_column_table->set_long_text( value = 'Texto Oper. Sub'(t02) ).
      ol_salv_column_table->set_medium_text( value = 'Texto Oper. Sub'(t02) ).
      ol_salv_column_table->set_short_text( value = 'Txt Op Sub'(t03) ).


      ol_salv_column_table ?= ol_salv_columns_table->get_column( 'TIPO' ).
      ol_salv_column_table->set_long_text( value = 'Tipo'(t06) ).
      ol_salv_column_table->set_medium_text( value = 'Tipo'(t06) ).
      ol_salv_column_table->set_short_text( value = 'Tipo'(t06) ).

    CATCH cx_salv_not_found.  .
  ENDTRY.

*   Exibir a ALV
  og_salv_table->display( ).

ENDFORM.                    " ZF_EXIBE_DADOS


{% endhighlight %}
