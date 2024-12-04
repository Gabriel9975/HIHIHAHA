#include <allegro5/allegro.h>
#include <allegro5/allegro_image.h>
#include <allegro5/allegro_primitives.h>
#include <allegro5/allegro_font.h>
#include <allegro5/allegro_ttf.h>
#include <allegro5/allegro_audio.h>
#include <allegro5/allegro_acodec.h>
#include <stdio.h>
#include <stdbool.h>
#include <stdlib.h>
#include <time.h>

#define ROWS 20
#define COLS 20
#define TILE_SIZE 30

typedef struct {
    int x, y;
    int direction;
    int lives;
    int score;
} Pacman;

typedef struct {
    int x, y;
    int direction;
} Ghost;

// Fonction pour vérifier si Pacman peut se déplacer
bool can_move(int x, int y, int labyrinthe[ROWS][COLS]) {
    if (x < 0 || x >= COLS || y < 0 || y >= ROWS)
        return false;
    return labyrinthe[y][x] != 1;
}

// Fonction pour dessiner le labyrinthe
void draw_maze(int labyrinthe[ROWS][COLS], ALLEGRO_BITMAP *wall, ALLEGRO_BITMAP *point) {
    for (int row = 0; row < ROWS; row++) {
        for (int col = 0; col < COLS; col++) {
            int tile = labyrinthe[row][col];
            if (tile == 1) {
                al_draw_bitmap(wall, col * TILE_SIZE, row * TILE_SIZE, 0);
            }
            else if (tile == 2) {
                al_draw_bitmap(point, col * TILE_SIZE + (TILE_SIZE/2 - al_get_bitmap_width(point)/2),
                                     row * TILE_SIZE + (TILE_SIZE/2 - al_get_bitmap_height(point)/2), 0);
            }
        }
    }
}

// Fonction pour dessiner une entité
void draw_entity(ALLEGRO_BITMAP *sprite, int x, int y) {
    al_draw_bitmap(sprite, x * TILE_SIZE, y * TILE_SIZE, 0);
}

int main(int argc, char **argv) {
    srand(time(NULL)); // Initialiser le générateur de nombres aléatoires

    // Initialisation d'Allegro
    if (!al_init()) {
        fprintf(stderr, "Échec de l'initialisation d'Allegro.\n");
        return -1;
    }

    // Initialisation des modules
    al_init_image_addon();
    al_init_primitives_addon();
    al_init_font_addon();
    al_init_ttf_addon();
    al_install_keyboard();
    al_install_audio();
    al_init_acodec_addon();

    // Création de la fenêtre
    const int SCREEN_WIDTH = COLS * TILE_SIZE;
    const int SCREEN_HEIGHT = ROWS * TILE_SIZE + 50; // Espace pour le HUD
    ALLEGRO_DISPLAY *display = al_create_display(SCREEN_WIDTH, SCREEN_HEIGHT);
    if (!display) {
        fprintf(stderr, "Échec de la création de l'affichage.\n");
        return -1;
    }

    // Chargement des images
    ALLEGRO_BITMAP *wall = al_load_bitmap("assets/images/wall.png");
    ALLEGRO_BITMAP *point = al_load_bitmap("assets/images/point.png");
    ALLEGRO_BITMAP *pacman_sprite = al_load_bitmap("assets/images/pacman.png");
    ALLEGRO_BITMAP *ghost_sprite = al_load_bitmap("assets/images/ghost.png");

    if (!wall || !point || !pacman_sprite || !ghost_sprite) {
        fprintf(stderr, "Échec du chargement des images.\n");
        al_destroy_display(display);
        return -1;
    }

    // Chargement des sons
    ALLEGRO_SAMPLE *eat_sound = al_load_sample("assets/sounds/eat.wav");
    ALLEGRO_SAMPLE *death_sound = al_load_sample("assets/sounds/death.wav");

    // Vérifier le chargement des sons
    if (!eat_sound || !death_sound) {
        fprintf(stderr, "Échec du chargement des sons.\n");
        al_destroy_bitmap(wall);
        al_destroy_bitmap(point);
        al_destroy_bitmap(pacman_sprite);
        al_destroy_bitmap(ghost_sprite);
        al_destroy_display(display);
        return -1;
    }

    // Initialisation des polices
    ALLEGRO_FONT *font = al_load_ttf_font("assets/fonts/arial.ttf", 24, 0);
    if (!font) {
        fprintf(stderr, "Échec du chargement de la police.\n");
        // Continuez même si la police n'est pas chargée
    }

    // Définition du labyrinthe
    int labyrinthe[ROWS][COLS] = {
        {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
        {1,2,2,2,2,2,1,2,2,2,2,2,1,2,2,2,2,2,2,1},
        {1,2,1,1,2,2,1,2,1,1,1,2,1,2,2,1,1,2,2,1},
        {1,2,1,1,2,2,1,2,1,1,1,2,1,2,2,1,1,2,2,1},
        {1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,1},
        {1,2,1,1,2,1,1,1,2,1,1,1,2,1,1,1,2,1,2,1},
        {1,2,2,2,2,2,2,2,2,0,0,2,2,2,2,2,2,2,2,1},
        {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1}
    };
    // Remplissez le reste du labyrinthe avec des chemins libres ou des points

    // Initialisation de Pacman
    Pacman pacman = {1, 1, 1, 3, 0}; // Position initiale, direction, vies, score

    // Initialisation des Fantômes
    int num_ghosts = 4;
    Ghost ghosts[4] = {
        {18, 1, 3},
        {18, 18, 3},
        {1, 18, 1},
        {10, 10, 0}
    };

    // Création d'un timer
    ALLEGRO_TIMER *timer = al_create_timer(1.0 / 60); // 60 FPS

    // Création d'une file d'événements
    ALLEGRO_EVENT_QUEUE *queue = al_create_event_queue();
    al_register_event_source(queue, al_get_display_event_source(display));
    al_register_event_source(queue, al_get_timer_event_source(timer));
    al_register_event_source(queue, al_get_keyboard_event_source());

    // Démarrer le timer
    al_start_timer(timer);

    // Boucle principale
    bool running = true;
    bool redraw = true;
    while (running) {
        ALLEGRO_EVENT event;
        al_wait_for_event(queue, &event);

        if (event.type == ALLEGRO_EVENT_TIMER) {
            // Gestion des entrées
            while(al_get_next_event(queue, &event)) {
                if(event.type == ALLEGRO_EVENT_KEY_DOWN) {
                    switch(event.keyboard.keycode) {
                        case ALLEGRO_KEY_UP:
                            if(can_move(pacman.x, pacman.y -1, labyrinthe)) {
                                pacman.y -=1;
                                pacman.direction = 0;
                                al_play_sample(eat_sound, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                            }
                            break;
                        case ALLEGRO_KEY_RIGHT:
                            if(can_move(pacman.x +1, pacman.y, labyrinthe)) {
                                pacman.x +=1;
                                pacman.direction = 1;
                                al_play_sample(eat_sound, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                            }
                            break;
                        case ALLEGRO_KEY_DOWN:
                            if(can_move(pacman.x, pacman.y +1, labyrinthe)) {
                                pacman.y +=1;
                                pacman.direction = 2;
                                al_play_sample(eat_sound, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                            }
                            break;
                        case ALLEGRO_KEY_LEFT:
                            if(can_move(pacman.x -1, pacman.y, labyrinthe)) {
                                pacman.x -=1;
                                pacman.direction = 3;
                                al_play_sample(eat_sound, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                            }
                            break;
                        case ALLEGRO_KEY_ESCAPE:
                            running = false;
                            break;
                    }
                }
            }

            // Mise à jour des fantômes
            for(int i = 0; i < num_ghosts; i++) {
                move_ghost(&ghosts[i], labyrinthe);
            }

            // Vérification des collisions
            for(int i = 0; i < num_ghosts; i++) {
                if(pacman.x == ghosts[i].x && pacman.y == ghosts[i].y) {
                    pacman.lives -=1;
                    al_play_sample(death_sound, 1.0, 0.0, 1.0, ALLEGRO_PLAYMODE_ONCE, NULL);
                    // Réinitialiser la position de Pacman
                    pacman.x = 1;
                    pacman.y = 1;
                    if(pacman.lives <=0) {
                        running = false;
                        printf("Game Over! Score: %d\n", pacman.score);
                    }
                }
            }

            // Collecte des points
            if(labyrinthe[pacman.y][pacman.x] == 2) {
                labyrinthe[pacman.y][pacman.x] = 0;
                pacman.score +=10;
            }

            // Vérifier la condition de victoire
            bool victoire = true;
            for(int row = 0; row < ROWS && victoire; row++) {
                for(int col = 0; col < COLS && victoire; col++) {
                    if(labyrinthe[row][col] == 2)
                        victoire = false;
                }
            }
            if(victoire) {
                running = false;
                printf("Vous avez gagné! Score: %d\n", pacman.score);
            }

            redraw = true;
        }
        else if(event.type == ALLEGRO_EVENT_DISPLAY_CLOSE) {
            running = false;
        }

        if(redraw && al_is_event_queue_empty(queue)) {
            // Dessiner à l'écran
            al_clear_to_color(al_map_rgb(0, 0, 0));

            // Dessiner le labyrinthe
            draw_maze(labyrinthe, wall, point);

            // Dessiner Pacman
            draw_entity(pacman_sprite, pacman.x, pacman.y);

            // Dessiner les fantômes
            for(int i =0; i < num_ghosts; i++) {
                draw_entity(ghost_sprite, ghosts[i].x, ghosts[i].y);
            }

            // Dessiner le HUD
            if(font) {
                char hud[50];
                sprintf(hud, "Score: %d   Vies: %d", pacman.score, pacman.lives);
                al_draw_text(font, al_map_rgb(255, 255, 255), 10, ROWS * TILE_SIZE + 10, 0, hud);
            }

            al_flip_display();
            redraw = false;
        }
    }

    // Nettoyage
    al_destroy_sample(eat_sound);
    al_destroy_sample(death_sound);
    al_destroy_bitmap(wall);
    al_destroy_bitmap(point);
    al_destroy_bitmap(pacman_sprite);
    al_destroy_bitmap(ghost_sprite);
    if(font)
        al_destroy_font(font);
    al_destroy_timer(timer);
    al_destroy_display(display);
    al_destroy_event_queue(queue);

    return 0;
}
