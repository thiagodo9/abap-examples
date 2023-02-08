# Exemplos de relatórios

## Exemplo 1 -  Relatório ALV básico usando funções

```abap
*&---------------------------------------------------------------------*
*& Report YDO9_REPORT_001
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT ydo9_report_001.

**********************************************************************
" Tabelas usadas na tela de seleção
**********************************************************************
TABLES:
  ydo9_alunos.

**********************************************************************
" Declaração de Constantes globais
**********************************************************************
CONSTANTS:
  c_error   TYPE sy-msgty VALUE 'E',
  c_success TYPE sy-msgty VALUE 'S'.

**********************************************************************
" Declaração de variáveis globais
**********************************************************************
DATA:
  t_students TYPE TABLE OF ydo9_alunos.

**********************************************************************
" Tela de Seleção
**********************************************************************
SELECTION-SCREEN BEGIN OF BLOCK a1 WITH FRAME TITLE TEXT-t01.

  PARAMETERS:
    p_id TYPE ydo9_alunos-id_student.

  SELECT-OPTIONS:
    s_name  FOR ydo9_alunos-full_name NO INTERVALS,
    s_birth FOR ydo9_alunos-birth_date NO-EXTENSION.

SELECTION-SCREEN END OF BLOCK a1.

**********************************************************************
" Evento Inicialização
**********************************************************************
* Este evento é executado no momento em que o programa é executado pela primeira vez
INITIALIZATION.

  PERFORM:
    f_initialization.

**********************************************************************
  " Evento Start-Of-Selection
**********************************************************************
* Este evento é o principal de um relatório. Toda a sua lógica princiapal deve
* ser feita aqui
START-OF-SELECTION.

  PERFORM:
    f_select_data,
    f_display_data.


**********************************************************************
  " Rotinas para Encapsulamento de Código
**********************************************************************

*&---------------------------------------------------------------------*
*& Form f_initialization
*&---------------------------------------------------------------------*
FORM f_initialization .

  " A título de exemplo, se o relatório for executado por outra pessoa que
  " não seja o meu usuário ele irá exibir uma mensagem de erro e
  " interromper o processamento.

  IF sy-uname NE 'THIAGO'.

    " Exemplo de exibição de mensagem de erro.
    MESSAGE TEXT-e01 TYPE c_success DISPLAY LIKE c_error.
    LEAVE PROGRAM.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form f_select_data
*&---------------------------------------------------------------------*
FORM f_select_data .

  " Exemplo de uma condição SE e um SE NÃO.
  IF p_id IS NOT INITIAL.

    " Exemplo de uma seleção no ABAP. Existem muitas variações possíveis.
    SELECT *
    INTO TABLE t_students
    FROM ydo9_alunos
    WHERE id_student EQ p_id
      AND full_name IN s_name
      AND birth_date IN s_birth.

  ELSE.

    SELECT *
    INTO TABLE t_students
    FROM ydo9_alunos
    WHERE full_name IN s_name
      AND birth_date IN s_birth.

  ENDIF.

  IF sy-subrc NE 0.
    MESSAGE TEXT-e02 TYPE c_success DISPLAY LIKE c_error.
    STOP.
  ENDIF.

ENDFORM.
*&---------------------------------------------------------------------*
*& Form f_display_data
*&---------------------------------------------------------------------*
FORM f_display_data .

  " Podemos declarar variáveis dentro das nossas rotinas. Elas são consideradas
  " locais no escopo.
  DATA:
    lt_fieldcat TYPE slis_t_fieldcat_alv.

  " Exemplo de uma chamada de função.
  CALL FUNCTION 'REUSE_ALV_FIELDCATALOG_MERGE'
    EXPORTING
      i_structure_name       = 'YDO9_ALUNOS'
    CHANGING
      ct_fieldcat            = lt_fieldcat
    EXCEPTIONS
      inconsistent_interface = 1
      program_error          = 2
      OTHERS                 = 3.

  IF sy-subrc <> 0.
    MESSAGE TEXT-e03 TYPE c_success DISPLAY LIKE c_error.
    STOP.
  ENDIF.

  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
    EXPORTING
      it_fieldcat   = lt_fieldcat
    TABLES
      t_outtab      = t_students
    EXCEPTIONS
      program_error = 1
      OTHERS        = 2.

  IF sy-subrc <> 0.
    MESSAGE TEXT-e04 TYPE c_success DISPLAY LIKE c_error.
    STOP.
  ENDIF.

ENDFORM.
```
