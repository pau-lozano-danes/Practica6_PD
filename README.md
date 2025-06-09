# Práctica 6: Buses de Comunicación II (SPI)

## Descripción
El objetivo de esta práctica es entender el funcionamiento del bus de comunicación **SPI (Serial Peripheral Interface)**, ampliamente utilizado en sistemas embebidos por su rapidez y sencillez. El bus SPI permite la comunicación **Full Duplex** entre un dispositivo maestro y uno o varios dispositivos esclavos, utilizando las siguientes líneas:

- **MOSI (Master Out Slave In)**
- **MISO (Master In Slave Out)**
- **SCK (Serial Clock)**
- **SS (Slave Select)**

Esta práctica se centra en el **ESP32-S3** e incluye dos ejercicios prácticos:
1. Lectura y escritura de una memoria SD.
2. Lectura de una etiqueta RFID con el módulo MFRC522.

Ambos dispositivos utilizan el bus SPI, lo que permite explorar de forma práctica cómo iniciar, enviar y recibir datos a través de esta interfaz.

El conocimiento adquirido en esta práctica es esencial para diseñar sistemas que requieren interconectar múltiples periféricos de alta velocidad.

---

## Ejercicio Práctico 1

### Descripción
En este ejercicio se trabaja con una tarjeta SD utilizando el bus SPI para realizar operaciones básicas de **lectura y escritura**. El programa:
- Inicializa la comunicación serie.
- Configura el bus SPI con el pin GPIO 10 como **Chip Select (CS)**.
- Escribe la frase **"Hola desde ESP32-S3"** en un archivo llamado `archivo.txt`.
- Lee y muestra el contenido del archivo en el monitor serie.

### Código:
```cpp
#include <SPI.h>
#include <SD.h>

File myFile;

void setup() {
    Serial.begin(115200);
    Serial.print("Iniciando SD...");

    if (!SD.begin(10)) {  // CS en GPIO 10
        Serial.println("No se pudo inicializar");
        return;
    }
    Serial.println("Inicialización exitosa");

    // Escritura en el archivo
    myFile = SD.open("/archivo.txt", FILE_WRITE);
    if (myFile) {
        myFile.println("Hola desde ESP32-S3");
        myFile.close();
        Serial.println("Escritura exitosa");
    } else {
        Serial.println("Error al abrir el archivo");
    }

    // Lectura del archivo
    myFile = SD.open("/archivo.txt");
    if (myFile) {
        Serial.println("Contenido del archivo:");
        while (myFile.available()) {
            Serial.write(myFile.read());
        }
        myFile.close();
    } else {
        Serial.println("Error al leer el archivo");
    }
}

void loop() {
    // No se realiza ninguna operación en el loop
}
```

---

## Ejercicio Práctico 2

### Descripción
Este ejercicio consiste en leer el identificador único (UID) de una tarjeta RFID utilizando el módulo MFRC522 conectado mediante el bus SPI. El programa:

-Inicializa la comunicación serie y el bus SPI.

-Configura los pines de control:

  -RST_PIN: GPIO 8

  -SS_PIN: GPIO 9

-Escanea continuamente tarjetas RFID.

-Muestra el UID de cada tarjeta detectada en el monitor serie.

### Codigo 

```cpp
#include <SPI.h>
#include <MFRC522.h>

#define RST_PIN 8    // Pin de reset del RC522
#define SS_PIN  9    // Pin SS (SDA) del RC522

MFRC522 mfrc522(SS_PIN, RST_PIN);  // Crear objeto RFID

void setup() {
    Serial.begin(115200);  // Iniciar la comunicación serial
    SPI.begin();           // Iniciar el bus SPI
    mfrc522.PCD_Init();    // Iniciar el módulo RFID
    Serial.println("Escaneando tarjetas RFID...");
}

void loop() {
    // Verificar si hay una nueva tarjeta presente
    if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
        Serial.print("UID de tarjeta: ");

        // Leer el UID de la tarjeta y mostrarlo en hexadecimal
        for (byte i = 0; i < mfrc522.uid.size; i++) {
            Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
            Serial.print(mfrc522.uid.uidByte[i], HEX);
        }
        Serial.println();

        mfrc522.PICC_HaltA();  // Detener la lectura de la tarjeta
    }
}
```
