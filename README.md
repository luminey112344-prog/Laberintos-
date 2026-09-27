#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <conio.h>
#include <windows.h>
#include <algorithm>
#include <string>

using namespace std;

int FILAS = 50;  
int COLS = 50;
const int VIEW = 21; 

// Estadísticas del Jugador (Se mantienen o mejoran entre partidas)
int playerX = 1, playerY = 1;
int level = 1;
int xp = 0;
int xpNeeded = 5;
int score = 0; 

// Valores base iniciales (mejoras permanentes)
int maxHpIncial = 20;
int baseDamageInicial = 2;
int pocionesIniciales = 1;
int bonusArmaInicial = 0;
string nombreArmaInicial = "Manos Desnudas";

// Variables de la partida actual
int hp;
int maxHp;
int baseDamage;
int potions;
int weaponDamageBonus;
string weaponName;

vector<vector<char>> world;

void ocultarCursorYPosicionar() {
    HANDLE hOut = GetStdHandle(STD_OUTPUT_HANDLE);
    COORD Position = {0, 0};
    SetConsoleCursorPosition(hOut, Position);
    
    CONSOLE_CURSOR_INFO cursorInfo;
    GetConsoleCursorInfo(hOut, &cursorInfo);
    cursorInfo.bVisible = FALSE;
    SetConsoleCursorInfo(hOut, &cursorInfo);
}

void inicializarJugador() {
    hp = maxHpIncial;
    maxHp = maxHpIncial;
    baseDamage = baseDamageInicial;
    potions = pocionesIniciales;
    weaponDamageBonus = bonusArmaInicial;
    weaponName = nombreArmaInicial;
    playerX = 1;
    playerY = 1;
    level = 1;
    xp = 0;
    xpNeeded = 5;
}

void generarMapa() {
    world.assign(FILAS, vector<char>(COLS, '#'));

    for (int i = 1; i < FILAS - 1; i++) {
        for (int j = 1; j < COLS - 1; j++) {
            int r = rand() % 100;
            if (r < 75) world[i][j] = '.'; 
            else world[i][j] = '#';        
        }
    }

    world[1][1] = '.';
    world[FILAS - 2][COLS - 2] = 'X';

    for (int i = 1; i < FILAS - 1; i++) {
        for (int j = 1; j < COLS - 1; j++) {
            if ((i == 1 && j == 1) || (i == FILAS - 2 && j == COLS - 2)) continue;

            if (world[i][j] == '.') {
                int r = rand() % 100;
                if (r < 4) world[i][j] = 'M';       
                if (r >= 4 && r < 6) world[i][j] = '$';  
                if (r >= 6 && r < 8) world[i][j] = '^';  
                if (r >= 8 && r < 11) world[i][j] = '*'; 
            }
        }
    }
    world[playerX][playerY] = '@';
}

void tiendaPostMuerte(int nivelAlMorir) {
    system("cls");
    bool enTienda = true;

    string tipoPocion = (nivelAlMorir < 5) ? "Pocion Bendita (Cura 18 HP)" : "Pocion Estandar (Cura 10 HP)";

    while (enTienda) {
        ocultarCursorYPosicionar();
        cout << "====================================================================\n";
        cout << "                    ▲ LA TIENDA DEL INFRAMUNDO ▲                    \n";
        cout << "====================================================================\n";
        cout << " Moriste en el Nivel: " << nivelAlMorir << "   |   Tu Oro disponible: " << score << " Monedas\n";
        cout << " Compra mejoras permanentes o equipo para tu siguiente encarnacion:\n";
        cout << "====================================================================\n\n";

        cout << " [1] Entrenar Vitalidad (+5 Max HP para siempre)    | Costo: 40 Oro\n";
        cout << " [2] Comprar " << tipoPocion << "           | Costo: 25 Oro\n";

        if (nivelAlMorir >= 5) {
            cout << " [3] COMPRA DE NIVEL ALTO: Mandoble Oscuro (+12 DMG) | Costo: 120 Oro\n";
            cout << " [4] COMPRA DE NIVEL ALTO: Filo del Vacio (+18 DMG)  | Costo: 200 Oro\n";
        } else {
            cout << " [3] COMPRA INICIAL: Daga de Hierro (+4 DMG)        | Costo: 35 Oro\n";
            cout << " [4] COMPRA INICIAL: Espada de Bronce (+6 DMG)      | Costo: 60 Oro\n";
            cout << " (Alcanza el nivel 5 en tu proxima vida para desbloquear mejores armas)\n";
        }
        
        cout << "\n [0] REGRESAR AL LABERINTO (Comenzar nueva partida con tus mejoras)\n";
        cout << " [Q] SALIR DEL JUEGO (Cerrar el programa definitivamente)\n";
        cout << "--------------------------------------------------------------------\n";
        cout << "Selecciona una opcion: ";

        char opc = getch();
        cout << "\n";

        if (opc == '1') {
            if (score >= 40) {
                score -= 40;
                maxHpIncial += 5;
                cout << "¡Tu vitalidad maxima ha aumentado permanentemente!                 \n";
                Sleep(800);
            } else {
                cout << "¡No tienes suficiente oro!                                         \n";
                Sleep(800);
            }
        } 
        else if (opc == '2') {
            if (score >= 25) {
                score -= 25;
                pocionesIniciales++;
                cout << "¡Compraste una pocion! Empezaras con ella en tu inventario.        \n";
                Sleep(800);
            } else {
                cout << "¡No tienes suficiente oro!                                         \n";
                Sleep(800);
            }
        } 
        else if (opc == '3') {
            if (nivelAlMorir >= 5) {
                if (score >= 120) {
                    score -= 120;
                    nombreArmaInicial = "Mandoble Oscuro";
                    bonusArmaInicial = 12;
                    cout << "¡Has adquirido el Mandoble Oscuro!                             \n";
                    Sleep(800);
                } else { cout << "Oro insuficiente.                                           \n"; Sleep(800); }
            } else {
                if (score >= 35) {
                    score -= 35;
                    nombreArmaInicial = "Daga de Hierro";
                    bonusArmaInicial = 4;
                    cout << "¡Has adquirido la Daga de Hierro!                              \n";
                    Sleep(800);
                } else { cout << "Oro insuficiente.                                           \n"; Sleep(800); }
            }
        } 
        else if (opc == '4') {
            if (nivelAlMorir >= 5) {
                if (score >= 200) {
                    score -= 200;
                    nombreArmaInicial = "Filo del Vacio";
                    bonusArmaInicial = 18;
                    cout << "¡Has adquirido el Filo del Vacio legendario!                   \n";
                    Sleep(800);
                } else { cout << "Oro insuficiente.                                           \n"; Sleep(800); }
            } else {
                if (score >= 60) {
                    score -= 60;
                    nombreArmaInicial = "Espada de Bronce";
                    bonusArmaInicial = 6;
                    cout << "¡Has adquirido la Espada de Bronce!                            \n";
                    Sleep(800);
                } else { cout << "Oro insuficiente.                                           \n"; Sleep(800); }
            }
        } 
        else if (opc == '0') {
            enTienda = false;
        }
        else if (opc == 'q' || opc == 'Q') {
            system("cls");
            cout << "Gracias por jugar. ¡Hasta la proxima encarnacion!\n";
            exit(0);
        }
        system("cls");
    }

    inicializarJugador();
    generarMapa();
}

void morir() {
    system("cls");
    cout << "==================================================\n";
    cout << "            ¡HAS CAIDO EN EL LABERINTO!           \n";
    cout << "==================================================\n";
    cout << " Tu alma viaja al inframundo...\n";
    cout << " Conservas tus monedas de oro para negociar.\n\n";
    cout << " Presiona cualquier tecla para entrar a la tienda...";
    getch();
    
    int nivelGuardado = level; 
    tiendaPostMuerte(nivelGuardado);
}

void drawHUD() {
    cout << "====================================================================\n";
    cout << " JUGADOR: @  |  NIVEL: " << level << "  |  XP: " << xp << "/" << xpNeeded << "\n";
    cout << " HP: " << hp << "/" << maxHp << " [";
    for (int i = 0; i < maxHp; i += 2) { 
        if (i < hp) cout << "I";
        else cout << ".";
    }
    cout << "]  |  ORO ACUMULADO: " << score << "\n";
    cout << " ARMA: " << weaponName << " (+" << weaponDamageBonus << " DMG)  |  POCIONES: " << potions << "\n";
    cout << "====================================================================\n";
}

void drawWorld() {
    int startX = max(0, playerX - VIEW / 2);
    int endX = min(FILAS, startX + VIEW);
    int startY = max(0, playerY - VIEW / 2);
    int endY = min(COLS, startY + VIEW);

    if (endX - startX < VIEW) startX = max(0, endX - VIEW);
    if (endY - startY < VIEW) startY = max(0, endY - VIEW);

    for (int i = startX; i < endX; i++) {
        for (int j = startY; j < endY; j++) {
            cout << world[i][j] << " ";
        }
        cout << "\n";
    }
    cout << "\n[WASD/Flechas] Mover | [H] Usar Pocion | [X] Salida\n";
}

void checkLevelUp() {
    while (xp >= xpNeeded) {
        xp -= xpNeeded;
        level++;
        xpNeeded = level * 5;
        maxHp += 5;
        hp = maxHp; 
        baseDamage += 2;
        system("cls");
        cout << "\n   ¡¡¡ SUBISTE DE NIVEL !!! Ahora eres Nivel " << level << "\n";
        cout << "   Tu vida maxima aumento y tu fuerza base ha crecido.\n";
        cout << "   Presiona cualquier tecla para continuar...";
        getch();
        system("cls");
    }
}

void usarPocion() {
    if (potions > 0) {
        if (hp == maxHp) {
            cout << "¡Ya tienes la vida al maximo!                      \n";
            Sleep(600);
            return;
        }
        potions--;
        int curacion = (level < 5) ? 18 : 10; 
        hp = min(maxHp, hp + curacion);
        cout << "¡Usaste una pocion! Recuperas " << curacion << " HP.               \n";
        Sleep(600);
    } else {
        cout << "¡No te quedan pociones!                            \n";
        Sleep(600);
    }
}

void abrirCofre() {
    system("cls");
    cout << "==================================================\n";
    cout << "               ¡ABRISTE UN COFRE!                 \n";
    cout << "==================================================\n";
    
    int r = rand() % 2;
    if (r == 0) {
        potions++;
        cout << "\n   ¡Encontraste una Pocion de Curacion!\n";
    } else {
        int oroEncontrado = (rand() % 40) + 20;
        score += oroEncontrado;
        cout << "\n   ¡El cofre tenia riquezas! + " << oroEncontrado << " de Oro.\n";
    }
    cout << "\nPresiona cualquier tecla para continuar...";
    getch();
    system("cls");
}

void battle() {
    system("cls");
    int monsterLevel = max(1, level + (rand() % 3 - 1));
    int monsterHP = 5 + (monsterLevel * 3);
    int monsterDamage = 1 + monsterLevel;
    int xpReward = monsterLevel * 3;

    cout << "==================================================\n";
    cout << "  ¡UN MONSTRUO LVL " << monsterLevel << " TE HA EMBOSCADO!   \n";
    cout << "==================================================\n";

    while (monsterHP > 0 && hp > 0) {
        cout << "\nTu HP: " << hp << "/" << maxHp << " | HP Monstruo: " << monsterHP << "\n";
        cout << "[F] Atacar   [D] Defender   [H] Usar Pocion (" << potions << " restantes)\n";
        
        char c = getch();
        int damageDealt = (rand() % 4) + baseDamage + weaponDamageBonus; 
        int damageTaken = (rand() % 3) + monsterDamage;

        if (c == 'f' || c == 'F') {
            monsterHP -= damageDealt;
            cout << "-> ¡Golpeas al monstruo por " << damageDealt << " de danio!\n";
        } 
        else if (c == 'd' || c == 'D') {
            damageTaken /= 2;
            if (damageTaken < 1) damageTaken = 1;
            cout << "-> Te defiendes firmemente.\n";
        } 
        else if (c == 'h' || c == 'H') {
            usarPocion();
        }

        // Si el jugador usó poción o atacó y el monstruo sigue vivo, el monstruo contraataca
        if (monsterHP > 1) {
            hp -= damageTaken;
            cout << "-> ¡El monstruo te ataca y te hace " << damageTaken << " de danio!\n";
        }

        // CONTROL DE MUERTE: Romper el bucle inmediatamente si el jugador muere
        if (hp <= 0) {
            break;
        }
    }

    // Procesar los resultados fuera del bucle de combate para evitar errores
    if (hp <= 0) {
        morir();
        return; 
    }

    if (monsterHP <= 0) {
        int oroGanado = monsterLevel * 6;
        score += oroGanado;
        xp += xpReward;
        
        cout << "\n==================================================\n";
        cout << " ¡VICTORIA! + " << xpReward << " XP | + " << oroGanado << " Oro\n";
        cout << "==================================================\n";
        getch();
        system("cls");
        checkLevelUp();
    }
}

void movePlayer(char dir) {
    int newX = playerX, newY = playerY;

    if (dir == 'w' || dir == 'W') newX--;
    if (dir == 's' || dir == 'S') newX++;
    if (dir == 'a' || dir == 'A') newY--;
    if (dir == 'd' || dir == 'D') newY++;
    if (dir == 'h' || dir == 'H') { usarPocion(); return; }

    if (dir == -32 || dir == 0) { 
        char flecha = getch();
        if (flecha == 72) newX--; 
        if (flecha == 80) newX++; 
        if (flecha == 75) newY--; 
        if (flecha == 77) newY++; 
    }

    if (world[newX][newY] != '#') {
        char destino = world[newX][newY];

        if (destino == 'M') {
            battle();
            return; // Detener movimiento para que se refresque el mapa tras la pelea/tienda
        }
        else if (destino == '$') abrirCofre();
        else if (destino == '*') {
            xp += 2;
            checkLevelUp();
        }
        else if (destino == '^') {
            hp -= 4;
            if (hp <= 0) { 
                morir(); 
                return; 
            }
        }
        else if (destino == 'X') {
            system("cls");
            cout << "==================================================\n";
            cout << "         ¡FELICIDADES, ESCAPASTE DEL LABERINTO!   \n";
            cout << "         Oro Total Acumulado: " << score << "\n";
            cout << "==================================================\n";
            exit(0);
        }

        world[playerX][playerY] = '.';
        playerX = newX; playerY = newY;
        world[playerX][playerY] = '@';
    }
}

int main() {
    srand(time(0));
    inicializarJugador();
    generarMapa();
    system("cls"); 

    while (true) {
        ocultarCursorYPosicionar();
        drawHUD();
        drawWorld();
        
        char c = getch();
        movePlayer(c);
    }
    return 0;
}
