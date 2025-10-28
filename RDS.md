# DiagnÃ³stico de conexiÃ³n RDP entre Windows 11 y Windows Server 2019 con aplicaciones VB6

Este documento describe las situaciones mÃ¡s comunes que pueden provocar desconexiones o errores al conectarse desde **Windows 11** a un **servidor de Escritorio Remoto (RDS) en Windows Server 2019**, especialmente cuando se ejecutan aplicaciones **legacy desarrolladas en VB6**.

---

## ðŸ§© Tabla de diagnÃ³stico: Windows 11 â†” Windows Server 2019 (RDS con apps VB6)

| **SituaciÃ³n / SÃ­ntoma** | **Causa probable** | **Posible soluciÃ³n o ajuste recomendado** |
|--------------------------|--------------------|-------------------------------------------|
| ðŸ”¹ **El cliente Windows 11 se conecta y se desconecta de inmediato, sin mostrar error.** | Incompatibilidad con *Network Level Authentication (NLA)* o protocolo TLS 1.3 entre cliente y servidor. | En el servidor, abrir **gpedit.msc** â†’ `ConfiguraciÃ³n del equipo > Plantillas administrativas > Componentes de Windows > Servicios de Escritorio remoto > Host de sesiÃ³n > Seguridad`. <br>âœ” Establecer â€œ**Nivel de seguridad de conexiÃ³n: RDP Security Layer**â€ <br>âœ” Deshabilitar â€œ**Requerir NLA**â€. <br>Reiniciar servicio: `net stop termservice && net start termservice`. |
| ðŸ”¹ **La sesiÃ³n RDP carga brevemente el fondo y luego se cierra sola.** | AceleraciÃ³n grÃ¡fica o *hardware rendering* incompatible (VB6 usa GDI y controles OCX antiguos). | En el cliente Windows 11: ConfiguraciÃ³n > Sistema > Pantalla > GrÃ¡ficos > ConexiÃ³n a Escritorio Remoto â†’ **Desactivar aceleraciÃ³n de hardware**. <br>TambiÃ©n editar `.rdp`: `redirectdrives:i:0`, `use multimon:i:0`, `desktopcomposition:i:0`. |
| ðŸ”¹ **RDP falla solo con ciertos usuarios, pero otros sÃ­ logran entrar.** | Perfil de usuario remoto daÃ±ado (NTUSER.DAT corrupto o permisos incorrectos). | En el servidor: renombrar `C:\Users\Usuario` â†’ `Usuario.old` y eliminar su clave en `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\<SID>`. Luego volver a iniciar sesiÃ³n para regenerarlo. |
| ðŸ”¹ **El cliente muestra â€œSe perdiÃ³ la conexiÃ³n con el servidor remotoâ€ inmediatamente.** | Cortes de sesiÃ³n por handshake TLS 1.3 no soportado o falta de compatibilidad en canal seguro. | Desactivar TLS 1.3 o limitar a 1.2: en registro del servidor (`regedit`) agregar: <br>`[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.3\Server]` â†’ `Enabled=0` (DWORD). Reiniciar. |
| ðŸ”¹ **VB6 se cierra o cuelga cuando se abre un formulario con grÃ¡ficos o controles OCX.** | Uso de GDI+ o DirectDraw bajo RDP con RemoteFX activado. | En el servidor: deshabilitar RemoteFX: `gpedit.msc > ConfiguraciÃ³n del equipo > Plantillas administrativas > Componentes de Windows > Remote Desktop Services > RemoteFX > ConfiguraciÃ³n > Deshabilitar`. <br>AdemÃ¡s, ejecutar las apps VB6 en compatibilidad Windows 7 y desactivar â€œOptimizar pantalla completaâ€. |
| ðŸ”¹ **El escritorio remoto muestra pantalla negra y luego se desconecta.** | PolÃ­ticas de sesiÃ³n o lÃ­mites de tiempo en el RDS. | En el servidor: `gpedit.msc > Servicios de Escritorio remoto > Host de sesiÃ³n > LÃ­mites de tiempo de sesiÃ³n`. <br>Configurar â€œSin lÃ­miteâ€ para sesiones activas y reconexiones. |
| ðŸ”¹ **Solo ocurre en Windows 11 22H2 o 23H2, no en clientes Windows 10.** | Actualizaciones recientes rompieron compatibilidad (p. ej. KB5036892). | Desinstalar temporalmente la actualizaciÃ³n problemÃ¡tica (`wusa /uninstall /kb:5036892`) o instalar parches mÃ¡s recientes de RDP. <br>Verificar que el servidor tenga KB5031258 o superior. |
| ðŸ”¹ **Los formularios VB6 abren lentos o con parpadeo en RDP.** | RDP redibuja GDI y ventanas por cambios de tema o animaciones. | En las propiedades de conexiÃ³n RDP â†’ pestaÃ±a â€œExperienciaâ€ â†’ Desactivar: *ComposiciÃ³n del escritorio*, *Temas visuales*, *Animaciones*. |
| ðŸ”¹ **DesconexiÃ³n aleatoria despuÃ©s de unos minutos sin actividad.** | Timeout de inactividad o polÃ­tica de energÃ­a. | `gpedit.msc > Servicios de Escritorio remoto > Host de sesiÃ³n > LÃ­mites de tiempo de sesiÃ³n` â†’ â€œNo definir lÃ­mite para sesiones desconectadasâ€. <br>En â€œEnergÃ­aâ€ del servidor, usar plan â€œAlto rendimientoâ€. |
| ðŸ”¹ **Mensajes â€œNo se pudo iniciar sesiÃ³n en el servicio de perfil de usuarioâ€.** | Perfil temporal por conflicto de antivirus o permisos en registro. | Deshabilitar antivirus temporalmente, corregir permisos en `C:\Users\Default` y verificar que `NTUSER.DAT` estÃ© Ã­ntegro. |

---

## ðŸ§° Recomendaciones adicionales para entornos VB6 en RDS 2019

- Ejecutar las aplicaciones VB6 como **RemoteApp** en lugar de sesiones completas RDP.  
  ðŸ‘‰ AsÃ­, si una aplicaciÃ³n falla, no desconecta toda la sesiÃ³n.
- Mantener instaladas las **actualizaciones de VB6 runtime redistributables (SP6)**.  
- Evitar **controladores de video modernos (NVIDIA/AMD) con aceleraciÃ³n RemoteFX**: usar el controlador estÃ¡ndar de Microsoft.
- Si usas **redirecciÃ³n de impresoras o puertos**, prueba desactivarlas (`redirectprinters:i:0`, `redirectcomports:i:0` en el archivo .rdp).

---

> ðŸ“„ Documento generado automÃ¡ticamente â€” Heiner G. Cambronero | 2025