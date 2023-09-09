# First-game-on-python
Моя 2д игра созданная с помощью python на основе лабиринта 

# подключение pygame, random и time
from pygame import*
from random import*
from time import time as timer

# тут можно поставить музыку для игры
mixer.init()
#mixer.music.load('')
#mixer.music.play()
#boom = mixer.sound('')

# загрузка изображений персонажей
img_back = 'back.jpg'
img_player = 'down1.png'
img_chest = 'gold.png'
img_win = 'win.png'
img_gun_bullet = 'gun_bullet.png' 
img_player_bullet = 'player_bullet.png'
img_eye = 'eye.png'
img_monster = 'monster.png'
img_lose = 'lose.png'
img_ghost = 'ghost.png'

# создание листов и загрузка в них изображения 
# animation
walk_right= [
    image.load('right1.png'),
    image.load('right2.png'),
    image.load('right3.png'),
    image.load('right4.png')
]
walk_left = [
    image.load('left1.png'),
    image.load('left2.png'),
    image.load('left3.png'),
    image.load('left4.png')
]
walk_up = [
    image.load('up1.png'),
    image.load('up2.png'),
    image.load('up3.png'),
    image.load('up4.png')
]
walk_down = [
    image.load('down1.png'),
    image.load('down2.png'),
    image.load('down3.png'),
    image.load('down4.png')
]
ghost_left = [
    image.load('ghost_left1.png'),
    image.load('ghost_left2.png'),
    image.load('ghost_left1.png'),
    image.load('ghost_left2.png')
]
ghost_right = [
    image.load('ghost_right1.png'),
    image.load('ghost_right2.png'),
    image.load('ghost_right1.png'),
    image.load('ghost_right2.png')
]

# создание классов наследников sprite.Sprite
class GameSprite(sprite.Sprite):
    def __init__(self,player_image, player_x, player_y, width, height, player_speed):
        sprite.Sprite.__init__(self)
        self.image = transform.scale(image.load(player_image), (width, height))
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
        self.left = False
        self.right = False
        self.up = False
        self.down = False
        self.count = 0
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))
# создание функции путь. Она задает движение всем обьектам
    def away(self):
        if self.side == "left":
            self.rect.x -= self.speed
        elif self.side == "right":
            self.rect.x += self.speed
        elif self.side == "up":
            self.rect.y -= self.speed
        elif self.side == "down":
            self.rect.y += self.speed

# класс стена создает на плоскости стену по указаным координатам, ширине и высоте, а также красит её            
class Wall(sprite.Sprite):
    def __init__(self, red, green, blue, wall_x, wall_y, wall_width, wall_height):
        super().__init__()
        self.red = red
        self.green = green
        self.blue = blue
        self.w = wall_width
        self.h = wall_height
        self.image = Surface((self.w, self.h))
        self.image.fill((red,green,blue))
        self.rect = self.image.get_rect()
        self.rect.x = wall_x
        self.rect.y = wall_y
    def draw_wall(self):
        window.blit(self.image, (self.rect.x, self.rect.y))
# создание классов врагов и один из них имеет функцию animation(но на данный момент она не доработана)
class Enemy(GameSprite):
    side = "left"
    def update(self):
        if self.rect.x >= 900:
            self.side = "left"
        if self.rect.x <= 300:
            self.side = "right"
        self.away()
    def animation(self):
        if self.count +1 >= FPS:
            self.count = 0
        if self.side == "right":
            window.blit(ghost_left[self.count // 5], (self.rect.x, self.rect.y))
            self.count += 1
        elif self.side == "left":
            window.blit(ghost_right[self.count // 5], (self.rect.x, self.rect.y))
            self.count += 1
        else:
            window.blit(self.image, (self.rect.x, self.rect.y))
     

class Enemy2(GameSprite):
    side = 'right'
    def update(self):
        if self.rect.x >= 900:
            self.side = "left"
        if self.rect.x <= 150:
            self.side = "right"
        self.away()

class Enemy3(GameSprite):
    side = "right"
    def update(self):
        if self.rect.x >= 750:
            self.side = "left"
        if self.rect.x <= 300:
            self.side = "right"
        self.away()

# создание класса игрок 
class Player(GameSprite): 
# функция update отвечает за его движение по нажатию клавиш
    def update(self):
        keys = key.get_pressed()
        if keys[K_a] and self.rect.x >= 10:
            self.rect.x -= self.speed
            self.left = True
            self.right = False
            self.up = False
            self.down = False
        elif keys[K_s]:
            self.rect.y += self.speed
            self.left = False
            self.right = False
            self.up = False
            self.down = True
        elif keys[K_w]:
            self.rect.y -= self.speed
            self.left = False
            self.right = False
            self.up = True
            self.down = False
        elif keys[K_d] and self.rect.x <= win_width - 50:
            self.rect.x += self.speed
            self.left = False
            self.right = True
            self.up = False
            self.down = False
        else:
            self.left = False
            self.right = False
            self.up = False
            self.down = False
            self.count = 0
# эти две функции отвечают за огонь в правую и левую сторону

    def leftfire(self):
        player_bullet = BulletLeft(img_player_bullet, self.rect.right, self.rect.centery, 25,25,7)
        bullets.add(player_bullet)
    def rightfire(self):
        player_bullet = BulletRight(img_gun_bullet, self.rect.left, self.rect.centery, 25,25,7)
        bullets.add(player_bullet)
#        teleport(self)
    def animation(self):
        if self.count +1 >= FPS:
            self.count = 0
        if self.left == True:
            window.blit(walk_left[self.count // 5], (self.rect.x, self.rect.y))
            self.count += 1
        elif self.right == True:
            window.blit(walk_right[self.count // 5], (self.rect.x, self.rect.y))
            self.count += 1
        elif self.up == True:
            window.blit(walk_up[self.count // 5], (self.rect.x, self.rect.y))
            self.count += 1
        elif self.down == True:
            window.blit(walk_down[self.count // 5], (self.rect.x, self.rect.y))
            self.count += 1
        else:
            window.blit(self.image, (self.rect.x, self.rect.y))

# два класса пули в них прописано что если пуля выйдет за пределлы экрана она пропадет
class BulletRight(GameSprite):
    def update(self):
        self.rect.x -= self.speed
        if self.rect.x < 0:
            self.kill()

class BulletLeft(GameSprite):
    def update(self):
        self.rect.x += self.speed
        if self.rect.x > win_width + 10:
            self.kill()

# эта функция должна отвечать за телепортацию персонажа если он доходит до края карты, но она тоже некоректно работает
def teleport(self):
    if self.rect.y <= 20:
        self.rect.y = 450
    if self.rect.y >= 480:
        self.rect.y = 0 

# создание окна и загрузка заднего фона 
back = image.load(img_back)
win_width = 1000
win_height = 500
display.set_caption("Game")
window = display.set_mode((win_width, win_height))
back = transform.scale(image.load(img_back), (win_width, win_height))
# 2 финальных изображения одио выйгрышное (если игрок дойдет до сундука), второе (если игрок коснется стен или врагов)
win = transform.scale(image.load(img_win), (win_width,win_height))
lose = transform.scale(image.load(img_lose), (win_width,win_height))

# Создание переменных и присваивание им их картинки с координатами, шириной и высотой
player = Player(img_player, 10, 420,80,80,5)
ghost = Enemy(img_ghost, 950,160,100,100,3)
eye = Enemy2(img_eye, 950,400,100,100,5)
monster = Enemy3(img_monster,260,0,100,100,5)
chest = GameSprite(img_chest,950,0,50,50,0)

# Создание стен
w1 = Wall(255, 165, 0,250,0,20,280)
w2 = Wall(255, 165, 0,100,200,20,300)
w3 = Wall(255, 165, 0,650,280,350,20)
w4 = Wall(255, 165, 0,800,0,20,120)
w5 = Wall(255,165,0, 250,280,200,20)

# Создание групп для стен,пуль и врагов
walls = sprite.Group()
bullets = sprite.Group()
enemy = sprite.Group()

# Добавление всех в группы
enemy.add(ghost)
enemy.add(eye)
enemy.add(monster)
walls.add(w1)
walls.add(w2)
walls.add(w3)
walls.add(w4)
walls.add(w5)

# Создание игрового цикла пока game = True and finish = False
game = True
finish = False
clock = time.Clock()
FPS = 20
while game:
    for e in event.get():
        if e.type == QUIT:
            game = False
        elif e.type == KEYDOWN:
            if e.key == K_TAB:
                player.leftfire()
            elif e.key == K_SPACE:
                player.rightfire()

# Пока finish != True будут отрисовываться спрайты                
    if finish != True:
        window.blit(back, (0,0))
        chest.reset()
        enemy.draw(window)
        player.update()
        player.animation()
        ghost.animation()
        enemy.update()
        bullets.draw(window)
        bullets.update()
        walls.draw(window)
        sprite.groupcollide(bullets, walls, True, False)
        sprite.groupcollide(bullets, enemy, True, True)

        if sprite.collide_rect(player, chest):
            finish = True
            window.blit(win, (0,0))
        if sprite.spritecollide(player, walls, False):
            finish = True
            window.blit(lose, (0,0))
        if sprite.spritecollide(player, enemy, False):
            finish = True
            window.blit(lose, (0,0))

            
    display.update()
    clock.tick(FPS)

    
