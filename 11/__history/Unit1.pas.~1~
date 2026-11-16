unit Unit1;

interface

uses
  Winapi.Windows, Winapi.Messages, System.SysUtils, System.Variants,
  System.Classes, Vcl.Graphics, Vcl.Controls, Vcl.Forms, Vcl.Dialogs,
  Vcl.StdCtrls, Vcl.ExtCtrls, Math,
  VclTee.Chart, VclTee.Series, VclTee.TeeProcs, VCLTee.TeEngine;

type
  TForm1 = class(TForm)
    // Поля ввода
    Label1: TLabel;
    Label2: TLabel;
    Label3: TLabel;
    Label4: TLabel;
    EditR: TEdit;
    EditRho: TEdit;
    EditHmax: TEdit;
    EditN: TEdit;
    // Кнопки
    ButtonCalc: TButton;
    ButtonClear: TButton;
    // Параметры
    LabelA: TLabel;
    LabelVt: TLabel;
    LabelTau: TLabel;
    // График и лог
    Chart1: TChart;
    MemoLog: TMemo;

    procedure FormCreate(Sender: TObject);
    procedure ButtonCalcClick(Sender: TObject);
    procedure ButtonClearClick(Sender: TObject);
  private
    FTimes: array of Double;
    FHeights: array of Double;
    FCount: Integer;
    Fa, FvTerm: Double;
    LineSeries1: TLineSeries;  // создаётся программно, не через дизайнер
  public
  end;

var
  Form1: TForm1;

implementation

{$R *.dfm}

const
  g_const = 9.81;
  rho0    = 1000.0;
  Cd      = 0.47;
  eps     = 1e-9;

{------------------------------------------------------------------}
{ f(t) = (v_t^2/a)*ln(ch(a*t/v_t)) - h                             }
{------------------------------------------------------------------}
function f_func(t, h, a, vt: Double): Double;
var
  arg: Double;
begin
  arg := a * t / vt;
  if arg > 20 then
    Result := vt * t - (vt * vt / a) * Ln(2) - h
  else
    Result := (vt * vt / a) * Ln(Cosh(arg)) - h;
end;

{------------------------------------------------------------------}
{ f'(t) = v_t * th(a*t/v_t)                                        }
{------------------------------------------------------------------}
function df_func(t, a, vt: Double): Double;
var
  arg: Double;
begin
  arg := a * t / vt;
  if arg > 20 then
    Result := vt
  else
    Result := vt * Tanh(arg);
end;

{------------------------------------------------------------------}
{ Метод Ньютона                                                    }
{------------------------------------------------------------------}
function SolveTime(h, a, vt: Double; var iterCount: Integer): Double;
var
  t, ft, dft: Double;
  iter: Integer;
begin
  if h <= 0 then
  begin
    Result := 0;
    iterCount := 0;
    Exit;
  end;

  t := h / vt;
  iter := 0;

  repeat
    Inc(iter);
    ft  := f_func(t, h, a, vt);
    dft := df_func(t, a, vt);
    if Abs(dft) < 1e-15 then Break;
    t := t - ft / dft;
  until (Abs(ft) < eps) or (iter > 100);

  iterCount := iter;
  Result := t;
end;

{------------------------------------------------------------------}
{ Создание формы                                                   }
{------------------------------------------------------------------}
procedure TForm1.FormCreate(Sender: TObject);
begin

  // Значения по умолчанию
  {EditR.Text    := '0.01';
  EditRho.Text  := '7800';
  EditHmax.Text := '10';
  EditN.Text    := '10';   }

  LabelA.Caption   := 'a   = ---';
  LabelVt.Caption  := 'v(t) = ---';
  LabelTau.Caption := 't = ---';

  // Настройка графика
  Chart1.Title.Text.Text := 'Зависимость времени падения t от глубины h';
  Chart1.Title.Font.Size := 12;
  Chart1.Title.Font.Style := [fsBold];
  Chart1.LeftAxis.Title.Caption := 't, с';
  Chart1.BottomAxis.Title.Caption := 'h, м';
  Chart1.Legend.Visible := False;
  Chart1.View3D := False;

  // Создаём серию ПРОГРАММНО (без дизайнера)
  LineSeries1 := TLineSeries.Create(Self);
  LineSeries1.ParentChart := Chart1;
  LineSeries1.Title := 't(h)';
  LineSeries1.SeriesColor := clRed;
  LineSeries1.LinePen.Width := 2;
  LineSeries1.Pointer.Visible := True;
  LineSeries1.Pointer.Style := psCircle;
  LineSeries1.Pointer.Size := 4;
  LineSeries1.Pointer.Brush.Color := clRed;

  FCount := 0;
end;



{------------------------------------------------------------------}
{ Кнопка "Вычислить"                                               }
{------------------------------------------------------------------}
procedure TForm1.ButtonCalcClick(Sender: TObject);
var
  r, rho, h_max: Double;
  N, i: Integer;
  h, t: Double;
  iterCount: Integer;
begin

    r     := StrToFloat(EditR.Text);
    rho   := StrToFloat(EditRho.Text);
    h_max := StrToFloat(EditHmax.Text);
    N     := StrToInt(EditN.Text);


  if r <= 0 then begin ShowMessage('Радиус должен быть > 0!'); Exit; end;
  if rho <= rho0 then begin ShowMessage('Плотность шарика должна быть > плотности воды!'); Exit; end;
  if h_max <= 0 then begin ShowMessage('Глубина должна быть > 0!'); Exit; end;
  if N <= 0 then begin ShowMessage('Число точек должно быть > 0!'); Exit; end;

  // Параметры задачи
  Fa     := (rho - rho0) * g_const / rho;
  FvTerm := Sqrt(8 * r * (rho - rho0) * g_const / (3 * Cd * rho0));

  LabelA.Caption   := Format('a   = %.4f м/с2', [Fa]);
  LabelVt.Caption  := Format('v(t) = %.4f м/с',  [FvTerm]);
  LabelTau.Caption := Format('t = %.4f с',    [FvTerm / Fa]);

  MemoLog.Clear;
  MemoLog.Lines.Add('=== Параметры задачи ===');
  MemoLog.Lines.Add(Format('r = %.4f м, rho = %.0f кг/м3', [r, rho]));
  MemoLog.Lines.Add(LabelA.Caption);
  MemoLog.Lines.Add(LabelVt.Caption);
  MemoLog.Lines.Add(LabelTau.Caption);
  MemoLog.Lines.Add('');

  SetLength(FTimes, N + 1);
  SetLength(FHeights, N + 1);
  FCount := N + 1;

  LineSeries1.Clear;

  for i := 0 to N do
  begin
    h := h_max * i / N;
    FHeights[i] := h;

    if h = 0 then
    begin
      t := 0;
      iterCount := 0;
    end
    else
      t := SolveTime(h, Fa, FvTerm, iterCount);

    FTimes[i] := t;

    LineSeries1.AddXY(h, t);

    MemoLog.Lines.Add(Format('h = %.3f м -> t = %.6f с (итераций: %d)',
                             [h, t, iterCount]));
  end;

  MemoLog.Lines.Add('');
  MemoLog.Lines.Add('=== Готово. Точек на графике: ' +
                    IntToStr(LineSeries1.Count) + ' ===');

  Chart1.Repaint;
end;

{------------------------------------------------------------------}
{ Кнопка "Очистить"                                                }
{------------------------------------------------------------------}
procedure TForm1.ButtonClearClick(Sender: TObject);
begin
  {EditR.Text    := '0,01';
  EditRho.Text  := '7800';
  EditHmax.Text := '10';
  EditN.Text    := '10';  }

  LabelA.Caption   := 'a   = ---';
  LabelVt.Caption  := 'v(t) = ---';
  LabelTau.Caption := 't = ---';

  LineSeries1.Clear;
  MemoLog.Clear;
  FCount := 0;
end;



end.
