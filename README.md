# -
"Проста гра 'Лабіринт', створена на Python"
from pygame import *

mixer.init()
mixer.music.load("back.mp3")
mixer.music.play(-1)
mixer.music.set_volume(0.1)
boom = mixer.Sound("gun.wav")
lose = mixer.Sound("gun.wav")





#клас-батько для інших спрайтів
class GameSprite(sprite.Sprite):
    #конструктор класу
    def __init__(self, player_image, player_x, player_y, size_x, size_y):
        # Викликаємо конструктор класу (Sprite):
        sprite.Sprite.__init__(self)
    
        #кожен спрайт повинен зберігати властивість image - зображення
        self.image = transform.scale(image.load(player_image), (size_x, size_y))

        #кожен спрайт повинен зберігати властивість rect - прямокутник, в який він вписаний
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
 
    #метод, що малює героя на вікні
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

#клас головного гравця
class Player(GameSprite):
    #метод, у якому реалізовано управління спрайтом за кнопками стрілочкам клавіатури
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_x_speed,player_y_speed):
        # Викликаємо конструктор класу (Sprite):
        GameSprite.__init__(self, player_image, player_x, player_y, size_x, size_y)

        self.x_speed = player_x_speed
        self.y_speed = player_y_speed
    ''' переміщає персонажа, застосовуючи поточну горизонтальну та вертикальну швидкість'''
    def update(self):  
        # Спершу рух по горизонталі
        if packman.rect.x <= win_width-80 and packman.x_speed > 0 or packman.rect.x >= 0 and packman.x_speed < 0:
            self.rect.x += self.x_speed
        # якщо зайшли за стінку, то встанемо впритул до стіни
        platforms_touched = sprite.spritecollide(self, barriers, False)
        if self.x_speed > 0: # йдемо праворуч, правий край персонажа - впритул до лівого краю стіни
            for p in platforms_touched:
                self.rect.right = min(self.rect.right, p.rect.left) # якщо торкнулися відразу кількох, то правий край - мінімальний із можливих
        elif self.x_speed < 0: # йдемо ліворуч, ставимо лівий край персонажа впритул до правого краю стіни
            for p in platforms_touched:
                self.rect.left = max(self.rect.left, p.rect.right) # якщо торкнулися кількох стін, то лівий край - максимальний
        if packman.rect.y <= win_height-80 and packman.y_speed > 0 or packman.rect.y >= 0 and packman.y_speed < 0:
            self.rect.y += self.y_speed
        # якщо зайшли за стінку, то встанемо впритул до стіни
        platforms_touched = sprite.spritecollide(self, barriers, False)
        if self.y_speed > 0: # йдемо вниз
            for p in platforms_touched:
                # Перевіряємо, яка з платформ знизу найвища, вирівнюємося по ній, запам'ятовуємо її як свою опору:
                if p.rect.top < self.rect.bottom:
                    self.rect.bottom = p.rect.top
        elif self.y_speed < 0: # йдемо вгору
            for p in platforms_touched:
                self.rect.top = max(self.rect.top, p.rect.bottom) # вирівнюємо верхній край по нижніх краях стінок, на які наїхали
    # метод "постріл" (використовуємо місце гравця, щоб створити там кулю)
    def fire(self):
        boom.play()
        bullet = Bullet('bullet.png', self.rect.centerx+3, self.rect.centery -10, 15, 20, 15)
        bullets.add(bullet)

#клас спрайту-ворога
class Enemy_h(GameSprite):
    side = "left"
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed, x1, x2):
        # Викликаємо конструктор класу (Sprite):
        GameSprite.__init__(self, player_image, player_x, player_y, size_x, size_y)
        self.speed = player_speed
        self.x1 =x1
        self.x2 =x2

   #рух ворога
    def update(self):
        if self.rect.x <= self.x1: 
            self.side = "right"
        if self.rect.x >= self.x2:
            self.side = "left"
        if self.side == "left":
            self.rect.x -= self.speed
        else:
            self.rect.x += self.speed

class Enemy_v(GameSprite):
    side = "up"
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed, y1, y2):
        # Викликаємо конструктор класу (Sprite):
        GameSprite.__init__(self, player_image, player_x, player_y, size_x, size_y)
        self.speed = player_speed
        self.y1 =y1
        self.y2 =y2

   #рух ворога
    def update(self):
        if self.rect.y <= self.y1: #w1.wall_x + w1.wall_width
            self.side = "down"
        if self.rect.y >= self.y2:
            self.side = "up"
        if self.side == "up":
            self.rect.y -= self.speed
        else:
            self.rect.y += self.speed

# клас спрайту-кулі
class Bullet(GameSprite):
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        # Викликаємо конструктор класу (Sprite):
        GameSprite.__init__(self, player_image, player_x, player_y, size_x, size_y)
        self.speed = player_speed
    #рух ворога
    def update(self):
        self.rect.x += self.speed
        # зникає, якщо дійде до краю екрана
        if self.rect.x > win_width+10:
            self.kill()

#Створюємо віконце
win_width = 1920
win_height = 1080
window = display.set_mode((win_width, win_height))
display.set_caption("Лабіринт")
back = transform.scale(image.load("jungle.jpg"), (win_width, win_height))

#Створюємо групу для стін
barriers = sprite.Group()

#створюємо групу для куль
bullets = sprite.Group()

#Створюємо групу для монстрів
monsters = sprite.Group()






#додаємо стіни до групи
barriers.add(GameSprite('platform2.png',win_width/2 - win_width/3, win_height/2, 600, 10))
barriers.add(GameSprite('platform2_v.png', 70, 750, 10, 300))
barriers.add( GameSprite('platform2_v.png', 70, 350, 10, 300))
barriers.add(GameSprite('platform2_v.png', 70, 10, 10, 200))
barriers.add(GameSprite('platform2_v.png', 320, 350, 10, 300))
barriers.add(GameSprite('platform2.png',win_width/5 - win_width/6, win_height/2, 380, 10))
barriers.add(GameSprite('platform2_v.png', 520, 540, 10, 300))
barriers.add(GameSprite('platform2_v.png', 320, 540, 10, 300))
barriers.add(GameSprite('platform2.png',win_width/5 - win_width/6, win_height/10, 200, 10))
barriers.add(GameSprite('platform2_v.png', 320, 540, 10, 300))
barriers.add(GameSprite('platform2.png',win_width/5 - win_width/6, win_height/1, 200, 10))
barriers.add(GameSprite('platform2_v.png', 320, 140, 10, 300))
barriers.add(GameSprite('platform2_v.png', 520, 10, 10, 460))
barriers.add(GameSprite('platform2_v.png', 720, 170, 10, 370))
barriers.add(GameSprite('platform2_v.png', 1320, 170, 10, 300))
barriers.add(GameSprite('platform2_v.png', 1220, 170, 10, 400))
barriers.add(GameSprite('platform2_v.png', 1820, 540, 10, 300))
barriers.add(GameSprite('platform2_v.png', 1620, 840, 10, 200))
barriers.add(GameSprite('platform2.png',win_width/1 - win_width/4, win_height/2, 390, 10))
barriers.add(GameSprite('platform2_v.png', 720, 170, 500, 10))
barriers.add(GameSprite('platform2_v.png', 520, 10, 500, 10))
barriers.add(GameSprite('platform2_v.png', 820, 10, 500, 10))
barriers.add(GameSprite('platform2_v.png', 1320, 10, 500, 10))
barriers.add(GameSprite('platform2_v.png', 1520, 10, 500, 10))
barriers.add(GameSprite('platform2_v.png', 1320, 170, 600, 10))
barriers.add(GameSprite('platform2_v.png', 1620, 840, 210, 10))
barriers.add(GameSprite('platform2_v.png', 1720, 940, 200, 10))
barriers.add(GameSprite('platform2_v.png', 1220, 540, 250, 10))
barriers.add(GameSprite('platform2_v.png', 1320, 470, 400, 10))
barriers.add(GameSprite('platform2_v.png', 1420, 390, 300, 10))
barriers.add(GameSprite('platform2_v.png', 1420, 290, 10, 100))
barriers.add(GameSprite('platform2_v.png', 1420, 290, 400, 10))
barriers.add(GameSprite('platform2_v.png', 1810, 180, 10, 120))
barriers.add(GameSprite('platform2_v.png', 720, 670, 10, 400))
barriers.add(GameSprite('platform2_v.png', 720, 670, 360, 10))
barriers.add(GameSprite('platform2_v.png', 1070, 270, 10, 400))
barriers.add(GameSprite('platform2_v.png', 920, 270, 10, 280))
barriers.add(GameSprite('platform2_v.png', 1070, 670, 300, 10))
barriers.add(GameSprite('platform2_v.png', 1070, 750, 300, 10))
barriers.add(GameSprite('platform2_v.png', 860, 750, 300, 10))
barriers.add(GameSprite('platform2_v.png', 860, 750, 10, 400))
barriers.add(GameSprite('platform2_v.png', 860, 860, 300, 10))
barriers.add(GameSprite('platform2_v.png', 1070, 860, 300, 10))
barriers.add(GameSprite('platform2_v.png', 1520, 670, 10, 400))
barriers.add(GameSprite('platform2_v.png', 1520, 670, 200, 10))
barriers.add(GameSprite('platform2_v.png', 1620, 760, 200, 10))









bonus = sprite.Group()
bonus.add(GameSprite("bonus.jpg", win_width - 180, win_height - 300, 50, 50))
bonus.add(GameSprite("bonus.jpg", 85, 20, 50, 50))
bonus.add(GameSprite("bonus.jpg", 1750, 200, 50, 50))
bonus.add(GameSprite("bonus.jpg", 800, 1000, 50, 50))
bonus.add(GameSprite("bonus.jpg", 1560, 1000, 50, 50))
bonus.add(GameSprite("bonus.jpg", 900, 810, 50, 50))
bonus.add(GameSprite("bonus.jpg", 800, 490, 50, 50))

num = 0



#створюємо спрайти
packman = Player('hero.png', 5, win_height - 80, 40, 40, 0, 0)

monster1 = Enemy_v('cyborgv3.png', win_width - 200, 200, 80, 80, 10, 300, 465)

monster2 = Enemy_h('cyborgv2.png', win_width - 80, 90, 80, 80, 20, 550, win_width-85)

monster3 = Enemy_h('cyborg.png', 0, 240, 80, 80, 10, 0, 250)

monster4 = Enemy_h('cyborgv2.png',300, 900, 80, 80, 10, 80, 640)

monster5 = Enemy_h('cyborg.png',700, 180, 80, 80, 10, 740, 1130)

monster6 = Enemy_h('cyborgv2.png',1030, 590, 80, 80, 10, 1100, 1740)

monster7 = Enemy_h('cyborg.png', 720,680, 70, 70, 10, 730, 1250)

monster8 = Enemy_h('cyborgv3.png', 340, 470, 70, 70, 15, 340, 650)

monster9 = Enemy_h('cyborgv3.png', 140, 30, 70, 70, 15, 141, 350)

final_sprite = GameSprite('pac-1.png', win_width - 85, win_height - 100, 85, 85)





#додаємо монстра до групи
monsters.add(monster1)
monsters.add(monster2)
monsters.add(monster3)
monsters.add(monster4)
monsters.add(monster5)
monsters.add(monster6)
monsters.add(monster7)
monsters.add(monster8)
monsters.add(monster9)
#змінна, що відповідає за те, як закінчилася гра
finish = False
#ігровий цикл
run = True
while run:

    #цикл спрацьовує кожну 0.05 секунд
    time.delay(50)
        #перебираємо всі події, які могли статися
    for e in event.get():
        if e.type == QUIT:
            run = False
        elif e.type == KEYDOWN:
            if e.key == K_LEFT:
                packman.x_speed = -10
            elif e.key == K_RIGHT:
                packman.x_speed = 10
            elif e.key == K_UP:
                packman.y_speed = -10
            elif e.key == K_DOWN:
                packman.y_speed = 10
            elif e.key == K_SPACE:
                packman.fire()


        elif e.type == KEYUP:
            if e.key == K_LEFT:
                packman.x_speed = 0
            elif e.key == K_RIGHT:
                packman.x_speed = 0 
            elif e.key == K_UP:
                packman.y_speed = 0
            elif e.key == K_DOWN:
                packman.y_speed = 0

#перевірка, що гра ще не завершена
    if not finish:
        #оновлюємо фон кожну ітерацію
        window.blit(back, (0, 0))#зафарбовуємо вікно кольором
        
        #запускаємо рухи спрайтів
        packman.update()
        bullets.update()

        #оновлюємо їх у новому місці при кожній ітерації циклу
        packman.reset()
        #рисуємо стіни 2
        bullets.draw(window)
        barriers.draw(window)
        final_sprite.reset()

        bonus.draw(window)
        if sprite.spritecollide(packman, bonus, True):
            num += 1

        sprite.groupcollide(monsters, bullets, True, True)
        monsters.update()
        monsters.draw(window)
        sprite.groupcollide(bullets, barriers, True, False)
        

        
        if sprite.spritecollide(packman, bonus, True):
            num +- 1


        #Перевірка зіткнення героя з ворогом та стінами
        if sprite.spritecollide(packman, monsters, False):
            finish = True
            mixer.music.stop()
            lose.play()
            img = image.load('game-over_1.png')
            window.blit(transform.scale(img, (win_width, win_height)), (0, 0))

        if sprite.collide_rect(packman, final_sprite ):
            finish = True
            img = image.load('thumb.jpg')
            window.blit(transform.scale(img, (win_width, win_height)), (0, 0))
    
    display.update()
