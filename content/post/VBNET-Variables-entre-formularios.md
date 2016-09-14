+++
categories = ["Dev", "lPinchol"]
date = "2016-04-11T16:55:01+02:00"
draft = false
featureimage = "img/avatar.png"
menu = ""
tags = ["lPinchol", "Dev"]
title = "VB.NET Variables entre formularios"

+++

<center>![001](/img/Dev/vb_net.png)</center>

>Como en cualquier lenguaje vb.net tiene sus constructores para poder pasar variables entre formularios. Aquí dejo un pequeño ejemplo de como puede crearse un constructor:

__________________________

``` C#
Public Class Form2
.
.
.
    Private texto As String
    Private numero As Integer


    ' Constructor Form2
    Public Sub New (ByVal miTexto As String, ByVal miNumero As Integer)

        Me.texto = miTexto
        Me.numero = miNumero

    End Sub
.
.
.

End Class
```
________________________

> Y la llamada pasadole los parámetros se realiza de esta manera:

________________________

``` C#
' Instancia
Dim miForm2 As New Form2("Este es el texto que paso", 2)

' Mostramos el form2
miForm2.Show()
```