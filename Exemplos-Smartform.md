# Exemplos de SmartForms

## Exemplo 1 -  Relatório executando um Smartform

```` abap
*&---------------------------------------------------------------------*
*& Report YDO9_REPORT_003
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ydo9_report_003.

DATA:
  v_name               TYPE char50,
  v_birth              TYPE char10,
  v_salary             TYPE char50,
  v_form_function      TYPE rs38l_fnam,
  s_output_options     TYPE ssfcompop,
  s_control_parameters TYPE ssfctrlop.

TABLES:
  pa0002, vbap.

SELECTION-SCREEN BEGIN OF BLOCK a1 WITH FRAME TITLE TEXT-001.
  PARAMETERS:
    p_name   TYPE pa0002-cname OBLIGATORY,
    p_birth  TYPE sy-datum OBLIGATORY,
    p_salary TYPE vbap-netwr OBLIGATORY,
    p_prev   AS CHECKBOX DEFAULT 'X'.
SELECTION-SCREEN END OF BLOCK a1.

START-OF-SELECTION.

  v_name = p_name.

  WRITE p_birth TO v_birth.

  WRITE p_salary TO v_salary.
  CONDENSE v_salary NO-GAPS.
  v_salary = |R$ { v_salary }|.

  " Primeiro, usamos a função SSF_FUNCTION_MODULE_NAME para obter
  " o nome da função referente ao nosso formulário
  CALL FUNCTION 'SSF_FUNCTION_MODULE_NAME'
    EXPORTING
      formname           = 'YDO9_FORM_001'
    IMPORTING
      fm_name            = v_form_function
    EXCEPTIONS
      no_form            = 1
      no_function_module = 2
      OTHERS             = 3.

  IF sy-subrc <> 0.
    " Seu formuário não existe. Abortar.
    STOP.
  ENDIF.

  s_output_options-tdnewid = abap_true.
  s_output_options-tddest  = 'LP01'.
  s_control_parameters-preview = abap_false.

  IF p_prev IS INITIAL.
    s_control_parameters-no_dialog = abap_true.
  ELSE.
    s_control_parameters-no_dialog = abap_false.
  ENDIF.

  " Por fim, chamamos a função encontrada acima passando os
  " parâmetros que achamos necessários
  CALL FUNCTION v_form_function
    EXPORTING
      control_parameters = s_control_parameters
      output_options     = s_output_options
      user_settings      = space
      i_name             = v_name
      i_birth_date       = v_birth
      i_salary           = v_salary
    EXCEPTIONS
      formatting_error   = 1
      internal_error     = 2
      send_error         = 3
      user_canceled      = 4
      OTHERS             = 5.

  IF sy-subrc EQ 0.
    MESSAGE 'Impressão realizada com sucesso' TYPE 'S'.
  ELSE.
    MESSAGE 'Erro ao realizar impressão' TYPE 'S'.
  ENDIF.
````
