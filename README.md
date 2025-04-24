# pingpong
from pygame import *
from random import choice

# вынесем размер окна в константы для удобства
# W - width, ширина
# H - height, высота
WIN_W = 900
WIN_H = 700
FPS = 60
WHITE = 255, 255, 255
size = 52
step = 5
direction = 'left'
speed = 2
x1 = 60
x2 = 100
GREEN = 0, 255, 0
RED = 255, 0, 0
YELLOW = 255, 195, 0
class GameSprite(sprite.Sprite):
    def __init__(self, img, x, y, w, h):
        super().__init__()
        self.image = transform.scale(
            image.load(img),
            # здесь - размеры картинки
            (w, h)
        )
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
    
    def draw(self, window):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def __init__(self, x, y, img, sizeX, sizeY, step):
        super().__init__(img, x, y, sizeX, sizeY)
        self.step = step
    def update(self, up, down):
        keys = key.get_pressed()
        if keys[up] and self.rect.y >= 0:
            self.rect.y -= self.step

        if keys[down] and self.rect.y <= WIN_H - self.rect.height:
            self.rect.y += self.step

class Ball(GameSprite):
    def __init__(self, x, y, img, sizeX, sizeY, speed):
        super().__init__(img, x, y, sizeX, sizeY)
        self.speed_x = speed * choice([-1.5, -1, 1, 1.5])
        self.speed_y = speed * choice([-1.5, -1, 1, 1.5])
    def update(self):
        if self.rect.x <= 0 or self.rect.x >= WIN_W - self.rect.width:
            self.speed_x *= -1
        self.rect.x += self.speed_x
        if self.rect.y <= 0 or self.rect.y >= WIN_H - self.rect.height:
            self.speed_y *= -1
        self.rect.y += self.speed_y
# создание окна размером 700 на 500
window = display.set_mode((WIN_W, WIN_H))
# создание таймера
clock = time.Clock()

# название окна
display.set_caption("Ping-pong")

mixer.init()
mixer.music.load('src/play.mp3')
mixer.music.play(-1)
mixer.music.set_volume(0.1)

font.init()
title_font = font.SysFont('Comic Sans', 70)
player_1_win = title_font.render('Игрок 1 выиграл!', True, GREEN)
player_2_win = title_font.render('Игрок 2 выиграл!', True, GREEN)

player_1 = Player(50, 250, 'src/player.png', 20, 100, step)
player_2 = Player(850, 250, 'src/player.png', 20, 100, step)
ball = Ball(350, 150, 'src/ball.png', 20, 20, step)

# задать картинку фона такого же размера, как размер окна
background = transform.scale(
    image.load("src/fon.jpg"),
    # здесь - размеры картинки
    (WIN_W, WIN_H)
)
finish = False
# игровой цикл
game = True
while game:
    if not finish:
        # отобразить картинку фона
        window.blit(background,(0, 0))
        player_1.draw(window)
        player_1.update(K_w, K_s)
        player_2.draw(window)
        player_2.update(K_UP, K_DOWN)
        ball.draw(window)
        ball.update()
        if sprite.collide_rect(ball, player_1):
            ball.speed_x *= -1
        if sprite.collide_rect(ball, player_2):
            ball.speed_x *= -1
        if ball.rect.x <= 0:
            window.blit(player_2_win, (50, 50))
            display.update()
            finish = True
        if ball.rect.x >= WIN_W - ball.rect.width:
            window.blit(player_1_win, (50, 50))
            display.update()
            finish = True
    else:
        time.delay(2000)
        player_1 = Player(50, 100, 'src/player.png', 20, 100, step)
        player_2 = Player(850, 100, 'src/player.png', 20, 100, step)
        ball = Ball(450, 350, 'src/ball.png', 20, 20, step)
        finish = False
    # слушать события и обрабатывать
    for e in event.get():
        # выйти, если нажат "крестик"
        if e.type == QUIT:
            game = False
    # обновить экран, чтобы отобрзить все изменения
    display.update()
    clock.tick(FPS)
