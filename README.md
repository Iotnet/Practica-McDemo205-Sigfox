# Práctica McDemo205 con Sigfox #
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
![McDemoImg](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Connection.png?raw=true)<br /> <br />
Al iniciar el programa McStudio, ir a la pestaña Tools > Devices.  <br />
![Devices](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Captura%20de%20pantalla%202017-01-18%20a%20las%207.10.12%20p.m..png?raw=true)<br />
Seleccionar Testboard Gateway y Connect Gateway. Después **retirar el puente del McDemo y volverlo a colocar en la alimentación USB** para reconocer el dispositivo. Después elegir el botón de Connect Device. Al conectarse se puede cerrar esta ventana. 
<br />
![ConnectDevice](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/ConnectDev.png?raw=true)<br />
<br /> 
### Primeros pasos con McDemo <br />
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

```
Shared Event SW1FallingEdge()
   LED2 = True
   Thread.Sleep(100000)
   Led2 = False
End Event
```
### Enviando un mensaje a Sigfox

Esta es la configuración básica para enviar un mensaje por Sigfox:<br />
```
Class SigfoxDemo
   Shared Event Button1FallingEdge()
      Lplan.SigfoxRadioZone(sigfoxradiozone.US)
      Led2= True
      Thread.Sleep(500000)
      Dim sfData As ListOfByte = New ListOfByte
      sfData.Add(0x40)
      sfData.Add(0x87)
      sfData.Add(0x1A)
      Lplan.Sigfox(sfData)
      LED2 = False
   End Event
End Class
```
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
El siguiente código nos da la ubicación del GNSS, el tiempo que tarda en obtener la ubicación y la envía por Sigfox. Las secciones comentadas aparecen ``'De esta forma``<br />
Al presionar el botón 1 encenderá el led2, si detecta una ubicación del GNSS encendera el led3 unos segundos y después encenderá. Si no detecta nada solo apagará el led2 después de 2 minutos (``GNSS_Timeout = 120s``).<br />
El GNSS necesita estar en exterior, estático y alrededor de 10 metros de edificios.<br />
    
```
Class SigfoxGNSS 
  'GNSS Configuration Constants 
  'GNSS Timeout = 120s
  Const GNSS_TIMEOUT_uS As Integer = 120000000 
  'GNSS minimum sats = 3
  Const GNSS_MIN_SAT_COUNT As Integer = 3 
 
  Shared Event SW1FallingEdge()
    'turn on LED2 to indicate GNSS acquisition started
    Led2 = True
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
```
<br />
### Personalizando variables en Sigfox
En esta sección se editará el JSON que se envía a través de Sigfox a otros clouds como Microsoft Azure, IBM Watson, etc.<br />
![Edit](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Edit.png?raw=true) <br />
<br /> 
En display type elige Custom y Custom configuration introduce: <br />
``lat::float:32:little-endian lon::float:32:little-endian  Timeout::uint:16`` <br />
Nombre:: tipo de var: Nº de bits: Bit mas significativo (big-endian por default)
<br /> <br />
![CustomPayload](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/CustomPl.png?raw=true) <br />
<br />
Ahora se puede agregar esta información en el payload de Sigfox. También al ir a la sección de mensajes se mostrarán los valores obtenidos:<br />
![CustomCallback](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/CustomMsg.png?raw=true) <br />


### Creando un callback en Sigfox
Ingresa a la sección de Device Type > Callbacks donde se encuentre el McDemo y crea uno nuevo.<br /> 
![NewCallback](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/NewCallback.png?raw=true) <br />
<br />
Elegir Azure Iot Hub. <br />
![IotHub](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/IotHub.png?raw=true) <br />
Agrega el Json Body que se enviará a Azure. <br /> 
```
{ 
"device" : "{device}",
"data" : "{data}",
"Latitud": "{customData#lat}",
"Longitud": "{customData#lon}",
"Timeout": "{customData#Timeout}",
"time" : {time},
"duplicate" : {duplicate},
"snr" : {snr},
"station" : "{station}",
"avgSignal" : {avgSnr},
"lat" : {lat},
"lng" : {lng},
"rssi" : {rssi},
"seqNumber" : {seqNumber}
}
```
<br />
Dejaremos Connection string en blanco por ahora para preparar el backend en Azure.
<br />
![Callback](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/callback.png?raw=true) <br />
<br />
## Setup de Microsoft Azure ##
Conectarse a https://portal.azure.com e iniciar una aplicación de IotHub. <br />
![NewHub](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/NewIotHub.png?raw=true) <br />
<br />
Elegir un nombre para el centro y los recursos (Tomar la suscripción gratuita si está disponible) y crear. <br />
![IotCenter](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/IotCenter.png?raw=true) <br /> 
<br />
Una vez creado entrar a las Directivas de Acceso compartido en el Hub.<br />
![Acces](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Access.png?raw=true) <br /> 
<br />
La cadena de conexión será el link para postear en el backend de Sigfox (**Connection string**). Pegarla y darle ok. <br /> 
![Acces](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Keys.png?raw=true) <br />
<br />
Regresando a Azure, ahora crearemos una instancia de Steam Analitics para crear un mapa en Power BI con los datos del GNSS.<br />
![Analitics](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/SteamA.png?raw=true) 
![Analitics2](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/SteamA2.png?raw=true) <br />
<br />
Agregar una nueva entrada <br />
![Analitics](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/Analitic.png?raw=true) <br />
<br />
Mantener el siguiente formato, grupo de consumidores default, formato Json, codificacion UTF-8. <br />
![Entradas](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/newInput.png?raw=true) <br />
Crear una nueva salida a Power BI. <br />
![Salidas](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/salidas.png?raw=true) <br />




