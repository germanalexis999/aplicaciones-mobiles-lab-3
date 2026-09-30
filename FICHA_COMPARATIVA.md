# Ficha Comparativa Completa — Trilogía AppTeca (3A vs 3B vs 3C)

## Tabla Comparativa General

| Pregunta / Aspecto | 3A — Modelo Tradicional (Views + XML) | 3B — MVVM + ViewModel + LiveData | 3C — Jetpack Compose + StateFlow |
| :--- | :--- | :--- | :--- |
| **1. ¿Cuántas piezas hicieron falta para mostrar la lista?** | **4 piezas**: Layout XML del ítem (`item_app.xml`), `AppViewHolder`, `AppAdapter` y configuración en `MainActivity`. | **5 piezas**: Layout XML del ítem, `AppViewHolder`, `AppAdapter`, `AppTecaViewModel` (con `LiveData`), y suscripción en `MainActivity`. | **1 sola función declarativa**: `LazyColumn` con invocación a `FilaApp`. Sin XMLs, sin ViewHolder, sin Adapter, sin `findViewById`. |
| **2. ¿Dónde vive el estado de la pantalla (búsqueda, modo, favoritos, navegación)?** | Disperso en Vistas XML (`EditText`), variables de Activity (`soloFavoritas`) y el singleton `Catalogo`. | Centralizado en `AppTecaViewModel` (`LiveData`) para la lista y el modo. Navegación imperativa mediante Intents. | Centralizado en `AppTecaViewModel` (`StateFlow`), **incluyendo la navegación (`appSeleccionada: StateFlow<App?>`)**. `rememberSaveable` para la búsqueda. |
| **3. ¿Qué sobrevivió y qué murió al rotar? ¿Y al morir el proceso?** | **Al rotar**: Murió `soloFavoritas`.<br>**Al morir proceso**: Murió todo. | **Al rotar**: Sobrevivió todo el estado del `ViewModel`.<br>**Al morir proceso**: Murió la RAM del proceso. | **Al rotar**: **Sobrevivió todo el estado** (incluyendo detalle abierto y pantalla activa).<br>**Al morir proceso**: Murió el ViewModel y singleton, pero `rememberSaveable` rescató el texto del buscador. |
| **4. ¿Qué costaría agregar un campo nuevo al ítem (por ej., un puntaje)?** | Modificar 7 lugares distintos entre Modelo, Catálogo, Layouts XML, ViewHolder, Adapter y Activities. | Mismas modificaciones de UI, aunque la capa de presentación (`ViewModel`) casi no se toca. | **Mínimo costo**: Agregar campo a `App.kt` y consumirlo directamente en los composables `FilaApp` y `DetalleApp`. |
| **5. ¿Qué se podría probar de esta lógica sin abrir un emulador?** | **Nada o muy poco**. Lógica acoplada a la `Activity` y al árbol de Vistas. | **Toda la lógica de presentación en ViewModel** mediante tests unitarios puros en JVM. | **Toda la lógica de ViewModel en JVM**, más previsualización directa en el IDE con `@Preview` y tests de UI livianos. |
| **6. ¿Qué se sintió mejor y qué peor de esta forma de trabajar?** | **Lo mejor**: Modelo conocido.<br>**Lo peor**: Sincronización manual, pérdida de estado al rotar. | **Lo mejor**: Eliminación del bug de rotación.<br>**Lo peor**: Necesidad de `onResume()` para refrescar la lista al no tener repositorio reactivo. | **Lo mejor**: Cero ceremonia XML, muerte natural de la "grieta del detalle", reactividad inmutable.<br>**Lo peor**: Necesidad de comprender inmutabilidad (`copy()`) para la recomposición. |

---

## Preguntas de Cierre

### Pregunta 1
**Un equipo mantiene una app Views+XML grande, estable y rentable. ¿Le recomendás reescribirla en Compose? ¿Qué le recomendás, y con qué de tu ficha lo fundamentás?**

* **Respuesta**: **No se recomienda la reescritura total desde cero**.
* **Fundamentación**: Reescribir una aplicación grande y madura genera un costo masivo de desarrollo sin aportar valor directo e inmediato al usuario final, con el riesgo elevado de introducir regresiones. Según fundamenta la **Fila 4 de la Ficha Comparativa**, el costo de modificar o reescribir pantalla por pantalla es elevado.
* **Recomendación**: Adopción gradual y pragmática mediante la **interoperabilidad nativa de Compose** (utilizando `ComposeView` para incorporar componentes en composables dentro de layouts XML existentes, o desarrollando únicamente las pantallas nuevas en Compose).

---

### Pregunta 2
**El mismo equipo arranca un producto nuevo desde cero. ¿Qué stack elegís y por qué — citando al menos tres filas de tu ficha?**

* **Respuesta**: Se elige **Jetpack Compose + MVVM con StateFlow** (Columna C de la ficha).
* **Fundamentación (citando la ficha)**:
  1. **Fila 1 (Cero Ceremonia y Mantenimiento de UI)**: Reducción drástica de código de infraestructura (una función `LazyColumn` sustituye 4 componentes tradicionales de `RecyclerView`), acelerando el desarrollo.
  2. **Fila 3 (Resiliencia del Estado y Navegación Declarativa)**: El manejo de estado centralizado en `ViewModel` y la navegación reactiva eliminan de forma nativa los bugs de recreación por giros de pantalla.
  3. **Fila 5 (Alta Testeabilidad y Previsualización con `@Preview`)**: Permite probar la lógica de negocio en JVM pura sin emulador y previsualizar componentes de UI en tiempo real dentro del IDE.
