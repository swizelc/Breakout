# Breakout
Breakout Game using PyGame
#------------imports--------------------
import pygame
import math
import random
pygame.init()

#---------global variables---------
screen = pygame.display.set_mode([600,400])
pygame.display.set_caption("Breakout!")
surface = pygame.Surface(screen.get_size())
CLOCK = pygame.time.Clock()
WHITE = (255,255,255)
BLACK = (0,0,0)
PINK = (255,192,203)
PURPLE = (177,156,217)
BLUE = (128,206,225)
PPINK =(231,84,128)
COLOURS = [PINK,PURPLE,BLUE,PPINK]
brick_w = 30
brick_h = 15
FONT = pygame.font.Font(None, 32)
ticks=pygame.time.get_ticks()

#---classes------
class Ball(pygame.sprite.Sprite):
    # speed of the ball at all times
    speed = 8
 
    # location of the ball
    x = 0
    y = 180
 
    #angle the ball comes in at in the beginning - in degrees
    direction = 135

    #dimentions of the ball
    w = 10
    h = 10
 
    def __init__(self):
        #call the parent class (Sprite) constructor
        super().__init__()
 
        #create the image of the ball
        self.image = pygame.Surface([self.w, self.h])
 
        #color the ball
        self.image.fill(WHITE)
 
        # Get rectangle object to collect its attributes
        self.rect = self.image.get_rect()
 
        # Get attributes: height and width of the screen
        self.screen_h = pygame.display.get_surface().get_height()
        self.screen_w = pygame.display.get_surface().get_width()
 
    def bounce(self, change):
       #Handles bounce off a horizontal surface ie the paddle
        self.direction = (180 - self.direction)
        self.direction -= change
 
    def update(self):
        #Update position of the ball
        # Sine and Cosine work in degrees, so we have to convert them
        direction_rad = math.radians(self.direction)
 
        # Change the position according to the speed and direction
        self.x += self.speed * math.sin(direction_rad)
        self.y -= self.speed * math.cos(direction_rad)
 
        # Move image to where the x and y are
        self.rect.x = self.x
        self.rect.y = self.y
 
        #  bounce off the top of the screen?
        if self.y <= 0:
            self.bounce(0)
            self.y = 1
 
        # bounce off the left of the screen?
        if self.x <= 0:
            self.direction = (360 - self.direction) % 360
            self.x = 1
 
        # bounce of the right side of the screen?
        if self.x > self.screen_w - self.w:
            self.direction = (360 - self.direction) % 360
            self.x = self.screen_w - self.w - 1
 
        # fall off the bottom edge of the screen?
        if self.y > 350:
            return True
        else:
            return False
 
class Paddle(pygame.sprite.Sprite):
 
    def __init__(self):
        # Call the parent's constructor
        super().__init__()
 
        self.w = 75
        self.h = 15
        self.image = pygame.Surface([self.w, self.h])
        self.image.fill((WHITE))
 
        self.rect = self.image.get_rect()
 
        self.rect.x = 0
        self.rect.y = 350 #350 is the cut off edge
 
    def update(self):
        #update position of the paddle
        # Get where the mouse is
        mouse = pygame.mouse.get_pos()
        #use the x coordinate of the mouse to make the bar move with it
        self.rect.x = mouse[0]-(self.w/2)
        #if it goes past the screen(right side) then keep it at the edge
        if self.rect.x > 525:
            self.rect.x = 525


class Brick(pygame.sprite.Sprite):
    def __init__(self, x, y):
        # Call the parent class (Sprite) constructor
        super().__init__()
 
        # Create the image of the block
        self.image = pygame.Surface([brick_w, brick_h])
 
        # Fill the image with color
        self.image.fill(COLOURS[random.randint(0,3)])
 
        # get the rectangle object that has the dimensions of the image
        self.rect = self.image.get_rect()
 
        self.rect.x = x
        self.rect.y = y
        

#takes user to end screen
#displays scores and points
#then quits  
def gameover(points):
    screen.fill(BLACK)
    text = "GAMEOVER!"
    txt_surface =FONT.render(text, True, WHITE)
    screen.blit(txt_surface,(230,150))
    score = "Points: "+ str(points)
    txt_surface = FONT.render(score,True,WHITE)
    screen.blit(txt_surface,(250,200))
    pygame.display.update() 
    while True:
        CLOCK.tick(30)
        screen.fill(BLACK)
        #if user exists the game by pressing the x
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()
                break

    
def play():
    points = 0
    lives = 5
    bricks = pygame.sprite.Group()
    balls = pygame.sprite.Group()
    total_sprites = pygame.sprite.Group()
    paddle = Paddle()
    total_sprites.add(paddle)

    ball = Ball()
    total_sprites.add(ball)
    balls.add(ball)

    #start points for first row of bricks
    top = 50

    #create 8 rows with 20 bricks each
    for i in range(8):
        for j in range(0,20):
            brick = Brick(j*(brick_w+2)+1,top)
            bricks.add(brick)
            total_sprites.add(brick)
        top+= brick_h +2
    #main loop    
    playing = True
    while playing:
        CLOCK.tick(30)
        screen.fill(BLACK)
        #if user exists the game by pressing the x
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                playing = False
                pygame.quit()
                quit()
        #enable the movement of the paddle        
        paddle.update()
        #returns True if ball falls of the bottom edge 
        death = ball.update()
        if death:
            #lose a life
            lives -= 1
            if lives ==0:
                #end game if lives are over
                gameover(points)
                playing = False
            else:
                #get a new ball
                balls.remove(ball)
                total_sprites.remove(ball)
                ball = Ball()
                total_sprites.add(ball)
                balls.add(ball)

        #if paddle and ball collide
        #make it bounce and change its direction
        #by the differece in the centres of the 2 objects
        if pygame.sprite.spritecollide(paddle,balls,False):
            change = (paddle.rect.x + paddle.w/2) - (ball.rect.x+ball.w/2)
            ball.rect.y = 349
            ball.bounce(change)

        #if the brick collides with the walls it just bounces
        # with the same angle opposite direction
        #and add ten points fro every brick eliminated (True removes the sprite)
        ball_brick = pygame.sprite.spritecollide(ball,bricks,True)
        if len(ball_brick)>0:
            for i in ball_brick:
                points += 10
            ball.bounce(0)
        #if all bricks are eliminated; end game
            if len(bricks)==0:
                playing = False
                gameover(points)
        # show user their points and  remaining lives continually
        text = "Points: " +str(points) + "   Lives: " +str(lives)
        txt_surface = FONT.render(text,True,WHITE)
        screen.blit(txt_surface,(0,370))       
                
        total_sprites.draw(screen)
        pygame.display.flip()
        
play()           
