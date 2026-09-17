Suivi de souris avec retard réglable

import pygame
import time

pygame.init()
screen = pygame.display.set_mode((900, 600))
clock = pygame.time.Clock()
delay_ms = 0
history = []

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_UP:
                delay_ms = min(200, delay_ms + 10)
            elif event.key == pygame.K_DOWN:
                delay_ms = max(0, delay_ms - 10)

    mx, my = pygame.mouse.get_pos()
    history.append((time.perf_counter(), mx, my))
    target = time.perf_counter() - delay_ms / 1000

    position = history[0][1:]
    for timestamp, x, y in reversed(history):
        if timestamp <= target:
            position = (x, y)
            break

    screen.fill((30, 30, 30))
    pygame.draw.circle(screen, (255, 255, 255), position, 15)
    pygame.display.flip()
    clock.tick(120)

pygame.quit()

Seuils des cinq participants

Participant

Seuil détecté

1

À mesurer

2

À mesurer

3

À mesurer

4

À mesurer

5

À mesurer

Comparaison

Le budget VR est d’environ 20 ms pour l’ensemble de la chaîne. Le seuil est plus bas dans un casque parce que le retard perturbe directement la correspondance entre le mouvement réel de la tête et le mouvement visuel perçu. Cette incohérence entre les informations visuelles et vestibulaires est particulièrement sensible en réalité virtuelle.