## Função YDO9_CHECK_USER

```abap
FUNCTION ydo9_check_user.
*"----------------------------------------------------------------------
*"*"Local Interface:
*"  IMPORTING
*"     VALUE(I_USER) TYPE  SYST_UNAME
*"  EXPORTING
*"     VALUE(E_USER_OK) TYPE  BOOLEAN
*"  EXCEPTIONS
*"      ERROR_INVALID_USER
*"----------------------------------------------------------------------

  " Verificar se usuário existe
  SELECT SINGLE COUNT(*)
    FROM usr01
    WHERE bname EQ I_user.

  IF sy-subrc NE 0.
    " Dispara exceção caso o usuário não exista.
    RAISE error_invalid_user.
  ENDIF.

  PERFORM: f_check_user USING i_user
                        CHANGING e_user_ok.

ENDFUNCTION.
```

## Include de rotinas
```abap
*----------------------------------------------------------------------*
***INCLUDE LYDO9_FUNCTIONSF01.
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*& Form f_check_user
*&---------------------------------------------------------------------*
*& text
*&---------------------------------------------------------------------*
*&      --> I_USER
*&      <-- E_USER_OK
*&---------------------------------------------------------------------*
FORM f_check_user  USING    VALUE(p_user) TYPE sy-uname
                   CHANGING p_user_ok TYPE boolean.

  IF p_user NE 'THIAGO'.
    p_user_ok = abap_true.
  ELSE.
    p_user_ok = abap_false.
  ENDIF.

ENDFORM.
```

## Programa para chamar a sua função
```abap
*&---------------------------------------------------------------------*
*& Report YDO9_REPORT_002
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ydo9_report_002.

DATA:
  v_user_ok TYPE boolean.

PARAMETERS:
  p_user TYPE sy-uname OBLIGATORY.

START-OF-SELECTION.

  CALL FUNCTION 'YDO9_CHECK_USER'
    EXPORTING
      i_user             = p_user
    IMPORTING
      e_user_ok          = v_user_ok
    EXCEPTIONS
      error_invalid_user = 1
      OTHERS             = 2.

  IF sy-subrc NE 0.
    MESSAGE 'Usuário não existe' TYPE 'S' DISPLAY LIKE 'E'.
    STOP.
  ENDIF.

  IF v_user_ok IS INITIAL.
    MESSAGE 'Usuário não foi validado com sucesso' TYPE 'S' DISPLAY LIKE 'W'.
  ELSE.
    MESSAGE 'Usuário validado com sucesso' TYPE 'S'.
  ENDIF.
```
