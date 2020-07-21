    DATA(lv_dt_vencimento) = get_dt_vencimento( ).

    CALL FUNCTION 'RE_ADD_MONTH_TO_DATE'
      EXPORTING
        months  = 1
        olddate = lv_dt_vencimento
      IMPORTING
        newdate = lv_dt_vencimento.

    CALL FUNCTION 'RP_LAST_DAY_OF_MONTHS'
      EXPORTING
        day_in            = lv_dt_vencimento
      IMPORTING
        last_day_of_month = rv_dt_vencimento_max.
