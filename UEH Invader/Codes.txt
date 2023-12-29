import pygame
from pygame import mixer
from pygame.locals import *
import random

#Intialize
pygame.init()
mixer.init()

#Screen
screen_width = 800
screen_height = 600
screen = pygame.display.set_mode((screen_width,screen_height))

#Captain and Icon
title = pygame.display.set_caption("UEH Invaders")
icon = pygame.image.load('spaceship.png')
pygame.display.set_icon(icon)

#Background
background = pygame.image.load('Terrain/background.png')
background_menu = pygame.image.load('Terrain/menu_background.png')
logo = pygame.image.load('Assets/uehlogo.png')

#Sounds
laser_sound = pygame.mixer.Sound('Sounds/laser.wav')
laser_sound.set_volume(0.1)

background_music = pygame.mixer.Sound('Sounds/background.mp3')
background_music.set_volume(0.25)

#Player
player = pygame.image.load('Assets/player.png')

#Enemies
AIchip = pygame.image.load('Assets/AIchip.png')
AIchip = pygame.transform.scale(AIchip, (50, 50))

evileye = pygame.image.load('Assets/evileye.png')
evileye = pygame.transform.scale(evileye, (60, 60))

bluewing = pygame.image.load('Assets/bluewing.png')
bluewing = pygame.transform.scale(bluewing, (90, 90))

darkspace = pygame.image.load('Assets/darkspace.png')
darkspace = pygame.transform.scale(darkspace, (120, 120))

silverarmor = pygame.image.load('Assets/silverarmor.png')
silverarmor = pygame.transform.scale(silverarmor, (70, 70))

toxicrobot = pygame.image.load('Assets/toxicrobot.png')
toxicrobot = pygame.transform.scale(toxicrobot, (50, 50))

pinkrobot = pygame.image.load('Assets/pinkrobot.png')
pinkrobot = pygame.transform.scale(pinkrobot, (70, 70))
#Lasers
player_laser = pygame.image.load('Assets/laser3.png')
AIchiplaser = pygame.image.load('Assets/AIchiplaser.png')
evileyelaser = pygame.image.load('Assets/evileyelaser.png')
evileyelaser = pygame.transform.scale(evileyelaser, (65, 60))
bluewingbullet = pygame.image.load('Assets/bluewingbullet.png')
bluewingbullet = pygame.transform.scale(bluewingbullet, (100, 90))
darkspacelaser = pygame.image.load('Assets/darkspacelaser.png')
silverarmorlaser = pygame.image.load('Assets/silverarmorlaser.png')
silverarmorlaser = pygame.transform.scale(silverarmorlaser, (70, 140))
toxicrobotlaser = pygame.image.load('Assets/toxicrobotlaser.png')
toxicrobotlaser = pygame.transform.scale(toxicrobotlaser, (60, 50))
pinkrobotbullet = pygame.image.load('Assets/pinkrobotbullet.png')
#Collide
def collide(obj1, obj2):
    offset_x = obj2.x - obj1.x
    offset_y = obj2.y - obj1.y
    return obj1.mask.overlap(obj2.mask, (offset_x, offset_y)) != None


class Bullet:
    def __init__(self, x, y, img):
        self.x = x + 15
        self.y = y
        self.img = img
        self.mask = pygame.mask.from_surface(self.img)

    def draw(self, window):
        window.blit(self.img, (self.x, self.y))

    def move(self, vel):
        self.y += vel

    def off_screen(self, height):
        return not(self.y <= height and self.y >= 0)

    def collision(self, obj):
        return collide(self, obj)


#Ship
class Ship:
    COOLDOWN = 15
    def __init__(self, x, y, health = 100):
        self.x = x
        self.y = y
        self.health = health
        self.ship_img = None
        self.bullet_img = None
        self.bullets = []
        self.cool_down_counter = 0

    def draw(self, window):
        window.blit(self.ship_img, (self.x, self.y))
        for bullet in self.bullets:
            bullet.draw(window)
    def cooldown(self):
        if self.cool_down_counter >= self.COOLDOWN:
            self.cool_down_counter = 0
        elif self.cool_down_counter > 0:
            self.cool_down_counter += 1
    def move_bullets(self, vel, obj):
        self.cooldown()
        for bullet in self.bullets:
            bullet.move(vel)
            if bullet.off_screen(screen_height):
                self.bullets.remove(bullet)
            elif bullet.collision(obj):
                obj.health -= 10
                self.bullets.remove(bullet)
    def shoot(self):
        if self.cool_down_counter == 0:
            bullet = Bullet(self.x-10, self.y-30, self.bullet_img)
            self.bullets.append(bullet)
            self.cool_down_counter = 1
            laser_sound.play()

    def get_width(self):
        return self.ship_img.get_width()

    def get_height(self):
        return self.ship_img.get_height()

class Player(Ship):
    def __init__(self, x, y, health=100):
        super().__init__(x, y, health)
        self.ship_img = player
        self.bullet_img = player_laser
        self.mask =pygame.mask.from_surface(self.ship_img)
        self.max_health = health

    def move_bullets(self, vel, objs):
        self.cooldown()
        for bullet in self.bullets:
            bullet.move(vel)
            if bullet.off_screen(screen_height):
                self.bullets.remove(bullet)
            else:
                for obj in objs:
                    if bullet.collision(obj):
                        objs.remove(obj)
                        if bullet in self.bullets:
                            self.bullets.remove(bullet)
    def healthbar(self, window):
        pygame.draw.rect(window, (255,0,0), (self.x, self.y + self.ship_img.get_height() + 10, self.ship_img.get_width(), 10))
        pygame.draw.rect(window, (0,255,0), (self.x, self.y + self.ship_img.get_height() + 10, self.ship_img.get_width() * (self.health/self.max_health), 10))

    def draw(self, window):
        super().draw(window)
        self.healthbar(window)
    
#Enemies
class Enemy(Ship):
    MOBS = {
                "Enemy1": (AIchip, AIchiplaser),
                "Enemy2": (evileye, evileyelaser),
                "Enemy3": (bluewing, bluewingbullet),
                "Enemy4": (darkspace, darkspacelaser),
                "Enemy5": (silverarmor, silverarmorlaser),
                "Enemy6": (toxicrobot, toxicrobotlaser),
                "Enemy7": (pinkrobot, pinkrobotbullet),
                }
    def __init__(self, x, y, enemy, health = 100):
        super().__init__(x, y, health)
        self.ship_img, self.bullet_img = self.MOBS[enemy]
        self.mask = pygame.mask.from_surface(self.ship_img)

    def move(self, vel):
        self.y += vel
    
    def shoot(self):
        if self.cool_down_counter == 0:
            bullet = Bullet(self.x-20, self.y, self.bullet_img)
            self.bullets.append(bullet)
            self.cool_down_counter = 1


def main():

    run = True
    fps = 60
    level = 0
    lives = 3
    main_font = pygame.font.SysFont("comicsans", 30)
    lost_font = pygame.font.SysFont("04B_19", 40)
    enemies = []
    wave_length = 1
    enemy_vel = 1

    player_vel = 5
    bullet_vel = 4

    player = Player(350, 500)

    clock = pygame.time.Clock()

    lost = False
    lost_count = 0

    background_music.play(-1)
    button_toggle = True
    music_button = pygame.Rect(740, 50, 100, 50)

    def redraw_window():
        #Screen blit
        screen.blit(background, (0,0))
        screen.blit(logo, (10,-30))

        #Texts
        lives_label = main_font.render(f"Lives: {lives}", 1, (255,255,255))
        level_label = main_font.render(f"Level: {level}", 1, (255,255,255))

        screen.blit(lives_label, (10, 90))
        screen.blit(level_label, (680, 4))

        #Draw the button
        music_button = pygame.image.load('Assets/mutemusic.png')
        screen.blit(music_button , (740 , 50))
        laser_button = pygame.image.load('Assets/mutelaser.png')
        screen.blit(laser_button , (740 , 120))
        for enemy in enemies:
            enemy.draw(screen)

        player.draw(screen)

        if lost:
            lost_label = lost_font.render("GAME OVER!!!", 1, (255,255,255))
            screen.blit(lost_label, (screen_width/2 - lost_label.get_width()/2, 350))

        pygame.display.update()
    while run:
        clock.tick(fps)
        redraw_window()
        if lives <= 0 or player.health <= 0:
            lost = True
            lost_count += 1
        if lost:
            if lost_count > fps * 3:
                run = False
            else:
                continue
        
        if len(enemies) == 0:
            level += 1
            wave_length += 5
            for i in range(wave_length):
                enemy = Enemy(random.randrange(50, screen_width-300), random.randrange(-1000, -100), random.choice(["Enemy1", "Enemy2", "Enemy3" , "Enemy4" , "Enemy5" , "Enemy6" , "Enemy7"]))
                enemies.append(enemy)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                quit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                if music_button.collidepoint(event.pos):
                    if button_toggle:
                        background_music.stop()
                    else:
                        background_music.play(-1)
                    button_toggle = not button_toggle

        #Player Movements
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and player.x - player_vel > 0:
            player.x -= player_vel
        if keys[pygame.K_RIGHT] and player.x + player_vel + player.get_width() < screen_width:
            player.x += player_vel
        if keys[pygame.K_UP] and player.y - player_vel > 0:
            player.y -= player_vel
        if keys[pygame.K_DOWN] and player.y + player_vel + player.get_height() + 15 < screen_height:
            player.y += player_vel
        if keys[pygame.K_SPACE]:
            player.shoot()

        for enemy in enemies[:]:
            enemy.move(enemy_vel)
            enemy.move_bullets(bullet_vel, player)

            if random.randrange(0, 2*60) == 1:
                enemy.shoot()

            if collide(enemy, player):
                player.health -= 10
                enemies.remove(enemy)
            elif enemy.y + enemy.get_height() > screen_height:
                lives -= 1
                enemies.remove(enemy)

        player.move_bullets(-bullet_vel, enemies)

#Game Loop
def main_menu():
    title_font = pygame.font.SysFont("04B_19.ttf", 60)
    run = True
    blink_timer = 0
    background_music.play(-1)
    while run:
        screen.blit(background_menu, (0,0))
        #Flashing Texts
        if blink_timer < 30:
            title_label = title_font.render("Click to start...", 1, (255,255,255))
            screen.blit(title_label, (screen_width/2 - title_label.get_width()/2, 490))
        elif blink_timer < 60:
            pass
        else: blink_timer = 0
        blink_timer += 1

        pygame.display.update()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
            if event.type == pygame.MOUSEBUTTONDOWN:
                background_music.stop()
                main()
    
    pygame.quit()


main_menu()