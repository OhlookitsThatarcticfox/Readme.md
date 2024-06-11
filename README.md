import pygame
from sys import exit
from random import randint, choice

class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.image.load('Graphics/chara.png').convert_alpha() 
        self.rect = self.image.get_rect(midbottom = (200,340))
        self.gravity = 0
        
        
    def player_input(self):
        keys = pygame.key.get_pressed()
        if keys [pygame.K_SPACE] and self.rect.bottom >= 340:
            self.gravity = -35
            
    
    def apply_gravity(self):
        self.gravity += 1
        self.rect.y += self.gravity
        if self.rect.bottom >= 340:
            self.rect.bottom = 340
            
    def update(self):
        self.player_input()
        self.apply_gravity()

class Obstacle(pygame.sprite.Sprite):
    def __init__(self,type):
        super().__init__()
        
        if type == 'fly':
            fly_surf = pygame.image.load('Graphics/sword.png').convert_alpha()
            self.frames = [fly_surf]
            y_pos = 210
        else:
            snail_surface = pygame.image.load('Graphics/Snail.png').convert_alpha()
            self.frames = [snail_surface]
            y_pos = 325
        
        self.animation_index = 0
        self.image = self.frames[self.animation_index]
        self.rect = self.image.get_rect(midbottom = (randint(900, 1100), y_pos))
        
        
    def update(self):
        self.rect.x -= 6
        self.destroy()
    
    def destroy(self):
        if self.rect.x <= -100:
            self.kill()
    
def display_score():
    current_time = int(pygame.time.get_ticks() / 1000) - start_time
    score_surf = test_font.render(("Score: "+ str(current_time)),False,(0,0,0))
    score_rect = score_surf.get_rect(center = (400,50))
    screen.blit(score_surf,score_rect)
    return current_time


def obstacle_movement(obstacle_list):
    if obstacle_list:
        for obstacle_rect in obstacle_list:
            obstacle_rect.x -= 5
            
            if obstacle_rect.bottom == 325:
                screen.blit(snail_surface,obstacle_rect)
            else:
                screen.blit(fly_surf,obstacle_rect)
                
        obstacle_list = [obstacle for obstacle in obstacle_list if obstacle.x > -100]
        
        return obstacle_list
    else:
        return []

def collision_sprite():
    if pygame.sprite.spritecollide(player.sprite, obstacle_group, False):
        obstacle_group.empty()
        return False
    else:
        return True 

pygame.init()
screen = pygame.display.set_mode((800,400))
pygame.display.set_caption('Snail')
clock = pygame.time.Clock()
test_font = pygame.font.Font('Graphics/Pixeltype.ttf', 50)
game_active = False
start_time = 0
score = 0

#Groups
player = pygame.sprite.GroupSingle()
player.add(Player())

obstacle_group = pygame.sprite.Group()

#BG
sky_surface = pygame.image.load('Graphics/Mountain.png').convert_alpha()
ground_surface = pygame.image.load('Graphics/Ground.png').convert_alpha()

obstacle_rect_list = []

#end screen
player_stand = pygame.image.load('Graphics/chara.png').convert_alpha()
player_stand = pygame.transform.rotozoom(player_stand,0,2)
player_stand_rect = player_stand.get_rect(center = (350,200))

game_name = test_font.render('Pixel Runner',False,(111,196,169))
game_name_rect = game_name.get_rect(center = (400,80))

game_message = test_font.render('Press space to run',False,(111,196,169))
game_message_rect = game_message.get_rect(center = (400,320))

# Timer
obstacle_timer = pygame.USEREVENT + 1
pygame.time.set_timer(obstacle_timer,1500)


while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            exit()
        
        if event.type == obstacle_timer and game_active:
            obstacle_group.add(Obstacle(choice(['fly', 'snail','snail', 'snail'])))
           
        else:
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                game_active = True
                
    
    if game_active:    
        screen.blit(sky_surface,(0,0))
        screen.blit(ground_surface,(0,300))
        score = display_score()
        
        player.draw(screen)
        player.update()
        
        obstacle_group.draw(screen)
        obstacle_group.update()
        
 
               #obstacle movement
        obstacle_rect_list = obstacle_movement(obstacle_rect_list)
        
        #collision
        game_active = collision_sprite()

    else:
        screen.fill((94,129,162))
        screen.blit(player_stand,player_stand_rect)
      
        score_message = test_font.render(("Your score: "+ str(score)),False,(111,196,169))
        score_message_rect = score_message.get_rect(center = (400,330))
        screen.blit(game_name,game_name_rect)
        
        if score == 0:
            screen.blit(game_message,game_message_rect)
        else:
            screen.blit(score_message,score_message_rect)
            
        start_time = int(pygame.time.get_ticks() / 1000)
    
    pygame.display.update()
    clock.tick(60)
