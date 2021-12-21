## Overview

Individual forms in a multi-form application can be registered as separate Glue42 applications. The forms to be registered as Glue42 apps must implement the [`IGlueApp`](../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglueapp) interface and the main form must be registered as a Glue42 application factory in order to create and register the forms.

## App Factories

To make a form a factory for Glue42 applications, implement the [`IAppFactory`](../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iappfactory) interface. The [`CreateApp`](../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iappfactory-createapp) method of the interface will be invoked by Glue42 whenever a new form is to be created as a Glue42 application:

```delphi
TMainForm = class(TForm, IAppFactory)
  ...
protected
  function CreateApp(const appDefName: WideString; state: GlueValue; const announcer: IAppAnnouncer): HResult; stdcall;
  ...
```

The form acting as a Glue42 application factory may be invisible.

After initializing Glue42, register the main form as an app factory. The following example demonstrates registering the main form as an app factory for two Glue42 applications:

```delphi
procedure TMainForm.InitializeGlue;
var
  ...
  appDef: GlueAppDefinition;
begin
  ...
  G42.Start(inst);
  ...
  // Register an application.
  ZeroMemory(@appDef, sizeof(appDef));
  appDef.Name := 'DelpiFormApp01';
  appDef.title := 'Delphi Form App 01';
  appDef.Category := 'COM Apps';
  G42.AppFactoryRegistry.RegisterAppFactory(appDef, Self);

  // Register another application.
  ZeroMemory(@appDef, sizeof(appDef));
  appDef.Name := 'DelpiFormApp02';
  appDef.title := 'Delphi Form App 02';
  appDef.Category := 'COM Apps';
  G42.AppFactoryRegistry.RegisterAppFactory(appDef, Self);
  ...
```

## Registering App Instances

After the main form has been registered as an [app factory](#app_factories), it can create and register child forms as separate Glue42 applications. The forms that are to be registered as Glue42 applications must first implement the [`IGlueApp`](../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iglueapp) interface and its methods:

```delphi
TApp01Form = class(TForm, IGlueApp)
  ...
protected
  function SaveState(out pRetVal: GlueValue): HResult; stdcall;
  function Initialize(state: GlueValue; const glueWindow: IGlueWindow): HResult; stdcall;
  function Shutdown: HResult; stdcall;
  ...
```

The following is a sample minimal implementation of the interface methods:

```delphi
function TApp01Form.Initialize(state: GlueValue;
  const glueWindow: IGlueWindow): HResult;
begin
  Result := S_OK;
end;

function TApp01Form.SaveState(out pRetVal: GlueValue): HResult;
begin
  Result := S_OK;
end;

function TApp01Form.Shutdown: HResult;
begin
  Result := S_OK;
  Close;
end;
```

Now the forms can be created and registered as Glue42 apps with the [`CreateApp`](../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iappfactory-createapp) method of the [`IAppFactory`](../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iappfactory) interface implemented by main form:

```delphi
function TMainForm.CreateApp(const appDefName: WideString; state: GlueValue; const announcer: IAppAnnouncer): HResult; stdcall;
var
  form01Inst: TApp01Form;
  form02Inst: TApp02Form;
begin
  if appDefName = 'DelphiFormApp01' then
  begin
    form01Inst := TApp01Form.Create(self);
    // Registering a form as a Glue42 app instance.
    announcer.RegisterAppInstance(form01Inst.Handle, form01Inst);
    Result := S_OK;
  end
  else if appDefName = 'DelphiFormApp02' then
  begin
    form02Inst := TApp02Form.Create(self);
    // Registering another form.
    announcer.RegisterAppInstance(form02Inst.Handle, form02Inst);
    Result := S_OK;
  end
  else
    Result := E_FAIL;
end;
```

*Note that the [`RegisterAppInstance`](../../../getting-started/how-to/glue42-enable-your-app/delphi/index.html#interfaces-iappannouncer-registerappinstance) method also registers the child form as a [Glue42 Window](../../windows/window-management/delphi/index.html).*