# Quickstart-McDemo205-Sigfox #
Este es un proyecto para enviar datos desde el McDemo205 a Sigfox, y cómo personalizar las variables desde el backend de Sigfox

Se usará la lectura del GNSS(GPS) embebido en el McDemo205, el sensor de temperatura, la red de Sigfox y Microsoft azure con Power BI para graficar los resultados.

## Antes de Empezar ##
El equipo/ software para empezar:
<br />
- McDemo205 de McThings
<br />
![McDemo205](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/McDemo205.jpg?raw=true)
<br />
- [McStudio](https://www.mcthings.com/s/mcStudioInstall-xfe2.msi "McStudio")  (Unicamente para windows por ahora) <br />
- Una cuenta en Sigfox: [https://backend.sigfox.com/](https://backend.sigfox.com/) Para México puedes obtener una cuenta escribiendo al correo contacto@iotnet.mx<br />
- Una cuenta de Microsoft Azure [https://portal.azure.com](https://portal.azure.com)

## Overview de McDemo205 ##
Verifica que el puente del McDemo esté alimentado por USB:
![McDemoImg](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Connection.png?raw=true)<br /> 
Al iniciar el programa McStudio, ir a la pestaña Tools > Devices.  <br />
![Devices](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Captura%20de%20pantalla%202017-01-18%20a%20las%207.10.12%20p.m..png?raw=true)<br />
Seleccionar Testboard Gateway y Connect Gateway. Después **retirar el puente del McDemo y volverlo a colocar en la alimentación USB** para reconocer el dispositivo. Después elegir el botón de Connect Device. Al conectarse se puede cerrar esta ventana. 
<br />
![ConnectDevice](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/ConnectDev.png?raw=true)<br />
<br /> 
### Primeros pasos con McDemo <br />
Pasa directamente al [proyecto de GNSS] (GNSS) <br />
Para entender la facilidad de [McScript](https://static1.squarespace.com/static/5644f11fe4b0d6ca7d80d351/t/57c61f35b8a79ba9708b2cc4/1472601916268/mc-ScriptUserGuide.pdf), el lenguaje usado por los productos de McThings, veamos un par de ejemplos incluidos en McStudio:
![ExampleProj](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/ExampleProj.png?raw=true) <br />
```
Class Demo
   Shared Event BlinkGreen() RaiseEvent Every 2000 milliSeconds
       Led2 = True
       Thread.Sleep(100000)
       Led2 = False
   End Event
   Shared Event BlinkRed() RaiseEvent Every 1500 milliSeconds
       Led3 = True
       Thread.Sleep(100000)
       Led3 = False
   End Event
End Class
```
<br /> <br />
Cambiamos ``LedGreen`` y ``LedRed`` por ``Led2`` y ``Led3`` respectivamente. Al crear un projecto automáticamente se genera la clase del nombre del proyecto. El tipo de evento ``Shared Event`` determina el alcance que tendrá, e.g. si las variables serán accesibles por otros eventos o sólo dentro de ese evento. Los leds se comportan como salidas digitales y tienen función de encendido y apagado (no pwm). <br /> <br />
Para correr el programa sólo hay que compilarlo, ejecutar y presionar el botón 1 : <br />
![BuildMcTh](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/BuildMcTh.png?raw=true) <br />
Usar los botónes no es mucho mas difícil<br />
``Shared Event SW1FallingEdge()``<br />
   ``LED2 = True``<br />
   ``Thread.Sleep(100000)``<br />
   ``Led2 = False``<br />
``End Event``<br />

### Enviando un mensaje a Sigfox
Esta es la configuración básica para enviar un mensaje por Sigfox:<br />
``Class SigfoxDemo``<br />
   ``Shared Event Button1FallingEdge()`` <br />
      ``Lplan.SigfoxRadioZone(sigfoxradiozone.US)``<br />
      ``Led2= True``<br />
      ``Thread.Sleep(500000)``<br />
      ``Dim sfData As ListOfByte = New ListOfByte``<br />
      ``sfData.Add(0x40)``<br />
      ``sfData.Add(0x87)``<br />
      ``sfData.Add(0x1A)``<br />
      ``Lplan.Sigfox(sfData)``<br />
      ``LED2 = False``<br />
   ``End Event``<br />
``End Class``<br />  
<br />
La primera parte nos dice que al presionar el botón 1 empezará el evento. El led es un indicador de que el evento está funcionando correctamente y ``Thread.Sleep(500000)`` es un tiempo para que no se apague inmediatamente.<br /> <br />
``Lplan.SigfoxRadioZone(sigfoxradiozone.US)`` Define la frecuencia a la que se enviarán los mensajes, la misma que se usa en US (902 MHz).<br /> <br />
``Dim sfData As ListOfByte = New ListOfByte`` Crea la lista de bytes que se enviarán por Sigfox. Se pueden enviar hasta 12 bytes por mensaje (24 caracteres en hexadecimal) en cada mensaje por Sigfox. Los bytes que se envían en este mensaje son ``0x40 0x87 0x1A`` <br />

### Sigfox Backend
En el portal de Sigfox https://backend.sigfox.com/welcome/news entramos a Device y damos click en el ID del dispositivo.<br />
![BuildMcTh](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Device.png?raw=true) <br /> <br />
En la pestaña de messages se puede ver el mensaje recibido, las estaciones que lo recibieron, la señal con la que llegó y el estado de los callbacks (ver más adelante). <br /> <br />
![BuildMcTh](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Msg.png?raw=true) <br /> <br />

### Obteniendo la ubicación por GNSS
El siguiente código nos da la ubicación del GNSS, el tiempo que tarda en obtener la ubicación y la envía por Sigfox. <br />
    
```
Class SigfoxGNSS 
  'GNSS Configuration Constants 
  'GNSS Timeout = 120s
  Const GNSS_TIMEOUT_uS As Integer = 120000000 
  'GNSS minimum sats = 3
  Const GNSS_MIN_SAT_COUNT As Integer = 3 
 
  Shared Event SW1FallingEdge()
    'turn on LED2 to indicate GNSS acquisition started
    Led2 = True``<br />
    Device.StartGPS(GNSS_TIMEOUT_uS, GNSS_MIN_SAT_COUNT)
  End Event
    
  Shared Event LocationDelivery()
    'Called when GNSS location acquired or timeout occurred
       'Get latitude
       Dim Lat As Float = Device.GetLatitude()
        
       'Get longitude
       Dim Lon As Float = Device.GetLongitude()
        
       'Get GNSS fix time
       Dim Time As Integer = Device.GetGpsFixTime()
       'Set GNSS In Seconds And set As Short 
       Dim timeSec As Float = Time / 1000000
       Dim timeSecShort As Short = timeSec.ToShort
        
       'Create list of bytes To send over Sigfox
       Dim SigfoxMsg As ListOfByte = New ListOfByte()
        
       'Turn not available location to 0.0
       If Lat.IsNaN() Then
          Lat = 0.0
       End If
       If Lon.IsNaN() Then
          Lon = 0.0
       End If
        
       'Add bytes to Sigfox Message
       SigfoxMsg.AddFloat(Lat)
       SigfoxMsg.AddFloat(Lon)
       SigfoxMsg.AddShort(timeSecShort)
        
        'If the GNSS got a location, send over Sigfox 
        If Lat <> 0.0 Then
            Lplan.Sigfox(SigfoxMsg)
            Led3 = True
            Thread.Sleep(7000000)
            Led3 = False
            Led2 = False
        Else
            Led2 = False
        End If
    End Event   
End Class 
``
<br />


  
## Setup de Microsoft Azure ##

