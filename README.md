import pygame
import math
import random
from pygame import mixer

# Initialize pygame
pygame.init()

# Create screen
screen = pygame.display.set_mode((800, 600))

# Background
background = pygame.image.load('spacebg.png')

# Sound
mixer.music.load("background.wav")
mixer.music.play(-1)

# Title and Icon
pygame.display.set_caption("Space Cat Invaders")
icon = pygame.image.load('kitty.png.png')
pygame.display.set_icon(icon)

# Player
playerImg = pygame.image.load('catgun.png')
playerX = 370
playerY = 480
playerX_change = 0

# Enemy
enemyImg = []
enemyX = []
enemyY = []
enemyX_change = []
enemyY_change = []
num_of_enemies = 6

for i in range(num_of_enemies):
    enemyImg.append(pygame.image.load('catenemy.png'))
    enemyX.append(random.randint(0, 736))
    enemyY.append(random.randint(50, 150))
    enemyX_change.append(4)
    enemyY_change.append(40)

# Bullet (List of bullets for multiple shots)
bulletImg = pygame.image.load('bullet.png')
bullets = []

# Score
score_value = 0
font = pygame.font.Font('freesansbold.ttf', 32)
textX = 10
textY = 10

# Game Over
over_font = pygame.font.Font('freesansbold.ttf', 64)

def show_score(x, y):
    score = font.render("Score : " + str(score_value), True, (255, 255, 255))
    screen.blit(score, (x, y))

def game_over_text():
    over_text = over_font.render("GAME OVER", True, (255, 255, 255))
    screen.blit(over_text, (200, 250))

def player(x, y):
    screen.blit(playerImg, (x, y))

def enemy(x, y, i):
    screen.blit(enemyImg[i], (x, y))

def fire_bullet(x, y):
    bullets.append([x, y])  # Add new bullet

def isCollision(enemyX, enemyY, bulletX, bulletY):
    distance = math.sqrt(math.pow(enemyX - bulletX, 2) + (math.pow(enemyY - bulletY, 2)))
    return distance < 27

def game_over():
    game_over_text()
    pygame.display.update()
    pygame.time.delay(2000)
    pygame.quit()
    quit()

def reset_enemy(i):
    enemyX[i] = random.randint(0, 736)
    enemyY[i] = random.randint(50, 150)

# Game Loop
running = True
while running:
    screen.fill((0, 0, 0))
    screen.blit(background, (0, 0))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        # Player movement
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                playerX_change = -5
            if event.key == pygame.K_RIGHT:
                playerX_change = 5
            if event.key == pygame.K_SPACE:
                bulletSound = mixer.Sound("laser.wav")
                bulletSound.play()
                fire_bullet(playerX + 16, playerY)

        if event.type == pygame.KEYUP:
            if event.key in (pygame.K_LEFT, pygame.K_RIGHT):
                playerX_change = 0

    # Update player position
    playerX += playerX_change
    playerX = max(0, min(playerX, 736))

    # Enemy movement
    for i in range(num_of_enemies):
        if enemyY[i] >= 440:
            game_over()
            running = False
            break

        enemyX[i] += enemyX_change[i]

        if enemyX[i] <= 0:
            enemyX_change[i] = 4
            enemyY[i] += enemyY_change[i]
        elif enemyX[i] >= 736:
            enemyX_change[i] = -4
            enemyY[i] += enemyY_change[i]

        # Collision detection
        for bullet in bullets[:]:
            if isCollision(enemyX[i], enemyY[i], bullet[0], bullet[1]):
                explosionSound = mixer.Sound("explosion.wav")
                explosionSound.play()
                score_value += 1
                bullets.remove(bullet)  # Remove bullet on hit
                reset_enemy(i)

        enemy(enemyX[i], enemyY[i], i)

    # Bullet movement
    for bullet in bullets[:]:
        bullet[1] -= 10
        if bullet[1] < 0:
            bullets.remove(bullet)  # Remove bullets that leave the screen
        else:
            screen.blit(bulletImg, (bullet[0], bullet[1]))  # Draw bullet

    # Draw player and score
    player(playerX, playerY)
    show_score(textX, textY)

    pygame.display.update()
