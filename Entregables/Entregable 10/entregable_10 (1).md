
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

Este programa implementa un sistema automático de regulación de presión en cuatro celdas neumáticas (A, B, C y D)...

---

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

## Fabricación digital

- Modelado digital completo con todos los elementos modelados.
- Archivos DXF y planos para corte láser.

## Plan de usabilidad basado en evidencias

### 1. Contexto de uso

Este dispositivo está diseñado para ser utilizado por usuarios con movilidad reducida...

### 2. Perfil del usuario

El usuario objetivo de este cojín inteligente es una persona adulta mayor...

### 3. Análisis de tareas

Para garantizar el uso correcto del cojín inteligente...

### 4. Criterios de éxito

- **Eficacia:** éxito ≥ 90%.
- **Eficiencia:** instalación &lt; 2 minutos.
- **Satisfacción:** escala SUS &gt; 70.
- **Seguridad:** tasa de incidentes = 0.

## Referencias bibliográficas

- Fadil, R., et al. (2022). https://doi.org/10.1016/j.medengphy.2019.06.006  
- Navickas, R., et al. (2022). https://www.mdpi.com/1660-4601/19/4/2195  
- Carrigan, W., et al. (2019). https://doi.org/10.1016/j.jtv.2022.04.004
