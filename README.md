# shooting_main1
import pygame
import random

from pygame import draw
from pygame.color import Color
from pygame.sprite import Sprite
from pygame.surface import Surface

pygame.init()

# 게임 화면 설정
image_list = ['backgroundse.png','backgroundse1.png']
size = (400, 500)
screen = pygame.display.set_mode(size)
pygame.display.set_caption("슈팅 게임")

#배경화면 클래스
class Background(Sprite):
    def __init__(self):
        self.sprite_image = image_list[0]
        self.image = pygame.image.load(self.sprite_image).convert()
        self.rect = self.image.get_rect()
        self.rect.y = 0
        self.dy = 1
        self.count = 0
        
        Sprite.__init__(self)

    def update(self):
        self.rect.y -= self.dy
        if self.rect.y == -400:
            self.rect.y = 0
            self.count += 1
            
        if self.count == 5:
            self.sprite_image = image_list[1]
            self.count = 0

# 플레이어 클래스
class Player(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.sprite_image = 'players.png'
        self.sprite_width = 45
        self.sprite_height = 40
        self.sprite_columns = 10
        self.fps = 32
        self.speed = 5
        self.score = 0
        self.image = Surface((self.sprite_width, self.sprite_height))
        self.rect = self.image.get_rect()

    def update(self):
        # 키 입력 처리
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.rect.x -= self.speed
        elif keys[pygame.K_RIGHT]:
            self.rect.x += self.speed
        if keys[pygame.K_UP]:
            self.rect.y -= self.speed
        elif keys[pygame.K_DOWN]:
            self.rect.y += self.speed
        
        # 화면을 벗어나지 않도록 위치 조정
        if self.rect.x < 0:
            self.rect.x = 0
        elif self.rect.x > 400:
            self.rect.x = 400
        if self.rect.y < 0:
            self.rect.y = 0
        elif self.rect.y > 500:
            self.rect.y = 500

# 적 클래스
class Enemy(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.sprite_image = 'enemys.png'
        self.sprite_width = 92
        self.sprite_height = 50
        self.sprite_columns = 10
        self.speed = random.randint(1, 5) #스피드 랜덤
        self.image = Surface((self.sprite_width, self.sprite_height))
        self.rect = self.image.get_rect()

    def update(self):
        self.rect.y += self.speed
        if self.rect.top > 500:
            self.rect.y = 0

# 총알 클래스
class Bullet(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.sprite_image = 'stone.png'
        self.sprite_width = 8
        self.sprite_height = 8
        self.sprite_columns = 4
        self.speed = -10
        self.image = Surface((self.sprite_width, self.sprite_height))
        self.rect = self.image.get_rect()

    def update(self):
        self.rect.y += self.speed
        if self.rect.bottom < 0:
            self.kill()

#이미지 그룹
background = Background()
background_group = pygame.sprite.Group()
background_group.add(background)

player = Player()
player_group = pygame.sprite.Group()
player_group.add(player)

bullet = Bullet()
bullet_group = pygame.sprite.Group()
bullet_group.add(bullet)

enemy = Enemy()
enemy_group = pygame.sprite.Group()
enemy_group.add(enemy)

#적 생성
for i in range(10):
    a = 0

# 게임 루프
running = True
clock = pygame.time.Clock()

while running:
    # 이벤트 처리
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                player.shoot()

    # 게임 상태 업데이트
    player_group.update()
    bullet_group.update()
    enemy_group.update()

    # 충돌 처리
    hits = pygame.sprite.groupcollide(enemy, bullet, True, True)
    for hit in hits:
        player.score += 10
        enemy = Enemy(enemy_group, random.randint(0, 500), random.randint(-500, -100))
        enemy_group.add(enemy)

    hits = pygame.sprite.groupcollide(player, enemy, False)
    
    if hits:
        running = False

    # 화면 그리기
    background_group.update()
    background_group.draw(screen)
    player_group.draw(screen)
    bullet_group.draw(screen)
    enemy_group.draw(screen)

    # 점수 출력
    font = pygame.font.Font(None, 30)
    score_text = font.render('Score: {}'.format(player.score), True, (0, 0, 0))
    screen.blit(score_text, (10, 10))

    # 화면 업데이트
    pygame.display.update()

    # 프레임 설정
    clock.tick(60)

pygame.quit()
