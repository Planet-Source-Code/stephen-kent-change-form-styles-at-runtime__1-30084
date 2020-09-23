<div align="center">

## Change Form Styles at Runtime


</div>

### Description

This sub-procedure will allow the developer to fairly easily switch between a form's border styles during runtime. Normally this isn't really possible because several of the attributes are read-only at runtime. This code overcomes those limitations.

I have only tested this with VB6, but since it is basically just API calls it should be able to work with any version that supports API calls.

Thanks to Fred_CPP for the tip on using SWP_FRAMECHANGED instead of resizing the form.
 
### More Info
 
For certain buttons to work such as those in the control box they need to be enabled in design time (even if they are then hidden at runtime) otherwise there will be no handlers linked to those buttons and they will be useless. This applies to the What's This Button, Min Button, Max Button, and the Control Box. (What's this button has same restrictions on it as it does when used normally)


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Stephen Kent](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/stephen-kent.md)
**Level**          |Beginner
**User Rating**    |5.0 (30 globes from 6 users)
**Compatibility**  |VB 6\.0
**Category**       |[Windows API Call/ Explanation](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/windows-api-call-explanation__1-39.md)
**World**          |[Visual Basic](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/visual-basic.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/stephen-kent-change-form-styles-at-runtime__1-30084/archive/master.zip)

### API Declarations

```
Private Declare Function SetWindowLong Lib "user32" Alias "SetWindowLongA" (ByVal hwnd As Long, ByVal nIndex As Long, ByVal dwNewLong As Long) As Long
Private Declare Function SetWindowPos Lib "user32" (ByVal hwnd As Long, ByVal hWndInsertAfter As Long, ByVal x As Long, ByVal y As Long, ByVal cx As Long, ByVal cy As Long, ByVal wFlags As Long) As Long
'Get/Set WindowLong Constants (only those used)
Private Const GWL_STYLE = (-16)
Private Const GWL_EXSTYLE = (-20)
'SetWindowPos Constants (only those used)
Private Const SWP_FRAMECHANGED = &H20 'The frame changed: send WM_NCCALCSIZE
Private Const SWP_NOMOVE = &H2
Private Const SWP_NOSIZE = &H1
'Dialog Styles (also present in the GWL_STYLE area)
Private Const DS_ABSALIGN   As Long = &H1
Private Const DS_SYSMODAL   As Long = &H2
Private Const DS_3DLOOK    As Long = &H4
Private Const DS_FIXEDSYS   As Long = &H8
Private Const DS_NOFAILCREATE  As Long = &H10
Private Const DS_LOCALEDIT   As Long = &H20 'Edit items get Local storage.
Private Const DS_SETFONT   As Long = &H40 'User specified font for Dlg controls
Private Const DS_MODALFRAME   As Long = &H80 'Can be combined with WS_CAPTION
Private Const DS_NOIDLEMSG   As Long = &H100 'WM_ENTERIDLE message will not be sent
Private Const DS_SETFOREGROUND  As Long = &H200 'not in win3.1
Private Const DS_CONTROL   As Long = &H400
Private Const DS_CENTER    As Long = &H800
Private Const DS_CENTERMOUSE  As Long = &H1000
Private Const DS_CONTEXTHELP  As Long = &H2000
'Window Styles (GWL_STYLE area)
Private Const WS_OVERLAPPED   As Long = &H0
Private Const WS_POPUP    As Long = &H80000000
Private Const WS_CHILD    As Long = &H40000000
Private Const WS_MINIMIZE   As Long = &H20000000
Private Const WS_VISIBLE   As Long = &H10000000
Private Const WS_DISABLED   As Long = &H8000000
Private Const WS_CLIPSIBLINGS  As Long = &H4000000
Private Const WS_CLIPCHILDREN  As Long = &H2000000
Private Const WS_MAXIMIZE   As Long = &H1000000
Private Const WS_CAPTION   As Long = &HC00000 'WS_BORDER | WS_DLGFRAME
Private Const WS_BORDER    As Long = &H800000
Private Const WS_DLGFRAME   As Long = &H400000
Private Const WS_VSCROLL   As Long = &H200000
Private Const WS_HSCROLL   As Long = &H100000
Private Const WS_SYSMENU   As Long = &H80000
Private Const WS_THICKFRAME   As Long = &H40000
Private Const WS_GROUP    As Long = &H20000
Private Const WS_TABSTOP   As Long = &H10000
Private Const WS_MINIMIZEBOX  As Long = &H20000
Private Const WS_MAXIMIZEBOX  As Long = &H10000
Private Const WS_TILED    As Long = WS_OVERLAPPED
Private Const WS_ICONIC    As Long = WS_MINIMIZE
Private Const WS_SIZEBOX   As Long = WS_THICKFRAME
'Extended Window Styles (GWL_EXSTYLE area)
Private Const WS_EX_DLGMODALFRAME As Long = &H1
Private Const WS_EX_NOPARENTNOTIFY As Long = &H4
Private Const WS_EX_TOPMOST   As Long = &H8
Private Const WS_EX_ACCEPTFILES  As Long = &H10
Private Const WS_EX_TRANSPARENT  As Long = &H20
Private Const WS_EX_MDICHILD  As Long = &H40
Private Const WS_EX_TOOLWINDOW  As Long = &H80
Private Const WS_EX_WINDOWEDGE  As Long = &H100
Private Const WS_EX_CLIENTEDGE  As Long = &H200
Private Const WS_EX_CONTEXTHELP  As Long = &H400
Private Const WS_EX_RIGHT   As Long = &H1000
Private Const WS_EX_LEFT   As Long = &H0
Private Const WS_EX_RTLREADING  As Long = &H2000
Private Const WS_EX_LTRREADING  As Long = &H0
Private Const WS_EX_LEFTSCROLLBAR As Long = &H4000
Private Const WS_EX_RIGHTSCROLLBAR As Long = &H0
Private Const WS_EX_CONTROLPARENT As Long = &H10000
Private Const WS_EX_STATICEDGE  As Long = &H20000
Private Const WS_EX_APPWINDOW  As Long = &H40000
```


### Source Code

```
Public Sub ChangeFormBorder(frmForm As Form, _
       ByVal eNewBorder As FormBorderStyleConstants, _
       Optional ByVal bClipControls As Boolean = True, _
       Optional ByVal bControlBox As Boolean = True, _
       Optional ByVal bMaxButton As Boolean = True, _
       Optional ByVal bMinButton As Boolean = True, _
       Optional ByVal bShowInTaskBar As Boolean = True, _
       Optional ByVal bWhatsThisButton As Boolean = False)
 Dim lRet As Long
 Dim lStyleFlags As Long
 Dim lStyleExFlags As Long
 'Initialize our flags
 lStyleFlags = 0
 lStyleExFlags = 0
 'If we want ClipControls then add that flag and change the form property
 If bClipControls Then
  lStyleFlags = lStyleFlags Or WS_CLIPCHILDREN
  frmForm.ClipControls = True
 Else
  frmForm.ClipControls = False
 End If
 'If we want the control box then add the flag (property is read-only)
 If bControlBox Then lStyleFlags = lStyleFlags Or WS_SYSMENU
 'If we want the max button then add the flag (property is read-only)
 If bMaxButton Then lStyleFlags = lStyleFlags Or WS_MAXIMIZEBOX
 'If we want the min button then add the flag (property is read-only)
 If bMinButton Then lStyleFlags = lStyleFlags Or WS_MINIMIZEBOX
 'If we want the form to show in taskbar then add the flag (property is read-only)
 If bShowInTaskBar Then lStyleExFlags = lStyleExFlags Or WS_EX_APPWINDOW
 'If we want the what's this button then add the flag (property is read-only)
 If bWhatsThisButton Then lStyleExFlags = lStyleExFlags Or WS_EX_CONTEXTHELP
 'If the form is an MDI Child form then add the flag (Don't want to screw up the form)
 If frmForm.MDIChild Then lStyleExFlags = lStyleExFlags Or WS_EX_MDICHILD
 'Now we need to set the flags for the border we are changing to
 Select Case eNewBorder
  Case vbBSNone
   lStyleFlags = lStyleFlags Or (WS_VISIBLE Or WS_CLIPSIBLINGS)
   'No change to extended style flags.
  Case vbFixedSingle
   lStyleFlags = lStyleFlags Or (WS_VISIBLE Or WS_CLIPSIBLINGS Or WS_CAPTION)
   lStyleExFlags = lStyleExFlags Or WS_EX_WINDOWEDGE
  Case vbSizable
   lStyleFlags = lStyleFlags Or (WS_VISIBLE Or WS_CLIPSIBLINGS Or WS_CAPTION Or WS_THICKFRAME)
   lStyleExFlags = lStyleExFlags Or WS_EX_WINDOWEDGE
  Case vbFixedDialog
   lStyleFlags = lStyleFlags Or (WS_VISIBLE Or WS_CLIPSIBLINGS Or WS_CAPTION Or DS_MODALFRAME)
   lStyleExFlags = lStyleExFlags Or (WS_EX_WINDOWEDGE Or WS_EX_DLGMODALFRAME)
  Case vbFixedToolWindow
   lStyleFlags = lStyleFlags Or (WS_VISIBLE Or WS_CLIPSIBLINGS Or WS_CAPTION)
   lStyleExFlags = lStyleExFlags Or (WS_EX_WINDOWEDGE Or WS_EX_TOOLWINDOW)
  Case vbSizableToolWindow
   lStyleFlags = lStyleFlags Or (WS_VISIBLE Or WS_CLIPSIBLINGS Or WS_CAPTION Or WS_THICKFRAME)
   lStyleExFlags = lStyleExFlags Or (WS_EX_WINDOWEDGE Or WS_EX_TOOLWINDOW)
 End Select
 'WS_VISIBLE makes sure the form is visible
 'WS_CLIPSIBLINGS makes sure that when there are other windows with the same relative family that they do not draw over each other.
 'WS_CAPTION provides the form's caption
 'WS_THICKFRAME makes the form sizable
 'DS_MODALFRAME allows dialog forms to have 3d effect
 'WS_EX_WINDOWEDGE is for the border around the form
 'WS_EX_DLGMODALFRAME says the window has a double border and may or may not have a caption
 'WS_EX_TOOLWINDOW says we need a shorter caption and smaller font
 'Change our styles
 lRet = SetWindowLong(frmForm.hwnd, GWL_STYLE, lStyleFlags)
 lRet = SetWindowLong(frmForm.hwnd, GWL_EXSTYLE, lStyleExFlags)
 'Signal that the frame has changed
 lRet = SetWindowPos(frmForm.hwnd, 0, 0, 0, 0, 0, SWP_NOMOVE Or SWP_NOSIZE Or SWP_FRAMECHANGED)
 'Make that we've changed the border in the form's property
 frmForm.BorderStyle = eNewBorder
End Sub
```

