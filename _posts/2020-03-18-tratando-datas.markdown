---
layout: post
title:  "Tratando datas"
date:   2020-03-18 09:34:03 -0300
categories: [abap]
tags: [abap, datatypes]
---

Existem algumas funções para tratamento de datas. A função abaixo verifica se um periodo se sobrepoe a outro período.

<more>

{% highlight abap %}

    CALL FUNCTION 'TTE_CHK_DTRNG_DATERANGE'
      EXPORTING
        datefrom           = lv_dezembro_ini
        dateto             = lv_dezembro_fim
        checkdatefrom      = ls_pa2001-begda
        checkdateto        = ls_pa2001-endda
      EXCEPTIONS
        date_range_overlap = 1
        OTHERS             = 2.

    IF sy-subrc <> 0.
* Possui licenca em dezembro
      cv_afastamento = abap_true.
      RETURN.
    ENDIF.

{% endhighlight %}
