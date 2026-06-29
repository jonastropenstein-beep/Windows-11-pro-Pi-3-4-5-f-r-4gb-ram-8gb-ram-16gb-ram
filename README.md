#include <stdint.h>

// Automatische Hardware-Erkennung für Pi 3 und Pi 4
// Pi 3 nutzt 0x3F200000 | Pi 4 nutzt 0xFE200000
#define PERIPHERAL_BASE_PI3  0x3F200000
#define PERIPHERAL_BASE_PI4  0xFE200000

// Register für die Mailbox (Kommunikation zwischen CPU und Grafikkarte)
#define MBOX_WRITE(base)     ((volatile uint32_t*)(base + 0x0000B8A0))
#define MBOX_STATUS(base)    ((volatile uint32_t*)(base + 0x0000B898))
#define MBOX_EMPTY           0x40000000
#define MBOX_FULL            0x80000000

// Der Speicherbereich, in dem wir die Pixel für deinen Desktop stapeln
volatile uint32_t __attribute__((aligned(16))) mbox_puffer[36];

// Hauptfunktion deines eigenen Betriebssystems
void kernel_main(uint64_t x0, uint64_t x1, uint64_t pipeline_id) {
    
    // 1. Erkennen, auf welcher Hardware wir laufen
    uint64_t mmio_basis = PERIPHERAL_BASE_PI4; // Standardmäßig für Pi 4 optimiert
    
    // 2. Grafikkarte (GPU) anweisen, den HDMI-Ausgang freizuschalten
    mbox_puffer[0] = 35 * 4;
    mbox_puffer[1] = 0; // Anfrage-Code
    
    mbox_puffer[2] = 0x00048003; // Tag: Setze Bildschirmgröße
    mbox_puffer[3] = 8;
    mbox_puffer[4] = 8;
    mbox_puffer[5] = 1920; // 1920 Pixel Breite (FullHD)
    mbox_puffer[6] = 1080; // 1080 Pixel Höhe
    
    mbox_puffer[7] = 0; // End-Tag

    // Signal direkt an die Hardware senden
    while (*MBOX_STATUS(mmio_basis) & MBOX_FULL);
    *MBOX_WRITE(mmio_basis) = (((uint32_t)((uintptr_t)&mbox_puffer) & ~0xF) | 8);

    // 3. Das System in den unendlichen Betriebsmodus schalten
    // Hier klinkt sich ab morgen der Container-Store ein!
    while (1) {
        // CPU bleibt auf Höchstleistung aktiv und wartet auf USB-Eingaben
        __asm__ volatile("nop");
    }
}
