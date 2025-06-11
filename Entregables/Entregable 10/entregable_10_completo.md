
# Entregable N° 10

## Modelado 3D - Avance Prototipado Electrónico

### Hardware Electrónico

- Diseño esquemático del circuito electrónico (Versión final según observaciones del entregable anterior)
- Ejecución del código de ESP32 con componentes físicos implementados.

#### Propuesta del código final

<pre><code class="language-c" style="overflow-x: auto; display: block; white-space: pre;">
#include &lt;Arduino.h&gt;

const int fsrPins[4][3] = {
  {34, 35, 32},  // Celda A
  {33, 25, 26},  // Celda B
  {27, 14, 12},  // Celda C
  {13, 23, 22}   // Celda D
};

const int bombaPins[4] = {4, 16, 17, 5};
const int valvInflaPins[4] = {18, 19, 21, 3};
const int valvDesinflaPins[4] = {1, 2, 15, 0};

const int umbral_alto = 200;
const unsigned long tiempo_inflado = 4000;

bool enInflado[4] = {false, false, false, false};
unsigned long tiempoInicio[4] = {0, 0, 0, 0};

void setup() {
  Serial.begin(115200);
  for (int i = 0; i &lt; 4; i++) {
    pinMode(bombaPins[i], OUTPUT);
    pinMode(valvInflaPins[i], OUTPUT);
    pinMode(valvDesinflaPins[i], OUTPUT);
    digitalWrite(bombaPins[i], LOW);
    digitalWrite(valvInflaPins[i], LOW);
    digitalWrite(valvDesinflaPins[i], HIGH);
  }
}

void loop() {
  for (int i = 0; i &lt; 4; i++) {
    int maxPresion = 0;
    for (int j = 0; j &lt; 3; j++) {
      int lectura = analogRead(fsrPins[i][j]);
      if (lectura &gt; maxPresion) {
        maxPresion = lectura;
      }
    }

    if (!enInflado[i] && maxPresion &gt;= umbral_alto) {
      Serial.print("Inflando celda "); Serial.println(i);
      digitalWrite(valvDesinflaPins[i], LOW);
      digitalWrite(valvInflaPins[i], HIGH);
      digitalWrite(bombaPins[i], HIGH);
      tiempoInicio[i] = millis();
      enInflado[i] = true;
    }

    if (enInflado[i] && millis() - tiempoInicio[i] &gt;= tiempo_inflado) {
      Serial.print("Desinflando celda "); Serial.println(i);
      digitalWrite(bombaPins[i], LOW);
      digitalWrite(valvInflaPins[i], LOW);
      digitalWrite(valvDesinflaPins[i], HIGH);
      enInflado[i] = false;
    }
  }

  delay(50);
}
</code></pre>

### Explicación del código

Este programa implementa un sistema automático de regulación de presión en cuatro celdas neumáticas (A, B, C y D) utilizando sensores FSR402 conectados a un microcontrolador ESP32. Cada celda cuenta con su propia bomba de inflado y dos válvulas: una para inflar y otra para desinflar. El objetivo es detectar cuándo una celda experimenta una presión elevada, inflarla por un tiempo determinado y luego permitir su desinflado para evitar zonas de alta presión que puedan causar úlceras por presión.

Primero se definen los pines analógicos del ESP32 conectados a los sensores FSR, agrupados de tres en tres por celda. También se definen los pines digitales encargados de controlar individualmente las bombas, las válvulas de inflado y las de desinflado. En la función `setup()`, estos pines se inicializan como salidas, se apagan las bombas y válvulas de inflado, y se abren las válvulas de desinflado por defecto, permitiendo que las celdas comiencen sin aire acumulado.

Durante la ejecución continua del programa (`loop()`), se realiza una lectura de los tres sensores de cada celda y se considera el valor más alto como representativo. Si la presión detectada supera un umbral establecido (por ejemplo, 200), el sistema inicia un ciclo de inflado: se cierra la válvula de desinflado, se abre la válvula de inflado y se enciende la bomba correspondiente. Este estado se mantiene durante un tiempo fijo (por ejemplo, 4 segundos).

Después del tiempo de inflado, el sistema apaga la bomba, cierra la válvula de inflado y vuelve a abrir la de desinflado, permitiendo que la celda libere presión. Este ciclo se repite constantemente para cada celda, de manera independiente, lo que permite una redistribución de la presión localizada.

Este enfoque ofrece un control automático, celda por celda, que responde únicamente a condiciones de presión elevadas. No se requieren entradas manuales ni comparaciones complejas entre zonas. El sistema está diseñado para ser simple pero efectivo, y puede adaptarse a integraciones más avanzadas como comunicación inalámbrica, visualización de datos o control remoto si se desea expandir su funcionalidad.

### Avance del prototipado electrónico (una sola celda)

<pre><code class="language-c" style="overflow-x: auto; display: block; white-space: pre;">
const int fsrPin = A0;
const int relePin = 13;

const int UMBRAL_ACTIVAR = 200;
const int UMBRAL_REINICIO = 150;

const unsigned long TIEMPO_INFLADO_MS = 4000UL;
bool bombaActiva = false;
unsigned long tiempoInicio = 0;

void setup() {
  pinMode(fsrPin, INPUT);
  pinMode(relePin, OUTPUT);
  digitalWrite(relePin, LOW);
  Serial.begin(9600);
}

void loop() {
  int lecturaFSR = analogRead(fsrPin);
  Serial.print("FSR: ");
  Serial.println(lecturaFSR);

  unsigned long ahora = millis();

  if (!bombaActiva && lecturaFSR >= UMBRAL_ACTIVAR) {
    digitalWrite(relePin, HIGH);
    bombaActiva = true;
    tiempoInicio = ahora;
    Serial.println("Inflando globo...");
  }

  if (bombaActiva && (ahora - tiempoInicio >= TIEMPO_INFLADO_MS)) {
    digitalWrite(relePin, LOW);
    bombaActiva = false;
    Serial.println("Fin inflado. Esperando reinicio...");
  }

  delay(100);
}
</code></pre>

*(Explicación técnica completa de este avance también se incluirá al continuar el archivo)...
