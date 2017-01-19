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
![McDemoImg](https://github.com/Iotnet/Quickstart-McDemo205-Sigfox/blob/master/Images/image.png?raw=true)<br /> 
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

Class Demo                                                        //Cada proyecto se define por una clase <br />
    Shared Event BlinkGreen() RaiseEvent Every 2000 milliSeconds  //El tipo de evento determina el alcance <br />
        Led2 = True                                               
        Thread.Sleep(100000)                                      //Tiempo que permanecerá encendido el Led2 <br />
        Led2 = False                
    End Event                                                     
    Shared Event BlinkRed() RaiseEvent Every 1500 milliSeconds
        Led3 = True
        Thread.Sleep(100000)
        Led3 = False
    End Event
End Class





## Setup de Microsoft Azure ##

