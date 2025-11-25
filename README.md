# proyecto-de-uacm
proyecto final de la materia introduccion
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// --- Constantes ---
#define MAX_MESAS 5
#define MAX_RESERVAS 50

// Usar enum para estados mejora la legibilidad y tipado
typedef enum {
    MESA_LIBRE = 0,
    MESA_OCUPADA = 1
} EstadoMesa;

// --- Estructuras ---
typedef struct {
    int id_mesa;
    EstadoMesa estado; // Usando el enum
} Mesa;

typedef struct {
    int id_reservacion;
    int id_cliente;
    int id_mesa_asignada;
    char hora_inicio[6]; // "HH:MM"
} Reservacion;

// --- Base de Datos (simulada) ---
Mesa mesas[MAX_MESAS];
Reservacion reservas[MAX_RESERVAS];
int total_reservas = 0;
int proximo_id_reservacion = 1001;

/**
 * @brief Inicializa todas las mesas del sistema, asignando ID y estado LIBRE.
 */
void inicializar_mesas() {
    for (int i = 0; i < MAX_MESAS; i++) {
        mesas[i].id_mesa = i + 1;
        mesas[i].estado = MESA_LIBRE;
    }
}

/**
 * @brief Imprime en consola el estado actual de todas las mesas.
 */
void consultar_disponibilidad() {
    printf("\n--- 📊 Disponibilidad de Mesas ---\n");
    for (int i = 0; i < MAX_MESAS; i++) {
        const char* estado_str = (mesas[i].estado == MESA_LIBRE) ? "LIBRE" : "OCUPADA";
        printf("Mesa %d: %s\n", mesas[i].id_mesa, estado_str);
    }
    printf("------------------------------------\n");
}

/**
 * @brief Verifica el formato básico de hora HH:MM y su rango (00:00 a 23:59).
 * @param hora Cadena de texto de la hora.
 * @return 1 si es válida, 0 si es inválida.
 */
int validar_hora(const char* hora) {
    int h, m;
    // Intentar leer 2 enteros separados por ':'
    if (sscanf(hora, "%2d:%2d", &h, &m) != 2) return 0;
    // Validar rangos
    if (h < 0 || h > 23) return 0;
    if (m < 0 || m > 59) return 0;
    return 1;
}

/**
 * @brief Intenta crear una nueva reservación asignando la primera mesa libre.
 * @param id_cliente ID único del cliente.
 * @param hora Hora de inicio de la reserva (HH:MM).
 * @return 1 si la reserva fue exitosa, 0 si falló.
 */
int solicitar_reservacion(int id_cliente, const char* hora) {
    if (!validar_hora(hora)) {
        printf("\n❌ Formato de hora inválido. Use HH:MM.\n");
        return 0;
    }

    int indice_mesa_encontrada = -1;

    // Buscar el índice de la primera mesa libre
    for (int i = 0; i < MAX_MESAS; i++) {
        if (mesas[i].estado == MESA_LIBRE) {
            indice_mesa_encontrada = i;
            break;
        }
    }

    // Si NO hay mesas libres
    if (indice_mesa_encontrada == -1) {
        printf("\n❌ No hay mesas disponibles.\n");
        return 0;
    }

    // --- 💾 Crear reservación y actualizar estado de la mesa ---
    mesas[indice_mesa_encontrada].estado = MESA_OCUPADA;

    // Crear y llenar la estructura de la reserva
    Reservacion r;
    r.id_reservacion = proximo_id_reservacion++;
    r.id_cliente = id_cliente;
    r.id_mesa_asignada = mesas[indice_mesa_encontrada].id_mesa;
    strncpy(r.hora_inicio, hora, 5); // 5 caracteres (HH:MM) + 1 para '\0' = 6. Se usa 5 para la copia.
    r.hora_inicio[5] = '\0'; // Asegurar el nulo terminador

    // Guardar en la "base de datos"
    reservas[total_reservas++] = r;

    printf("\n✅ Reserva Confirmada!\n");
    printf("ID Reserva: %d\n", r.id_reservacion);
    printf("Cliente: %d\n", id_cliente);
    printf("Mesa Asignada: %d\n", r.id_mesa_asignada);
    printf("Hora: %s\n", r.hora_inicio);

    return 1;
}

// --- Programa Principal ---
int main() {
    inicializar_mesas();

    printf("===========================================\n");
    printf("      SIMULACION DE SISTEMA DE RESERVAS\n");
    printf("===========================================\n");

    consultar_disponibilidad();

    // --- Solicitud al usuario ---
    int cliente_id;
    char hora_reserva[6];

    printf("\n--- ✍️ Solicitar Nueva Reserva ---\n");
    printf("Ingrese su ID de Cliente: ");
    if (scanf("%d", &cliente_id) != 1) {
        printf("Error al leer ID de cliente.\n");
        return 1;
    }
    // Limpiar el buffer de entrada antes de leer la hora
    while (getchar() != '\n'); 

    printf("Ingrese la hora (HH:MM): ");
    if (scanf("%5s", hora_reserva) != 1) {
         printf("Error al leer la hora.\n");
        return 1;
    }
    // Asegurar el nulo terminador, aunque scanf("%5s") lo hace si la entrada no es más larga.
    hora_reserva[5] = '\0'; 

    // Crear reserva
    solicitar_reservacion(cliente_id, hora_reserva);

    // Mostrar estado actualizado
    printf("\n[ADMINISTRADOR: Estado después de la reserva]\n");
    consultar_disponibilidad();

    // Simulación de cancelación de mesa 1
    printf("\n--- 🚫 Simulación de Cancelar Reserva ---\n");
    // Se usa el índice 0 para la Mesa 1
    if (mesas[0].estado == MESA_OCUPADA) { 
        mesas[0].estado = MESA_LIBRE;
        printf("🚫 Reserva Cancelada. Mesa 1 liberada.\n");
    } else {
        printf("Mesa 1 ya estaba libre.\n");
    }

    consultar_disponibilidad();

    // Mantener ventana abierta
    printf("\nPresione ENTER para salir...");
    while (getchar() != '\n'); // Limpia buffer restante
    getchar(); // Espera la tecla ENTER

    return 0;
}
